# Vue.js 정복 캠프 6기

출석번호 : 6976

* [강의 자료](https://joshua1988.github.io/vue-camp/)

* [강의 GitHub Repository](https://github.com/joshua1988/vue-camp)



## 오늘의 내용

* 라우터 코드 스플리팅 & 다이나믹 컴포넌트
* 라우터 네비게이션 가드
* 페이지별 권한 관리
* 테스트 코드
* 수업 이후 학습 로드맵
  * Vue Composition
  * TypeScript
  * Unit Teesting



## 코드 스플리핑

* [코드 스플리핑 React 관련 자료]([https://velog.io/@odini/Code-Splitting%EC%BD%94%EB%93%9C-%EC%8A%A4%ED%94%8C%EB%A6%BF%ED%8C%85](https://velog.io/@odini/Code-Splitting코드-스플릿팅))
* https://joshua1988.github.io/vue-camp/advanced/code-splitting.html

<뷰에서 자주 쓰이는 테크닉>

router에서 아래와 같이 path에 매칭되는 component에 바로 import 해줄 수 있다

```js
import Vue from 'vue';
import VueRouter from 'vue-router';
import LoginView from '../views/LoginView';
import HomeView from '../views/HomeView';
import PostAddView from '../views/PostAddView';

Vue.use(VueRouter);

const router = new VueRouter({
  routes: [
    {
      path: '/',
      redirect: '/login'
    },
    {
      path: '/signup',
      component: () => import('../views/SignupView.vue'),
    },
    {
      path: '/login',
      component: LoginView,
    },
    {
      path: '/home',
      component: HomeView
    },
    {
      path: '/add',
      component: PostAddView
    }
  ]
});

export default router;
```



## 네비게이션 가드

```js
import Vue from 'vue';
import VueRouter from 'vue-router';
import store from '../store';

Vue.use(VueRouter);

const router = new VueRouter({
  routes: [
    {
      path: '/',
      redirect: '/login'
    },
    {
      path: '/signup',
      component: () => import('../views/SignupView.vue'),
    },
    {
      path: '/login',
      component: () => import('../views/LoginView.vue'),
    },
    {
      path: '/home',
      component: () => import('../views/HomeView.vue'),
      beforeEnter(to, from, next) {
        console.log('메인 화면 진입 전');
        if (store.state.token) {
          next();
        } else {
          next('/login');
        }
      }
    },
    {
      path: '/add',
      component: () => import('../views/PostAddView.vue')
    }
  ]
});

// 모든 라우터에 대해서 가드 처리 (공통 로직 등)
router.beforeEach((to, from, next) => {
   next();
});

export default router;
```



## Nuxt

Vue 프레임워크의 프레임워크다!



## Composition

* https://composition-api.vuejs.org/



## TypeScript

https://www.typescriptlang.org/

타입 설정, 신택스 단 체킹 가능! 생산성이 올라감!



자바스크립트로 개발할때는..... 기본적으로 타입이 없다보니 다 Any라 불안...

```js
// javascript
/**
 * @param User user
 */
 function setUser(user) {

 }
```

이런식으로 파라미터를 일일이 지정하고 커멘드를 추가해줘야 외부에서 사용할 때 그나마 개발자의 의도를 파악할 수 있었음... 하지만 여전히 구조적으로 아무거나 넣을 수 있는 상태이다보니 불안...



타입스크립트는 이런 문제점들 해결! => 혼란방지! => 협업시 매우 좋음

```typescript
import axios from 'axios';

interface User {
  username: string;
  password: number;
}

type UserResponse = {
  user: string;
};

function createUser(userData: User): Promise<UserResponse> {
  return axios.post('users/1');
}

createUser({
  username: 'dddd',
  password: 1234,
})
.then(res => res.user)
.catch(err => err);
```

파라미터 타입, 리턴타입 등 설정가능

파라미터 타입과 다른 값을 넣으면 에러뿜음



https://kr.vuejs.org/v2/guide/typescript.html





## PWA

예전보다 훨씬 좋아짐!

앞으로 네이티브 기능들을 훨씬 더 많이 지원할 예정이라 알아보면 매우 좋다!