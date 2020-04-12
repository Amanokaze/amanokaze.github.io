---
layout: post
title:  "Spring Boot 연동을 위한 Vue.js 개발환경 세팅"
date:   2020-04-13
image: 'img023_01.png'
categories: [Integration]
tags: [spring,boot,vue,js,vue.js,vuesax,postgresql,setting,overview,develoment,environment,스프링,부트,뷰,세팅,개발,환경,개발환경]
---


안녕하세요. 거의 한 달만에 새로운 글을 작성하게 되었네요.

지난 글인 Spring Boot + Vue.js + PostgreSQL 개발 환경 세팅에 이어서 이번 글은 Spring Boot 연동을 위한 Vue.js 환경 설정에 대한 글을 써보겠습니다.
지난 글이 전체 요약 글이라면, 이번 글부터는 세부적인 세팅 글이 될 것으로 생각됩니다.

* [Spring Boot + Vue.js + PostgreSQL 개발환경 세팅(Overview)](/blog/SB-VJ-PS-Setting/)

이번 글 작성을 위해서 가장 참고가 많이 된 글은 다음과 같습니다.  다시한번 감사의 말을 전하겠습니다.

* [BezKoder - Vue.js Setting](https://bezkoder.com/jwt-vue-vuex-authentication/)



#### 1. 개요

글 순서에 있어서 여러 의문이 있을 수 있습니다. Overview 다음 왜 Spring Boot가 아니고 Vue.js부터 작성할까.
일반적으로 Back-end를 먼저 구축하고 Front-end를 나중에 구축하지 않나? 

물론 그것도 맞습니다. 하지만 실제 개발에 앞서서 틀을 구축하기 위해서는 Front-end 환경을 먼저 만들어 놓은 다음에 Back-end 환경에서 Front-end 환경을 연동한 후 실행하는 절차로 거칩니다.
실제로 웹 애플리케이션을 가동하는 것도 Front-end가 아니라 Back-end에서 가동하기 때문에 틀을 만들 때에도 Back-end에서 최종적으로 완성된 틀을 갖춰야 합니다.

그렇게 해서 틀을 어느 정도 갖춘다면, 추가 기능 개발은 Back-end에서 어느 정도 만들고 Front-end에서 완성하는 순서로 진행할 수 있겠죠.

앞선 글에서는 Vue.js에서 만들어야 할 페이지를 다음과 같이 나타냈습니다.

* 회원가입 화면
* 로그인 화면
* 메인 화면(회원권한 설정된 화면)

그렇다면 당연히 회원가입 화면, 로그인 화면, 메인 화면을 만들어야 되겠죠?


#### 2. Vue.js 환경 구축

Vue.js는 Front-end 환경이기 때문에, Bootstrap을 지원하는 다양한 Theme가 있습니다. 하지만 이들 테마마다 주어진 양식도 다르고, 원하지 않는 모듈을 포함해서 일일히 빼는 작업도 하는 소요가 만만치 않습니다.
그렇기 때문에 이 글에서는 특정 테마를 사용하지 않고 최소한의 기본 기능만 가지고 Vue.js 환경을 구축하고자 합니다.

먼저 프로젝트를 진행할 디렉토리를 생성한 후, 빈 프로젝트를 생성하겠습니다.
여기에서는 npm과 vue를 사용하는데, 이들 프로그램은 미리 설치가 되어 있어야 합니다.

빈 프로젝트 생성은 일반적으로 다음 명령어를 사용합니다.
{% highlight Shell %}
vue init webpack my-project
{% endhighlight %}

여기서 my-project는 프로젝트명입니다. 다른 이름 입력해도 됩니다.

![Vue Install]({{ '/assets/img/img023_01.png' | prepend: site.baseurl }})

설치가 다 되면 기본 환경이 세팅된 것을 확인할 수 있습니다.

![Vue Dir]({{ '/assets/img/img023_02.png' | prepend: site.baseurl }})

설치가 다 되면 친절하게도 아래와 같이 다음 명령어를 이용해서 실행해 보라고 나옵니다. 실행해 볼게요.

![Vue Comment]({{ '/assets/img/img023_03.png' | prepend: site.baseurl }})

실행하면 localhost:8080으로 접속하라고 나옵니다.

![Vue Execution]({{ '/assets/img/img023_04.png' | prepend: site.baseurl }})

접속하면 잘 나옵니다.

![Vue Result]({{ '/assets/img/img023_05.png' | prepend: site.baseurl }})

여기까지가 사실상 튜토리얼이였고요. 본격적인 것은 이제부터 시작되겠죠?

#### 3. 구현 순서 및 필요 모듈 설치

우리가 구현해야 할 화면은 회원가입, 로그인, 로그인 인증 여부가 포함된 메인화면입니다.
이에 따라 구현되어야 할 순서는 다음과 같습니다.

* 처음에 들어갔을 때 로그인이 되어 있는 지 확인이 필요하다. 안 되어 있다면 로그인 화면이 나오고, 되어 있다면 메인 화면이 첫 화면으로 나온다.
* 로그인 정보를 입력 후 전송했을 때 처리가 필요하다.
* 회원가입 정보를 입력 후 전송했을 때 처리가 필요하다.

여기에서 로그인/회원가입 정보 입력 후 전송하는 부분은 당연히 Front-End에서 처리하지 않고 Back-End에서 처리합니다. 
물론 Firebase나 Auth2를 이용해서 Front-End에서 자체적으로 로그인이나 회원가입을 시도했을 때 원격 회원관리 서버로 값을 주고받는 형태로 로그인을 수행할 수는 있으나, 
여기는 Spring Boot + PostgreSQL 연동을 하기 때문에 회원정보는 DB에서 관리합니다. 그러므로 Back-End로의 전송은 필수가 되겠죠?

그리고 로그인이 정상적으로 되었을 경우, 로그인 상태를 관리하기 위해서는 프로젝트 전반적으로 사용하는 변수 및 함수에 대한 관리도 필요합니다.
이를 store라고 하며, Vuex라는 모듈을 사용하여 관리가 가능합니다.

또한 최근에는 웹페이지를 구축할 때 단순히 기능만 가지고 구축하기 보다는 어떤 일정한 템플릿을 바탕으로 해서 구축합니다.
그래서 Front-end 테마를 사용하는 경우도 많지만, 이 글에서는 특정 테마를 사용하지는 않을 예정이므로 Front-end 디자인을 조금 더 쉽게 개발하기 위해서 Bootstrap을 사용합니다.
그러므로 Bootstrap도 같이 설치를 합니다.

이를 토대로 했을 때 추가로 설치해야 하는 모듈은 다음과 같습니다.

* vuex: Vue.js의 store 관리를 위한 모듈
* axios: HTTP 통신을 위한 모듈
* vee-validate: Form 검증을 위한 모듈
* bootstrap/bootstrap-vue: 웹사이트 제작 프레임워크

위 모듈을 추가 설치할 때 주의점은 다음과 같습니다.
다른 모듈은 최신버전으로 찾아서 설치해도 되지만, vee-validate는 현재 출시된 버전이 3버전이지만 이 글에서는 2버전으로 사용함을 알려드리니 참고바랍니다.

Vue.js 프로젝트에서 설치된 모듈을 나타내는 파일은 package.json입니다. 위에서 설치된 프로젝트에 이들 모듈이 있는지 확인하겠습니다.
확인은 dependencies 부분에서 보시면 됩니다.


{% highlight JSON %}
  "dependencies": {
    "vue": "^2.6.11",
    "vue-router": "^3.1.6"
  },
{% endhighlight %}

없네요. 이제 추가를 하겠습니다.

{% highlight JSON %}
  "dependencies": {
    "vue": "^2.6.11",
    "vue-router": "^3.1.6",
    "vuex": "^3.1.3",
    "axios": "^0.19.2",
    "bootstrap": "^4.4.1",
    "bootstrap-vue": "^2.11.0",
    "vee-validate": "^2.2.15"
  },
{% endhighlight %}

그리고 다시 npm 설치를 진행하겠습니다.
특별한 버전 충돌 등이 없다면 정상적으로 설치가 진행될 것입니다.


#### 4. Vue.js 프로젝트 구조

추가 모듈 구성까지 끝났으면 각 화면을 구현합니다.
하지만 화면 구현에 앞서서 전체 프로젝트의 구조를 먼저 파악해야 합니다. 생각보다 구현해야 할 기능이 많기 때문에 프로젝트 구조를 바탕으로 각각에 기능을 구현해야 하기 때문입니다.

Vue.js 프로젝트는 외부 테마를 사용하거나 위에서처럼 직접 환경을 생성할 경우 프로젝트 별로 구성 환경이 달라질 수는 있지만, 기본적으로 프로젝트 개발에 사용되는 디렉토리는 소스코드를 다루는 
src/ 디렉토리에서 관리합니다.

src/ 디렉토리는 다음과 같은 구조로 구성되어 있습니다.

![Vue src Files]({{ '/assets/img/img023_06.png' | prepend: site.baseurl }})

src/ 디렉토리는 위와 같이 다음 파일 및 디렉토리로 구성되어 있습니다.

* App.vue: 프로젝트의 메인 템플릿 파일, 실제로는 수정할 부분이 거의 없으며 가동 시 실행되는 첫 번째 파일로 볼 수 있음
* amin.js: 이 파일이 프로젝트의 메인 기능을 담당하는 파일로, 애플리케이션 내에서 사용할 모듈 및 컴포넌트를 지정하는 파일입니다.

main.js 파일을 한번 간단히 보면 여러 가지 지정 부분이 있고, 맨 아래에는 Vue를 생성하는 부분이 있습니다.

src/main.js
{% highlight JavaScript %}
import Vue from 'vue'
import App from './App'
import router from './router'

Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})
{% endhighlight %}

