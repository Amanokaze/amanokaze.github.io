---
layout: post
title:  "AWS Elastic Beanstalk에서 HTTPS 사용하기"
date:   2019-10-15
image:  "assets/img/img004_01.png"
categories: [Books]
tags: [AWS, Cloud, Django, Amazon, books, 장고, 클라우드, Application, 애플리케이션, Authenticate, 인증, 인증서, HTTPS, Python, 파이썬]
---

이 글은 이번 책에는 현재 없는 내용이지만, 차후 2판 인쇄를 할 때 들어갈 수도 있는 내용임을 먼저 사전에 알려드리고자 합니다.

이번에 책을 집필하면서 많은 내용을 수록하였습니다만, 추가적으로 알아두면 좋을 내용도 일부 확인한 후에 이 블로그를 통해서 지속적으로 업데이트하는 방향으로 하겠습니다.

이번 글은 AWS Elastic Beanstalk에서 HTTPS를 사용하는 방법에 대해서 다룹니다.

최근 모든 웹 사이트는 HTTP가 아닌 HTTPS를 권장하는 추세이며, 이는 좀 더 보안이 강화된 웹사이트의 연결을 위한 조치로 볼 수 있습니다. 이에 따라 Django 웹 애플리케이션으로 구축된 사이트에서도 HTTPS를 적용하는 방법에 대해서 다룰 예정인데요. 여기에서 하나 참고해야 할 부분은, 이 내용의 주된 출처는 AWS 공식 기술문서에 이미 기재된 내용임을 먼저 알려드리고자 하며, 다만 이 블로그에서는 상세한 이미지나 순서까지도 같이 추가를 하여 한 번에 나타내므로 더욱 알아보기 쉽게 제공하고자 합니다.

출처: AWS 기술문서

ACM 공인인증서 생성:
<https://docs.aws.amazon.com/ko_kr/acm/latest/userguide/gs-acm-request-public.html>

 
공인 인증서 요청 - AWS Certificate Manager

