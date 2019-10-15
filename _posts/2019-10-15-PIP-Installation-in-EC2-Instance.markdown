---
layout: post
title:  "DB Instance 연결을 위한 보안 설정"
date:   2019-10-15
---
미편집
<p>안녕하세요.</p>

<p>AWS / Django 웹 애플리케이션 개발 관련하여 책에서 설명이 부족했던 부분을 알려드리기 위해서 글을 쓰니 참고하시기 바랍니다. 이 부분도 추후에 2쇄가 발행된다면 반영토록 하겠습니다.</p>

<p>책 134~138p를 보면, virtualenv 가상환경을 생성하는 부분이 있습니다.</p>

<p>가상환경 생성 순서는</p>

1. pip3로 virtualenv를 설치한다.
2. virtualenv --version 으로 버전을 확인한다.
3. virtualenv [가상환경명] 을 입력하여 신규 가상환경을 생성한다.
4. source [가상환경명]/bin/activate 를 실행하여 가상환경으로 들어간다.

<p>이 순서로 구성되어 있는데, 놀랍게도 3번부분이 빠져있었고, 4번 부분 예제 코드가 잘못 기재가 되어 있었습니다.</p>

<p>해당 사항에 대해서는 다시 한번 양해를 부탁드리겠습니다.</p>

<p>가상환경 생성은 위 3번의 내용대로 'virtualenv [가상환경명]'의 형태로 입력하면 가상환경이 생성되며,
생성된 가상환경은 아래와 같이 나타납니다.</p>

``` Python3
virtualenv ve
```

<img src='/assets/img/img002_01.png' />

<p>그리고 138p의 가상환경 들어가는 부분의 코드는 </p>

{% highlight python3 %}
source ve/bin/activate
{% endhighlight %}
 

<p>위 코드이며, 설명까지도 자세히 되어 있었는데, 책에서는 'source'가 빠진 채로 've/bin/activate'로만 입력되어 있었습니다. 해당 부분에 대해서는 더불어 양해 바라겠습니다.</p>
 

<p>서적 이용에 참고하시기 바랍니다.</p>

<p>감사합니다.</p>