App 파일은 App.vue 파일을 불러오는 데 사용되는 용도이며, router를 불러오는 부분이 있습니다.

실제로 프로젝트에서 가장 많이 다루게 되는 부분도 router로, 프로젝트 내 모든 웹페이지에 대한 전체 설정 및 경로를 지정하는 부분으로 보시면 됩니다.
그 외에도 프로젝트에서 사용할 함수 및 변수를 관리하는 store라는 것도 존재하지만, 기본 프로젝트에는 store가 없기 때문에 아래에서 추가하는 순서로 하겠습니다.

* router: 애플리케이션 내 모든 웹페이지 경로 매핑 및 전체 페이지에서 실행할 코드를 나타내는 부분
* store: 애플리케이션 내에서 사용할 함수, 변수 등을 지정하고 관리하는 부분

router하고 store의 개념은 반드시 이해를 해야 웹 애플리케이션을 개발할 수 있으니 참고 바랍니다.


#### 5. Router

router는 src/ 디렉토리에 router.js 파일로 관리하는 경우도 있지만, 위 프로젝트에서는 router/index.js 파일에서 관리하도록 설정되어 있습니다.

src/router/index.js
{% highlight JavaScript %}
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    }
  ]
})
{% endhighlight %}

위와 같이 HelloWorld를 불러오고 루트 경로로 지정하도록 되어 있습니다.
하지만 HelloWorld를 보기 위해서 이 글을 쓰는 것은 아니기 때문에 메인화면, 로그인, 회원가입 페이지를 router에 추가하겠습니다.

