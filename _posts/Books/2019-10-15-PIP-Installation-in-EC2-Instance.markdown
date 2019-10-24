---
layout: post
title:  "AWS EC2 인스턴스에서 PIP 설치하기"
date:   2019-10-15
image: '/assets/img/img005_01.png'
categories: [Books]
tags: [AWS, Cloud, Django, Amazon, books, 장고, 클라우드, Application, 애플리케이션, pip, pip3, Python, 파이썬 ]
---

Python 및 PIP 설치와 관련해서 참고로 알려드릴 사항이 있어서 글을 써봅니다.

제가 집필한 책에서도 그렇고, 일반적으로는 Python3를 설치하면 pip3도 설치할 수 있습니다.

{% highlight Shell %}
$ sudo apt install python3
$ sudo apt install python3-pip
{% endhighlight %}

하지만, AWS에서 제공하는 EC2 인스턴스 중 일부 운영체제의 버전에서는 python3 설치는 정상적으로 되나, pip3가 설치되지 않는 경우가 발생하는 것으로 확인되었습니다.

![PIP Error]({{ '/assets/img/img005_01.png' | prepend: site.baseurl }})


위와 같이 'Unable to locate package python3-pip'가 나오면서 설치가 되지 않을 수도 있습니다.

하지만 해결방법은 의외로 간단하니 참고바랍니다.

python3를 설치한 후, APT 패키지 관리자를 업데이트-업그레이드를 해주시기 바랍니다.

{% highlight Shell %}
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install python3-pip
{% endhighlight %}

위와 같이 패키지 업그레이드까지 완료된 후, 다시 pip3를 설치하면 정상적으로 설치가 이루어집니다.

![PIP Completed]({{ '/assets/img/img005_02.png' | prepend: site.baseurl }})


Python3 및 PIP3 설치 시 참고하시기 바라겠습니다.

감사합니다.