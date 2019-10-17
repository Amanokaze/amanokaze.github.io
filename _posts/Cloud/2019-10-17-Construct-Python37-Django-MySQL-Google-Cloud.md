---
layout: post
title:  "Google Cloud에서 Python3.7 - Django - MySQL 환경 구축"
date:   2019-10-17
categories: [Cloud]
tags: [Python, 파이썬, Python37, Google, Cloud, Django, 장고, MySQL, SQL, 구글, 클라우드]
---

 
이번에 쓸 글은 Google Cloud에서 Python3.7과 Django - MySQL 환경을 구축하는 내용입니다.

 

배포 환경은 아니고 개발 환경을 구축하는 부분까지 다루었으며, 버전은 다음과 같습니다.

 

* VM 인스턴스: Ubuntu 18.04 LTS
* Python: 3.7
* Django: 2.2
* MySQL: 5.7

 

Google Cloud에서 Django 개발 환경을 구축하기 위해서는 Compute Engine을 사용해야 합니다.

여기서는 순수하게 환경을 구축하는 법만 다룰 예정이므로, Compute Engine의 VM 인스턴스와 MySQL의 SQL 인스턴스는 이미 생성되어 있다고 가정하겠습니다. 

 

Ubuntu 18.04 LTS에서는 Python 3.6이 기본으로 설치되어 있습니다.

하지만 추후에 다룰 Google Cloud에서 Django 웹 애플리케이션을 App Engine에 배포하기 위해서는 Python 3.7버전을 사용하는 것을 권장하고 있습니다. 그러므로 Python 3.7 설치를 진행하는 것이니 참고 바랍니다.

 

 

### 1. Python 3.7 설치
 

Python 3.7 설치를 위해서 먼저 다음 명령어를 수행합니다.

 
{% highlight Shell %}
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository ppa:deadsnakes/ppa
$ sudo apt update
$ sudo apt-get install python3.7
{% endhighlight %}

여기까지 진행하면 Python 3.7 설치가 완료됩니다.

그러나 Python 3.6도 여전히 있고, 기본 Python으로 인식하고 있습니다. 실제로 pip3를 설치를 해도 pip 호환 Python 버전은 3.6으로 인식됩니다.

 

그래서 pip3를 설치하기 전에 Python 기본 버전 변경이 우선되어야 합니다.

 

 

### 2. Python 기본 버전 변경

기본 버전 변경을 위한 설정 방법은 다음과 같습니다.

 
{% highlight Shell %}
$ sudo update-alternatives --config python
update-alternatives: error: no alternatives for python
{% endhighlight %}

하지만 에러가 발생하죠.

 

그러므로 update-alternatives에 Python 3.6과 3.7을 모두 등록해야 합니다.

 
{% highlight Shell %}
$ sudo update-alternatives --install /usr/bin/python3 python /usr/bin/python3.6 1
$ sudo update-alternatives --install /usr/bin/python3 python /usr/bin/python3.7 2
{% endhighlight %}

Ubuntu 18.04에서 설치하면 Python 3.6은 /usr/bin/python3.6으로, 3.7은 /usr/bin/python3.7로 설치되니 참고바랍니다. 만약 위치가 다르면 whereis python3.6 이런 식으로 찾으시면 됩니다.

 

이제 버전 선택할게요.

 
{% highlight Shell %}
$ sudo update-alternatives --config python
There are 2 choices for the alternative python (providing /usr/bin/python).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /usr/bin/python3.7   2         auto mode
  1            /usr/bin/python3.6   1         manual mode
  2            /usr/bin/python3.7   2         manual mode

Press <enter> to keep the current choice[*], or type selection number: 2
{% endhighlight %}

2번 선택하시면 됩니다. 이러면 버전 선택은 모두 완료되었습니다.

 

 

### 3. PIP / VEnv / Django 설치
 

이제 Python 버전도 3.7이 기본 버전으로 되었으니 다음 설치를 진행합니다.

PIP3 설치하시고요.

 
{% highlight Shell %}
$ sudo apt-get install python3-pip
{% endhighlight %}


Django 가상환경 구축을 위한 VENV(구 Virtualenv)도 설치하고 가상환경 생성까지 진행합니다.

 
{% highlight Shell %}
$ sudo apt-get install python3.7-venv
$ python3 -m venv venv
{% endhighlight %}


다음은 가상환경 들어가셔서 Django 2.2 설치까지 같이 진행하고, 프로젝트도 생성합니다.

 
{% highlight Shell %}
$ source venv/bin/activate
(venv)$ pip install django==2.2
(venv)$ django-admin startproject testproj
{% endhighlight %}


여기까지는 아무 이상 없습니다. 기존 설치방법과 동일합니다. 이제 Django에서 MySQL을 사용하기 위한 설정을 진행하겠습니다.

 

 

 

### 4. mysqlclient 설치
 

일반적으로 Django에서 MySQL을 사용하기 위해서는 

 
{% highlight Shell %}
$ sudo apt-get install  default-libmysqlclient-dev