그리고 로그인 여부에 따라서 메인화면/로그인화면으로 갈 것인지를 결정하는 것도 router에서 설정합니다.
왜냐하면 router는 모든 웹페이지에 대한 설정을 하는 부분이기 때문에 이 부분도 아래에 결정해줘야 하기 때문입니다.

웹페이지를 열기 전에 수행하는 동작은 beforeEach() 함수를 사용하며, 이 함수를 사용하기 위해서는 router 상의 경로 정보를 하나의 변수로 지정하고, 해당 변수에 대한 동작을 수행해야 합니다.
그러므로 여기에서는 router = new Router() 형태로 선언하고 router.beforeEach() 함수를 사용하도록 하겠습니다.

수정된 소스코드는 아래와 같습니다.

src/router/index.js
{% highlight JavaScript %}
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import Home from '@/components/Home'
import Login from '@/components/Login'
import Register from '@/components/Register'

Vue.use(Router)

const router = new Router({
  mode: 'history',
  base: process.env.BASE_URL,
  routes: [
    {
      path: '/HelloWorld',
      name: 'HelloWorld',
      component: HelloWorld
    },
    {
      path: '/',
      name: 'home',
      component: Home
    },
    {
      path: '/login',
      name: 'Login',
      component: Login,
    },
    {
        path: '/register',
        name: 'Register',
        component: Register,
    }
  ]
})

router.beforeEach((to, from, next) => {
  const publicPages = ['Login','Register'];
  const authRequired = !publicPages.includes(to.name);
  const loggedIn = localStorage.getItem('user');
    
  if (authRequired && !loggedIn) {
    router.push({ name: 'Login', query: { to: to.path }});
  } else {
    next();
  }
});

export default router
{% endhighlight %}

로그인, 회원가입은 로그인 여부를 확인하지 않는 페이지이기 때문에 publicPages 변수의 값으로 추가하고, 인증을 요구하지 않도록(authRequired) 합니다.
그리고 loggedIn 변수는 localStoreage의 'user' 키 값이 있을 경우 해당 키의 value에 넣는 형태인데, localStoreage에 값을 추가하는 부분은 인증서비스 구현 시 추가할 예정입니다.

