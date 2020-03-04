---
layout: post
title:  "Spring Boot + Vue.js + PostgreSQL 개발환경 세팅(Overview)"
date:   2020-03-04
image: 'img022_01.png'
categories: [Integration]
tags: [spring,boot,vue,js,vue.js,vuesax,postgresql,setting,overview,develoment,environment,스프링,부트,뷰,세팅,개발,환경,개발환경]
---


안녕하세요.

이번 글은 Spring Boot + Vue.js + PostgreSQL 개발 환경 세팅을 위한 전체 요약 글을 써보겠습니다.
말 그대로 요약 글이기 때문에 세부 설명은 이 글에서는 생략될 것으로 생각되니 참고바랍니다.

이번에 신규 환경을 구축하면서 여러 가지 자료를 수도 없이 찾아보았습니다만, 실제로 도움이 되는 자료는 얼마 없었습니다.
하지만 그나마 가장 도움이 될만한 자료를 찾아서 이를 기반으로 구축할 수 있었습니다. 다시한번 감사의 말을 전하겠습니다.

참고 자료 안내

* [BezKoder - Spring Boot / Vue.js Overview](https://bezkoder.com/spring-boot-vue-js-authentication-jwt-spring-security/)
* [BezKoder - Spring Boot Setting](https://bezkoder.com/spring-boot-jwt-authentication/)
* [BezKoder - Vue.js Setting](https://bezkoder.com/jwt-vue-vuex-authentication/)

일단 환경 구축에 앞서서, 많은 사람들이 고민을 해 봅니다.

Spring Boot를 써 본 사람도 있을 것이고 Vue.js를 써 본 사람도 있지만 그 둘을 어떻게 연동할까?

왜냐하면 이들 개발환경은 각각 독립된 개발환경이기 때문에 연동을 한다는 것이 잘 상상이 가지 않을 수도 있기 때문이랄까요.

먼저 개발환경을 세팅하기 전에 가장 먼저 알아야 할 사실이 있습니다. 그것은 바로 백엔드(Back-end)와 프론트엔드(Front-end)에 대한 이해입니다.
이 둘 간의 구조를 이해해야 개발환경 세팅이 가능해지겠죠.

![Overview Process]({{ '/assets/img/img022_01.png' | prepend: site.baseurl }})

전체를 요약하면 위와 같이 구성이 가능합니다.

* Vue.js에서 Front-end를 담당하고
* JWT(Java Web Token)을 사용하여 Front-end와 Back-end를 통신하고
* Spring Boot에서 Back-end를 담당하고
* DB 연결을 위해서 MyBatis / JPA를 사용하며
* PostgreSQL을 사용하여 데이터를 관리한다

결국은 위와 같은 구조로 가야 연동이 가능하겠죠.

물론 각 층 간의 통신을 꼭 JWT라던가 MyBatis / JPA를 사용한다던가 그래야 할 필요는 없습니다. 각각의 개발환경에 맞게 사용해도 되겠죠.
다만 이 블로그에서는 당연히 해당 수단을 기반으로 작성할 예정이니 참고하시기 바라겠습니다.

그렇다면 어떤 순서로 개발해야 할까요. 답은 이미 나와 있습니다.

#### 1. PostgreSQL DB Server를 구축한다.
#### 2. Spring Back-end 환경을 구축한다.
#### 3. Vue.js Front-end 환경을 구축한다.

네. 그렇습니다. 구축 순서는 당연히 최하단에서 최상단으로 구축되어야 합니다. 

먼저 PostgreSQL을 사용하여 DB 서버를 구축하는 것은 여기에서는 다루지 않겠습니다. 왜냐하면 별도의 설정 없이 가장 기본적인 세팅으로도 가능하기 때문입니다.

![PostgreSQL Setting]({{ '/assets/img/img022_02.png' | prepend: site.baseurl }})

Localhost에서 사용할 PostgreSQL Server를 설치하고 계정을 생성한 후, Table 스키마 및 Function 간단한 것 몇 개만 만들면 그것으로 끝입니다.
이 부분은 별도로 설명하지 않아도 괜찮을 것으로 생각됩니다.

다음은 Spring Back-end 환경과 Vue.js Front-end 환경을 구축해야 되겠죠?

하지만 구축에 앞서서 가장 중요한 것이 있습니다.
Front-end와 Back-end의 경계를 어디까지로 둬야 할 것인가. 무엇을 구분해야 할 것인가.
이 글에서는 여기에 대한 구분까지만 하겠습니다.

일단 환경 구축에 있어서 여러 가지 테스트를 수행할 예정입니다.
Hello World 이런거 출력하는 테스트는 인터넷 뒤지면 어딜 가나 다 나오기 때문에 그런 글을 쓸 생각은 당연히 없습니다.

통합 개발환경 구축에 있어서 수행할 테스트는 다음과 같습니다.

* 회원가입 및 로그인
* 테이블 값 호출 및 데이터 생성

PostgreSQL하고 연동할 웹 개발환경을 만드는데 위 부분이 없으면 그것은 진정한 테스트라고 볼 수 없겠죠.

위 사항을 기반으로 했을 때 Spring Boot로 구현할 부분과 Vue.js로 구현할 부분은 그럼 어떻게 나누어질까요. 이 부분은 위 테스트 단위를 바탕으로 해서 한 번 보겠습니다.

#### 회원가입 및 로그인

먼저 필요한 구성요소를 보겠습니다.

* 회원가입 페이지
* 로그인 페이지
* 회원가입 처리
* 로그인 처리
* 로그인 되었을 때만 보여주는 페이지 및 권한 설정

아마도 이렇게 5가지로 구성될 것입니다.

Vue.js로 만들어야 할 페이지는 어떤 페이지일까요.

* 회원가입 화면
* 로그인 화면
* 메인 화면(회원권한 설정된 화면)

그렇다면 Spring Boot로 만들어야 할 기능은 무엇일까요.

* 회원가입 
    + Request 수신
    + 비밀번호 암호화
    + 회원가입 유효성 검증
    + 회원가입 여부 처리 결과
* 로그인
    + Request 수신
    + 로그인 유효성 검증
    + 로그인 처리 결과
* 메인 화면
    + Request 수신
    + 현재 로그인되어있는 지 여부 및 권한 확인 후 Response

이 정도가 될 것으로 보입니다.


#### 테이블 값 호출 및 데이터 생성

이 부분에 대한 구성요소도 크게 다르지 않습니다.

* 일반 데이터 호출 페이지
* 특정 값 입력 페이지
* 요청에 따라 특정 값 호출 페이지

페이지를 3개로 구분한 이유는 다음과 같습니다.

먼저 일반 데이터 호출은 GET 방식의 Request에 따른 결과를 나타내는 반면,
특정 값 입력 및 요청에 따른 특정 값 호출은 POST 방식의 Request에 따른 결과를 나타냅니다.
테스트를 하려면 두 가지 방식을 모두 테스트해야 하기 때문에 구성한 것으로 보시면 됩니다.

이 페이지는 Vue.js와 Spring Boot로 어떤 것을 구분해야 할 지가 너무나도 명확하므로 생략하겠습니다.

이 정도에서 전체 소개를 마치고, 각각의 기능을 어떤 방식으로 구현하는지를 이어서 다루겠습니다.
