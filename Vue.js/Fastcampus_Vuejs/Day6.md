# Vue.js 정복 캠프 6기

출석번호 : 1373

* [강의 자료](https://joshua1988.github.io/vue-camp/)

* [강의 GitHub Repository](https://github.com/joshua1988/vue-camp)



## 6회차 수업 자료 안내

- [lodash](https://lodash.com/)
- [Node.js Path API](https://nodejs.org/api/path.html)
- [즉시 실행 함수](https://developer.mozilla.org/en-US/docs/Glossary/IIFE)
- [웹팩 로더 문서](https://webpack.js.org/loaders/)
- [웹팩 플러그인 문서](https://webpack.js.org/plugins/)
- [해커뉴스 사이트](https://news.ycombinator.com/newest)
- [bind() API - MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
- [뷰 라우터 공식 사이트](https://router.vuejs.org/)
- [Vuex 소개](https://joshua1988.github.io/vue-camp/vuex/concept.html)
- [Apache Cordova - 하이브리드앱 개발 프레임워크](https://cordova.apache.org/docs/en/latest/reference/cordova-plugin-file/index.html)
- [Vue Native - 모바일 앱 개발 프레임워크](https://vue-native.io/)
- [웹으로 로컬 파일 시스템 접근할 수 있는 API(현재 구현 중임) 소개 하는 글](https://web.dev/native-file-system/)



## 기타 자료

* 네이티브 로컬 저장소에 접근해야 하는 경우
  * [Vue Native](https://vue-native.io/)
  * Apache Cordova



## Webpack

https://youtu.be/WQue1AN93YU

* [웹팩 핸드북](https://joshua1988.github.io/webpack-guide/)

* 웹팩 강의 듣고오기
* 주말 예습 권장
  * (필수) 웹팩 강좌
  * (선택) Vue.js 중급 강좌 (전반 60분 정도 학습... 다 들으시면 굿)
  * (선택) 웹팩 가이드
  * (선택) 지금까지 안내드린 자료들 복습 (자바스크립트 위주)



```bash
$ npm init -y
```





```js
// webpack.config.js
var path = require('path');

module.exports = {
  // mode: 'production',
  mode: 'none',
  entry: './index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        // 오른쪽 로더부터 먼저 불러짐 => 순서를 바꾸면 오동작
        use: ['style-loader', 'css-loader', 'sass-loader']
      }
    ]
  },
}
```



* [webpack loaders](https://webpack.js.org/loaders/) 에서 원하는 로더 사용법을 확인하고 사용하면 된다

* [webpack plugin](https://webpack.js.org/plugins/) 에서 원하는 플러그인을 확인하고 사용하면 된다

### dev-server

* [webpack dev server]([https://joshua1988.github.io/webpack-guide/tutorials/webpack-dev-server.html#%EC%9B%B9%ED%8C%A9-%EB%8D%B0%EB%B8%8C-%EC%84%9C%EB%B2%84](https://joshua1988.github.io/webpack-guide/tutorials/webpack-dev-server.html#웹팩-데브-서버))

```bash
$ npm init -y
$ npm i webpack webpack-cli webpack-dev-server html-webpack-plugin -D
pa
```

package.json에 serve 추가

```json
{
  // ...
  "scripts":  {
    "serve" : "webpack-dev-server"
  }
}
```

```bash
$ npm run serve
```

localhost:9000 에 접속하면 화면이 뜸

* dev-server는 빌드결과를 인메모리에 올려놓고 접근함

   => 빌드 속도가 훨씬 빠르고 변경된 내용을 바로바로 확인할 수 있음

### 정리

![](https://joshua1988.github.io/webpack-guide/assets/img/diagram.519da03f.png)

1. **Entry** 속성은 웹팩을 실행할 대상 파일. 진입점
2. **Output** 속성은 웹팩의 결과물에 대한 정보를 입력하는 속성. 일반적으로 `filename`과 `path`를 정의
3. **Loader** 속성은 CSS, 이미지와 같은 비 자바스크립트 파일을 웹팩이 인식할 수 있게 추가하는 속성. 로더는 오른쪽에서 왼쪽 순으로 적용
4. **Plugin** 속성은 웹팩으로 변환한 파일에 추가적인 기능을 더하고 싶을 때 사용하는 속성. 웹팩 변환 과정 전반에 대한 제어권을 갖고 있음