---
layout: post
title:  "Google Cloud Vision API 사용 예제"
date:   2019-10-17
image:  "assets/img/img009_01.png"
categories: [Cloud]
tags: [Python, 파이썬, AWS, Amazon, 아마존, EC2, Instance, 인스턴스, Vision, API, 비전, Ubuntu, 우분투, Google, Cloud, SDK, 구글, 클라우드]
---

 
지금 쓰려는 글은 Google Cloud에도 나와 있는 내용이고, 유사한 내용으로 다른 블로그에도 있을 것입니다.

그럼에도 불구하고 Vision API 사용예제를 쓰려는 이유는 제가 추후 진행할 연구의 선행과제로 다루기 위함입니다.

 

이 점 참고하시기 바라며, 저는 왠만해서 공식 사이트의 레퍼런스를 가지고 와서 글을 쓰지 다른 블로그의 내용을 참고해서 글을 쓰는 일은 없을 것이며 혹여나 있더라도 출처를 명확히 밝히고 있으니 참고바랍니다.

 

Vision API를 사용하기 위해서 딱 필요한 절차만 언급하고, 그 전의 사전 절차는 모두 생략하니 더불어 참고바랄게요.

 

### 1. API 인증을 위한 서비스 계정 키 생성

일단 Vision API를 사용하려면, API 사용을 위한 인증이 있어야 합니다.

먼저 메뉴의 '<font color=#F00><b>API 및 서비스 - 사용자 인증 정보</b></font>' 여기로 들어가시면 됩니다.

 

Google Cloud 관련 문서에 보면 중간중간 Credentials 라는 영어단어가 나오는데, 그거 전부 아래 사진의 '사용자 인증 정보' 이 부분에 해당되는 부분이니 참고하시면 됩니다.

![Credential Menu](/assets/img/img009_01.png)
 
 

그래서 사용자 인증 정보를 만들어야 하는데, '서비스 계정 키', 'API 키' 두개 다 만들어야 합니다. 다만 API 키는 당장 만들지 않아도 큰 문제 없으며, 아래 언급대로 특정 권한 확인용으로만 사용되므로 <u>이 글에서는 다루지 않겠습니다.</u>

![Create Info](/assets/img/img009_02.png)


 

#####서비스 계정 키: 새 서비스 계정 선택해 주시고요.

#####키 유형: JSON 으로 선택해주시기 바랍니다.

 

JSON으로 해야 REST로 API 사용할 수 있으므로 필수입니다.

![Create Service Accout](/assets/img/img009_03.png)

 


'생성'까지 누르면 JSON 파일을 다운로드합니다. 이제 API 사용 인증을 위한 서비스 계정 키 생성이 완료됐습니다.

 

 

### 2. 서비스 계정 키 환경 변수 추가

다음은 서비스 계정 키를 환경 변수에 추가해야 합니다. 그런데 환경 변수에 추가하려면 서비스 계정 키가 실행하려는 호스트에 있어야 되겠죠?

 

리눅스에서는 환경 변수 추가를 export 명령어로 다음과 같이 수행합니다.

{% highlight Shell %}
export GOOGLE_APPLICATION_CREDENTIALS="[경로]"
{% endhighlight %}

실제 사용 예제는 다음과 같겠죠.

{% highlight Shell %}
export GOOGLE_APPLICATION_CREDENTIALS="/home/ubuntu/google/google_key.json"
{% endhighlight %}

사실 Windows는 안해봐서 잘 모르겠습니다만, 아마 내컴퓨터 - 고급 - 환경변수 여기서 추가하는게 아닐까 싶습니다. 

여기서 JSON 파일은 위에서 다운로드받은 그 JSON 파일이 맞으니 주의 바라겠습니다.

 

 

### 3. Vision API 예제 코드 실행

Vision API 예제 코드는 다른 것에서 고민할 필요 없습니다. Google Cloud에 다 나와 있으니 해보시면 됩니다. 
REST API를 사용하여 실행할 예정이므로, 명령어는 curl을 사용할게요.

Google Cloud 예제는 다음과 같습니다.

 

<https://cloud.google.com/vision/docs/ocr#vision-detect-text-cli-curl>

 
위 구글 문서 중 텍스트 감지 - Curl 명령어로 사용한 예제를 실행해 보겠습니다.

한글이 잘 나오는지 확인하기 위해서 예제에 있는 URI 말고, 인터넷에 돌아다니는 배너광고를 하나 넣겠습니다.

 

<https://ssl.pstatic.net/tveta/libs/1249/1249571/7842d5f1c61a007c932e_20191004171838293.jpg>

![Speakingmax Banner](/assets/img/img009_04.png)

<small><b><font color=#F00>※ 스피킹맥스 배너광고 절대 아닙니다! 위 이미지 파일이 예제 파일입니다. 어차피 클릭해도 안넘어갑니다.</font></b></small>

 

 
{% highlight Curl %}
curl -X POST \
     -H "Authorization: Bearer "$(gcloud auth application-default print-access-token) \
     -H "Content-Type: application/json; charset=utf-8" \
     --data "{
      'requests': [
        {
          'image': {
            'source': {
              'imageUri': 'https://ssl.pstatic.net/tveta/libs/1249/1249571/7842d5f1c61a007c932e_20191004171838293.jpg'
            }
          },
          'features': [
            {
              'type': 'TEXT_DETECTION'
            }
          ]
        }
      ]
    }" "https://vision.googleapis.com/v1/images:annotate"
{% endhighlight %}


그래서 실행하면, JSON 형태로 출력합니다. 위 배너광고하고 비교해봤을 때, 한글 추출 잘 해내는걸 확인할 수 있습니다.

![Contains Hangul Result](/assets/img/img009_05.png)

만약에 서비스 계정 키가 생성되어있지 않다면?

 
![Permission Failed](/assets/img/img009_06.png)


이런 식으로 Permission Denied가 나올 것입니다. 그래서 서비스 계정 키를 생성해야 하니 참고 바랍니다.

 

 

REST API 말고 Python 언어를 사용해서 Vision API를 사용하는 예제도 실행해봤었는데, 한글 지원이 잘 안되는 문제가 있어서 추가 확인이 필요할 것으로 보입니다. 다만 이번 Vision API 관련해서는 REST API를 활용하는 부분이 향후 연구할 과제와 더욱 부합할 것으로 판단되므로 Python에서 억지로 Vision API를 구현하기보다는 REST API로 구현하고 Python에서 연동하는 방안으로 고려해서 진행할 예정입니다.

 

사실 이미 생각한 연구 과제도 어느 정도 윤곽은 잡은 상태입니다만, 여기에서 굳이 공개하기보다는 추후 개발이 진행되는대로 관련 기술을 이 블로그에 지속적으로 공유할 예정이니 참고하시기 바랍니다.

 

 

이상으로 REST API를 사용하여 curl 명령어로 Vision API를 사용하는 예제에 대해서 마치도록 하겠습니다.