그렇게 해서 로그인 여부 확인이 필요하고(authRequired), 로그인이 되어 있지 않으면(!loggedIn) 로그인 페이지로 강제 이동하도록 합니다.

router에 대한 부분은 이 정도로 설명하면 될 것 같습니다.


#### 6. Store

다음은 store입니다. store는 위에서 설명했듯이 프로젝트에서 사용할 변수, 함수를 관리하는 모듈로, 프로젝트 생성 시에는 store가 별도로 없습니다.
이에 따라 store를 추가하겠습니다.

![Store Added]({{ '/assets/img/img023_07.png' | prepend: site.baseurl }})

store의 구성요소는 action, getter, mutation, state로 되어 있으며, 필요에 따라서는 getter나 state가 없는 경우도 있습니다.
이 글에서는 getter를 필요로 하지 않으므로 생략하며, 아래와 같이 코드를 추가하겠습니다.

src/store/index.js
{% highlight JavaScript %}
import Vue from 'vue'
import Vuex from 'vuex'
import AuthService from '../services/auth.service';

Vue.use(Vuex)

const user = JSON.parse(localStorage.getItem('user'));
const initialState = user
  ? { status: { loggedIn: true }, user }
  : { status: { loggedIn: false }, user: null };

const state = {
  initialState
}

const mutations = {
  loginSuccess(state, user) {
    state.initialState.status.loggedIn = true;
    state.initialState.user = user;
  },
  loginFailure(state) {
      state.initialState.status.loggedIn = false;
      state.initialState.user = null;
  },
  logout(state) {
      state.initialState.status.loggedIn = false;
      state.initialState.user = null;
  },
  registerSuccess(state) {
      state.initialState.status.loggedIn = false;
  },
  registerFailure(state) {
      state.initialState.status.loggedIn = false;
  }   
}

const actions = {
  login({ commit }, user) {
    return AuthService.login(user).then(
        user => {
        commit('loginSuccess', user);
        return Promise.resolve(user);
        },
        error => {
        commit('loginFailure');
        return Promise.reject(error);
        }
    );
  },
  logout({ commit }) {
      AuthService.logout();
      commit('logout');
  },
  register({ commit }, user) {
    return AuthService.register(user).then(
        response => {
        commit('registerSuccess');
        return Promise.resolve(response.data);
        },
        error => {
        commit('registerFailure');
        return Promise.reject(error);
        }
    );
}}

const store = new Vuex.Store({
  strict: true, // process.env.NODE_ENV !== 'production',
  mutations,
  state,
  actions,
})

export default store
{% endhighlight %}

크게 나누면 다음과 같습니다.

* 로그인/회원가입 처리를 위한 auth.service 모듈 import
* 로그인/회원가입/로그인 여부 확인을 위한 action, mutation, state 설정 

그런데 auth.service 모듈은 사실 Vue.js 설치 시 불러오는 외부 모듈이 아니라 직접 정의해야 하는 모듈입니다.
auth.service에서 Spring Boot와 연동하는 부분을 진행해야 하므로, 아래에서 자세히 다루겠습니다.

store 구성요소에 대한 코드에 대해서 세부적으로 설명하고 싶지만 너무 늘어질 것 같아서 코드만 올리도록 하며, 궁금하신 부분은 질문 주시면 언제든지 상세히 답변 드리겠습니다.
(절대로 설명을 못해서 안쓰는 것이 아닙니다)

위와 같이 store 파일을 작성했으면, 여기서 끝나는 것이 아니라 main.js에서도 store를 추가해야 합니다.
router는 이미 기본으로 추가가 되어있지만, store는 신규로 생성했기 때문입니다.

src/main.js
{% highlight JavaScript %}
import Vue from 'vue'
import App from './App'
import router from './router'
import store from './store'

Vue.config.productionTip = false

new Vue({
  el: '#app',
  router,
  store,
  components: { App },
  template: '<App/>'
})
{% endhighlight %}

#### 7. HTTP 통신

바로 위의 store에서는 로그인/회원가입 처리를 위해서 auth.service 모듈을 불러온다고 하였습니다. 그렇다면 auth.service 모듈에서는 어떤 방식으로 처리할까요?

이 글의 서두에도 언급하였지만, 구축하고자 하는 웹 애플리케이션은 단순히 Vue.js에서 로그인 및 회원가입을 자체적으로 처리하지 않습니다. 
Front-end 환경인 Vue.js에서 로그인/회원가입 정보를 전송하면, Back-end 환경인 Spring Boot에서는 이를 PostgreSQL과 연결하여 정보에 대한 결과를 받아오는 방식으로
각 환경 간에 전송을 수행한다 하였습니다.

