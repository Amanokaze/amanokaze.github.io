---
layout: post
title:  "오타 수정 예정 안내"
date:   2020-04-06
categories: [Books]
tags: [AWS, Cloud, Django, Amazon, books, 장고, 클라우드, Application, 애플리케이션, Python, 파이썬]
---

#### ※ 최근 수정일: 2020-04-06
#### 일부 코드는 highlight를 그대로 적용하면 문법 충돌로 오류가 발생하므로 Github Gist에서 코드를 가져오니 참고바랍니다.
#### 생각보다 오타가 정말 많네요. 사실 책을 집필하는 과정에서 예제 프로그램을 급하게 작성하다 보니까 일부 수정된 내용을 책에 미반영된 사항이 있어서 그런 것이니 양해 바라며, 더욱 많은 사랑 바라겠습니다. 감사합니다.

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
<script src="https://gist.github.com/Amanokaze/1b6f1dd507d8fbe924581659f8a5c311.js"></script>
<ul>	
	<li>205P: 하단 예제 파일명 test_proj/urls.py -> test_proj/settings.py로 변경합니다.</li>
	<li>216P: 맨 아래에 있는 코드를 다음과 같이 변경합니다.</li>
</ul>
<script src="https://gist.github.com/Amanokaze/17001bb97c1a05adb3e67d09f6417afe.js"></script>
<ul>
	<li>219P: default_if_none의 표현방법을 아래와 같이 변경합니다.</li>
</ul>
<script src="https://gist.github.com/Amanokaze/d0d9b429133db553285334f28a7ea90e.js"></script>
<ul>
	<li>220P: dicsort를 dictsort로 모두 변경하고, dicsortreversed를 dictsortreversed로 변경합니다.</li>
	<li>227P: [표 12-1]로 기재된 것을 [표 13-1]로 변경합니다.</li>
	<li>236P: difference() 항목에서 결과는 "John's diary", "Emma's novel"로 수정합니다.</li>
	<li>239P: exists() 항목 예제 자체는 문제가 없으나, 아래와 같이 수정하는 것이 조금 더 바람직 할 것으로 생각되어 아래와 같이 수정합니다.</li>
</ul>
<script src="https://gist.github.com/Amanokaze/13e1f8665ccc865c5b85ed61f3e8d8b0.js"></script>
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
	<li>287P: 상단 예제 코드에 DB Engine의 NAME 부분이 'awsdjangoprojdb'로 되어 있는데 'awsdjangodb'가 맞습니다. 정정하겠습니다.</li>
	<li>294P: create_user() 메소드의 phone_number를 phone으로 수정합니다.</li>
	<li>295P: board_images 테이블은 존재하지 않는 테이블입니다. 집필 중간 과정에서 생성했다가 빼기로 한 테이블인데 해당 부분 내용이 남아있었던 점 양해 바랍니다. 코드는 아래와 같이 수정되니 참고 바라며, 첨부 이미지는 빼도록 하겠습니다.<br>그리고 이와 관련하여 추가로 입력할 내용이 있습니다. 이 부분은 <a href="https://amanokaze.github.io/blog/Customize-Model-Field/">다음 게시물</a>에서 다루겠습니다.</li>
</ul>

{% highlight Shell %}
(ve)$ python manage.py inspectdb board_categories boards board_replies board_likes >> boardapp/models.py
{% endhighlight %}

