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
 
