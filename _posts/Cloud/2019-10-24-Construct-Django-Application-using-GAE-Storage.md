---
layout: post
title:  "Google App Engine / Storage를 사용한 Django 웹 애플리케이션 구축"
date:   2019-10-24
categories: [Cloud]
tags: [Python, 파이썬, Python37, Google, Cloud, Django, 장고, 구글, 클라우드, GAE, App, Engine, Storage, 스토리지, 앱, 엔진]
---

 
안녕하세요.

이번에 쓸 글은 Google Cloud 환경에서 Google App Engine과 Google Cloud Storage를 사용한 Django 웹 애플리케이션을 구축해 보는 글을 쓰겠습니다.

 

구현 기능은 이미지 다중 첨부가 구현된 애플리케이션입니다.

Google Cloud에서 STATIC, MEDIA 파일을 모두 배포하고 관리하기 위해서는 가장 적합한 예제가 이미지 다중 첨부 기능이 아닐까 싶기 때문입니다. 

덧붙여 말하면, 백엔드 기능에 집중했기 때문에 프론트엔드쪽은 거의 작업하지 않은 점 참고바랍니다.

 

설명할게 매우 많은 글입니다. 그래서 군더더기는 빼고 핵심 부분만 요약해서 쓰겠습니다.

물론 그래도 내용이 매우 많을 것으로 예상되네요.

 

 

 

### 1. 개요
Google App Engine은 실제 배포용 애플리케이션을 위한 클라우드 서비스입니다.

Django 애플리케이션 개발을 완료하면 이를 실제로 배포하기 위한 서비스로 볼 수 있죠.

일반적으로 App Engine에 배포한다는 것은, Compute Engine의 VM 인스턴스에서 개발한 애플리케이션을 배포할 때 사용되는 것을 뜻하기도 합니다.

AWS에서는 이와 유사한 서비스로 Elastic Beanstalk가 있으니 참고하시면 됩니다.


 
 

Google Cloud Storage는 이름에서도 알 수 있듯이, 말 그대로 파일 저장소를 뜻합니다. 

AWS에서는 S3와 완전히 동일한 개념으로, 버킷 단위로 파일을 저장하고 관리합니다.

 

Django 애플리케이션 구축을 완료하면 애플리케이션 서버를 만들고 배포를 진행합니다.

하지만 단순히 애플리케이션을 배포하는 데 그치는 것이 아니라, 애플리케이션에 사용되는 파일들을 관리할 공간이 별도로 필요합니다. 그래서 Storage와 연동도 당연히 필요한 것이고요.

 

Django에서 Storage에 관리되는 파일은 크게 두 가지 유형이 있습니다. Static 파일과 Media 파일이 있는데요.

Static 파일은 웹 애플리케이션의 JS, CSS, Image 등 구성 요소로 들어가는 정적 파일이고,

Media 파일은 웹 애플리케이션의 대용량 파일 또는 사용자가 첨부하는 파일 등을 관리하는 파일입니다.

 

이것과 관련된 기술문서는 찾아보면 다수 있습니다. 구글 클라우드에서도 설명서까지 나와있고요.


 
하지만 구글 클라우드의 설명서는 Static 파일만 다루고 있으며, 그조차 별도로 관리되기 보다는 애플리케이션의 한 일부분으로만 관리되고 있습니다. 반대로 다른 문서에서는 구글 App Engine에서의 배포방법을 다루지 않고 환경설정을 하는 부분만 다루고 있습니다. 그래서 이 두가지를 합친 설정을 적용하는데 충돌이 발생해서 어려움이 있었습니다.

 

그래서 이들을 망라한 방법을 제시하고자 하니, 저와 비슷한 서비스를 구축하고자 하는 분들에게 참고가 되었으면 합니다.

 

이와 더불어, 이 문서를 작성한 데 참고했던 사이트는 다음과 같습니다.

 

Google App Engine Django 배포 공식문서 - <https://cloud.google.com/python/django/appengine?hl=ko>
Django-storage 패키지 공식문서: <https://django-storages.readthedocs.io/en/latest/backends/gcloud.html>
Tariq Al-Sadoon 블로그: <https://medium.com/swlh/preparing-your-django-application-for-google-cloud-run-7c8cb7b7464b>
 

그 외 다른 사이트도 참고했지만, 실질적으로 도움이 된 사이트는 위 세 곳만 있어서 언급하니 참고바랍니다.

 

 

이번에 올릴 게시물은 단순히 이용 방법만 설명하는 것이 아닌 전체 소스코드를 배포했습니다. 소스코드는 아래 사이트에서 제공하니 더불어 참고하시기 바랍니다.

 

 

<https://github.com/Amanokaze/gcloud_django_deploy/>


### 2. Google Compute Engine 설정

Django 애플리케이션 배포는 Google Compute Engine의 VM인스턴스에서 이루어집니다.

사용 환경은 다음과 같습니다.

 

* OS: Ubuntu 18.04 LTS
* Python: 3.7
* Django: 2.2
* MySQL: 5.7

이미 VM 인스턴스가 있다면 현재 사용중인 인스턴스를 그대로 사용해도 되고, 신규로 생성해도 문제는 없습니다. 다만 VM 인스턴스에서 실행되는 Django 애플리케이션은 배포 환경이 아닌 개발 환경이라는 점에서, 개발 환경에서 정상적으로 Django가 실행되는 지 확인도 더불어 필요합니다. 이를 위한 설정 부분만 간단히 진행하겠습니다.

 

