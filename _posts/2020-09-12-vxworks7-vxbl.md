---
layout: post
title: "vxworks7 의 오동작"
date: 2020-09-12
excerpt: "me"
category: vxworks
tags: [vxworks]
comments: true
---

vxworks7 으로 진행하는 프로젝트를 맡게되었습니다.
6.9 에서 확 바뀌었다는 느낌이 들더군요. 디렉토리 구조, 프로젝트 생성법, 
configuration 방법등 많은 것이 바뀐 것 처럼 보였습니다.
그 중 vxbl 이라는 bootloader를 이용하여 kernel을 load하게끔 되어있길래 살펴보았습니다.


### vxbl

#### 부트로더의 역할

<figure>
	<a href="http://farm9.staticflickr.com/8426/7758832526_cc8f681e48_b.jpg"><img src="http://farm9.staticflickr.com/8426/7758832526_cc8f681e48_c.jpg"></a>
	<figcaption><a href="http://www.flickr.com/photos/80901381@N04/7758832526/" title="Morning Fog Emerging From Trees by A Guy Taking Pictures, on Flickr">Morning Fog Emerging From Trees by A Guy Taking Pictures, on Flickr</a>.</figcaption>
</figure>

SBC에 심어놓은 bootloader 라면 꼭 해줘야 하는일이 있을겁니다. POWERPC기준으로 썰을 풀자면
당연히 flash(IFC) 와 DDR설정을 해 주어야 할거구요, 해당 영역에 TLB와 WINDOW를 열어줘야 할겁니다.
변태같은 CCSBAR 위치를 다른 구성요소들과 겹치지 않게 잡아줘야 하구요 system clock 도 재설정하고, console 띄우려면 uart도 초기화하고 등등... RCW는 논외로 하죠. 

-딱 잠에서 깨자마나 커널을 flash 에서 끄집어 내서 DDR로 집어던지기 위해 필요한 작업.- 혹은
-정신을 다시 차리고 커널을 flash 에서 끄집어 내서 DDR로 집어던지기 위해 필요한 작업.- 

이 작업들을 해 줘야 합니다. vxbl은 이 작업에 특화되어있는듯 보입니다. 
왜 이런표현이 썼냐하면 그외에는 아무것도 없기때문입니다.
ethernet, PCI, 그외 peripheral들 뭐든지 불필요하다싶은건 보이지 않습니다. ethernet은 좀 넣...


#### T 시리즈의 architecture

아... 왜 없을까 했더니 아니 뭐 그래도 이해는 안된다만 T 와 P 시리즈에 DPAA 가 적용되었습니다. 
간단히 말하면 phy 와 mac 만 가지고는 통신이 불가하다는 이야기 입니다. 
내부 데이터 path 가 좀 복잡해 졌죠. 

A simplified view of the dpaa_eth interfaces mapped to FMan MACs:
{% highlight html %}
dpaa_eth       /eth0\     ...       /ethN\
driver        |      |             |      |
-------------   ----   -----------   ----   -------------
     -Ports  / Tx  Rx \    ...    / Tx  Rx \
FMan        |          |         |          |
     -MACs  |   MAC0   |         |   MACN   |
           /   dtsec0   \  ...  /   dtsecN   \ (or tgec)
          /              \     /              \(or memac)
---------  --------------  ---  --------------  ---------
    FMan, FMan Port, FMan SP, FMan MURAM drivers
---------------------------------------------------------
    FMan HW blocks: MURAM, MACs, Ports, SP
---------------------------------------------------------
{% endhighlight %}


The dpaa_eth relation to the QMan, BMan and FMan:
{% highlight html %}
            ________________________________
dpaa_eth   /            eth0                \
driver    /                                  \
---------   -^-   -^-   -^-   ---    ---------
QMan driver / \   / \   / \  \   /  | BMan    |
           |Rx | |Rx | |Tx | |Tx |  | driver  |
---------  |Dfl| |Err| |Cnf| |FQs|  |         |
QMan HW    |FQ | |FQ | |FQs| |   |  |         |
           /   \ /   \ /   \  \ /   |         |
---------   ---   ---   ---   -v-    ---------
          |        FMan QMI         |         |
          | FMan HW       FMan BMI  | BMan HW |
            -----------------------   --------
{% endhighlight %}
			
vxbl에서 ethernet 통신이 가능하려면 이게 다 들어가야하니 귀찮았던건지 다이어트한 모습을 유지하고 싶었던건지
하여튼 말쑥한 부트로더의 모습이었습니다.


