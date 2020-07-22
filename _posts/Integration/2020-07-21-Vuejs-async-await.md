---
layout: post
title:  "Vue.js의 Async - Await 사용"
date:   2020-07-21
image: 'img025_01.png'
categories: [Integration]
tags: [Vue,Vue.js,Asnyc,Await,Promise,Resolve,Reject,ES6]]
---


안녕하세요. 거의 두 달만에 쓰는 글입니다.
그동안 프로젝트를 수행하느라 블로그에 글을 쓸 시간도 없었네요.
이제 프로젝트가 마무리 단계라서 그동안 프로젝트 수행 과정에서 나온 여러 가지 자료를 올리고자 하니 참고하시면 좋겠습니다.

#### 1. Async - Await란?

쉽게 요약하면 Javascript의 ES6 문법에서 사용하는 비동기 처리 패턴입니다.

요즘 Front-end 개발을 하는 데 있어서 Ajax를 사용하는 것은 너무나도 구식의 방법이 되어버렸죠. 실제로 사용도 복잡하고, Ajax를 대체할 수 있는 비동기 처리 방식이 다수 등장했기 때문이랄까요.

그런데 사실 Ajax 다음에 나온 것이 Async - Await는 아닙니다. 그 중간에는 Promise라는 것이 있는데, Promise에 대한 이해를 먼저 해야 Async - Await를 이해할 수 있습니다.

Promise 역시 ES6 문법에서 사용되는 객체로, Async - Await는 쉽게 말하자면 Promise를 간소화한 것으로 이해하시면 될 것 같습니다.


#### 2. Promise

Front-end에서 어떤 Framework를 사용하느냐에 따라 다르지만, 일반적으로 Vue.js에서는 API 호출을 할 때 axios 모듈을 사용합니다.

그런데 axios 모듈에서의 request / response는 비동기로 처리되기 때문에 어떤 처리 순서를 지정하지 않으면 request로 요청을 보내고 나서 response로 응답도 받기 전에 다음 구문을 수행해 버리기 때문에 원하는 결과를 받아오지 못하겠죠.

{% highlight Javascript %}
testFunction() {
	const response = axios.get('api/test')
	console.log(response)
}
{% endhighlight %}

이런 문장이 있다고 합시다. 맨 아래에 axios에 대한 결과값인 response를 보여주려고 하는데, 결과는 원하는 결과값이 아닌 전혀 이상한 값이 나올 것입니다.

{% highlight JSON %}
Promise {<pending>}
	__proto__: Promise
	[[PromiseStatus]]: "resolved"
	[[PromiseValue]]: Object
{% endhighlight %}

왜냐. Javascript에서의 문장 처리 순서는 axios()를 사용해서 request를 보내는 연산을 수행하고 response를 언제 받는지는 잘 모르겠고, 다음 문장을 수행하기 때문입니다.
그래서 이러한 순차적 처리를 위해서 Promise를 사용하는 것이며, 다음과 같이 사용합니다.

{% highlight Javascript %}
testFunction() {
	axios.get('api/test')
	.then(function(response) {
		console.log(response)
	})
}
{% endhighlight %}

간단하죠? .then()만 붙이면 그냥 결과값을 불러옵니다.

하지만 여기서 주의해야 할 점은, 순차적으로 처리하는 부분은 .then() 구문 내에서만 한정됩니다. 그게 무슨 소리고 하면, 아래와 같습니다.

{% highlight Javascript %}
testFunction() {
	axios.get('api/test')
	.then(function(response) {
		console.log(response)
	})

	console.log('Test')
}
{% endhighlight %}

뭐가 먼저 나올까요?

console.log('Test') 여기 결과인 'Test'가 먼저 나옵니다.

왜냐하면, response는 axios로 요청했을 때 결과가 나오면 그걸 받는거고, 위에서 언급했듯이 axios로 request만 하면 response가 오든지 말든지 다음 문장을 수행하기 때문입니다. 그래서 아래 문장 먼저 수행하고, response가 온 다음에 .then() 구문을 수행합니다.

이러면 이제 슬슬 뭐가 문제인지 보이겠죠?
바로 API를 여러 번 호출할 때 문제가 발생할 수 있습니다.

아래 예제를 볼까요?

{% highlight Javascript %}
testFunction() {
	axios.get('api/test')
	.then(function(response) {
		console.log(response)

		axios.get('api/test2')
		.then(function(response) {
			console.log(response)		
		})
	})
}
{% endhighlight %}

