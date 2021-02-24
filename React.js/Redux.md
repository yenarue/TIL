## `redux` vs `mobx`

![](https://github.com/yenarue/images/blob/master/Frontend/react_vs_redux.png?raw=true)

- 리덕스 : 초보에게 좋음
  - 코드량이 좀 많음 ⇒ 생산성이 약간 떨어질 수 있음
- 몹엑스 : 리액트 생태계, 리액트를 자유자재로 다룰 수 있는 사람들에게 좋음
- next.js 에 redux 붙이기가 좀 까다로움
  - 래퍼가 있다 : https://github.com/kirill-konshin/next-redux-wrapper

## 대략적인 리덕스 흐름도

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/37f02f59-c3e8-4c4d-82bd-a6146d3c7480/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/37f02f59-c3e8-4c4d-82bd-a6146d3c7480/Untitled.png)

- 액션, 리듀서 하나씩 다 만들어줘야함;;; 뷰엑스 쓰다가 이거보니 갑갑하네
- 액션들이 리덕스에 기록되어 쌓임
  - 디버깅 할 때 편함
- 리듀서
  - 불변성 Immutability
- 스토어 (Store) : State + Reducer