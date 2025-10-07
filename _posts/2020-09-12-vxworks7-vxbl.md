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


문제를 풀기 위해 먼저 두 부트로더의 실행 환경을 비교하는 것부터 시작했습니다. `fw_printenv` 값과
vxWorks bootline 을 그대로 복사해 오고, MAC 주소·VLAN 같은 사소한 설정까지 u-boot 과 일치시키며
변수를 최대한 제거했습니다. 그래도 인터페이스가 `UP` 된 뒤에 패킷이 한 장도 나가지 않길래 Wind River
네트워크 포팅 가이드를 다시 읽으면서 초기화 경로를 추적했습니다.

추적 과정에서는 vxWorks 에서 제공하는 `ipcom_syslogd` 와 `logDevConnect` 를 이용해 부팅 로그를 최대한
끌어냈습니다. 특히 DPAA 관련 드라이버가 메모리를 어떻게 소비하는지 보고 싶어 QMan/BMan 초기화 지점에
직접 printk 를 삽입했습니다. 그 결과 vxbl 경로에서만 QMan 이 참조하는 FQD 영역이 `0x00000000` 으로
표기되는 메시지를 잡을 수 있었습니다. 런타임에서 기본값을 쓰고 있다는 얘기였고, u-boot 대비 누락된
설정을 의심할 수 있었습니다.

u-boot BSP 는 RCW 값에 맞춰 DPAA carve-out 을 자동으로 계산해 주지만, vxbl BSP 는 그런 후처리를 하지
않는다는 사실을 문서에서 확인했습니다. `_wrLinuxQorIQPxxx.dts` 를 기반으로 만들어 둔 기존 디바이스 트리를
확인해 보니 `fsl,bman-mem`, `fsl,qman-mem` 노드가 비어 있었습니다. 아래 로그는 그때 캡처한 것입니다.

```
[vxbl boot] qman_init(): using cgrid base 0x00000000
[vxbl boot] qman_init(): FQD base   0x00000000
```

결국 디바이스 트리에 carve-out 을 직접 적어 넣어야 했습니다. BSP 의 DT 소스에 다음과 같은 영역을 추가하고
`vxprj` 에서 `DTB_REBUILD` 를 수행한 뒤 이미지를 다시 만들어 올렸습니다. 아래 단계대로 진행하면 재현할 수
있습니다.

1. **디바이스 트리 원본 수정** – 워크스페이스의 `board/` 또는 `prj_boot/` 디렉토리 아래에 있는
   `_wrLinuxQorIQPxxx.dts` 를 열어 아래 carve-out 정의를 붙여 넣습니다. 기존에 동일한 노드가 있다면
   `fsl,*-mem` 속성만 채워 줍니다.
2. **DTB 재생성** – 프로젝트 루트에서 `vxprj -s <workspace>.wpj prj_dtb rebuild` 혹은 GUI 의 *Build →
   Rebuild Project* 를 실행해 DTB 를 다시 만듭니다. 로그에 `DTB_REBUILD` 단계가 포함되어야 합니다.
3. **부트 이미지 갱신** – 생성된 DTB 를 `vxprj bootimage` 또는 `make export` 로 다시 패키징한 뒤 타겟 보드의
   플래시나 TFTP 서버에 업로드합니다.
4. **부팅 확인** – vxbl 로 부팅해 `ipcom_syslogd` 로그와 `ifconfig -a` 를 확인합니다. QMan/BMan 드라이버가
   carve-out 주소를 올바르게 인식하면 TX/RX 카운터가 정상적으로 증가합니다.

```dts
        bman@31a000 {
                compatible = "fsl,bman", "simple-bus";
                status = "okay";
                fsl,bman-mem = <0x00 0x17f00000 0x00 0x00100000>;
        };

        qman@318000 {
                compatible = "fsl,qman", "simple-bus";
                status = "okay";
                fsl,qman-mem = <0x00 0x18000000 0x00 0x00100000>;
        };
```

재부팅한 뒤에는 QMan/BMan 드라이버가 정상적으로 attach 되면서 `ifconfig` 에서 TX/RX 카운터가 오르기
시작했습니다. 결국 vxbl 은 커널을 메모리로 올려 주는 일에만 집중하는 만큼, 주변 리소스 선언은 사용자가
직접 챙겨야 한다는 교훈을 다시 확인했습니다. 동일한 문제가 다시 발생하면 위 순서대로 디바이스 트리와
이미지를 재작성해 주면 됩니다.

마지막으로 정리하면 다음과 같습니다.

1. 부트로더를 바꿀 때는 bootline·환경 변수뿐 아니라 DT 설정까지 함께 비교해야 한다.
2. DPAA 기반 보드에서는 QMan/BMan carve-out 영역이 없으면 드라이버 초기화 자체가 실패한다.
3. 로그가 부족하면 직접 printk 를 넣어서라도 근거를 확보하라. 삽질을 끝내는 가장 빠른 길이다.

이제 남은 건 VXWorks 네트워크 스택 튜닝인데, 그건 다음 글에서 이어가려 합니다.