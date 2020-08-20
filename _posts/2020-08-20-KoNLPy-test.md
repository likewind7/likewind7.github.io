---
layout: post
title: "KONLP 한국어 처리 패키지"
date: 2020-08-20
excerpt: "https://datascienceschool.net/view-notebook/a0237ff8f13a454c96072f868c01bc30/"
categories : [NLP]
tags: [nlp, konlp]
comments: true
---



이 번 post 역시 데이터 사이언스 스쿨에서 참조하여 작성합니다.

<https://datascienceschool.net/view-notebook/a0237ff8f13a454c96072f868c01bc30/>



## KONLP 한국어 처리 패키지

역시 있었습니다. 코엔엘파이라고 발음 한다고 하며 한국어 정보처리를 위한 파이썬 패키지 입니다.



### 한국어 말뭉치
KoNLPy 에서는 대한민국 헌법 말뭉치인 kolaw와 국회법안 말뭉치인 kobill을 제공합니다. 각 말뭉치가 포함하는 파일의 이름은 fields 메서드로 알 수 있고 open 메서드로 해당파일의 텍스트를 읽어들입니다.

다음과 같은 error 가 발생하였습니다.

{% highlight css %}
Python 3.6.8 (tags/v3.6.8:3c6b436a57, Dec 24 2018, 00:16:47) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import matplotlib
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "C:\Users\jhlee\AppData\Roaming\Python\Python36\site-packages\matplotlib\__init__.py", line 174, in <module>
    _check_versions()
  File "C:\Users\jhlee\AppData\Roaming\Python\Python36\site-packages\matplotlib\__init__.py", line 159, in _check_versions
    from . import ft2font
ImportError: DLL load failed: 지정된 모듈을 찾을 수 없습니다.
{% endhighlight %}

사용중이던 환경은 
- windows 10
- python 3.6.8

찾아본 여러가지 방법들이 있었으나 모두 실패 하고 **Python 3.8.5 로 업그레이드** 후에 그냥 동작했습니다.

It will look something like this:

{% capture images %}
	https://likewind7.github.io/image/matplotlib.png
{% endcapture %}
{% include gallery images=images caption="matplotlib text test" cols=3 %}




## 워드클라우드

wordcloud 라이브러리를 통해 사용 단어 빈도수에 따라 시각화가 가능합니다.
{% highlight css %}
import nltk
emma_raw = nltk.corpus.gutenberg.raw("austen-emma.txt")
from nltk.tokenize import RegexpTokenizer
retokenize = RegexpTokenizer("[\w]+")
from nltk.tag import pos_tag
sentence = "Emma refused to permit us to obtain the refuse permit"

from nltk.tokenize import word_tokenize
tagged_list = pos_tag(word_tokenize(sentence))
tagged_list
from nltk import Text
text = Text(retokenize.tokenize(emma_raw))
from nltk import FreqDist
stopwords = ["Mr.", "Mrs.", "Miss", "Mr", "Mrs", "Dear"]
emma_tokens = pos_tag(retokenize.tokenize(emma_raw))
names_list = [t[0] for t in emma_tokens if t[1] == "NNP" and t[0] not in stopwords]
fd_names = FreqDist(names_list)

from wordcloud import WordCloud
wc = WordCloud(width=1000, height=600, background_color="white", random_state=0)
#plt.imshow(wc.generate_from_frequencies(fd_names))
#plt.axis("off")
#plt.show()
import matplotlib
matplotlib.pyplot.imshow(wc.generate_from_frequencies(fd_names))
matplotlib.pyplot.axis("off")
matplotlib.pyplot.show()
{% endhighlight %}

{% capture images %}
	https://likewind7.github.io/image/wordcloud.png
{% endcapture %}
{% include gallery images=images caption="word cloud test" cols=3 %}