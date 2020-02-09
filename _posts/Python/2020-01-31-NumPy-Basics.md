---
layout: post
title:  "NumPy 기본 및 생성"
date:   2020-01-31
categories: [Python]
tags: [Python, 파이썬, NumPy, 넘파이, 모듈, 연산, numpy, 기본, 생성, creation, basic]
---

안녕하세요. 벌써 2020년이 되고 설날도 지났네요. 새해 복 많이 받으시고요. 원래 작년 12월 말에 쓰려고 했던 글인데 한 달이 다 되어가서야 쓰게 되었네요.

이전에 NumPy 소개 글에 이어서 이번 글에는 NumPy 기본부분과 NumPy 배열을 생성하는 내용을 다루도록 하겠습니다. 제 글에 있는 모든 내용은 아래 사이트의 내용을 Reference로 하여 작성하였으니 참고 바랍니다.

#### 자료형

Numpy에서 사용하는 자료형은 ndarray라는 자료형을 사용합니다. ndarray 자료형은 배열 구조형으로, 기본적으로 다차원 배열을 사용합니다.

{% highlight Python %}
[[1., 0., 0.],
 [0., 1., 2.]]
{% endhighlight %}

하지만 일반 자료형인 리스트, 튜플로도 변환 가능하니 참고하시면 됩니다.


### 배열 생성 방법

배열을 생성하는 방법은 여러 가지가 있습니다.

특정 값으로 배열을 생성하는 방법이 있고, 정수 형태의 순차값으로 생성하는 방법도 있고, 0이나 1로 값을 채우는 방법도 있습니다. 물론 그 외에도 시작값과 끝 값을 지정하고 개수를 지정하면 일정 간격으로 사이 값으로 채워지는 경우도 있습니다.

{% highlight Python %}
a = np.array([10, 20, 30])  # [10, 20, 30]
{% endhighlight %}

특정 값으로 구성된 배열을 생성합니다. 특정 값은 단일 값일 수도 있지만 대체로 리스트, 튜플 등이 들어갑니다. 

{% highlight Python %}
b = np.arange(8)    # [0, 1, 2, 3, 4, 5, 6, 7]
{% endhighlight %}

0부터 입력한 개수의 값이 순차적으로 입력됩니다. for문에서 주로 사용되는 range() 함수와 같은 용도입니다. 함수 이름 외우는것도 Array의 'a', 그리고 range' 이런 식으로 외우면 되겠죠.

{% highlight Python %}
c = np.zeros(5) # [0, 0, 0, 0, 0]
{% endhighlight %}

입력 개수만큼 0으로 채웁니다. AI에서 많이 사용하니까 이 함수는 반드시 알아두는 것이 좋습니다.

{% highlight Python %}
d = np.ones(2)  # [1, 1, 1, 1, 1]
{% endhighlight %}

zeros가 0이면 ones는 1이겠죠? 같은 범주로 이해하시면 됩니다.

{% highlight Python %}
e = np.linspace(10, 12, num=5)
{% endhighlight %}

linspace는 arange를 확장한 메소드로 보시면 됩니다. 시작값, 끝값, 배열 요소 개수 형태로 들어가며, 나머지 값은 시작값과 끝값의 사이값으로 규칙적으로 들어갑니다. 
arange가 정수 배열이면, linspace는 정수/실수 배열로 보면 되겠죠?

물론 그 외에도 zeros_like, ones_like, empty, empty_like, fromfunction, fromfile, random 등이 있지만, 대표적으로는 위 다섯 가지를 많이 사용합니다.



### 배열 기본 속성 및 차원

위 예제에서는 1차원 배열을 예로 들었지만, ndarray 배열은 사실 다차원 배열입니다. 그래서 이와 관련된 기본 속성에 대해서도 간단히 알아보겠습니다.

{% highlight Python %}
a = np.array([[10, 20, 30],[40, 50, 11]])
print(a.ndim)   # 2
print(a.shape)  # (2, 3)
print(a.size)   # 6
{% endhighlight %}

* ndim은 몇 차원인지를 나타냅니다.
* shape는 배열 형태를 나타냅니다. 지금은 2x3 형태이므로 (2,3)으로 출력됩니다.
* size는 전체 크기를 나타냅니다.

ndarray는 요소와 형태가 독립적입니다. 이 부분은 매우 중요합니다.
2차원 2x3 배열을 2차원 3x2 또는 1x6으로 바꿀 수도 있고, 1차원 배열인 6으로 바꿀 수도 있습니다.

즉 요소의 내용 및 순서는 동일하더라도, 형태만 맞으면 자유롭게 변환이 가능합니다. 물론 형태가 맞지 않으면 변환도 안되므로 유의 바랍니다.

ndarray의 형태를 바꾸는 함수는 다음과 같습니다.

{% highlight Python %}
b = a.reshape(3,2)  # [10, 20], [30, 40], [50, 60]
{% endhighlight %}

reshape 메소드를 사용하면 형태를 바꿀 수 있으며, 이 함수는 numpy에서 가장 많이 사용하는 메소드이므로 반드시 알아두시는 것이 좋습니다.

또한 ndarray는 shape만 동일하면 일반적인 사칙연산도 가능합니다. 지난번 글에서 나온 예제가 그 내용으로 볼 수 있습니다.

{% highlight Python %}
import numpy as np

a = np.array([10, 20, 30, 40])
b = np.array([3, 5, 2, 1])

c = a + b		# [13 25 32 41]
d = a - b		# [ 7 15 28 39]
e = a * b		# [ 30 100 60 40]
f = a / b		# [ 3.33333333 4. 15. 40. ]
g = a % b		# [1 0 0 0]
{% endhighlight %}

Numpy에는 수많은 수학 내장함수가 있습니다. 숫자를 다루는 모듈이다 보니 숫자 함수가 없으면 안되겠죠? 

{% highlight Python %}
import numpy as np
a = np.array([10, 20, 30, 40])
b = np.array([0, 1, 2, 3])
c = np.exp(b)		# [ 1. 2.71828183 7.3890561 20.08553692]
d = np.sqrt(a)		# [3.16227766 4.47213595 5.47722558 6.32455532]
e = np.add(a,b)		# [10 21 32 43]
{% endhighlight %}

그 외에도 수많은 함수가 있는데, NumPy Reference를 찾으면 확인이 가능하니 참고 바랍니다.

가장 기초적인 내용은 여기서 마치며, 다음은 NumPy 배열인 ndarray를 어떻게 다루는지를 설명하겠습니다.
