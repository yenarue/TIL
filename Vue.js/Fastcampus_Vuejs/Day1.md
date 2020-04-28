# Vue.js 정복 캠프 6기

출석번호 : 3215

* [강의 자료](https://joshua1988.github.io/vue-camp/)

* [강의 GitHub Repository](https://github.com/joshua1988/vue-camp)



# 본격적인 강의 시작 전, 환경 설정하기

## 개발 환경

- [Chrome](https://www.google.com/intl/ko/chrome/)
- [Git](https://git-scm.com/downloads)
- [Visual Studio Code](https://code.visualstudio.com/)
- [Node.js LTS 버전(v10.x 이상)](https://nodejs.org/ko/)
- [Vue.js Dev Tools](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)
- [Windows] cmder

## VSCode 플러그인 목록

- 색 테마 : [Night Owl](https://marketplace.visualstudio.com/items?itemName=sdras.night-owl)
- 파일 아이콘 테마 : [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme)
- 뷰 확장 플러그인 : [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur)
- 뷰 코드 스니펫 : [Vue VSCode Snippets](https://marketplace.visualstudio.com/items?itemName=sdras.vue-vscode-snippets)
- 문법 검사 : ESLint, [TSLint](https://marketplace.visualstudio.com/items?itemName=eg2.tslint)
- 실습 환경 보조 : [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer)
- 기타
  - [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode), [Project Manager](https://marketplace.visualstudio.com/items?itemName=alefragnani.project-manager), [Auto Close Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-close-tag), [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens), [Atom Keymap](https://marketplace.visualstudio.com/items?itemName=ms-vscode.atom-keybindings), [Jetbrains IDE Keymap](https://marketplace.visualstudio.com/items?itemName=isudox.vscode-jetbrains-keybindings) 등



# 전반적인 프론트엔드 배경지식

이번 수강생분들은 나를 포함하여 웹개발 베이스인 분들이 적은 편이라 한다.

그래서 배경지식을 더 알려주셨음! 넘나 좋은 것!

* [the state of JavaScript 2017](https://2017.stateofjs.com/2017/front-end/results/)
* [the state of JavaScript 2018](https://2018.stateofjs.com/front-end-frameworks/overview/)
* [the state of JavaScript 2019](https://2019.stateofjs.com/front-end-frameworks/)



## Vue.js로 개발된 사이트들

* [삼성생명](https://www.samsunglife.com/main/PDP-MAMAI000000M)
* SSG 모바일 좌측 네비게이션 화면
  * 전체 프로젝트를 변경하지 않아도, 일부 화면만 Vue.js로 부분적으로 적용했음
  * 점진적인 적용이 가능하다!



# Vue.js 간단 페이지 만들기

* DEV용

  * 사람이 읽기 쉽게 되어있다.
  * 용량이 좀 더 크다 => 전반적으로 느리다

  ```html
  <!-- development version, includes helpful console warnings -->
  <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  ```

* PRD용

  * 난독화 + minify가 되어있다
  * 용량이 좀 더 작다 => 전반적으로 빠르다
  * 파일이 로드되는 시간이 빠르면 빠를수록 페이지 렌더링도 빠르게 이루어지니까

  ```html
  <!-- production version, optimized for size and speed -->
  <script src="https://cdn.jsdelivr.net/npm/vue"></script>
  ```



![](https://v1.vuejs.org/images/mvvm.png)