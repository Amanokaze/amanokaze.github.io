---
layout: post
title:  "AWS 머신러닝 Amazon Sagemaker 노트북 인스턴스 생성"
date:   2019-10-17
categories: [ML]
tags: [Python, 파이썬, Python37, Google, Cloud, Django, 장고, MySQL, SQL, 구글, 클라우드]
---

 
 
Amazon Sagemaker – Jupyter Notebook 사용 방법에 대해서 글을 써 보겠습니다.

글을 설명하기 앞서서 말씀드리자면, 본문 첨부된 이미지에 http://onikaze.tistory.com 썸네일이 있는 것을 확인할 수 있는데, 해당 블로그는 제가 그 블로그의 이미지를 무단 도용한 것이 아니라, 그 블로그가 제 예전 블로그였고 이 글 역시 예전 블로그에 썼던 글을 이 곳으로 이관한 것입니다. 이 점 참고 바라겠습니다.

Amazon Sagemaker는 웹 개발 도구인 Jupyter Notebook과 JupyterLab를 사용하여 Amazon에서 제공하는 머신러닝 API를 사용할 수 있게 제공하는 하나의 서비스로 보시면 됩니다. 자세한 부분은 아래 홈페이지를 참고하시면 관련된 문서가 있으니 참고하시면 될 것입니다.

<https://aws.amazon.com/ko/sagemaker/>
 

Amazon에서 머신러닝과 관련하여 AWS에서 서비스를 제공한 것은 2017년이였던 것 같습니다. 그래서 AWS Management Console에서 'Machine Learning'이라는 메뉴도 있었고요. 다만 머신러닝은 미국 동부 오하이오주 리전에서만 사용이 가능했었기 때문에 실제로 써 본적은 없었습니다.

하지만 AWS Machine Learning 서비스는 이젠 AWS에서 더 이상 제공하지 않습니다. 왜냐하면 AWS에서는 이젠 더 이상 머신러닝이 하나의 메뉴가 아니라, 하나의 카테고리로 상위 계층으로 올렸고, 머신러닝 관련하여 많은 서비스를 제공하고 있기 때문입니다. 아래 사진을 보시면 AWS Machine Learning을 들어갔을 때 나오는 초기화면입니다만, 상단 경고 문구와 같이 'AmazonML is no longer available to new customers.'라고 되어 있고, 대신 머신러닝을 위한 기초적인 서비스인 SageMaker로 이동하라는 형태로 나타나 있습니다. 

![Sagemaker Dashboard]({{ '/assets/img/img010_01.png' | prepend: site.baseurl }})
 

AWS의 머신러닝이 이처럼 카테고리로 재편되고 다양한 서비스를 제공하기 시작하면서, 리전 역시 미국 동부 오하이오주 리전에 제한되지 않고, 수많은 리전에서 사용할 수 있도록 하였고, 다행스럽게도 서울 리전(ap-northeast-2)에서도 머신러닝을 이용할 수가 있습니다.

![Sagemaker Menu]({{ '/assets/img/img010_02.png' | prepend: site.baseurl }})

 

AWS에서 제공하는 머신러닝 서비스는 위와 같이 매우 다양하게 구성되어 있으며, 이번 글에서는 첫 번째 서비스인 Amazon SageMaker에서 Jupyter Notebook을 어떻게 사용하는 지에 대해서 다루도록 하겠습니다.

 

Amazon SageMaker는 아래 화면과 같이 기계 학습 모델을 대량으로 빌드, 훈련 및 배포하는 서비스로, 머신러닝을 위해서 기계학습을 하기 위한 서비스를 제공합니다. 기계학습은 머신러닝을 위한 기본 과정으로, SageMaker에서는 Jupyter Notebook 기반의 노트북 인스턴스를 생성하고, AWS에서 제공하는 머신러닝 API도 더불어 제공하여 기계학습을 더욱 쉽게 할 수 있도록 합니다. 물론 SageMaker의 노트북 인스턴스는 다른 AWS 서비스와 마찬가지로 사용량에 따라 요금이 부과되므로 이 점도 참고하시면 될 것 같습니다.

일단 노트북 인스턴스를 생성하겠습니다. 위 사진의 우측 상단 ‘노트북 인스턴스 생성’을 눌러서 들어갑니다.