아래 방법은 Compute Engine의 VM 인스턴스가 이미 있다는 것을 전제로 하니 참고 바랍니다.


 
 

먼저 Compute Engine의 VM 인스턴스로 이동합니다. 이동하면 내부IP와 외부IP가 모두 조회될 것입니다.

 


위 내부/외부 IP 주소는 아래 Django 개발환경 서버 실행 시 모두 사용될 주소입니다.

그러므로 미리 확인하고 기록하시기 바랍니다. 

 

그리고 인스턴스를 클릭하고 들어간 후 HTTP/HTTPS 트래픽 허용 체크여부를 확인합니다. 

체크가 되어있으면 넘어가시고 안되어 있으면 체크 후 저장해주시면 됩니다.

 


 

일단 이 정도까지 진행한 후, 다음으로는 메뉴로 가서 'VPC 네트워크' - '방화벽 규칙'으로 이동합니다.

 

 


 

방화벽 규칙으로 가면 기본으로 사용되는 포트 목록이 나타납니다. 하지만 8000번 포트는 허용 포트번호에 없습니다.

Django는 일반적으로 8000번 포트를 사용하기 때문에 8000번 포트를 열어야 합니다. 물론 다른 포트를 사용하기를 원하신다면 해당 포트로 진행해도 문제는 없습니다. 

'방화벽 규칙 만들기'를 눌러서 진행하겠습니다.

 


 

이름 대충 입력해 주시고요.

 


 

다음으로 기본값은 그대로 둔 상태로 아래로 내린 다음에 

 

* 대상: 네트워크의 모든 인스턴스
* IP 범위: 모든 범위(0.0.0.0/0)
* tcp 포트: 8000번

입력 후 만들기를 진행하겠습니다.

 


다 만들었으면 아래와 같이 추가된 것을 확인할 수 있을 것입니다.

 


 

여기까지만 하면 VM 인스턴스에 대한 설정은 끝났습니다. 이제 인스턴스에 접속해서 Django 애플리케이션 구축을 위한 패키지 설치를 진행하겠습니다. 

 

 

### 3. 주요 패키지 설치
Google App Engine 배포를 위해서는 Python 3.7을 사용해야 하니, 다른 환경은 어떤 걸 써도 무방하나 Python은 반드시 3.7로 사용해야 합니다. Google Compute Engine의 Ubuntu VM 인스턴스에서 Python3.7로 환경설정하는 법은 바로 아래 글에 있으니 참고하시면 됩니다.

 

<https://amanokaze.github.io/blog/Construct-Python37-Django-MySQL-Google-Cloud/>
 

위 글에서 Google Compute Engine에서 Python3.7 설정하는 법은 다루었으니, Python3.7 설치방법은 이 글에서는 생략하도록 하고, Django 웹 애플리케이션 구축에 필요한 패키지 설치를 진행하겠습니다.

 

먼저 VM 인스턴스에서 설치할 패키지입니다.

위 2개 패키지는 Django와 MySQL 연동을 위한 기본 라이브러리입니다.

 
{% highlight Shell %}
$ sudo apt-get install python3.7-dev lib-mysqlclient-dev
{% endhighlight %} 
 

다음은 가상환경 생성 후 패키지를 설치하겠습니다.

 
{% highlight Shell %}
$ python3 -m venv venv
(venv)$ pip install --upgrade pip
(venv)$ pip install django==2.2
(venv)$ pip install mysqlclient
(venv)$ pip install django-storages[google]
(venv)$ pip install Pillow
{% endhighlight %} 
 

패키지에 대한 간략한 설명은 다음과 같습니다.

 

* django, mysqlclient: 생략
* django-storages[google]: Google Cloud Storage 연동을 위한 패키지
* Pillow: Media 파일 첨부를 위한 패키지
 

필요 패키지 설치가 완료되었으면, 이제 Django 프로젝트 및 앱을 생성하도록 하겠습니다.

 

{% highlight Shell %}
(venv2)$ django-admin startproject imageproj
(venv2)$ cd imageproj/
(venv2)/imageproj$ django-admin startapp imageapp
{% endhighlight %} 
 

프로젝트가 생성되었으면, 다음으로 DB 연동을 위한 MySQL 인스턴스 생성 및 연동을 진행하겠습니다.

 

 

### 4. MySQL 설치 및 설정
SQL 인스턴스 설치도 이 글에서 생략하려고 했으나, SQL 관련해서 추가 설정해야 할 부분이 있으므로 SQL 인스턴스 설치부터는 이 글에서 같이 다루겠습니다.

 

SQL 인스턴스는 Google Cloud Console 메뉴에서 다음 위치에 있습니다.

 


첫 화면의 '인스턴스 생성'을 먼저 눌러주시고,

 


MySQL을 선택해주세요.

 



인스턴스 ID는 임의대로 입력해주시고, 루트 비밀번호도 반드시 입력해주시기 바라고요.

리전은 편하실대로 하셔도 되는데, 우리나라에서 제일 가까운 곳이 도쿄이므로, 도쿄(asia-northeast1)를 선택해주세요.

나머지는 기본 설정 그대로 합니다.

 


생성이 완료되면 대시보드에 다음과 같이 인스턴스가 조회됩니다. 인스턴스 이름을 눌러서 들어갑니다.

 


 