그러므로 auth.service 모듈은 실제 로그인이나 화원가입을 직접적으로 처리하는 역할을 하지 않습니다. 
대신 입력 정보를 Back-end인 Spring Boot로 전송하는 역할만 수행합니다.

이러한 정보 전송에는 HTTP 통신을 사용하여 전송되며, Vue.js에서는 axios 모듈을 사용하여 HTTP 전송을 수행합니다.

auth.service는 AuthService라는 클래스(Class)를 생성하여 로그인 및 회원가입 기능을 메소드(method)로 가지고 있습니다. 그래서 각 기능을 수행할 때 store에서는 로그인 및 회원가입을 수행하도록 되어 있고요.

하나의 독립된 서비스로 관리된다는 점에서 services 디렉토리를 생성하고 해당 디렉토리에 auth.service.js를 다음과 같이 생성하겠습니다.

src/services/auth.service.js
{% highlight JavaScript %}
import axios from 'axios';

const API_URL = 'http://localhost:8080/api/auth/';

class AuthService {
  login(user) {
    return axios
      .post(API_URL + 'signin', {
        username: user.username,
        password: user.password
      })
      .then(response => {
        if (response.data.accessToken) {
          localStorage.setItem('user', JSON.stringify(response.data));
        }

        return response.data;
      });
  }

  logout() {
    localStorage.removeItem('user');
  }

  register(user) {
    return axios.post(API_URL + 'signup', {
      username: user.username,
      email: user.email,
      password: user.password
    });
  }
}

export default new AuthService();
{% endhighlight %}

막상 코드 보면 특별한 부분은 없습니다.

API_URL 변수는 전송할 때 사용되는 URL 주소로, 이 주소는 Vue.js가 아닌 Spring Boot에서 받을 주소이므로 이 글에서는 추가적으로 언급 않겠습니다.

또한 axios 모듈을 사용해서 login / register 메소드를 수행하는 것을 알 수 있으며, 그 외에도 로그아웃도 같이 수행한다는 것을 알 수 있습니다.
다만 로그아웃의 경우는 특별한 기능이 아닌 localStorage에 저장된 'user' 키값을 삭제하는 것이 전부이므로 이 글에서는 추가로 다루지 않겠습니다.

#### 8. 사용자 객체(User) 모델 

바로 위에서는 AuthService 클래스를 통해서 사용자 인증 정보를 axios를 사용해서 전송한다는 것을 확인할 수 있었습니다.

그런데 사용자 정보가 한 두가지도 아니고 여러 개가 있는데(사실은 3개입니다만), 위 코드를 자세히 보시면 이러한 정보를 하나의 변수(user)에서 관리한다는 것을 확인할 수 있습니다.
여기에서 사용된 user 변수를 보면 user.username, user.email, user.password라는 세부 속성값을 가지고 있지만, 이러한 속성값을 가지기 위해서는 user라는 객체를 별도로 생성해서 관리해야 합니다.

로그인을 수행할 때 localStorage에 저장되는 user 키값은 JSON 형태로 저장되기 때문에 저장되는 내용에는 문제가 없습니다만, login(user) 및 register(user) 메소드의 파라미터로 들어가는 user는 어떠한 규격을 가져야 하고,
이러한 규격을 정의하기 위한 모델이 별도로 필요합니다.

그래서 아래에 로그인 화면 구현 부분에서도 다룰 것이지만, 로그인 정보 역시 user라는 객체를 선언해서 해당 객체를 store에 전송하는 방식으로 구성되어 있습니다.

user 객체를 정의하기 위해서, 이 글에서는 models라는 디렉토리를 생성한 후, user.js 파일을 생성하여 아래와 같이 간단히 정의해 보겠습니다.

src/models/user/js
{% highlight JavaScript %}
export default class User {
    constructor(username, password, email) {
      this.username = username;
      this.password = password;
      this.email = email;
    }
}
{% endhighlight %}

예상은 했겠지만, 이게 전부입니다. user 객체 선언할 때 constructor() 생성자를 사용하여 각 속성값만 매핑시키면 간단합니다.
이제 이 모델을 사용해서 로그인 및 회원가입 기능을 구현하겠습니다.


#### 9. 로그인 화면 구현