![Sagemaker Dashboard]({{ '/assets/img/img010_03.png' | prepend: site.baseurl }})

먼저 노트북 인스턴스 설정입니다. 기본적인 이름, 유형 등을 입력합니다. 기본값으로 입력하고 넘어갈게요.

![Sagemaker Creation 1]({{ '/assets/img/img010_04.png' | prepend: site.baseurl }})

다음은 권한 및 암호화입니다. 첫 번째는 IAM역할로, 처음 생성할 경우에는 기존 역할이 없을 것입니다. 새 역할 생성을 눌러줍니다.

![Sagemaker Creation 2]({{ '/assets/img/img010_05.png' | prepend: site.baseurl }})

새 역할은 다음과 같이 권한 설정을 물어봅니다. 여기에서는 AWS 스토리지인 S3 버킷에 대한 권한을 물어보는 부분이 나와있습니다. 기본적으로 머신러닝을 수행하기 위해서는 훈련(Training) 데이터와 검증(Valdation) 데이터가 있어야 하며, 머신러닝을 위한 데이터는 주로 대용량의 데이터를 다루는 경우가 많기 때문에, 이러한 데이터를 저장하기 위해서는 S3 버킷과의 연동은 당연히 필수겠죠. 하지만 아무 S3 버킷을 선택할 수는 없기 때문에 어떤 유형의 버킷에 권한을 부여할 것인지를 결정하는 것으로 보시면 됩니다.

권한 설정은 편하실 대로 결정하시면 될 것 같습니다.

![Sagemaker Creation IAM]({{ '/assets/img/img010_06.png' | prepend: site.baseurl }})

그 외 부분은 기본값으로 놔둡니다. 세부 설정은 좀 더 다양하고 고급스러운 기능을 필요로 할 때 지정하며, 노트북 인스턴스 생성 후 수정이 가능하므로 참고하시면 될 것 같습니다.

![Sagemaker Creation 3]({{ '/assets/img/img010_07.png' | prepend: site.baseurl }})

노트북 인스턴스 생성까지 누르면 생성을 진행합니다. 생성에 걸리는 시간은 3분 정도 걸립니다.

![Sagemaker Creation In-progress]({{ '/assets/img/img010_08.png' | prepend: site.baseurl }})

 

생성이 완료되면 다음과 같이 'InService'로 상태가 변경되고, 오른쪽에 'Open Jupyter / Open JupyterLab' 링크가 있는 것을 확인할 수 있습니다. 각각을 눌러서 초기화면이 어떻게 나오는지를 간단히 살펴볼게요.

![Sagemaker Creation Completed]({{ '/assets/img/img010_09.png' | prepend: site.baseurl }})

 

먼저 Jupyter입니다. Python Jupyter Notebook을 실행했을 때의 초기화면과 동일하게 나타나네요. 지원하는 언어는 R, Sparkmagic, Python 등 다양한 언어를 지원합니다.

![Jupyter]({{ '/assets/img/img010_10.png' | prepend: site.baseurl }})

 

다음은 JupyterLab입니다. 역시 Jupyter Notebook과 같은 형태의 언어를 지원하며, JupyterLab 초기화면과 동일한 형태로 나타납니다.

![Jupyterlab]({{ '/assets/img/img010_11.png' | prepend: site.baseurl }})

 

그러면 다시 Jupyter Notebook으로 돌아가서, Sagemaker Examples 탭으로 들어가볼게요. 여기를 들어가면 Sagemaker에서 기계학습을 위한 다양한 예제가 있습니다. 일단 Sagemaker에서 지원하는 Amazon Algorithms 메뉴에는 다음과 같은 예제가 있습니다.

![Jupyter Examples]({{ '/assets/img/img010_11.png' | prepend: site.baseurl }})

  

그 외 다른 예제 유형의 카테고리는 다음과 같습니다. 카테고리만 14개가 있을 정도니 예제가 엄청 많이 있다는 사실을 유추할 수 있겠죠.

![Jupyter Examples 2]({{ '/assets/img/img010_13.png' | prepend: site.baseurl }})

Sagemaker 노트북 인스턴스 생성은 여기까지입니다. 다음 글은 Sagemaker 노트북 인스턴스에서 제공하는 예제를 실행해보도록 하겠습니다. 감사합니다.