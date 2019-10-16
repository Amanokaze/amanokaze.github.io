---
layout: post
title:  "Category 데이터 Insert"
date:   2019-10-15
image:  "assets/img/img003_01.png"
category: [Books]
tags: [AWS, Cloud, Django, Amazon, books, 장고, 클라우드, Application, 애플리케이션, MySQL, Table, 테이블, Python, 파이썬]
---
안녕하세요. AWS 클라우드 기반 Django 웹 애플리케이션 서적의 Part 3 웹 애플리케이션 개발 및 배포 관련하여 추가 입력 사항이 있어서 알려드립니다.
 
이 부분은 현재 1판으로 나온 책의 291페이지의

#### Chapter 16. 웹 애플리케이션 환경 설정
##### 001. Django 환경 설정
##### 2. DB 생성 및 테이블 구성

파트의 뒤에 들어갈 예정이며, 들어갈 부분은 '<font color="#FF0000"><B>3. 설정 데이터 구성</B></font>' 이라는 주제로 하겠습니다.
이 파트는 책의 일부에 추가로 넣어야 하는 부분인 관계로, 책을 집필할 때 썼던 문체로 작성하겠습니다.

 
<hr/>

### 3. 설정 데이터 구성

#### 1) 설정 데이터 개요
 

일반적으로 회원정보 및 게시판 데이터는 웹페이지에서 사용자로부터 입력을 받아서 데이터를 저장하고 관리하도록 되어 있다. 하지만 일부 데이터의 경우에는 환경 설정 시 미리 입력해야 하는 경우도 있다. 이 책에서는 이러한 데이터를 '설정 데이터'라 하며, 이를 구성한다. 

설정 데이터는 Chapter 21의 '003. 관리자 페이지'에서 나타난 것과 같이 관리자 페이지에 접속한 후 직접 입력하는 방법이  가장 일반적으로 사용되며, 그 외에도 DB 인스턴스에 접속하여 직접 데이터를 삽입(Insert)하는 방법도 있다. 관리자 페이지에서 데이터를 직접 입력하는 방법은 Chapter 21에 상세히 나와 있으므로, 이 챕터에서는 DB 인스턴스에서 데이터를 삽입하는 방법을 다룬다.

앞서 생성한 테이블은 회원정보 및 게시판 테이블로, 회원정보는 auth_user 외에 추가로 관리하는 테이블이 없는 관계로, 이와 관련된 설정 데이터는 없다. 하지만 게시판 테이블은 게시물 카테고리(board_categories) 테이블의 데이터를 설정 데이터로 다룬다. 게시물(board), 게시판 댓글(board_replies), 게시판 추천(board_likes) 테이블은 웹페이지에서 입력된 데이터를 저장하는 용도로 사용되는 반면, 게시판 카테고리(board_categories) 테이블은 미리 카테고리가 사전에 구성이 되어 있어야 하기 때문에 역시 해당 테이블에 들어갈 데이터를 설정 데이터로 한다.

 

#### 2) 게시판 카테고리 데이터 입력
 

Chapter 15의 259P 웹페이지 설계 부분을 보면, 웹페이지 설계 시 고려사항으로 기본 메뉴를 소개, 공지사항, 자유게시판, 대화형 게시판, 로그인/회원가입 등으로 구성하도록 되어 있다. 여기에서 소개 페이지는 게시판이 아닌 반면, 공지사항, 자유게시판, 대화형 게시판은 게시판 메뉴이므로, 게시판 카테고리 역시 이를 감안하여 각각의 데이터를 입력한다.


앞서 추가한 게시판 카테고리 테이블의 컬럼에 따라, 입력할 데이터는 다음 [표]와 같이 구성한다.

| category_type<br/>(카테고리 유형) | category_code<br/>(카테고리 코드) | category_name<br/>(카테고리 이름) | category_desc<br/>(카테고리 설명) | list_count<br/>(페이지당 출력 글) | authority<br/>(권한 값) |
|:---:|:---:|:---:|:---:|:---:|:---:|
| normal | notice | 공지사항 | 홈페이지의 주요 사항을 알리는 공간입니다.| 20 | 1 |
| normal | free | 자유게시판 | 원하는 주제의 글을 자유롭게 올릴 수 있는 공간입니다. | 10 | 0 |
| communication | comm | 대화형 게시판 | SNS와 유사한 형태로 실시간으로 글을 올리는 공간입니다. | 5 | 0 |

위 표를 토대로 DB 인스턴스에 데이터를 삽입하는 구문은 다음과 같다.

{% highlight SQL %}
insert into board_categories(category_type, category_code, category_name, category_desc, list_count, authority)  values ('normal', 'notice', '공지사항', '홈페이지의 주요 사항을 알리는 공간입니다.', 20, 1);
insert into board_categories (category_type, category_code, category_name, category_desc, list_count, authority)  values ('normal', 'free', '자유게시판', '원하는 주제의 글을 자유롭게 올릴 수 있는 공간입니다.', 10, 0);
insert into board_categories (category_type, category_code, category_name, category_desc, list_count, authority)  values ('communication', 'comm', '대화형 게시판', 'SNS와 유사한 형태로 실시간으로 글을 올리는 공간입니다.', 5, 0);
{% endhighlight %}

생성된 데이터는 다음 [그림] 과 같이 나타난다.

![Table Insert Result](/assets/img/img003_01.png)


id는 자동으로 생성되므로, 데이터 삽입 시 입력하지 않아도 자동으로 생성되며, creation_date 역시 현재 시간으로 자동으로 생성된다. 그 외의 데이터는 위 구문에 따라 직접 입력한 내용으로, 실제 게시판을 구성할 때에 해당 데이터를 사용하여 게시물 데이터를 다루고 웹페이지에도 표시하는 역할을 수행한다.

설정 데이터 입력까지 완료하였으면, 웹 애플리케이션 구축을 위한 모든 사전 준비는 마쳤다.

 
<hr/>
 

이상입니다.

 

처음 집필하는 책이다 보니까, 실제 내용을 글로 옮길 때 누락된 부분이 일부 자꾸 발생하네요.

다시한번 양해를 부탁드리며, 이렇게 블로그를 통해서 독자들하고 지속적으로 소통할 수 있도록 하겠습니다.