$ pip install mysqlclient
{% endhighlight %}

 

를 사용해서 설치합니다만, Python 3.7에서 위와 같이 설치했다가는 다음과 같은 에러를 봅니다. 

(올바른 방법이 아니므로 위 부분은 일부러 코드블럭을 안했습니다. 주의하세요!!)

 
{% highlight Shell %}
(venv)$ pip install mysqlclient
Collecting mysqlclient
  Using cached https://files.pythonhosted.org/packages/4d/38/c5f8bac9c50f3042c8f05615f84206f77f03db79781db841898fde1bb284/mysqlclient-1.4.4.tar.gz
Installing collected packages: mysqlclient
  Running setup.py install for mysqlclient ... error
    ERROR: Command errored out with exit status 1:
     command: /home/onikaze_books/venv/bin/python3.7 -u -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-6r2u7i3e/mysqlclient/setup.py'"'"'; __file__='"'"'/tmp/pip-install-6r2u7i3e/mysqlclient/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' install --record /tmp/pip-record-xwitjcp5/install-record.txt --single-version-externally-managed --compile --install-headers /home/onikaze_books/venv/include/site/python3.7/mysqlclient
         cwd: /tmp/pip-install-6r2u7i3e/mysqlclient/
    Complete output (31 lines):
    running install
    running build
    running build_py
    creating build
    creating build/lib.linux-x86_64-3.7
    creating build/lib.linux-x86_64-3.7/MySQLdb
    copying MySQLdb/__init__.py -> build/lib.linux-x86_64-3.7/MySQLdb
    copying MySQLdb/_exceptions.py -> build/lib.linux-x86_64-3.7/MySQLdb
    copying MySQLdb/compat.py -> build/lib.linux-x86_64-3.7/MySQLdb
    copying MySQLdb/connections.py -> build/lib.linux-x86_64-3.7/MySQLdb
    copying MySQLdb/converters.py -> build/lib.linux-x86_64-3.7/MySQLdb
    copying MySQLdb/cursors.py -> build/lib.linux-x86_64-3.7/MySQLdb
    copying MySQLdb/release.py -> build/lib.linux-x86_64-3.7/MySQLdb
    copying MySQLdb/times.py -> build/lib.linux-x86_64-3.7/MySQLdb
    creating build/lib.linux-x86_64-3.7/MySQLdb/constants
    copying MySQLdb/constants/__init__.py -> build/lib.linux-x86_64-3.7/MySQLdb/constants
    copying MySQLdb/constants/CLIENT.py -> build/lib.linux-x86_64-3.7/MySQLdb/constants
    copying MySQLdb/constants/CR.py -> build/lib.linux-x86_64-3.7/MySQLdb/constants
    copying MySQLdb/constants/ER.py -> build/lib.linux-x86_64-3.7/MySQLdb/constants
    copying MySQLdb/constants/FIELD_TYPE.py -> build/lib.linux-x86_64-3.7/MySQLdb/constants
Setting up libpython3.7:amd64 (3.7.4-1+bionic3) ...
Setting up libpython3.7-dev:amd64 (3.7.4-1+bionic3) ...
Setting up python3.7-dev (3.7.4-1+bionic3) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Processing triggers for libc-bin (2.27-3ubuntu1) ...
{% endhighlight %}

그러면 어떻게 해야 하냐.

만약에 default-libmysqlclient-dev 패키지가 이미 설치되어 있다면 삭제를 먼저 해주시고요. (설치 안되어있으면 안해도 됩니다)

 
{% highlight Shell %}
$ sudo apt remove lib-mysqlclient-dev
{% endhighlight %}


다음과 같이 python3.7-dev를 설치해야 합니다.

 

{% highlight Shell %}
$ sudo apt-get install python3.7-dev default-libmysqlclient-dev
{% endhighlight %}


위 설치가 완료되면 mysqlclient도 정상적으로 설치할 수 있습니다.

 

{% highlight Shell %}
(venv)$ pip install mysqlclient
Collecting mysqlclient
  Using cached https://files.pythonhosted.org/packages/4d/38/c5f8bac9c50f3042c8f05615f84206f77f03db79781db841898fde1bb284/mysqlclient-1.4.4.tar.gz
Installing collected packages: mysqlclient
  Running setup.py install for mysqlclient ... done
Successfully installed mysqlclient-1.4.4
{% endhighlight %}
 

이상으로 Google Cloud에서 Python3.7 기반의 Django - MySQL 설치 및 설정방법을 마치도록 하겠습니다.

 

 

이 글을 쓰기까지 도움받은 문서가 있어서 같이 첨부해드리니, 더불어 참고하시면 좋을 것 같습니다.

 

Python 3.7 설치 - <https://aliwo.github.io/swblog/linux/ubuntu/ubuntu-new-python/#>

Python 버전 선택 - <https://codechacha.com/ko/change-python-version/>

MysqlClient for Python 3.7 - <https://stackoverflow.com/questions/51117503/python-3-7-failed-building-wheel-for-mysql-python>
