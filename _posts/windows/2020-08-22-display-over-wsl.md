---
layout: post
title: "GUI over wsl"
date: 2020-08-22
excerpt: "https://m.blog.naver.com/monocho/221114374493"
categories : [windows]
tags: [wsl, gui]
comments: true
---

## WSL 찬양
WSL 기능을 무척이나 좋아하는 편입니다. 
* 멀티부팅이 필요가 없고 
* 그 무거운 vxware를 돌리지 않아도 되며
* build 및 많은것들을 위 2가지 예의 번거로움 없이 할 수 있습니다.
* 이점 대비 성능 감소가 큰 편은 아니라고 생각 합니다. [^1]

[^1]: <https://awesometic.tistory.com/223>
<linux system 과 wsl 환경의 build성능 비교>

그러나.... 최근 목표를 위한 환경구축을 하다보니 한가지가 불편했습니다. 그건 바로 GUI.
build 및 python 수행등을 wsl 에서 하고 있었는데 display 가 안되니까 어쩔 수 없이 windows환경을 사용하게 되었습니다. 그래서 찾아본 결과.


## xming X window server 
xming 이라는 xwindow server 를 통해 graphic 출력이 가능했습니다. <https://likewind7.github.io/x-server-xming/> 참고. 제 경험상 설치 후 동작하지 않는 이유가 두 가지 있었는데 하나는 display 포트 였고 나머지 하나는 권한문제였습니다. 포트는 xming log 에 나와있는대로 수정해서 해결하였고 권한은 xming의 옵션에 -ac 를 추가하여 해결하였습니다. xeyes 라는 테스트 어플리케이션이 정상동작하는 것까지 확인하였습니다.



## python tk
목적이 python 의 graphic 출력 이었으므로 python code를 구동시켜봤습니다만 동작하지 않았습니다.
이건 tkinter 라는 python 의 표준 GUI interface 를 설치하지 않았기 때문이었습니다. 설치후 다음과 같이 동작함을 확인하였습니다.


{% capture images %}
https://likewind7.github.io/image/python_gui_over_wsl.png
{% endcapture %}
{% include gallery images=images caption="<python gui over wsl>" cols=3 %}