들어가면 아래의 '공개 IP주소'와 '인스턴스 연결 이름' 두 가지를 모두 확인합니다.

먼저 복사하신 후 메모장 등에 미리 기록해주세요.

 


다음으로 '연결' 탭으로 이동해주신 후, '네트워크 추가'를 눌러줍니다.

 


그리고 주소를 입력합니다. 

특정 호스트에서만 접속하려면 해당 호스트의 IP를 입력해주시고, 모든 곳에서 접속 가능하게 하려면 아래 사진과 같이 '0.0.0.0/0'을 입력하시면 됩니다. 입력 후 '저장'을 누르면 등록이 완료됩니다.

 

'0.0.0.0/0'을 입력하면 경고 문구가 나오는데, 해당 사항은 참고 정도만 해주시면 됩니다.

 


추가로 말씀드리면, DB 인스턴스의 외부 접근이 염려된다고 생각하실 경우에는, 위 방법처럼 새로운 네트워크를 추가하지 마시고 Cloud SQL 프록시를 사용해서 VM 인스턴스 내부에서 연결해주시면 됩니다. 해당 방법은 App Engine 표준환경 설정법에 나와있으니 아래 링크에서 확인하시면 됩니다.

 

<https://cloud.google.com/python/django/appengine?hl=ko#install_the_sql_proxy>


 
 

다음은 사용자 설정입니다. root를 그대로 사용할 수도 있지만, 일반적으로 root 계정으로 뭔가 하는 것은 찝찝할 수 있으므로, root를 대신할 사용자 게정을 생성합니다. 이 계정은 Django 환경설정에서도 사용될 계정이니 참고 바랍니다.

사용자 설정은 '사용자 계정' 탭으로 가서 아래와 같이 만들어주시면 됩니다.

 


 

다음은 DB 생성입니다. Django에서 MySQL을 연결하려면 DB명이 반드시 있어야 하므로, DB 생성도 여기에서 바로 진행하도록 하겠습니다. '데이터베이스' 탭으로 이동해주신 후, '데이터베이스 만들기'를 눌러주세요.

 


원하는 이름 입력해주시고요.


만들면 완료됩니다.

 


MySQL 접속 테스트를 하겠습니다. 위에 언급했듯이 Cloud SQL 프록시를 사용할 경우에는 위 링크대로 테스트해주시고, 그렇지 않고 이 글에 언급한대로 공걔 IP를 통해서 접속할 경우에는 공개 IP주소로 접속하시면 됩니다.

MySQL 접속 및 사용은 MySQL Workbench를 사용해도 되고, VM인스턴스에서 실행할 경우에는 mysql-client를 설치해주신 후 테스트해보셔도 됩니다. 이 글에서는 mysql-client로 접속해보도록 하겠습니다.

 

만약 VM인스턴스에 MySQL이 설치되어있지 않다면, 아래와 같이 진행하시면 됩니다.

다만 이 패키지는 필수는 아니고 MySQL Workbench 프로그램으로 대체가 가능하므로 위 설치 패키지에는 쓰지 않았으니 참고 바랍니다.

{% highlight Shell %}
$ sudo apt install mysql-client
{% endhighlight %} 
 

접속 방법은 아래와 같이 하시면 됩니다.

IP 주소는 아까 위에서 기록했던 '공개 IP 주소'를 참고하시면 됩니다.

{% highlight Shell %}
$ mysql -h [IP주소] -u [사용자명] -p
Enter Password:
{% endhighlight %} 
 

이 글에서는 파일 첨부까지 같이 진행할 예정이므로, 파일 첨부를 위한 DB Table도 같이 생성합니다.

DB 사용 후 테이블을 아래와 같이 아주 간단히 생성하면 됩니다.

 

{% highlight Shell %}
mysql> use imagedb
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
Database changed
mysql> create table t_images (
    -> id integer primary key auto_increment,
    -> image varchar(300) not null,
    -> creation_date datetime
    -> );
Query OK, 0 rows affected (0.02 sec)
{% endhighlight %} 
 

### 5. Django - MySQL 연동
MySQL 설치가 완료되었으니, 이제 Django와 연동해야 되겠죠?

 

DB 연동과 관련된 부분은 Django의 [프로젝트 디렉토리]/settings.py 파일에 있습니다. 처음 설치하면 # Database 항목에 SQLite가 있는데, 그걸 지워주시고 다음과 같이 변경합니다.

 

{% highlight Python %}
# Database
# https://docs.djangoproject.com/en/2.2/ref/settings/#databases


if os.getenv('GAE_APPLICATOIN', None):
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': '<DB명>',
            'USER': '<사용자명>',
            'PASSWORD': '<비밀번호>',
            'HOST': '/cloudsql/<인스턴스 이름>',
        }
    }
else:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': '<DB명>',
            'USER': '<사용자명>',
            'PASSWORD': '<비밀번호>',
            'PORT': '3306',
            'HOST': '<공개 IP 주소>',
        }
    }
{% endhighlight %} 
 

여기서부터 일반적인 설정과는 조금 다르다는 것을 알 수 있을 것입니다.

 