갑자기 코드가 지저분해졌습니다. 당혹스럽죠?

그래서 Promise만 가지고는 안되겠는지, async await를 사용하여 그 해결답안을 제시하겠습니다.


#### 3. Async - Await

먼저 async는 현재 사용할 함수를 비동기로 처리하겠다는 선언자입니다.
다음으로 await는 비동기로 순차 처리하기 위해서 수행할 API에 붙이는 선언자입니다.
말로만 설명하면 어렵겠죠? 아래 예제를 보겠습니다.

{% highlight Javascript %}
async testFunction() {
	const response = await axios.get('api/test')
	console.log(response)
}
{% endhighlight %}

위에 보시다시피 함수 앞에 async가 붙었습니다.
그리고 axios 함수에는 앞에 await가 붙었죠.

맨 처음 예제에서는 response에 이상한 값이 나왔지만, 위와 같이 사용하게 되면 원하는 값이 나타납니다.

await를 사용하여 API 함수에 대한 request를 보내고 다음 문장을 수행하는 것이 아니라, request에 대한 response까지 모두 다 받은 다음에 다음 문장을 수행하기 때문입니다.

그러면 API를 여러 번 호출한다면?

{% highlight Javascript %}
async testFunction() {
	const response = await axios.get('api/test')
	console.log(response)
	const response2 = await axios.get('api/test')
	console.log(response2)	
}
{% endhighlight %}

첫 번째 axios 함수를 사용하고 response를 출력하고, 두 번째 axios 함수를 사용한 다음 response를 출력합니다. 맨 위의 Promise의 마지막 예제와 동일합니다.

그런데 코드를 비교해도 뭔가 굉장히 깔끔해졌죠?
.then()을 괜히 중첩해서 사용하지 않아도 되고, 가독성도 좋아졌습니다.

이것이 바로 async - await의 올바른 사용법입니다.


#### 4. 반복문과 async - await

그런데 이게 또 반복문이 들어가면 골치아파지기 시작합니다.
일반적인 반복문인 forEach를 써보겠습니다.

{% highlight Javascript %}
testFunction() {
	const data = [1,2,3,4,5]

	data.forEach(async (e,i) => {
		const response = await axios.get('api/test', e)
		console.log(response)
	})
}
{% endhighlight %}

순차적으로 잘 나올까요? 결과는 아닙니다.

일단 await의 사용 범위는 해당 구문이 들어간 함수에 한정되어 있습니다. 
자세히 보시면 await는 forEach() 내부에서 수행되기 때문에, await도 testFunction()이 아닌 forEach() 내부에서 선언된 것을 확인할 수 있습니다.

이 부분이 매우 중요합니다. 그것이 바로 순차적으로 잘 안나오는 이유이기 때문입니다.

* 일단 forEach() 반복문을 시작해요. 그래서 axios로 request를 보냅니다. 
* console.log(response)는 당연히 axios의 response 값으로 올바르게 출력은 합니다.

하지만 여기서 중요한 것은 await 다음 구문은 axios의 response를 받아온 다음에 수행되는 것이 올바르나, 반복문 자체는 그렇지 않다는 것입니다.

다시 말하자면 첫 번째 반복문으로 axios 함수를 수행을 한 다음에, 바로 두 번째 반복문으로 넘어가고, 또 세 번째, 네 번째, 다섯 번째 반복문을 수행한다는 것입니다.

![ForEach Async-Await]({{ '/assets/img/img025_01.png' | prepend: site.baseurl }})

위 그림을 보면
* 1-1을 실행한 다음에는 1-2를 수행하는 것이 맞고
* 2-1을 실행한 다음에는 2-2를 수행하는 것이 맞고
* 3-1을 실행한 다음에는 3-2를 수행하는 것이 맞습니다.

하지만
* 1-2를 수행한 다음에 2-1을 수행하지 않고
* 2-2를 수행한 다음에 3-1을 수행하는 것이 아닙니다.
* 2-1은 1-1을 수행한 다음에, 
* 3-1은 2-1을 수행한 다음에 수행됩니다.

이러면 데이터가 엄청나게 꼬여버려서 뒤죽박죽 나오게 됩니다.
forEach 뿐만 아니라 map 함수를 사용해도 결과는 마찬가지고요.

그래서 사용되는 것이 reduce입니다.
reduce는 원래 수학 연산에 사용되는 반복문이지만, async - await 처리에 활용이 가능하며, 예제는 아래와 같습니다.