로그인 화면은 테마가 적용되었느냐 여부에 따라 다르지만, 이 글에서는 Bootstrap-vue 모듈을 추가하였기 때문에, Bootstrap이 적용된 로그인 화면을 아래와 같이 간단히 구성하겠습니다.
디자인은 전혀 고려하지 않은 코드이므로 이 점 양해 바라겠습니다.

로그인 화면은 router에서 components/Login으로 설정하였습니다. 확장자가 빠져 있지만, Vue.js에서는 router의 파일 경로를 .vue 확장자를 사용하므로
Login.vue 파일을 만들면 됩니다. 이에 따라 components/Login.vue 파일을 아래와 같이 만들어 보겠습니다.

src/components/Login.vue
{% highlight XML %}
<template>
  <div>
    <b-form @submit="handleLogin">
      <b-row>
        <b-col sm="3"><label for="username">Username:</label></b-col>
        <b-col sm="9">
          <b-form-input
            id="username"
            v-model="user.username"
            type="text"
            required
            placeholder="Username"
          ></b-form-input>
        </b-col>
      </b-row>

      <b-row>
        <b-col sm="3"><label for="password">Password:</label></b-col>
        <b-col sm="9">
          <b-form-input
            id="password"
            v-model="user.password"
            type="password"
            required
            placeholder="Password"
          ></b-form-input>
        </b-col>
      </b-row>
      <b-button type="submit" variant="primary">Submit</b-button>
      <b-button variant="danger" @click="registerUser">Register</b-button>
    </b-form>
    <b-card class="mt-3" header="Form Data Result">
      <pre class="m-0">{{ message }}</pre>
    </b-card>    
  </div>
</template>
{% endhighlight %}

위 태그는 전통적인 HTML 태그는 아닙니다. bootstrap-vue 모듈에서 사용하는 태그로 대체되었으며, Vue.js에서 사용하는 속성값은 정상적으로 작동하니 참고하고 사용하시면 됩니다.

* Form 태그(<b-form>): @submit="handleLogin"
  + 전송 시 handleLogin 함수를 수행합니다.

* Input 태그(<b-form-input>): v-model="user.username" / "user.password"
    + Username, Password 입력에 사용되는 변수로, Vue.js에서는 v-model로 변수 이름을 지정합니다.

* Button 태그(<b-button>): @click="registerUser"
    + 위 코드에 의하면 registerUser 함수를 수행하겠죠?

사실 태그 부분은 테마나 사용 형식에 따라서 다르게 사용해도 상관없습니다. <form> 태그에서 submit할 때 수행할 함수로 handleLogin을 사용해도 되고요.
다만 여기서 중요한 것은 입력 폼에서 값을 전송할 때에는 v-model 속성을 사용해야 한다는 것만 참고하시면 됩니다.

다음은 스크립트 부분입니다. 스크립트는 당연히 태그 아래쪽에 바로 삽입해서 사용합니다.

{% highlight JavaScript %}
<script>
import User from '../models/user';

export default {
    name: 'Login',
    data() {
        return {
            user: new User('', '', ''),
            loading: false,
            message: ''
        }
    },
    computed: {
        loggedIn() {
            return this.$store.state.initialState.status.loggedIn;
        }
    },
    created() {
        if (this.loggedIn) {
            this.$router.push('/');
        }
    },
    methods: {
        handleLogin(evt) {
            evt.preventDefault();
            console.log(this.user.username);
            this.loading = true;
            this.$validator.validateAll().then(isValid => {
                if(!isValid) {
                    this.loading = false;
                    return;
                }

                if (this.user.username && this.user.password) {
                    this.$store.dispatch('login', this.user).then(
                        () => {
                            this.$router.push('/');
                        },
                        error => {
                            this.loading = false;
                            this.message = 
                            (error.response && error.response.data) || error.message || error.toString();
                        }
                    );
                }
            });
        },

        registerUser() {
            this.$router.push({ name: 'Register'});
        }

    }
}
</script>
{% endhighlight %}

코드를 일일히 다 설명하기보다는 특징적인 부분 중심으로 설명하겠습니다.

* new User()로 User에 대한 객체를 선언하는 부분이 있습니다. 바로 위에 언급한 user 객체를 불러와서 선언하는 부분입니다.

* 로그인이 되었는지 여부는 store에서 initialState 변수를 사용해서 확인합니다. 

* $validator를 사용하는 부분이 있는데, 이는 Form이 valid한지를 검증하는 부분입니다. 이 부분의 사용을 위해서 vee-validate 모듈을 사용하는 것입니다.
참고로 vee-validate 모듈 3버전에서는 위 검증 함수가 정상적으로 동작하지 않는 문제가 있으니 참고 바랍니다.