일반적으로 Django의 settings.py 파일에서 DB 설정을 할 때에는 DATABASES 항목에 간단히 입력만 하고 끝나지만, 위 코드에서는 os.getenv('GAE_APPLICATION', None) 조건문을 사용했습니다. 여기서 "GAE_APPLICATION'은 Googla App Engine Application을 줄인 말로, App Engine하고 연동했을 때, 즉 배포했을 때는 if 절 조건을 수행하고, 반대로 개발 환경일 경우에는 else 절 조건을 수행합니다. 이 부분은 개발환경과 배포환경의 설정을 다르게 하겠다는 뜻이므로 반드시 참고하시기 바랍니다.source 

 

설정을 완료했으면, DB가 변경되었으므로 migration과 개발환경 서버를 실행해서 이상유무를 먼저 확인하겠습니다.

 
{% highlight Shell %}
(venv2)/imageproj$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying sessions.0001_initial... OK

(venv2)/imageproj$ python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...
System check identified no issues (0 silenced).
October 11, 2019 - 07:14:09
Django version 2.2, using settings 'imageproj.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
{% endhighlight %}

다행히도 아무 이상이 없는 것으로 확인되었습니다.

이제 이미지 첨부를 위해서 Django 코드를 최대한 간단하게 생성하겠습니다.

지금부터가 진짜입니다.

 

 

### 6. Django 이미지 첨부 소스코드 작성

#### 1) imageproj/settings.py 

settings.py에는 여러 부분을 미리 추가해야 합니다.

 

개발 환경에서 실행한 결과를 조회하고, 배포 환경에서도 실행 결과를 조회해야 하므로, 접근 호스트 설정을 모든 호스트로 하겠습니다.

 
{% highlight Python %}
ALLOWED_HOSTS = ["*"]
{% endhighlight %} 
 

다음은 imageapp 앱을 추가했으므로, 아래와 같이 수정합니다.

{% highlight Python %}
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'imageapp',
]
{% endhighlight %} 
 

맨 아래의 STATIC과 MEDIA URL, Root를 설정합니다. 참고로 아래 경로는 개발환경에서만 사용되는 경로로, 배포환경에서는 경로를 변경해야 합니다. 

변경이 귀찮으시다면 'if DEBUG:' 분기를 사용해서 나눠주셔도 됩니다.

 

{% highlight Python %}
STATIC_URL = '/static/'
MEDIA_URL = '/media/'
MEDIA_ROOT = 'media'
{% endhighlight %} 
 

#### 2) imageproj/urls.py 

imageapp을 URL에 추가하고, MEDIA, STATIC 파일 첨부도 모두 진행해야 하므로 urlpatterns에 아래 문구도 같이 추가해줍니다.

{% highlight Python %}
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('imageapp/', include('imageapp.urls')),
] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
{% endhighlight %} 

#### 3) imageapp/models.py

앞서 생성한 테이블을 모델에 등록해야 되겠죠.

모델 등록을 편리하게 하는 스크립트 명령어인 inspectdb를 먼저 사용합니다.

 
{% highlight Shell %}
(venv)/imageproj$ python manage.py inspectdb t_images > imageapp/models.py
{% endhighlight %}

그리고 models.py 파일을 다음과 같이 수정해줍니다.

 
{% highlight Python %}
from django.db import models
from django.utils import timezone

class TImages(models.Model):
    image = models.ImageField(upload_to="%Y%m%d", blank=True)
    creation_date = models.DateTimeField(blank=True, null=True, default=timezone.now)
    class Meta:
        managed = False
        db_table = 't_images'
{% endhighlight %}
 

image 필드를 보면, 기본으로 CharField로 생성되어 있지만, 이를 ImageField로 반드시 바꿔줘야 합니다. 그래야 파일 첨부가 됩니다. upload_to 의 경우는 저장 경로를 설정하는 부분이므로, 가장 무난하게 날짜 포맷으로 써주시면 됩니다.

 

timezone을 import하는 이유는 creation_date의 값을 현재시간(timezone.now)으로 하기 위함입니다. timezone을 안쓰실 경우에는 import를 하지 않아도 됩니다.

 

 

#### 4) imageapp/forms.py

다음은 Form을 생성하겠습니다. forms.py 는 필수 생성 파일은 아니지만, 파일 다중 첨부를 위한 양식으로 표현해야 하기 때문에, forms.py에서 명시하는 것이 더욱 명확하므로, 아래와 같이 간단하게 file_field만 선언하겠습니다.

 
{% highlight Python %}
from django import forms

class FileFieldForm(forms.Form):
    file_field = forms.FileField(widget=forms.ClearableFileInput(attrs={'multiple': True}))
{% endhighlight %}


#### 5) imageapp/views.py

이제 뷰를 생성하겠습니다.

 
{% highlight Python %}
from django.shortcuts import render
from django.template.context_processors import csrf
from imageapp.models import *
from imageapp.forms import FileFieldForm
from django.views.generic.edit import FormView

class UploadFileView(FormView):
    form_class = FileFieldForm
    template_name = 'upload_image.html'
    success_url = '.'
    
    def post(self, request, *args, **kwargs):
        form_class = self.get_form_class()
        form = self.get_form(form_class)
        files = request.FILES.getlist('file_field')
        if form.is_valid():
            for f in files:
                imgfile = TImages(image=f)
                imgfile.save()
            
            return self.form_valid(form)
        else:
            return self.form_invalid(form)
        
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['imgdata'] = TImages.objects.all()
        return context
{% endhighlight %} 

View에 대해서 간단히 설명하면 다음과 같습니다.

 

