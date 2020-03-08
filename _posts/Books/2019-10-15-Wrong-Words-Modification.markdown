---
layout: post
title:  "오타 수정 예정 안내"
date:   2020-02-17
categories: [Books]
tags: [AWS, Cloud, Django, Amazon, books, 장고, 클라우드, Application, 애플리케이션, Python, 파이썬]
---

#### ※ 최근 수정일: 2020-03-09

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
	<li>182P: boards 테이블에 값을 입력하는 부분과 auth_user 테이블에 값을 입력하는 부분이 있는데, 두 부분 위치를 바꾸겠습니다.</li>
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
	<li>197P: 가운데 views.py 코드에서 5번째 줄을 다음과 같이 수정합니다.</li>
</ul>
```python
05: 	return render(request, 'boardsview.html', \{ 'board': boardList \})
```
<ul>	
	<li>205P: 하단 예제 파일명 test_proj/urls.py -> test_proj/settings.py로 변경합니다.</li>
	<li>216P: 맨 아래에 있는 코드를 다음과 같이 변경합니다.</li>
</ul>
```html
<a href="\{% url 'fbv' %\}">FBV List Link</a>
```
<ul>
	<li>219P: default_if_none의 표현방법을 아래와 같이 변경합니다.</li>
</ul>
{% highlight HTML %}
{{ value|default_if_none:"empty" }}
{% endhighlight %}
<ul>
	<li>220P: dicsort를 dictsort로 모두 변경하고, dicsortreversed를 dictsortreversed로 변경합니다.</li>
	<li>227P: [표 12-1]로 기재된 것을 [표 13-1]로 변경합니다.</li>
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
	<li>287P: 상단 예제 코드에 DB Engine의 NAME 부분이 'awsdjangoprojdb'로 되어 있는데 'awsdjangodb'가 맞습니다. 정정하겠습니다.</li>
	<li>295P: board_images 테이블은 존재하지 않는 테이블입니다. 집필 중간 과정에서 생성했다가 빼기로 한 테이블인데 해당 부분 내용이 남아있었던 점 양해 바랍니다. 코드는 아래와 같이 수정되니 참고 바라며, 첨부 이미지는 빼도록 하겠습니다.<br>그리고 이와 관련하여 추가로 입력할 내용이 있습니다. 이 부분은 <a href="https://amanokaze.github.io/blog/Customize-Model-Field/">다음 게시물</a>에서 다루겠습니다.</li>
</ul>

{% highlight Shell %}
(ve)$ python manage.py inspectdb board_categories boards board_replies board_likes >> boardapp/models.py
{% endhighlight %}

<ul>
	<li>316P: 위쪽 코드의 5Line 부분의 id="email_id" id="email_id"로 중복으로 들어갔습니다. 하나를 빼겠습니다. Github 소스코드도 확인결과 중복으로 들어간 것으로 확인되어 수정하였으니 참고 바랍니다.</li>
	<li>327P: 코드의 1 Line에 request → redirect로 정정</li>
	<li>332P: 코드의 41 Line에 </h5>로 끝나는 태그가 있습니다만, </h4>로 변경해야 합니다. Github Source Code에도 똑같이 변경사항 반영했습니다.</li>
	<li>341P: boardapp/static/boardapp/assets/suser.js → user.js 로 파일명 변경합니다. 단순 오타입니다.</li>
	<li>409, 410P: 소스코드 부분의 board_comm_list.html → board_comm_view.html로 정정합니다.</li>
	<li>501P: # DEBUG = True → DEBUG = True (# 기호 제거)</li>
</ul>