* 로그인 여부 확인을 위해서는 정보 전송이 필요합니다. 이를 위해서 store의 dispatch() 메소드를 사용하여 login을 수행합니다. 이 부분은 AuthService까지 연결되어 Spring Boot로 전송됩니다.

* 로그인 사용자, 비밀번호가 일치하면 메인화면(/)으로 이동하고, 그렇지 않으면 에러메시지를 화면 하단에 출력합니다.

위와 같은 절차면 로그인은 어느 정도 갖춰졌다고 보시면 됩니다. 

실제로 동작 수행은 하지 않을 것입니다. 왜냐하면 Spring Boot 부분을 아직 구현하지 않았기 때문입니다.
하지만 AuthService에서 지정한 API_URL 주소로 제대로 전송되었는 지 정도만 확인하면 간단합니다.

현재까지 구현한 내용을 바탕으로 한 번 정상적으로 동작하는 지 확인해 보겠습니다.

먼저 서버 가동 주소인 http://localhost:8080 으로 들어가겠습니다.

router에서는 메인화면을 Home으로 지정했습니다만, 로그인이 되어 있지 않으면 로그인 화면으로 이동하므로 로그인화면으로 자동 이동됩니다.

![Vue Login]({{ '/assets/img/img023_08.png' | prepend: site.baseurl }})

로그인 정보를 입력 후 전송을 수행합니다. 위 파일에서는 로그인 결과를 아래에 표시하도록 했습니다.

![Vue Login Result]({{ '/assets/img/img023_09.png' | prepend: site.baseurl }})

위와 같이 에러가 나옵니다. 

에러가 나오는 것이 잘못된 것은 아닙니다. 앞서 언급한대로, 로그인 전송 시 대상 페이지는 Spring Boot에서 구현할 예정이고 아직 만들어지지 않았기 때문에 나타난 에러입니다.
실제로 에러 메시지를 보더라도, 'Cannot POST /api/auth/signin'이라고 나타났는데, 실제로 저 페이지는 아직 만들지도 세팅하지도 않았습니다.

하지만 해당 페이지로 전송을 시도한다는 것만 보더라도 AuthService까지 올바르게 전송한다는 것을 확인할 수 있었습니다.


#### 10. 회원가입 화면 구현

회원가입도 로그인 화면과 크게 다르지 않습니다. router에서 Register로 설정해 놓았으므로, components/Register.vue 파일을 만들면 되며, 아래와 같이 만들어 보겠습니다.

src/components/Register.vue
{% highlight XML %}
<template>
  <div>
    <b-form @submit="handleRegister">
      <b-row>
        <b-col sm="3"><label for="username">Username:</label></b-col>
        <b-col sm="9">
          <b-form-input
            id="username"
            v-model="user.username"
            type="text"
            required
            placeholder="Username"
          ></b-form-input>
        </b-col>
      </b-row>
      <b-row>
        <b-col sm="3"><label for="password">Password:</label></b-col>
        <b-col sm="9">
          <b-form-input
            id="password"
            v-model="user.password"
            type="password"
            required
            placeholder="Password"
          ></b-form-input>
        </b-col>
      </b-row>      <b-row>
        <b-col sm="3"><label for="confirm_password">Confirm Password:</label></b-col>
        <b-col sm="9">
          <b-form-input
            id="confirm_password"
            v-model="confirm_password"
            type="password"
            required
            placeholder="Password"
          ></b-form-input>
        </b-col>
      </b-row>
      <b-row>
        <b-col sm="3"><label for="email">E-mail:</label></b-col>
        <b-col sm="9">
          <b-form-input
            id="email"
            v-model="user.email"
            type="email"
            required
            placeholder="E-mail"
          ></b-form-input>
        </b-col>
      </b-row>
      <b-button type="submit" variant="primary">Register</b-button>
    </b-form>
    <b-card class="mt-3" header="Form Data Result">
      <pre class="m-0">{{ message }}</pre>
    </b-card>    
  </div>
</template>
{% endhighlight %}

회원가입 양식도 크게 다르지 않습니다. 로그인과 거의 동일한 양식을 사용하므로 부연설명은 생략합니다.
스크립트 부분으로 넘어가겠습니다.

{% highlight JavaScript %}
<script>
import User from '../models/user';