FormView를 상속한 UploadFileView 에서 모든 동작이 이루어지도록 하였습니다.
뒤의 템플릿에서도 확인되겠지만, UploadFileView 하나에서 모든 것이 처리됩니다. 즉 다중 이미지 첨부를 위해서 입력하는 페이지와 입력 처리 완료 후 보여주는 페이지 모두 동일합니다.
그러므로 post() 메소드를 사용해서 폼 입력값을 전송했을 때 처리하는 부분도 나타내야 하지만, 처리 완료 후 첨부파일을 보여주는 부분도 같이 작성해야 하므로 get_context_data() 메소드도 같이 사용했습니다.
 

#### 6) imageapp/urls.py

위의 imageproj/urls.py와 당연히 다른 파일입니다. 위 urls.py는 프로젝트의 URL 설정 파일이고, 지금 나타낼 urls.py는 imageapp에 대한 앱 URL 설정 파일입니다. 이미지 업로드 처리 템플릿 주소만 구현할 것이므로 내용은 간단합니다.

먼저 사실 위 코드에서는 jQuery를 사용할 일이 전혀 없습니다. 하지만 말 그대로 예제 코드이므로 같이 넣었습니다.


 
그리고 스타일이나 어떤 다른 요소가 전혀 없습니다. 하지만 이 글의 서두에도 언급했듯이, 백엔드에 집중해서 나타낼 것이므로 프론트엔드는 최소한의 기능 외에 어떠한 요소도 없으니 참고하시기 바랍니다.

 

jQuery 다운로드는 공식 홈페이지에서 진행하시면 되며, 해당 파일은 /imageapp 디렉토리의 /static/js/ 디렉토리에 저장해 주시면 됩니다. 물론 웹에서 바로 사용할 수도 있지만, Static 파일이 최소 1개는 있어야 추후 Static 이동 테스트 시 결과 확인을 하기 더욱 용이해지기 때문이니 참고 바랍니다.

 

 

 

### 7. 개발환경 테스트

이제 개발환경에서 테스트를 진행하겠습니다.

 

앞서 Compute Engine 설정할 때 내부/외부 IP 주소를 모두 기록하라고 언급했었습니다. 이제 그 주소를 드디어 사용할 때가 왔습니다.

 

클라우드 가상 머신(VM)에서 Django 개발환경에서 구축한 내용을 조회하기 위한 방법은 다음과 같습니다.

Django 개발환경 실행 시 localhost(127.0.0.1)가 아닌, 클라우드 VM에 할당된 IP 주소를 사용한다.
외부 클라이언트에서는 외부 IP 주소를 사용하여 접속한다.
클라우드 VM에서는 외부 IP 주소로 접속 시, 이를 내부 IP 주소와 연결하여 내부 서버의 내용으로 응답한다.
더욱 자세한 내용은 아래 글을 참고하셔도 됩니다. 아래 글은 AWS EC2 인스턴스에 대한 내용이지만, Google Compute Engine VM에서도 동일하게 적용됩니다.

 

<https://amanokaze.github.io/blog/Way-to-See-Django-Development-Env-Using-Browser-in-EC2-Instance/>

 

Django 개발환경을 다음과 같이 실행합니다.

 
{% highlight Shell %}
(venv)/imageproj$ python manage.py runserver <내부 IP주소>:<포트번호>
{% endhighlight %} 
 

Django 개발환경을 내부 IP주소로 실행한 후, 클라이언트 PC에서는 외부 IP 주소를 사용하면 위 언급된 것과 같이 내부/외부 IP가 연동되어 정상 실행이 가능합니다.

 

이제 웹 브라우저를 열어서 접속하겠습니다.

 

Image 업로드 화면입니다. 

이제 여기에 이미지 3개를 동시 첨부한 후 업로드를 진행하겠습니다. 

 


코드가 정상적으로 실행되면, DB에 파일이름이 들어가고, MEDIA 디렉토리에도 이미지 파일 3개가 정상적으로 업로드가 완료되었습니다. 업로드 완료된 파일은 바로 조회하도록 설계했으므로 아래와 같이 조회할 수 있습니다.

 


첨부 이미지 3개는 모두 재사용 가능한 License 사진이므로 참고 바랍니다
 

개발환경 테스트는 모두 완료되었으니, 이제 배포환경을 위한 설정으로 넘어가겠습니다.

 

 

 

### 8. 배포환경 설정

사실 1번~7번까지는 별 내용 없습니다. 이 부분만 놓고 보면 인터넷에 있는 다른 유사 문서를 통해서도 나타나 있는 부분이고, 단순히 Compute Engine에서 Django 애플리케이션 설정 및 테스트했다 이게 전부니까요.

 

아마 제 글을 검색해서 들어오신 분들이 가장 관심이 갈만한 요소는 지금 시작되는 8번부터의 내용이 아닐까. 그렇게 생각됩니다. 사실 저도 여기서부터 쓸 내용 때문에 이 글을 쓰게 된 것이고, Github에 프로젝트로 올리기까지 한 것이고요.

 

 

먼저 배포환경과 개발환경의 차이점부터 쓰겠습니다.

 

클라우드 서비스에서의 Django 배포 환경은 웹 애플리케이션 서버와 파일 스토리지를 필요로 합니다.

그래서 이 글의 제목도 'Googla App Engine / Storage를 사용한' 이라고 언급했고요.

 

즉, 배포환경에서는 웹 애플리케이션 서버 및 스토리지 설정 및 연동이 가장 중요합니다.

 

