---
layout: post
title:  "EC2 인스턴스(Ubuntu)에서 Google Cloud SDK 설치하기"
date:   2019-10-17
image:  "assets/img/img008_01.png"
categories: [Cloud]
tags: [Python, 파이썬, AWS, Amazon, 아마존, EC2, Instance, 인스턴스, Ubuntu, 우분투, Google, Cloud, SDK, 구글, 클라우드]
---

AWS EC2 인스턴스에서 Google Cloud SDK 설치하는 법을 다루도록 하겠습니다.

설치할 인스턴스 OS는 Ubuntu로 할게요.

 

사실 설치하는 방법은 간단합니다. Google에서 하라는대로 하면 되니까요.

 

이 페이지에 나와있는대로 하면 됩니다. 구글 클라우드 공식 사이트입니다.

<https://cloud.google.com/sdk/docs/downloads-apt-get?hl=ko>

 

그런데 여기 있는대로만 하라면 이 글이 필요가 없겠죠?

하지만 다행히도(?) AWS EC2 인스턴스에서는 Google Cloud SDK 설치를 위한 추가 절차가 사실 필요합니다.

 

그러면 위 사이트에 있는 순서대로 설치를 진행하면서 알려드리도록 하겠습니다.

 

일단 각 구문 세부 설명은 생략할게요. 위 사이트에 다 있는 내용입니다.

 

1. 환경변수 생성

{% highlight Shell %}
export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)"
{% endhighlight %}

2. 패키지 소스 URI 추가

{% highlight Shell %}
echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
{% endhighlight %}
 

3. Google Cloud 공개키 가져옴

{% highlight Shell %}
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
{% endhighlight %}

![Google CLoud Private Key Import](/assets/img/img008_02.png)

4. Google Cloud SDK 설치

{% highlight Shell %}
sudo apt-get update && sudo apt-get install google-cloud-sdk
{% endhighlight %}

![Google Cloud SDK Installation](/assets/img/img008_01.png)
<small>(앞 15줄만 캡쳐, 실제로 내용 더 있습니다)</small>

5. Google Cloud 시작

{% highlight Shell %}
gcloud init
{% endhighlight %}

사실 4번까지는 순조롭게 됩니다. 그런데 여기가 문제입니다.

이 부분도 아무 문제없이 설치되었으면 이 글 쓰지도 않았을 겁니다.

![Required Login in gcloud](/assets/img/img008_03.png)


로그인하고 설치하라고 나오죠. 그리고 아래 주소 들어가라고 나오죠.

 

이게 무슨 뜻이냐면,

 

**1. 구글 클라우드 서비스 이용하고 프로젝트까지 생성해야 한다.**

**2. EC2 인스턴스에 브라우저를 볼 수 있어야 한다.**

 

위 두가지가 선행작업으로 되어 있어야 합니다. 그렇지 않으면 진행 안 될 것입니다.

 

먼저 첫 번째, 구글 클라우드 서비스 이용 및 프로젝트 생성 방법은 Google Cloud SDK를 이용하려는 고객이면 왠만해서 다 생성까지 되어 있을 것입니다.

 

그 다음 두 번째, EC2 인스턴스에서 브라우저를 열어서 사용하는 방법은 크게 두 가지가 있습니다. 


 
1) putty, Xming을 설치한 후 putty에서 X11 터미널 사용 설정을 한 후 Xming을 사용해서 Firefox를 설치한 후 사용

2) MobaXterm을 설치한 후 X11 사용 설정 체크 후 Firefox를 설치한 후 사용

 

**사실 위 두 가지 설치 방법은 인터넷 찾으면 확인할 수 있으므로 방법에 따라 진행해주셔도 되고요,**

**아니면 제가 최근에 집필한 책에도 상세히 나와 있으니 참고하시면 될 것 같습니다.**

(이럴 때 책 홍보라도 해 봤습니다 양해 부탁드려요)

<table class="tt-plugin-interpark" style="background: #fff; border: 1px solid #e0e0e0; width: 408px;" border="0" cellspacing="0">
<tbody>
<tr>
<td style="padding: 9px;">
<div style="float: left; width: 320px; height: 24px; vertical-align: middle; overflow: hidden;"><a style="float: left; line-height: 24px; font-weight: bold; letter-spacing: -1px; color: #444 !important; text-decoration: none !important; background: url('//t1.daumcdn.net/tistory_admin/static/images/icon_ipark_book.gif') no-repeat; padding-left: 30px;" href="http://book.interpark.com/blog/integration/product/itemDetail.rdo?prdNo=316045261&amp;refererType=8303&amp;bookblockname=bpmain_in&amp;booklinkname=wg_search_AE95511CE5E756D3BF8BD10A7E6AAA00F786CE8B53E9F4070FBB98A441CD2B22&amp;key=AE95511CE5E756D3BF8BD10A7E6AAA00F786CE8B53E9F4070FBB98A441CD2B22" target="_blank" rel="noopener">AWS 클라우드 기반의 Django 웹 애플리케이션</a><span style="float: left; font: 11px dotum, sans-serif; color: #777; letter-spacing: -1px; padding-left: 10px; line-height: 24px;">신성진</span></div>
<a style="float: right; line-height: 24px; margin-left: 20px; background: url('//t1.daumcdn.net/tistory_admin/static/images/icon_ipark_detail.gif') no-repeat 0 6px; overflow: hidden; display: block; width: 44px; height: 24px; text-indent: -1000em;" href="http://book.interpark.com/blog/integration/product/itemDetail.rdo?prdNo=316045261&amp;refererType=8303&amp;bookblockname=bpmain_in&amp;booklinkname=wg_search_AE95511CE5E756D3BF8BD10A7E6AAA00F786CE8B53E9F4070FBB98A441CD2B22&amp;key=AE95511CE5E756D3BF8BD10A7E6AAA00F786CE8B53E9F4070FBB98A441CD2B22" target="_blank" rel="noopener">상세보기</a></td>
</tr>
</tbody>
</table>


어쨌든, 이대로 하면 다음과 같이 실행할 수 있습니다.

 
![gloud execution](/assets/img/img008_04.png)

Firefox를 백그라운드로 실행해주시고, 그런 다음 아래 사이트가 나타나면 사이트를 복사해주신 후 Firefox에서 붙여넣기 해주시기 바랍니다. 그래서 거기서 인증 하라는 대로 해주시고 완료하면, 다음과 같이 결과가 나타날 것입니다.

![firefox authorization](/assets/img/img008_05.png)


중간에 프롬프트에서 간혹 아래와 같이 나오는 경우 있습니다만, 무시하셔도 됩니다. 브라우저에서 진행만 완료되면 문제없습니다.

![Ignore Warning](/assets/img/img008_06.png)


완료되면 다음과 같이 나타날 것입니다.

![gcloud login completed](/assets/img/img008_07.png)


프로젝트 선택하시고요. 없으면 새로 만들어도 됩니다. 그 다음 리전 선택하는 부분 나오는데, 이거는 사실 안해도 상관은 없을 것 같습니다.

 

참고로 말씀드리자면, Google Cloud에서 아직 한국쪽은 없습니다. 그러므로 가장 가까운 일본쪽을 하셔야 하고요. asia-northeast1 이 도쿄, asia-northeast2가 오사카입니다.

도쿄에서 거의 대부분의 Google Cloud 서비스를 지원하므로 asia-northeast1을 추천합니다.

![gcloud installation](/assets/img/img008_08.png)


설치가 완료되었네요. 잘 이용하시기 바랍니다.

 

글 마치겠습니다.