<ul>
	<li>307P: 8~9Line 부분에서 'boardapp/assets/css/'를 'boardapp/assets/js'로 수정합니다.</li>
	<li>316P: 위쪽 코드의 5Line 부분의 id="email_id" id="email_id"로 중복으로 들어갔습니다. 하나를 빼겠습니다. Github 소스코드도 확인결과 중복으로 들어간 것으로 확인되어 수정하였으니 참고 바랍니다.</li>
	<li>320P: 319P에서는 회원가입 취소 시 'cancelUserRegister()'로 입력되어 있는데 책에는 'cancelMemberRegister()'로 명시되어 있습니다. 맥락을 맞추기 위해서 'cancelUserRegister()'로 정정합니다. Github 코드에도 cancelUserRegister()로 되어 있는 부분 확인했습니다.</li>
	<li>324P: 맨 위의 user_register.js를 user.js로 정정, request.POST['id']를 request.POST['username']으로 정정, user_id를 username으로 정정</li>
	<li>327P: 코드의 1 Line에 request → redirect로 정정</li>
	<li>332P: 코드의 41 Line에 </h5>로 끝나는 태그가 있습니다만, </h4>로 변경해야 합니다. Github Source Code에도 똑같이 변경사항 반영했습니다.</li>
	<li>341P: boardapp/static/boardapp/assets/suser.js → user.js 로 파일명 변경합니다. 단순 오타입니다.</li>
	<li>348P: 게시물 조회 수 및 추천 수 → 게시물 댓글 및 추천 수</li>
	<li>356P: '이전' 부분 글씨체가 다르게 나타났습니다. 이건 출판사쪽에 문의해서 정정요구 하겠습니다.</li>
	<li>360P: 98Line의 article.id 바로 오른쪽에 중괄호 닫기 부분이 띄워쓰기가 되어 있지 않습니다. 코드 상 문제는 없으나 정정하겠습니다.</li>
	<li>360P: 실제 코드와 상이한 부분이 있어서 아래와 같이 수정합니다. 교재 상 88Line과 98Line이 빠져있습니다. Github 코드에는 이미 적용되어 있으므로 수정사항 없습니다.</li>
</ul>
<script src="https://gist.github.com/Amanokaze/5b83e343b9feadf6c24e96275968b19d.js"></script>
<ul>
	<li>364P: 아래 코드의 3Line을 다음과 같이 변경합니다. Github 코드에도 똑같이 변경 반영했습니다.</li>
</ul>
<script src="https://gist.github.com/Amanokaze/7b351ab310986d379358c63b961dc61c.js"></script>
<ul>
	<li>385, 386P: 소스코드를 전면 수정합니다. Github의 코드와 동일하게 구현해야 하므로 아래와 같이 정정하겠습니다.</li>
</ul>
<script src="https://gist.github.com/Amanokaze/0cb85f3e1203aef993bd363206e254aa.js"></script>
<ul>
	<li>386, 387P: 소스코드를 전면 수정합니다. Github의 코드와 동일하게 구현해야 하므로 아래와 같이 정정하겠습니다.</li>
</ul>
<script src="https://gist.github.com/Amanokaze/5c4253e38224612ce437a854e6c99183.js"></script>
<ul>
	<li>403P: 13Line 부분 내용을 다음과 같이 수정합니다.
		<p><i>category 변수를 사용하여 카테고리에 대한 정보를 board_category 변수에 저장하는 부분이다. 일반 게시판 글쓰기 처리에서는 category 변수를 request.POST 변수를 사용하여 가져오나, 
		여기에서는 파라미터 변수로 category를 가져와서 사용한다는 차이가 있으니 이에 유의한다.</i></p>
	</li>
	<li>403P: 위의 15 Line 코드를 아래와 같이 수정합니다. Github에 올라온 코드와 다르게 적혀서 있어서 정정하니 참고바랍니다.</li>	
</ul>
<script src="https://gist.github.com/Amanokaze/4078b280d20ac19172a617c76d7b1213.js"></script>
<ul>
	<li>405P: board_comm_list.html의 3Line 부분을 아래와 같이 조정합니다. 오타는 아니지만 article.title을 불러오는 부분이 없기 때문에 대화형 게시판의 카테고리명을 Title로 나타내는 것이 더욱 나을 것 같습니다. 이 부분은 Github 코드에도 동일하게 수정하였습니다.</li>
</ul>
<script src="https://gist.github.com/Amanokaze/8a80ec778840436e86385c122f40c8b2.js"></script>
<ul>
	<li>410P: 소스코드 부분의 board_comm_list.html → board_comm_view.html로 정정합니다.</li>
	<li>412P, 413P: 코드 부분에 굵은 표시가 왜 나왔는 지 모르겠네요. 편집 과정에서 실수가 있었던 것 같습니다. 출판사 측에 문의해서 조정하겠습니다.</li>
	<li>412P: 맨 아래에 board_delete_res() → board_delete_result()</li>
	<li>418P: 출판사 편집 과정에서 오류가 있었던 것으로 추정됩니다. 12Line 부분을 한 칸 내려주시기 바랍니다.</li>
	<li>485P: AbstractUser를 AbstractBaseUser로 수정합니다.</li>
	<li>501P: # DEBUG = True → DEBUG = True (# 기호 제거)</li>
</ul>