{% highlight Javascript %}
testFunction() {
	const data = [1,2,3,4,5]

	data.reduce((previous, current) => {
		return previous.then(async () => {
			await axios.get('api/test', current)
			console.log(response)
		})
	}, Promise.resolve())
}
{% endhighlight %}


reduce() 함수를 사용했을 떄 왜 async - await 처리가 가능한 지를 살펴보려면, 먼저 reduce()에 대한 이해가 우선되어야 합니다.
기본적으로 reduce() 함수는 previous와 current를 파라미터로 받으며, 수학 연산에서는 다음과 같이 사용됩니다.

{% highlight Javascript %}
testFunction() {
	const data = [1,2,3,4,5]
	data.reduce((previous, current) => previous + current)
}
{% endhighlight %}

여기서 연산하는 순서를 요약하자면,

* previous는 reduce() 반복문을 수행하면서 현재 이전까지 수행한 결과를 나타냅니다.
* current가 현재 반복문의 element입니다.

이를 참고하여 계산한다면,

* 첫 번째 반복: previous - undefined / current - 1 / return - 1
* 두 번째 반복: previous - 1 / current - 2 / return - 3
* 세 번째 반복: previous - 3 / current - 3 / return - 6
* 네 번째 반복: previous - 6 / current - 4 / return - 10
* 다섯 번째 반복: previous - 10 / current - 5 / return - 15

최종 결과는 15로 나타나며, 어떤 과정에 의해서 나오는지를 이제는 이해할 수 있을 것입니다.
반복을 수행할 때마다 return value가 나타나며, 해당 value는 다음 반복의 previous로 들어가서 또 다시 연산을 수행합니다.

위 예제는 reduce() 함수의 인자값을 1개로 하였을 때이고, 2개로 할 수도 있습니다.

{% highlight Javascript %}
testFunction() {
	const data = [1,2,3,4,5]
	data.reduce((previous, current) => { return previous + current }, 10)
}
{% endhighlight %}

여기에서 10은 초기값을 나타냅니다. 
바로 위 예제에서는 첫 번째 반복의 previous가 undefined였는데, 이를 10으로 대체한 것으로 보시면 됩니다.
최종 결과는 당연히 10 + 1 + 2 + 3 + 4 + 5 = 25가 나타나겠죠.

그렇다면 다시 원래 보려고 했던 예제로 가보겠습니다.

{% highlight Javascript %}
testFunction() {
	const data = [1,2,3,4,5]

	data.reduce((previous, current) => {
		return previous.then(async () => {
			await axios.get('api/test', current)
			console.log(response)
		})
	}, Promise.resolve())
}
{% endhighlight %}

왜 여기에서는 초기값으로 Promise.resolve()를 넣었을까요.
그것은 previous 변수의 유형을 결정하기 위한 것이 목적입니다.

인수 값이 1개가 들어가게 되었을 때 previous의 첫 번째 반복문의 값은 undefined라고 언급했습니다.
그런데 undefined 변수에서 previous.then()을 사용하면 과연 저것이 Promise 객체를 사용한다고 생각할까요? 
Javascript에서는 그런 식으로 인식하지 않습니다. 

그래서 previous 변수의 초기값을 Promise.resolve()로 선언하게 되면, 첫 번째 값은 빈 값이지만 Promise 객체형 값을 가진다는 것을 인식하게 됩니다.
아까 Promise를 다룰 때 resolove()에 대해서는 언급하지 않았는데, 이 부분은 앞서 Promise에서 .then()을 사용 완료했다라는 것과 동일한 기능을 수행하며,
만약 return 값이 있을 경우에는 resolve() 함수의 파라미터로 return 값도 들어가게 됩니다.

위와 같은 형태로 되면, 반복 수행 결과는 다음과 같습니다.

* 첫 번째 반복: previous - Promise.resolve() / 1을 파라미터로 한 axios request 및 response를 포함한 결과
* 두 번째 반복: previous - 첫 번쨰 반복 수행 결과 / 2를 파라미터로 한 axios request 및 response를 포함한 결과
* 세 번째 반복: ...

위와 같이 수행하면 반복문에 의해서 API 함수를 호출하더라도 순차적으로 호출할 수가 있습니다.

오랫만에 블로그에 글을 쓰려니 눈앞이 깜깜하고 정신이 살짝 없긴 합니다만, 그동안은 프로젝트에 매진하느라 뜸했던 점 양해 바라며 앞으로는 프로젝트 수행 중 공유할 만한 자료 위주로 지속적으로 공유하겠습니다.