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

##  xming X window server 
xming 이라는 xwindow server 를 통해 graphic 출력이 가능했습니다. <https://likewind7.github.io/x-server-xming/> 참고



계속