공인 인증서 요청 다음 단원에서는 ACM 콘솔 또는 AWS CLI를 사용하여 공인 ACM 인증서를 요청하는 방법을 설명합니다. 인증서 요청 시 문제가 있으면 인증서 요청 문제 해결을 참조하십시오. .IO 도메인에 대한 인증서 요청 시 문제가 있으면 .IO 도메인 문제 해결을 참조하십시오. 사설 인증 기관(CA)을 사용하여 사설 인증서를 요청하려면 사설 인증서 요청 단원을 참조하십시오. 콘솔을 사용하여 공인 인증서 요청 ACM 공인 인증서를 요청하려면(콘솔

docs.aws.amazon.com
Elastic Beanstalk의 HTTPS 사용:
<https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/configuring-https.html>

 
Elastic Beanstalk 환경에 대한 HTTPS 구성 - AWS Elastic Beanstalk

Elastic Beanstalk 환경에 대한 HTTPS 구성 Elastic Beanstalk 환경에 대한 사용자 지정 도메인 이름을 구매하고 구성한 경우, HTTPS를 사용하여 사용자가 안전하게 웹 사이트에 연결할 수 있습니다. 도메인 이름이 없는 경우, 자체 서명된 인증서가 있는 HTTPS를 개발 및 테스트 용도로 사용할 수 있습니다. 사용자 데이터 또는 로그인 정보를 전송하는 애플리케이션에서 HTTPS는 필수입니다. Elastic Beanstalk 환

docs.aws.amazon.com
이번 게시물은 위 2개의 페이지에서 참조하였음을 다시한번 알려드립니다.

 

**※ 전제조건: 본인 소유의 도메인을 반드시 하나 가지고 있어야 하며, Route 53에서 관리하고 있는 도메인이면 더욱 좋습니다.**

 

### 1. ACM 공인인증서 생성

ACM이라 함은 AWS Certificate Manager의 줄임말로, AWS의 인증서 관리를 위한 서비스입니다. 이 서비스에서는 공인인증서, 사설인증서 등을 모두 생성할 수 있으며, 여기서 생성된 공인인증서를 통해서 HTTPS를 사용하도록 구성되어 있습니다.

ACM을 사용하기 위해서는 서비스에 그냥 'acm'이라고 검색하면 'Certificate Manager'가 팝업으로 뜹니다. 그 곳으로 들어가면 됩니다.

![Certificate Manager](/assets/img/img004_01.png)

시작하기가 2개가 있을 겁니다. 첫번째 '인증서 프로비저닝'을 선택합니다.

![Authenticate Provisioning](/assets/img/img004_02.png)

인증서 요청은 공인, 사설 인증서가 있으며, 공인인증서를 선택합니다.

![Authenticate Request](/assets/img/img004_03.png)

이제 도메인 이름을 추가합니다.

![Add a domain name](/assets/img/img004_04.png)


저는 이미 도메인을 보유하고 있으므로, 보유하고 있는 도메인을 입력합니다.

여기서 이름 입력할 때에는 도메인 이름 앞에 '*' 기호를 넣어 줘야, 여러 호스트의 도메인을 모두 사용할 수 있습니다.

![Add a domain name - asterisk](/assets/img/img004_05.png)


그리고 하나 더 추가해야 합니다.

앞서 *.awsdjangoboard.com 을 사용했지만, 이 이름은 awsdjangoboard.com, 즉 호스트가 없는 도메인은 포함하지 않기 때문에,  호스트가 없는 도메인에 대해서도 추가를 해야 합니다.

![Add a domain name - none](/assets/img/img004_06.png)

다음은 검증 절차입니다. 입력한 도메인이 본인이 소유하고 있는 도메인인지를 검증해야 되겠죠.

검증 방법은 DNS 검증과 e-mail 검증이 있는데, DNS 검증은 DNS 구성 수정 권한이 있을 때 검증하는 방식이고, e-mail  검증은 도메인 등록 시 사용했던 e-mail을 사용하는 방법입니다. Route 53에서 도메인을 관리하고 있다면 DNS 검증을 권장합니다.

![Verification Method](/assets/img/img004_07.png)

이제 입력된 내용을 확인합니다. 이상이 없으면 '확인 및 요청'을 눌러서 진행합니다.

![Next Step](/assets/img/img004_08.png)

이제 요청 진행 중입니다.

검증 보류라고 한 이유는 간단합니다. DNS 검증을 하겠다고 했지, 실제 검증한 것이 아니기 때문에 검증이 완료된 것이 아니므로 보류라고 나타난 것입니다. 이제 실제 검증을 위해서 계속 진행합니다.

![Verification Next Step](/assets/img/img004_09.png)

다음 화면으로 이동한 다음에 도메인을 확인하면, 아직 검증 미완료 상태가 유지된 채로 나옵니다.

하지만 도메인의 DNS 구성에 CNAME 기록을 추가하여 검증할 수 있으며, 검증 이름 및 값은 아래와 나옵니다. 다행스럽게도, 저 이름과 값은 확인하지도 외우지 않아도 됩니다. 왜냐하면 아래의 파란색 'Route 53에서 레코드 생성' 버튼을 누르면 해당 CNAME 기록을 곧바로 추가할 수 있기 때문입니다.

파란색 'Route 53에서 레코드 생성'을 눌러줍니다.

![Add a record set](/assets/img/img004_10.png)

그러면 다음과 같은 화면이 나오면서 '생성'을 누르면 바로 추가할 수 있습니다.

![Create a record set](/assets/img/img004_11.png)

이제 DNS 검증 준비는 모두 완료되었습니다. 실제 검증이 완료되기까지는 시간이 조금 걸리므로 기다려야 합니다.

![Wait a record set](/assets/img/img004_12.png)

한 3분~5분 정도 지나면 검증이 완료되었다고 하면서 내용을 자동으로 갱신해서 보여줍니다. 

![Verification Completed](/assets/img/img004_13.png)

Route 53을 이동해도 정상적으로 등록된 것을 확인할 수 있으니 더불어 참고 바랍니다.

![Route 53](/assets/img/img004_14.png)

 

### 2. Elastic Beanstalk에 ACM 인증서 등록

이제 Elastic Beanstalk로 이동합니다. 

![Elastic Beanstalk](/assets/img/img004_15.png)

애플리케이션을 선택한 후, 좌측의 '구성'으로 들어갑니다.

![EB Configuration](/assets/img/img004_16.png)

아래로 조금만 내리면 '로드 밸런서'가 있습니다. 로드밸런서의 '수정'을 눌러서 진행합니다.

![LB Modification](/assets/img/img004_17.png)

첫 화면에서 아마 기본 HTTP 포트인 80만 등록되어 있을 것입니다. '리스너 추가'를 눌러서 추가를 진행해 줍니다.

![Add a LB Listener](/assets/img/img004_18.png)

추가 사항은 아래와 같이 입력해 줍니다. 

여기에서 리스너 포트/프로토콜은 443/HTTPS로 해야 하고, 인스턴스 포트/프로토콜은 80/HTTP로 해야 합니다.

저도 이 부분에서 실수가 있어서 인스턴스 포트/프로토콜도 동일하게 HTTPS로 했었는데, 반드시 HTTP로 해야 합니다. 아래와 같이 해야 하는 이유는, Elastic Beanstalk를 사용한 웹 애플리케이션이 기본은 HTTP 포트를 사용하지만 HTTPS를 사용할 때에도 HTTP와 동일한 페이지로 접속하기 위해서 연결시켜주는 개념으로 보시면 됩니다.

그리고 아래 SSL 인증서를 위에서 생성했던 ACM 인증서를 선택합니다. ACM 인증서를 발급했던 이유도 Elastic Beanstalk에서 HTTPS 추가 시 인증서를 선택하기 위한 사전 작업으로 보시면 됩니다.

![Class Load Balancer Listener](/assets/img/img004_19.png)

이제 거의 다 왔습니다. '생성 대기중'으로 표시된 것을 확인할 수 있습니다. 이제 적용하러 아래로 스크롤합니다.

![LB 443](/assets/img/img004_20.png)

우측 하단의 '적용'을 눌러서 적용시켜줍니다.

![LB Mod Completed](/assets/img/img004_21.png)

이제 Elastic Beanstalk 애플리케이션을 다시 실행합니다. 재실행하는 데 평균 1~3분이 걸리니 기다려야 되겠죠.

![EB Dashboard](/assets/img/img004_22.png)

적용이 완료되었네요. HTTPS를 넣어서 'https://www.awsdjangoboard.com'을 실행해 보겠습니다.

![Enter a site](/assets/img/img004_23.png)

정상적으로 HTTPS로도 웹사이트가 열린 것을 확인할 수 있습니다.

![Completed](/assets/img/img004_24.png)

 

이상으로 Elastic Beanstalk에서 HTTPS를 사용하는 방법에 대해서 마치도록 하겠습니다.

이 내용은 만약 2판 발행하게 될 경우 잘 정리해서 추가 반영하겠습니다.

