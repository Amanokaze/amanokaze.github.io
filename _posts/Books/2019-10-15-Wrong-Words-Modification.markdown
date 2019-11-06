---
layout: post
title:  "오타 수정 예정 안내"
date:   2019-11-06
categories: [Books]
tags: [AWS, Cloud, Django, Amazon, books, 장고, 클라우드, Application, 애플리케이션, Python, 파이썬]
---

#### ※ 최근 수정일: 2019-11-06

서적 1판1쇄에서 현재까지 나타난 오타 안내입니다.

오타는 2판 발행 시 수정 적용하겠습니다.

<ul>
	<li>11P: 사물인터넷(Internet of Thins) → 사물인터넷(Internet of Things)</li>
	<li>137P: 맨 위의 코드의 두 번째 둘이 다음 줄로 이동되지 않았습니다. 다음 구문이 맞습니다.</li>
</ul>

{% highlight Shell %}
$ cat .profile
$ source ~/.profile
$ virtualenv --verision
{% endhighlight %}

<ul>
	<li>169P: set variables → show variables</li>
	<li>181P: 테이블 선언 부분의 컬럼명을 감싸는 기호가 Grave(`) 기호를 사용해야 하는데 Apostrophe(') 기호가 들어갔습니다. 이 부분 주의 바라겠습니다.</li>
	<li>181P: boards 테이블 생성에서 이름 표기가 잘못된 부분도 같이 확인했습니다. 수정된 부분은 'category_id'와 'user_id' 위치를 바꾸었으며, 다음과 같이 수정하겠습니다.</li>
</ul>

{% highlight SQL %}
create table `boards` (
	`id` int(10) not null auto_increment,
	`category_id` int(10) not null,
	`user_id` int(10) not null,
	`title` varchar(300) not null,
	`content` text not null,
	`registered_date` datetime default current_timestamp,
	`last_update_date` datetime default null,
	`view_count` int(10) default '0',
	`image` varchar(255) default null,
	PRIMARY KEY (`id`),
	FOREIGN KEY (`category_id`) REFERENCES `board_categories` (`id`),
	FOREIGN KEY (`user_id`) REFERENCES `auth_user` (`id`)
)
{% endhighlight %}

<ul>
	<li>241P: AuthUser 모델 클래스의 필드에서 사용자 정의 필드가 있는 것이 확인되었습니다. 사용자 정의 필드는 PART 3에서 추가되지만, PART 2에서는 추가가 안 된 상태로 그대로 표시하는 것이 맞으므로 해당 부분을 삭제해서 다음과 같이 나타내겠습니다.</li>
</ul>

{% highlight Python %}
03: class AuthUser(models.Model):
04: 	password = models.CharField(max_length=128)
05: 	last_login = models.DateTimeField(blank=True, null=True)
06: 	is_superuser = models.IntegerField()
07: 	username = models.CharField(unique=True, max_length=150)
08: 	first_name = models.CharField(max_length=30, blank=True, null=True)
09: 	last_name = models.CharField(max_length=150)
10: 	email = models.CharField(max_length=254)
11: 	is_staff = models.IntegerField(blank=True, null=True)
12: 	is_active = models.IntegerField(blank=True, null=True)
13: 	date_joined = models.DateTimeField()
14:
15: 	class Meta:
16: 		managed = False
17: 		db_table = 'auth_user'
{% endhighlight %}

<ul>
	<li>243P: views.py의 7Line - template_file → template_name</li>
	<li>245P: testapp/templates/boardsfunctionview.html → testapp/templates/boards_list_fbv.html</li>
	<li>295P: board_images 테이블은 존재하지 않는 테이블입니다. 집필 중간 과정에서 생성했다가 빼기로 한 테이블인데 해당 부분 내용이 남아있었던 점 양해 바랍니다. 코드는 아래와 같이 수정되니 참고 바라며, 첨부 이미지는 빼도록 하겠습니다.<br>그리고 이와 관련하여 추가로 입력할 내용이 있습니다. 이 부분은 [다음 게시물](https://amanokaze.github.io/blog/Customize-Model-Field/)에서 다루겠습니다.</li>
</ul>

{% highlight Shell %}
(ve)$ python manage.py inspectdb board_categories boards board_replies board_likes >> boardapp/models.py
{% endhighlight %}

<ul>
	<li>501P: # DEBUG = True → DEBUG = True (# 기호 제거)</li>
</ul>