export default {
  name: 'Register',
  data() {
    return {
      user: new User('', '', ''),
      submitted: false,
      successful: false,
      message: '',
      confirm_password: '',
      isTermsConditionAccepted: true
    };
  },
  computed: {
    loggedIn() {
      return this.$store.state.initialState.status.loggedIn;
    }
  },
  mounted() {
    if (this.loggedIn) {
      this.$router.push('/');
    }
  },
  methods: {
    handleRegister(evt) {
      evt.preventDefault();
      this.message = '';
      this.submitted = true;
      this.$validator.validate().then(isValid => {
        if (isValid) {
          this.$store.dispatch('register', this.user).then(
            data => {
              this.message = data.message;
              this.successful = true;
            },
            error => {
              this.message =
                (error.response && error.response.data) ||
                error.message ||
                error.toString();
              this.successful = false;
            }
          );
        }
      });
    }
  }
};
</script>
{% endhighlight %}

회원가입도 로그인과 마찬가지로 vee-validate를 사용하여 폼을 검증하고, store의 dispatch() 함수를 사용하여 'login' 대신 'register'를 사용하여 전송합니다.
비밀번호 일치여부를 검증하는 부분이 빠져있기는 한데, 이 부분 구현은 어렵지 않으므로 직접 스스로 해보시기 바랍니다.

회원가입 화면도 한번 실행해 보겠습니다.

![Vue Register]({{ '/assets/img/img023_10.png' | prepend: site.baseurl }})

가입정보를 간단히 입력 후 전송해 보겠습니다.

![Vue Register Result]({{ '/assets/img/img023_11.png' | prepend: site.baseurl }})

'Cannot POST /api/auth/signup'이라고 나타납니다.

역시 정상적인 에러입니다.
/api/auth/signup 페이지 역시 Spring Boot에서 만들어질 페이지로, 아직 만들어지지 않았기 때문에 발생한 에러이며, AuthService로 제대로 폼이 전송되었다는 것을 확인할 수 있었습니다.

이것으로 로그인과 회원가입 페이지 구성은 완료되었습니다. Spring Boot에서 연결해서 실행한다면 정상적으로 동작할 것이니 크게 걱정하지는 않으셔도 됩니다.


#### 11. 메인화면 페이지 구성

마지막으로 메인화면입니다만, 크게 구현할 기능은 없습니다.

메인화면에서는 로그인이 되면 이동하는 것으로 설정하였습니다만, 이 부분은 router에서 이미 다 구현하였습니다.
다시 한번 router에서 어떻게 구현하였는지 볼까요?

{% highlight JavaScript %}
router.beforeEach((to, from, next) => {
  const publicPages = ['Login','Register'];
  const authRequired = !publicPages.includes(to.name);
  const loggedIn = localStorage.getItem('user');
    
  if (authRequired && !loggedIn) {
    router.push({ name: 'Login', query: { to: to.path }});
  } else {
    next();
  }
});
{% endhighlight %}

router의 맨 아래에 있었던 코드로, Login, Register가 아닌 다른 페이지의 경우에는 로그인 여부를 체크하고, 되어있지 않으면 Login으로 이동한다고 명시가 이미 되어 있습니다.

router에서는 Home에 대한 설정을 했지만, 이 부분으로 인하여 이미 로그인 여부를 체크하였고요.

그렇다고 해서 Home을 만들지 않으면 에러가 발생하므로, 빈 파일을 다음과 같이 만들겠습니다.

src/components/Home.vue
{% highlight XML %}
<template>
  <div>
  </div>
</template>
{% endhighlight %}

정말 아무 내용도 없죠? 하지만 파일 자체는 존재해야 하므로 만들어야 하니 참고 바랍니다.

별도의 화면 표시는 하지 않겠습니다. 
로그인이 동작하지 않는데, 메인화면이 나타날 리는 없을테니까요.

물론 추후에 작성할 글에 의해서 메인화면에는 DB에 이미 저장된 데이터를 불러오는 기능을 추가할 예정이지만, 이 부분은 Spring Boot로 Vue.js와 연동하는 기능을 넣은 후에 구현할 예정입니다.
그러므로 다음에 이어서 작성할 글이 어떻게 보면 진짜 본론이 될 수 있으니 참고 바라겠습니다.

이상 글 마치겠습니다. 
여러 가지 Vue.js 환경에서 테스트하고 어쩌고 하느라 글 쓰고 검증하는데에만 거의 일주일이 걸렸네요.
하지만 다음에 이어서 쓸 글은 좀 더 정제된 글로 찾아뵙겠습니다.

궁금한 점 있으면 사소한 것도 좋으니 댓글 환영합니다.