#### 어짜피 커널하고는 상관없음 ^^"
네 뭐 부트로더에 뭐가 있고 뭐가 없다한들 저랑은 아무상관 없다고 믿었습니다.
kernel 만 잘 던져주면 된다 싶었죠. 
그런데 이게.. vxbl 로 kernel 부팅을 하니 ethernet 이 안됩니다. 
u-boot 을 통해 부팅했을 때 아무이상없이 동작하길래 "됩니다~" 하고 보고했는데
같은 kernel을 사용했음에도 동작을 안해요. 걍 안되요 
큭..ㅠㅠ 아 왜..냐고 
SGMII 로 연결된 phy에 접근도 되고  auto negotiation 도 마쳤고 memac도 잡혀서 ifconfig 하면 
빵끗웃는데 통신만 안됩니다. 
부트로더에서 peripheral들 초기화 해봤자 kernel 올라가면서 drvier에서 다시 초기화 하므로 저언~혀 
상관없다고 생각했는데 안되요. 

이유는 두 가지로 귀결되는데 
- 부트로더에서 집어던지는 일 외에 해줘야할 일이 더 있다. ( 이런게 있다는 것을 받아들일 수 없음. )
- DTS에 (아참! vxworks7에 DTB 적용되었음) DPAA관련 configuration이 있는데 내가 이걸 적용안했다.


분명 kernel 부팅 되면서 관련 driver들 probe attach init 까지 되는데 막무가내로 동작을 안합니다.
뭐이런 쓰ㄹ...

boot argument 똑같게 맞춰놓았고 kernel 도 동일하므로 부트로더에 따라서 동작이 갈린다는 얘기가 되는데 받아들일 수가 없더군요. 



#### 삽질의 시작


Apply the `half` class like so to display two images side by side that share the same caption.

{% highlight html %}
<figure class="half">
    <a href="/images/image-filename-1-large.jpg"><img src="/images/image-filename-1.jpg"></a>
    <a href="/images/image-filename-2-large.jpg"><img src="/images/image-filename-2.jpg"></a>
    <figcaption>Caption describing these two images.</figcaption>
</figure>
{% endhighlight %}

And you'll get something that looks like this:

<figure class="half">
	<a href="http://placehold.it/1200x600.JPG"><img src="http://placehold.it/600x300.jpg"></a>
	<a href="http://placehold.it/1200x600.jpeg"><img src="http://placehold.it/600x300.jpg"></a>
	<figcaption>Two images.</figcaption>
</figure>

#### Three Up

Apply the `third` class like so to display three images side by side that share the same caption.

{% highlight html %}
<figure class="third">
	<img src="/images/image-filename-1.jpg">
	<img src="/images/image-filename-2.jpg">
	<img src="/images/image-filename-3.jpg">
	<figcaption>Caption describing these three images.</figcaption>
</figure>
{% endhighlight %}

And you'll get something that looks like this:

<figure class="third">
	<img src="http://placehold.it/600x300.jpg">
	<img src="http://placehold.it/600x300.jpg">
	<img src="http://placehold.it/600x300.jpg">
	<figcaption>Three images.</figcaption>
</figure>

### Alternative way

Another way to achieve the same result is to include `gallery` Liquid template. In this case you
don't have to write any HTML tags – just copy a small block of code, adjust the parameters (see below)
and fill the block with any number of links to images. You can mix relative and external links.

Here is the block you might want to use:

{% highlight liquid %}
{% raw %}
{% capture images %}
	http://vignette2.wikia.nocookie.net/naruto/images/9/97/Hinata.png
	http://vignette4.wikia.nocookie.net/naruto/images/7/79/Hinata_Part_II.png
	http://vignette1.wikia.nocookie.net/naruto/images/1/15/J%C5%ABho_S%C5%8Dshiken.png
{% endcapture %}
{% include gallery images=images caption="Test images" cols=3 %}
{% endraw %}
{% endhighlight %}

Parameters:

- `caption`: Sets the caption under the gallery (see `figcaption` HTML tag above);
- `cols`: Sets the number of columns of the gallery.
Available values: [1..3].

It will look something like this:

{% capture images %}
	http://vignette2.wikia.nocookie.net/naruto/images/9/97/Hinata.png
	http://vignette4.wikia.nocookie.net/naruto/images/7/79/Hinata_Part_II.png
	http://vignette1.wikia.nocookie.net/naruto/images/1/15/J%C5%ABho_S%C5%8Dshiken.png
{% endcapture %}
{% include gallery images=images caption="Test images" cols=3 %}
