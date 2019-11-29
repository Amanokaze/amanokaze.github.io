---
layout: post
title:  "Windows 운영체제에서 Django와 MySQL 연동"
date:   2019-11-29
image: '/assets/img/img009_02.png'
categories: [Django]
tags: [Python, 파이썬, Django, MySQL, 장고, Windows, 운영체제, 연동]
---

Django 애플리케이션을 개발할 때, 어떤 환경에서 개발하느냐는 항상 중요한 이슈 중 하나로 볼 수 있습니다. 

일반적으로 Python 개발은 Linux 운영체제에서 진행하는 것이 최저화되어 있고, 저 또한 Django 개발을 하면서 AWS EC2 인스턴스나, GCE(Google Compute Engine) 운영체제 환경에서 작업을 했었는데, 이 때 사용됐던 운영체제도 전부 Ubuntu 리눅스였고요.

하지만 사실은 Windows 운영체제에서 개발하는 경우가 거의 대부분으로 볼 수 있습니다. Cloud 환경에서 개발할 때에는 Linux 운영체제를 사용하는 것이 간편하지만, Cloud 환경이 아닌 로컬 환경에서 개발을 한다면 굳이 Linux를 사용해야 할 필요는 없기 때문일까요.
물론 로컬 환경을 리눅스에서 진행하거나 Mac OS에서 진행하는 경우도 종종 볼 수 있겠고요. 실제로 제가 책을 출간하고 나서 들어온 문의 중 대부분은 Django 환경 설정에 관한 문의였었는데, 
문의 내용 중 하나는 Mac OS에서 Django 애플리케이션 환경설정을 하는 데 어려움이 있다라는 문의도 있곤 했습니다.

Windows에서 Django 환경 구축을 할 때에는 아무래도 로컬 환경이다 보니 Cloud 환경과 유사하면서도 버전관리가 주기적으로 이루어지는 것이 중요하기 때문에 Github를 사용해서 Repository를 관리하기도 하고, 
아무래도 GUI 운영체제 중에서는 가장 편리하게 사용할 수 있다보니 그와 잘 어울리는 IDE로 VS Code를 사용하고 있습니다. 그래서 어제도 이와 관련된 환경 설정 글도 올렸고요.

하지만 MySQL하고 연동하는 부분이 순조롭지는 않았습니다. 과연 무엇이 문제였을까요.

MySQL을 사용하려면 서버가 있어야겠죠. 가상환경(Virtualenv)에서 MySQL 연결을 위해서는 mysqlclient 모듈을 설치해야 합니다.

{% highlight Shell %}
$ pip install mysqlclient
{% endhighlight %}

Linux에서는 apt 패키지 관리자 등을 통해서(아마 다른 리눅스에서는 yum을 사용하겠지만) libmysqlclient-dev 를 설치해야하만 mysqlclient가 설치가 됩니다. 
하지만 Windows에서는 별 문제없이 설치가 잘 됩니다. 그렇다면 mysqlclient가 과연 올바르게 작동을 할까요?

먼저 settings.py에 MySQL을 연동하고,

{% highlight Python %}
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'dev',
            'USER': 'root',
            'PASSWORD': 'root',
            'PORT': '3306',
            'HOST': '12.34.56.78',
        }
    }
{% endhighlight %}    

models.py 파일에 모델을 연결한 후,

{% highlight Python %}
from django.db import models

class Test(models.Model):
    code = models.CharField(max_length=100)
    name = models.CharField(max_length=100)
    description = models.CharField(max_length=300, blank=True, null=True)

    def __str__(self):
        return '%s (%s)' % (self.name, self.code)

    class Meta:
        managed = False
        db_table = 'test'
{% endhighlight %} 

Django Python Shell에서 테스트도 해 보겠습니다.

{% highlight Python %}
>>> from collection.models import *
>>> a = Test.objects.all()
>>> print(a)
&lt;QuerySet [&lt;Test: game (game)>]>
>>> 
{% endhighlight %}

아주 잘 나옵니다.

이제 관리자 페이지(Admin)에 연결한 다음 실행해 보겠습니다.

{% highlight Python %}
from django.contrib import admin
from collection.models import Test

admin.site.register(Test)
{% endhighlight %}

![Admin Site Test]({{ '/assets/img/img017_01.png' | prepend: site.baseurl }})

택도 없네요. 아무것도 나오지 않습니다.

왜 그럴까요. 이유는 간단합니다.
**Windows에서 mysqlclient 패키지는 정상적으로 설치되었지만, libmysqlclient-dev를 설치하지 않았기 때문에 생기는 현상입니다. 결국은 Windows에서도 Linux와 동일하게 MySQL Connect를 위한 라이브러리를 설치해야 되겠죠?**

그러면 이제 Windows에서 MySQL Installer를 설치하러 갑니다.

<https://dev.mysql.com/downloads/installer/>

![MySQL Install]({{ '/assets/img/img017_03.png' | prepend: site.baseurl }})

화면에 64비트가 없고 32비트만 있죠? 상관없습니다. 일단 다운받으시고 실행하세요. 왜 상관없는 지는 아래에 이어서 나옵니다.

실행하면 다음 화면이 나타날 것입니다.

![MySQL Installer]({{ '/assets/img/img017_04.png' | prepend: site.baseurl }})

저는 MySQL Workbench가 설치되어 있어서 나오는 것이고요. 설치가 안되어 있다면 공란으로 나올 것입니다. 우측 상단의 'Add...'를 눌러주세요.

![MySQL Installer - Python]({{ '/assets/img/img017_05.png' | prepend: site.baseurl }})

위와 같이 
**MySQL Connectors - Connector/Python - Connection/Python 8.0 - Connector/Python8.0.18 - X64**
를 선택해 주시면 됩니다.

MySQL 홈페이지에서는 32비트 Installer를 설치하라고 나오지만, 어차피 Installer를 실행하면 안에 설치해야 할 라이브러리는 64비트로 선택해서 설치할 수 있습니다.
그래서 위에서 다운로드를 어떤것을 받아도 상관없다라고 언급한 것입니다. 

그리고 위 사진에서 보면 Connection/Python8.0이라고 나오는데, 이건 Python 버전이 8.0이라는 뜻이 아니라, Connector의 버전이 8.0이라는 뜻이므로 크게 신경쓰지 않으셔도 됩니다.

이제 오른쪽 화살표를 눌러서 옮겨주신 후, 설치를 진행합니다.

![MySQL Installer - Python Install]({{ '/assets/img/img017_06.png' | prepend: site.baseurl }})

Execute 누르시고요. Next - Finish까지 누르면 완료 됩니다.

![MySQL Installer - Python Install Completed]({{ '/assets/img/img017_07.png' | prepend: site.baseurl }})

설치된 결과가 위와 같이 나올 것입니다. 

이제 이 상태에서 Admin 사이트를 다시 들어가도록 할게요.

![Modified Django Admin]({{ '/assets/img/img017_02.png' | prepend: site.baseurl }})

이제서야 잘 나오는 것을 확인할 수 있었네요. 앞으로는 Windows에서도 Django 애플리케이션 개발 시 MySQL 연동을 문제없이 할 수 있을 것입니다.


요약하겠습니다.

* Windows에서도 MySQL을 사용하기 위해서는 libmysqlclient가 필요하다.
* MySQL 홈페이지에서 Installer 다운로드 후 설치한다.
* 모든 설치가 완료되면 가상환경(Virtualenv)에서도 pip로 mysqlclient를 설치한다.

간단하죠?

Windows에서도 Django 개발을 잘 하길 바라겠습니다.