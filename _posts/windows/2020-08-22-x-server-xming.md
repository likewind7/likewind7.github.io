---
layout: post
title: "Windows X server for linux"
date: 2020-08-22
excerpt: "https://m.blog.naver.com/monocho/221114374493"
categories : [windows]
tags: [wsl, gui]
comments: true
---

## Windows에서 실행되는 네이티브 프로그램

Xming은 X 윈도우 시스템 디스플레이 서버에 다양한 샘플 X 도구 및 응용 프로그램 (기존의 것들)을 제공합니다. 또한 디스플레이 서버에 일련의 글꼴을 제공합니다. 디스플레이 서버는 여러 언어를 지원합니다. 또한 OpenGL GLX 3D 그래픽 확장 및 Mesa 3D 기능을 제공합니다. 다른 컴퓨터에서 X11 세션을 전달하기 위해 Xming을 사용하여 안전한 방식으로 SSH (Secure Shell)를 실행할 수 있습니다. 디스플레이 서버는 Linux에서 크로스 컴파일되며 MinGW 컴파일러 제품군이 있습니다. 또한 유명한 Pthreads-Win32 멀티 스레딩 라이브러리도 함께 제공됩니다.
<https://xming.softonic.kr/> 에서 발췌


##  x 의 아키텍쳐
x 는 client-server architecture 로 design 되어있습니다. application 이 client 가 되며 서버와의 통신을 통해 요청및 수신을 합니다. <https://www.tldp.org/HOWTO/XWindow-Overview-HOWTO/arch-overview.html> 에는 "client 가 그림을 어떻게 그리는지 알 필요가 없다는 것이 이점이다" 라고 되어있네요. x 에서 y 까지 라인을 그려라 라는 식의 명령줄을 전달하면 된다고 하네요. graphic driver 를 포함한 하위 레이어를 신경 쓸 필요가 없다는 거겠죠 

<figure>
	<a href="https://likewind7.github.io/image/x_client_server_arch.png"></a>
	<figcaption><a href="https://likewind7.github.io/image/x_client_server_arch.png" title="xming architecture">xming architecture</a>.</figcaption>
</figure>

## 설치및 간단한 테스트
https://sourceforge.net/projects/xming/ 에서 xmig server 설치

{ %raw% }
sudo systemd-machine-id-setup
sudo dbus-uuidgen — ensure
cat /etc/machine-id
{ %endraw% }

cat 명령어를 통해 machine-id 를 확인합니다. 대충 영어와 숫자가 섞여있는 문자열이 나오면 맞다고 보시면 됩니다.

{ %raw% }
sudo apt -y install x11-apps xfonts-base xfonts-100dpi xfonts-75dpi xfonts-cyrillic
{ %endraw% }
x window 의 font를 설치합니다.

{ %raw% }
export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0.0
{ %endraw% }
user의 .bashrc 에 위의 내용을 입력합니다. 
0.0 이라고 되어있는 항목은 xming 의 view_log 옵션을 보고
"winMultiWindowXMsgProc - DISPLAY=127.0.0.1:0.0" 라고 되어있는 부분을 통해 파악하면 될 것 같다는 추정을 해봤습니다.

No protocol specified Error: Can't open display: 등의 오류가 날 수 있는데 이 때는 xming 의 아이콘에서 -ac 옵션을 넣어주시면 됩니다. 

xming 의 옵션을 보면 
{ %raw% }
Usage...
Xming [:<display-number>] [option]
-a #                   mouse acceleration (pixels)
-ac                    disable access control restrictions
-audit int             set audit trail level
-auth file             select authorization file
{ %endraw% }
라고 되어있는데 매우 찝찝하지만 풀어주고 해결했습니다.


<figure>
	<a href="https://likewind7.github.io/image/xming_eye.png"></a>
	<figcaption><a href="https://likewind7.github.io/image/xming_eye.png" title="xming xeye test">정면을 볼 수 없는 눈동자</a>.</figcaption>
</figure>