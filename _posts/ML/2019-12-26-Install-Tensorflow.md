---
layout: post
title:  "Ubuntu 18.04에서 Tensorflow 설치하기(Using WSL)"
date:   2019-12-26
image: '/assets/img/img015_01.png'
categories: [ML]
tags: [Python, 파이썬, ATensorflow, 텐서플로우, 설치, Installation, install, 텐서플로]
---

 
 이번에 딥러닝 연구를 위해서 텐서플로(Tensorflow)를 설치하려고 하는데, 생각보다 쉽게 안 돼서 몇몇 자료를 모아서 설치했던 글을 써볼까 합니다.

 먼저 환경부터 살펴보겠습니다.
 
 * OS: Ubuntu 18.04
    + 운영체제는 Ubuntu지만 실제 환경은 WSL입니다.
    + 기존에 사용했던 Google Compute Engine, AWS EC2 Instance가 기간이 만료가 되어서
    + Cloud 환경도 좋지만 로컬 환경(localhost)을 아예 배제할 수는 없습니다. 
    + 다만 Windows에서 실행하는 것보다는 WSL로 Ubuntu를 설치해서 리눅스 환경에서 작업하는게 좀 더 편한 것 같습니다.

* Python: 3.7
    + 원래 3.8로 하려고 했습니다.
    + 그러나 Tensorflow는 로컬에서 설치 시, Python3.8에서 Tensorflow를 아직 제대로 지원하지 않습니다.
    + 그래서 3.7에서 가상환경을 생성했습니다.
    + 당연히 venv도 3.7입니다.

* Tensorflow: 2.0.0
    + 최신버전이 2.0.0입니다. 
    + 버전 지정은 안하고 자동 설치로도 됩니다.
    + Tensorflow 공식 홈페이지(<https://www.tensorflow.org/>)에서도 2.0 버전 설치를 권장합니다.
    + 그러나 기존 라이브러리나 문법은 1.15 버전 등에 최적화되어 있어서 문법 조정이 필요할 수 있습니다.

* Numpy: 1.16.5
    + Tensorflow 2.0 일부 문법은 Numpy 1.18 버전에서 실행 시 Warning 메시지가 나옵니다.
    + 그래서 1.17보다 낮은 버전 설치가 필요합니다.

그러면 위 내용을 토대로 설치해보도록 하겠습니다.


### 1. Python 3.7 설치

아래 글을 참조하시면 되겠습니다. 제가 쓴 글입니다.

<https://amanokaze.github.io/blog/Construct-Python37-Django-MySQL-Google-Cloud/>

요약하자면 이렇습니다.
* Python 3.7 설치 및 버전 설정
* PIP 설치
* venv 설치 및 가상환경 진입

코드는 아래와 같습니다. 설명은 위 글에 이미 자세히 나와 있어서 코드만 넣을게요.

{% highlight Shell %}
$ sudo update-alternatives --install /usr/bin/python3 python /usr/bin/python3.6 1
$ sudo update-alternatives --install /usr/bin/python3 python /usr/bin/python3.7 2
$ sudo update-alternatives --config python
There are 2 choices for the alternative python (providing /usr/bin/python).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /usr/bin/python3.7   2         auto mode
  1            /usr/bin/python3.6   1         manual mode
  2            /usr/bin/python3.7   2         manual mode

Press <enter> to keep the current choice[*], or type selection number: 2

$ sudo apt-get install python3-pip
$ sudo apt-get install python3.7-venv
$ python3 -m venv venv
$ source venv/bin/activate
{% endhighlight %}

### 2. PIP, setuptools upgrade, numpy downgrade, wheel install

이거저거 다 해봤는데, 결국은 아에 처음부터 위와 같이 해 주는 것이 가장 좋습니다.
안 그러면 결국 에러납니다.

{% highlight Shell %}
$ pip install --upgrade pip
$ pip install setuptools --upgrade
$ pip install wheel
$ pip install "numpy<1.17"
{% enghighlight %}

pip, setuptools 업그레이드, wheel은 Tensorflow 설치 시 필요한 패키지입니다. 이 작업을 미리 해 주지 않으면 Tensorflow 설치 시 에러나 경고 메시지가 나타납니다. 

numpy 다운그레이드는 Tensorflow 설치에 큰 영향은 없습니다. 하지만 tensorflow를 import할 때 경고 메시지가 나타납니다. 문법 차이에 따른 경고 메시지이므로, 맞춰주는 것이 좋습니다.


### 3. Tensorflow 설치

이제 준비가 다 되었습니다. 위에 쓴 대로 자동설치하겠습니다.

{% highlight Shell %}
pip install tensorflow
{% endhighlight %}