# Vue.js 정복 캠프 6기

출석번호 : 0414

* [강의 자료](https://joshua1988.github.io/vue-camp/)
* [강의 GitHub Repository](





https://caniuse.com/





https://meetup.toast.com/posts/174



## es6

https://joshua1988.github.io/vue-camp/es6/const-let.html

### export

```js
// math.js
export function sum(a,b) {
  return a + b;
}

export let pi = 3.14;

// 방법2
// export const sum = (a, b) => a + b;

// 방법3
// export { sum }

--------------------------

// app.js
import { 가져올 변수, 함수 } from '파일 경로';
import { sum, pi } from './math.js';
```



## webpack

https://youtu.be/WQue1AN93YU

* [웹팩 핸드북](https://joshua1988.github.io/webpack-guide/)

* 웹팩 강의 듣고오기
* 주말 예습 권장
  * (필수) 웹팩 강좌
  * (선택) Vue.js 중급 강좌 (전반 60분 정도 학습... 다 들으시면 굿)
  * (선택) 웹팩 가이드
  * (선택) 지금까지 안내드린 자료들 복습 (자바스크립트 위주)