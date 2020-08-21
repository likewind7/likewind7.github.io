---
layout: post
title: "KONLP 한국어 처리 패키지"
date: 2020-08-20
excerpt: "https://datascienceschool.net/view-notebook/a0237ff8f13a454c96072f868c01bc30/"
categories : [nlp]
tags: [nlp, konlp]
comments: true
---



이 번 post 역시 데이터 사이언스 스쿨에서 참조하여 작성합니다.

<https://datascienceschool.net/view-notebook/a0237ff8f13a454c96072f868c01bc30/>



## KONLP 한국어 처리 패키지

역시 있었습니다. 코엔엘파이라고 발음 한다고 하며 한국어 정보처리를 위한 파이썬 패키지 입니다.



### 한국어 말뭉치
KoNLPy 에서는 대한민국 헌법 말뭉치인 kolaw와 국회법안 말뭉치인 kobill을 제공합니다. 각 말뭉치가 포함하는 파일의 이름은 fields 메서드로 알 수 있고 open 메서드로 해당파일의 텍스트를 읽어들입니다.



### Mecab
무언가 시작을 하니 알 수 없는 것들이 쏟아지기 시작합니다. ㅎㄷㄷ
nltk 의 stemmer 처럼 한국어의 형태소를 다룰 수 있는 Mecab 라이브러리 가 있는데
이 게 windows에서 동작하지 않습니다.

- raise Exception('The MeCab dictionary does not exist at "%s". Is the dictionary correctly installed?\nYou can also try entering the dictionary path when initializing the Mecab class: "Mecab(\'/some/dic/path\')"' % dicpath)


위와 같은 에러메시지가 출력 됩니다. 이 때 코앤엘 파이 (KoNLPy)의 mecab 대신 은전한닢 (eunjeon) 프로젝트의 mecab-ko 를 사용하면 된다고 하여 동작시켜 보았습니다. 

<https://victorydntmd.tistory.com/264>
