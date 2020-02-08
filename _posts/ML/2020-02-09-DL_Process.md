---
layout: post
title:  "딥러닝 신경망 - (1) 딥러닝 프로세스 구현"
date:   2020-02-08
image: 'img019_01.png'
categories: [ML]
tags: [ML, DL, Deep, Learning, 딥러닝, 머신러닝, 프로세스, Process, 케라스, 신경망, Keras, Neuron, Network, MNIST, Model, 모델]
---


이번 글은 머신러닝/딥러닝 소개에 이어서 딥러닝 연구를 위한 신경망에 대해서 다루도록 하겠습니다.

기본적인 자료는 프랑소와 솔레가 저술한 '케라스 창시자에게 배우는 딥러닝(길벗)' 서적의 내용을 토대로 따왔으며, 제가 직접 그린 이미지도 있지만 책의 이미지도 일부 따왔음을 먼저 알려드립니다.
이번 글은 상기 서적의 2장을 다루었으나, 책의 내용을 참고하여 제 기준에서 나름대로 정리한 내용이므로 참고 바랍니다. <b>당연히 본문에 있는 코드도 해당 책의 코드이므로 역시 참고 바랍니다.</b>

다만 너무 많은 내용을 자세히 설명하면 저작권에 문제가 될 소지가 있으므로 구체적으로는 다루지 않고 대략적으로만 다루니 양해 바랍니다.

[케라스 창시자에게 배우는 딥러닝 서적 클릭](http://www.yes24.com/Product/Goods/65050162)


#### 예제 모델

딥러닝을 한 번이라도 접하셨다면, MNIST를 모를 리는 없을 것입니다. 만약 모르신다면 인터넷에 조금만 검색해도 나오므로 참고바랍니다.
이 글에서도, 참고서적에서도 역시 MNIST 모델을 바탕으로 예제가 진행될 것입니다.

![MNIST]({{ '/assets/img/img019_01.png' | prepend: site.baseurl }})

Keras에서 MNIST 모델을 불러오는 방법은 아래와 같습니다.

{% highlight Python %}
from keras.datasets import mnist
(train_images, train_labels), (test_images, test_labels) = mnist.load_data()
{% endhighlight %}

훈련 데이터(Train data)와 테스트 데이터(Test data)를 불러온 후 위와 같이 각각 저장하는 형태겠지요.

그런 다음 신경망을 다음과 같이 만듭니다.

{% highlight Python %}
from keras import models, layers

network = models.Sequential()
network.add(layers.Dense(512, activation='relu', input_shape=(28*28,)))
network.add(layers.Dense(10, activation='softmax')
{% endhighlight %}

이거를 진짜 아무것도 모르는 사람들을 위한 기준으로 한번 써보면 이 정도입니다. 깊게 안들어갑니다.

* model을 만들어요. 무슨 모델일진 모르지만 일단 만들어요.
* 모델에 들어갈 내용을 넣어요(add() 메소드).
* 층(Layer)을 만들어요(layers.Dense() 메소드). 위 코드는 두 번 나왔으니까 층이 2개가 되겠죠.
* relu가 뭐고 softmax가 뭐고 그런건 몰라요. 아무튼 만들어요.

지난 번 글에서는 딥러닝의 전체 프로세스를 간단히 소개한 그림이 있었습니다. 그 그림을 다시 넣어볼게요.

![DL Process]({{ '/assets/img/img018_08.png' | prepend: site.baseurl }})

위 그림의 '층'이라고 된 부분 있죠? 위 코드는 그냥 그걸 만드는 부분이라고 보면 됩니다.

그리고 그림 아래에는 뭐가 있죠. 손실함수, 옵티마이저가 있습니다. 그러면 아 다음 코드는 손실함수와 옵티마이저를 결정하는 코드가 나오겠구나 그렇게 생각하면 간단해집니다.

{% highlight Python %}
network.compile(optimizer='rmsprop', loss='categorical_crossentropy', metrics=['accuracy'])
{% endhighlight %}

이렇게 하면 위 딥러닝 프로세스에서 진행할 층, 손실함수, 옵티마이저는 미리 다 만들어 놓은겁니다.

다음은 뭐가 필요할까요. 테스트/훈련 데이터도 MNIST로 받아왔고, 딥러닝 모델도 만들었으니까 이제 그 두개를 연결해야겠죠.
코드를 다시 한 번 보세요. 각각 만들기만하고 연결시킨 것은 없었으니까 연결해야 한다는 뜻입니다.

그래서 연결을 하기 위해서는 모델에 알맞게 크기를 변경하는 작업이 필요합니다.
MNIST 모델의 train_images, train_labels, test_images, test_labels 변수는 전부 다 NumPy 변수로, reshape() 메소드를 사용하면 알맞는 크기로 변경이 되겠죠?

<b>책에서는 크기를 변경하는 코드도 있지만, 책의 모든 코드를 다 우겨넣으면 안될 것 같아서 이 부분 예제 코드는 생략합니다.</b>

그리하여 어찌저찌 크기 변경까지 다 끝났으면, 이제 위에서 말한 실제 연결하는 명령어를 써 보겠습니다.

{% highlight Python %}
network.fit(train_images, train_labels, epochs=5, batch_size=128)
{% endhighlight %}

요약하면 train_images, train_labels를 입력해서 넣어서 학습하겠다 그런 뜻으로 보시면 됩니다.
뒤에 epochs, batch_size가 나오는데, 사실 이 부분은 굉장히 중요한 파라미터입니다만 이번 글에서 다루기 보다는 뒤에서 별도로 다루겠습니다.

여기까지입니다.
딥러닝에 대한 전체 프로세스를 지난 글에서 간단히 언급했었는데, 그것을 실제 코드로 어떻게 구현하였나. 그 부분에 보다 중점을 두었고요.
이제 다음 글에서는 신경망 관련해서 이어서 써 보도록 하겠습니다.