이에 따라 달라지는 환경 설정 요소가 몇몇 존재하며, Google App Engine, Google Cloud Storage를 기준으로 환경 설정할 요소를 다음과 같이 나타내겠습니다. (서비스 제조사에 따라 구성 환경 요소가 약간씩 다르니 참고 바랍니다)

 

STATIC, MEDIA 파일 저장을 위한 Storage 지정 및 권한 설정
Google App Engine 배포에 사용되는 파일 준비
이렇게 두 가지 정도로 요약할 수 있으며, 각각에 대해서 기술하겠습니다.

 

 

### 9. Storage 지정 및 권한 설정
#### 1) imageproj/settings.py
먼저 권한 설정을 위한 패키지부터 불러오겠습니다.

 

{% highlight Python %}
import os
from google.oauth2 import service_account
{% endhighlight %} 


import os는 원래 있던 코드입니다.

아래에 from google.oauth2 import service_account만 추가하면 됩니다.

이 패키지를 사용해야 Storage에 파일을 저장하기 위한 권한을 설정할 수 있습니다.

 

 

배포할 때에는 DEBUG는 False로 지정하는 것이 기본이겠죠. 사실 이 부분은 Storage 보다는 App Engine 배포에 사용되는 설정이지만, settings.py 파일 변경을 진행해야 하므로 같이 하겠습니다.

 

{% highlight Python %}
DEBUG = False
{% endhighlight %} 
 

다음은 MEDIA, STATIC 파일 설정입니다. 위에서 지정했던 부분은 모두 주석처리하고 아래와 같이 입력해주시면 됩니다.

 
{% highlight Python %}
# Development Env Variable
#STATIC_URL = '/static/'
#MEDIA_URL = '/media/'
#MEDIA_ROOT = 'media'

GS_CREDENTIALS = service_account.Credentials.from_service_account_file(
    "secrets/bucket-admin.json"
)
DEFAULT_FILE_STORAGE = 'config.storage_backends.GoogleCloudMediaStorage'
STATICFILES_STORAGE = 'config.storage_backends.GoogleCloudStaticStorage'
GS_PROJECT_ID = '<google cloud project id>'
GS_MEDIA_BUCKET_NAME = '<media bucket name>'
GS_STATIC_BUCKET_NAME = '<static bucket name>'
STATIC_URL = 'https://storage.googleapis.com/{}/'.format(GS_STATIC_BUCKET_NAME)
MEDIA_URL = 'https://storage.googleapis.com/{}/'.format(GS_MEDIA_BUCKET_NAME)
{% endhighlight %}

요약하자면, MEDIA, STATIC 파일이 어느 Storage에 연결되고, 어느 Bucket을 사용할 것인지를 지정하는 것으로 보시면 됩니다. 그리고 Storage 접근 시 어떤 권한을 가지고 접근할 것인가도 같이 명시되어 있습니다. 

GS_CREDENTIALS는 Storage 접근을 위한 권한에 사용되는 환경변수이며, 그 외의 환경 변수는 Storage 저장을 위한 위치를 나타내는 변수입니다. 


 
 

Media/Static 파일 버킷은 각각 다른 버킷으로 설정하는 것이 좋습니다. Static 파일은 읽기 전용으로 관리되어야 하는 반면, Media 파일은 첨부파일 저장을 위해서 읽기/쓰기로 관리되는 것이 최적이기 때문입니다. Google Cloud Storage의 버킷 권한 설정은 버킷 단위로 할당되는 것이 일반적이므로, 역시 각각의 버킷으로 구성하는 것이 좋습니다.

 

이제 여기서 추가로 생성해야 할 파일이 두 개 있습니다. 

 

첫 번째는 storage_backends.py 파일입니다. 

DEFAULT_FILE_STORAGE, STATICFILES_STORAGE 변수의 값이 어떤 스토리지를 사용할 것인가를 나타내는 부분인데, 위 값에 의하면 config/storage_backends.py 파일의 GoogleCLoudMediaStorage / GoogleCloudStaticStorage 클래스에서 지정하기 때문입니다. 물론 storage_backends.py 경로나 파일이름은 자유롭게 변경할 수 있되, 변경하더라도 위 설정 값과 일치해야 하는 점 참고 바랍니다.

 

두 번째는 bucket-admin.json 파일입니다. 

이 파일은 앞서 언급했듯이 Storage 접근 권한을 위한 파일이며, GS_CREDENTIALS 변수에서 선언한 내용의 파일에서 권한을 관리하기 때문입니다.

 

그럼 각 파일 설정부분도 다루도록 하겠습니다.

 

 

#### 2) config/__init__.py

Django에서 신규 디렉토리를 생성하고 해당 디렉토리 내 Python 파일을 사용할 경우에는 기본적으로 해야 할 절차가 있습니다. 그것은 바로 __init__.py 파일 생성인데요. 이 파일이 있어야 특정 디렉토리의 파일 클래스를 가지고 와서 사용할 수 있습니다.

 

config/__init__.py 파일은 생성만 하고 내용은 아무것도 입력하지 않습니다. 파일이 존재하는 것만으로도 제 역할을 다 한 것입니다.

 

 

#### 3) config/storage_backends.py

이제 스토리지 지정을 위한 파일을 생성합니다. 파일 내용은 위의 참고 문서 중 하나인 Tariq Al-Sadoon 블로그의 내용으로 하겠습니다.

