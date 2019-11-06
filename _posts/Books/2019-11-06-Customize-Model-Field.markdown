---
layout: post
title:  "Django InspectDB 사용 후 날짜 필드를 현재 시간으로 수정하는 방법"
date:   2019-11-06
categories: [Books]
tags: [AWS, Cloud, Django, Amazon, books, 장고, 클라우드, Application, 애플리케이션, Python, 파이썬]
---

안녕하세요. 책 관련하여 추가 수정사항이 있어서 글을 올리게 되었으며, Django에서 InspectDB 스크립트를 사용하여 DB 테이블을 Models.py 파일에 입력할 때 수정해야 할 사항이 누락된 것이 있어서 추가로 입력하니 참고 바랍니다.

책 295P에 보시면 다음과 같은 부분이 있습니다. (실제 책에는 board_images도 있지만 이 부분은 오타인 관계로 제외 예정입니다)

{% highlight Shell %}
(ve)$ python manage.py inspectdb board_categories boards board_replies board_likes >> boardapp/models.py
{% endhighlight %}

여기서 InspectDB를 사용하여 models.py에 모델 클래스 및 필드를 생성할 때에는 외래키 및 기본값만 들어가게 됩니다.

하지만 파일 첨부에 사용되는 FileField, ImageField 등과 날짜 입력에 사용되는 DateField, TimeField, DateTimeField의 경우에는 상황에 따라 일부 내용을 수정해야 합니다.

책에 있는 P295의 소스코드에서 변경이 필요한 필드를 한번 살펴보도록 할게요.

{% highlight Python %}
08: creation_date = models.DateTimeField(default=timezone.now)
09: last_update_date = models.DateTimeField(default=timezone.now)
...
20: registered_date = models.DateTimeField(default=timezone.now)
21: last_update_date = models.DateTimeField(default=timezone.now)
...
23: image = models.ImageField(upload_to="images/%Y/%m/%d", blank=True)
...
34: registered_date = models.DateTimeField(default=timezone.now)
35: last_update_date = models.DateTimeField(default=timezone.now)
...
45: registered_date = models.DateTimeField(default=timezone.now)
{% endhighlight %}

#### 이미지 필드 수정

이미지 필드 수정은 다행히도 297P의 'Image 필드' 부분 설명에 어떤 방식으로 수정해야 할 지가 정확하게 명시가 되어 있습니다.
그러므로 이 부분에 대해서는 특별히 언급하지 않겠습니다.

#### 날짜 필드 수정

날짜 필드 역시 이미지 필드 수정사항과 동일한 형식으로 추가 입력하니 참고 바라겠습니다.

- DateTime 필드(8,9,20,21,34,35,45 Line): 게시판 관련 모델의 'creation_date','last_update_date','registered_date' 필드는 inspectdb 스크립트로 생성할 경우, 위 코드에 나타난 값이 아닌 '(blank=True, null=True)'의 형태로 생성된 것을 확인할 수 있다. DB에서 컬럼을 생성할 경우, 이들 날짜 필드에 대한 별다른 제약 조건이 들어가지 않았기 때문에 모델 필드를 생성할 때에도 역시 공백값을 기본값으로 한 것으로 볼 수 있다. 

게시판 모델에서 사용되는 날짜를 수동으로 입력할 경우에는 이 부분을 그대로 두어도 문제가 없으나, 이 책에서는 게시판 모델에 데이터를 입력할 경우 처음 입력 시간 및 최종 수정 시간은 모두 현재 시간을 기준으로 입력하도록 설계할 예정이므로, 현재 시간을 나타낼 수 있는 값으로 기본 값을 설정한다. Django에서는 현재 시간을 입력할 때 django.utils 패키지의 timezone을 사용하며, 이에 따라 날짜와 관련된 필드는 모두 'default=timezone.now'으로 변경하도록 한다.

서적 이용에 참고 바라겠습니다.