<https://medium.com/swlh/preparing-your-django-application-for-google-cloud-run-7c8cb7b7464b>

 
{% highlight Python %}
from django.conf import settings
from storages.backends.gcloud import GoogleCloudStorage
from storages.utils import setting
from urllib.parse import urljoin


class GoogleCloudMediaStorage(GoogleCloudStorage):
    """GoogleCloudStorage suitable for Django's Media files."""

    def __init__(self, *args, **kwargs):
        if not settings.MEDIA_URL:
            raise Exception('MEDIA_URL has not been configured')
        kwargs['bucket_name'] = setting('GS_MEDIA_BUCKET_NAME')
        super(GoogleCloudMediaStorage, self).__init__(*args, **kwargs)

    def url(self, name):
        """.url that doesn't call Google."""
        return urljoin(settings.MEDIA_URL, name)


class GoogleCloudStaticStorage(GoogleCloudStorage):
    """GoogleCloudStorage suitable for Django's Static files"""

    def __init__(self, *args, **kwargs):
        if not settings.STATIC_URL:
            raise Exception('STATIC_URL has not been configured')
        kwargs['bucket_name'] = setting('GS_STATIC_BUCKET_NAME')
        super(GoogleCloudStaticStorage, self).__init__(*args, **kwargs)

    def url(self, name):
        """.url that doesn't call Google."""
        return urljoin(settings.STATIC_URL, name)
{% endhighlight %}

파일이 좀 길긴 한데, 특별한 내용은 없습니다. settings의 MEDIA_URL과 STATIC_URL을 가지고 오는 부분과, 초기 선언 시 GS_MEDIA_BUCKET_NAME과 GS_STATIC_BUCKET_NAME을 가지고 와서 설정하는 부분입니다. 해당 변수는 이미 위에서 선언했기 때문에 그대로 사용하면 됩니다.

 

 

#### 4) Storage 권한 설정 및 secret/bucket-admin.json

앞서서는 Google Cloud Storage에 Media/Static 파일을 저장하고 불러오기 위해서 버킷을 지정했습니다. 특히 Google Cloud Storage를 외부의 다른 서비스에서 사용할 때, 서비스 계정을 별도로 설정하지 않으면 Storage에 접근할 때 에러가 발생합니다. 

 

이 부분은 Google Cloud Storage와 Amazon S3의 가장 큰 차이점이라고도 볼 수 있습니다. Amazon S3는 버킷 사용권한만 설정하면 바로 접근이 가능한 반면, Google Storage는 버킷 사용권한을 설정하더라도 서비스 계정을 설정하지 않으면 사용을 할 수 없습니다. App Engine에 배포되는 Django 웹 애플리케이션은 웹 페이지 접속 시 해당 페이지에 대한 Media/Static 파일을 호출할 때마다 애플리케이션에 등록된 서비스 계정을 확인한 후 접근 이상 유무를 판단하기 때문입니다.

 

제가 이미지 다중 첨부 코드를 작성하고 테스트하면서 가장 많이 애먹었던 부분도 이 부분이였습니다. 왜 설정할것 다 했는데도 어째서 Google Storage의 파일을 읽는 부분에서 자꾸 에러가 발생하는가를 확인하다가 위와 같은 사실을 알고, 이를 위해서 따로 파일 설정을 해야 한다는 것을 말이죠.

 

바로 다음에는 secret/bucket-admin.json 파일을 생성하고 등록하는 방법을 소개할 예정입니다. 이 파일은 바로 위에서 언급된 서비스 계정에 대한 정보가 있는 json 파일입니다.

 

이 파일을 생성하기 위해서는 Google Cloud 메뉴의 API 및 서비스 - 사용자 인증 정보로 먼저 들어갑니다.

 


사용자 인증 정보는 API키, OAuth 2.0 클라이언트, 서비스 계정 키 세 종류가 있습니다. 아래 화면은 제가 이미 생성했던 키이므로 무시하셔도 좋고, 신규로 '사용자 인증 정보 만들기'를 선택합니다.

 


앞서 Storage에 대한 서비스 계정을 설정해야 한다고 했으므로 '서비스 계정 키'를 선택합니다. 


서비스 계정 생성을 위한 정보는 다음과 같이 입력합니다.

 

* 서비스 계정: 새 서비스 계정
* 이름: 임의의 이름
* 역할: 프로젝트 - 소유자 (유사 다른 권한을 선택해도 됩니다)
* 키 유형: JSON

생성되면, JSON 파일을 다운로드받을 수 있습니다. JSON 파일의 형태는 다음과 같습니다.

 
{% highlight JSON %}
{
  "type": "service_account",
  "project_id": "<Project ID>",
  "private_key_id": "<Key ID>",
  "private_key": "<Key>",
  "client_email": "test-468@<프로젝트명>.iam.gserviceaccount.com",
  "client_id": "<Client ID>",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "<X509 Cert Url>"
}
{% endhighlight %}
 

생성된 파일은 수정을 하면 안됩니다. 그대로 사용해야 합니다.

하지만 여기서 참고해야 할 부분으로, "client_email" 이 부분은 일단 다른 곳에 기록해야 합니다. 추후 Storage 권한 생성 시 client_email 이 사용되기 때문입니다.

 

이 파일을 내용 그대로 해서, secret 디렉토리를 생성한 후, bucket-admin.json 파일로 저장합니다.

 

위와 같이 파일을 생성하면 웹 애플리케이션에서 Storage에 접근하기 위한 준비는 마쳤습니다. 하지만 Storage에서도 버킷 별로 접근 권한을 설정하고 있으며, 서비스 계정이 접근했을 때 어떤 권한을 부여할 것인가도 등록을 해야 합니다. 그러므로 Storage의 서비스 계정 권한도 바로 부여하겠습니다.

 

먼저 생성된 버킷 목록을 확인한 후, 맨 오른쪽의 점선 3개 부분을 클릭합니다. 맨 첫번째의 '버킷 권한 수정'을 클릭합니다.

 


버킷 권한 수정을 클릭하면 오른쪽에 창이 하나 나옵니다. 기본으로 생성된 권한이 있지만, 모두 무시하고 '구성원 추가'를 누릅니다.

 


 

아까 JSON 파일의 e-mail 주소 기록하라고 했었죠? 그 E-mail 주소를 '새 구성원' 부분에 입력합니다. 역할은 '스토리지 - 저장소 관리자'로 선택합니다. 

 


위와 같이 설정하면 웹 애플리케이션과 Cloud Storage 지정 및 권한 설정을 모두 마치게 되며, 정상적인 연동도 가능해집니다.

 

 

#### 5) Google Cloud Storage 연동 확인
 

이제 설정을 마쳤으면 실제로 파일이 복사되는지를 확인해야겠죠.

Media 파일은 배포를 한다고 해서 스토리지로 복사되지는 않습니다. 애플리케이션을 사용하면서 신규로 생성되거나 삭제되는 추가적인 파일이지, 웹 애플리케이션을 구성하는 파일은 아니기 때문입니다.

하지만 Static 파일은 정반대로 웹 애플리케이션을 구성하는 데 필요한 파일입니다. 그러므로 배포가 이루어질 경우에는 지정된 Storage 버킷으로 파일이 복사됩니다. 

 

Static 파일을 복사하는 법은 Django 배포를 한 번이라도 해보셨던 분들은 모두 아시겠지만, 아래와 같이 이용합니다.

 

{% highlight Shell %}
(venv)/imageproj$ python manage.py collectstatic
{% endhighlight %} 

이 명령어는 개발환경에서 사용된 모든 Static 파일을 settings.py에서 지정한 경로로 모두 복사한다는 것을 뜻합니다. 위에서는 Static 파일 버킷 설정을 하였으므로, 실제 복사가 되었는지 확인해봐야겠죠.

 

위와 같이 Django 애플리케이션을 개발했다면 복사되는 파일은 다음과 같습니다.

 

admin 모듈에서 사용되는 static 파일
위 예제에서 추가된 jquery 파일
실제 복사되었는지 여부는 Static 버킷에서 확인하면 되겠죠.

 


정상적으로 생성되었네요.

 

여기까지 Django 애플리케이션과 Storage 연동은 모두 마쳤습니다.

하지만 연동만 했다고 웹 애플리케이션이 배포되는 것은 아니죠. 이제 실제 배포를 위해서 추가 파일을 만들겠습니다.

 

 

 

### 10. Google App Engine 배포를 위한 파일 준비
 

이 부분은 쉽습니다. Google App Engine 배포를 위해서 제공하는 파일을 그대로 사용하면 됩니다.

 

먼저 App Engine에서 요구하는 패키지 설치를 위해서 requirements.txt를 생성합니다.

 

{% highlight Shell %}
(venv)/imageproj$ pip freeze > requirements.txt
{% endhighlight %} 
 

다음은 app.yaml 파일과 main.py 파일을 생성합니다. 이 2개의 파일은 Google App Engine 배포에만 사용하는 특수 파일로, 다음과 같이 입력합니다.

 

app.yaml 파일

{% highlight YAML %}
runtime: python37

handlers:
- url: /static
  static_dir: static/

- url: /.*
  script: auto
{% endhighlight %} 
 

main.py 파일

{% highlight Python %}
from imageproj.wsgi import application

app = application
{% endhighlight %} 
 

그리고 마지막으로 배포 명령어를 실행합니다. 

 

{% highlight Shell %}
(venv)/imageproj$ gcloud app deploy
{% endhighlight %} 

배포가 완료되면 App Engine에도 신규 웹 애플리케이션이 생성된 것을 확인할 수 있습니다.

웹 애플리케이션은 프로젝트에 명시된 웹페이지 주소로 입력하면 들어갈 수 있으며, 이 글에서 설명한 대로 하면 에러가 발생할 것입니다. 메인 페이지를 전혀 설정하지 않았기 때문입니다. 그러므로 '/imageapp/upload_image/' 를 붙인 후 들어가면 정상적으로 실행되는 것을 확인할 수 있을 것입니다.

 


이상입니다.

예상대로 글이 엄청 길었네요.

 

Google Cloud에서 웹 애플리케이션을 구축하기 위해서는 App Engine과 Storage 사용 및 연동은 거의 필수라고 봐야 합니다. 그 중에서 Django 웹 애플리케이션 구축 및 배포를 위해서 여러 방법들이 제시되어 있지만, 아쉽게도 모든 요구사항을 만족할 만한 글이 없었다고 판단되어 제가 직접 github에 프로젝트 소스코드도 올리고 이렇게 글도 쓰게 되었습니다.

 

저와 같은 것을 만들려는 분들께 도움이 되었으면 좋겠습니다.

감사합니다.
