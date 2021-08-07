# Redux 둘러보기

* 목표 : 플럭스를 기반으로 애플리케이션 안에서 변경된 데이터가 어떻게 흘러가는지 더 명확히 표현해주도록 하는 것

## Overview

* 액션, 액션생성기, 저장소(스토어), 액션 객체
* 애플리케이션 상태를 불변 객체 하나로 표현 => 플럭스의 개념을 더 단순화
* 리듀서 (reducer) : 현재 상태와 액션에 다라 새로운 상태를 반환하는 함수
  * `(상태, 액션) => 새 상태`

## [세가지 원칙](https://ko.redux.js.org/understanding/thinking-in-redux/three-principles)

1. **진실은 하나의 근원으로부터** - 애플리케이션의 모든 State는 하나의 Store 안에 하나의 객체 트리(State Tree) 구조로 저장된다
2. **상태는 읽기 전용이다** - 상태를 변화시키는 유일한 방법은 Action 객체를 전달하는 방법뿐이다
3. **변화는 순수함수로 작성되어야 한다** - Reducer : Action에 의해 State Tree가 어떻게 변화하는지 지정한다

# Redux 구성요소들

##[상태 (State)](https://ko.redux.js.org/understanding/thinking-in-redux/glossary/#%EC%83%81%ED%83%9C)

> "애플리케이션의 모든 상태(State)는 하나의 저장소(Store)에, 하나의 객체 트리 구조로 저장된다" - [Redux Three Principles](https://ko.redux.js.org/understanding/thinking-in-redux/three-principles)

리액트, 플럭스에서는 가능하면 상태를 더 적은 객체에 담으라고 권장한다. 심지어 리덕스에서는 아예 규칙으로 지정하였다. => "애플리케이션의 상태를 변경 불가능한 한 객체 안에 저장해야 한다"

* 상태를 하나의 객체에 모음으로써 애플리케이션에서 **상태를 보는 관점을 단순하게 유지**할 수 있다.
  * 개별 컴포넌트마다 상태를 가지고 있게 되면 파편화, 과한 분산으로 인하여 추적이 번잡스러워진다. 타고타고 내려가서 봐야하니까.

## [액션 (Action)](https://ko.redux.js.org/understanding/thinking-in-redux/glossary/#%EC%95%A1%EC%85%98)

리덕스에서 상태를 갱신할 수 있는 유일한 방법.

상태를 변경불가능한 객체 안에 저장하므로 상태를 바꿀 때는 객체 전체를 바꿔치기 해야한다. 그 바꿔치기 할 대상과 그 명령을 정의하는 놈이 Action 이다.

* 어떤 부분을 바꿀지 지시
* 변경에 필요한 데이터 제공

* 로그성으로 히스토리가 쌓이기 때문에 변화 이력에 대한 로깅처럼 활용도 가능

* 액션은 동사. 동사위주로 사고하라.
* 액션 유형 (action type) : `type`  필드 필수
  * 어떤 형태의 액션이 행해질지 표시
  * 상수
  * 웬만하면 문자열로 사용하기 (직렬화 가능하므로)

* 페이로드 (payload) : 상태 변화에 필요한 데이터



## [리듀서 (Reducer, 리듀싱 함수)](https://ko.redux.js.org/understanding/thinking-in-redux/glossary/#%EB%A6%AC%EB%93%80%EC%84%9C)

리덕스는 함수, 즉, 리듀서로 모듈화를 한다. 상태 트리는 한 객체로, 모듈화는 리듀서로! 당연 순수함수임 => 핫리로딩, 시간여행(time-travel) 가능

```typescript
//(현재 상태, 액션) => 새로운 상태
type Reducer<S, A> = (state: S, action: A) => S
```

* 상태트리 중 특정 부분을 갱신하기 위해 만든 함수
  * 상태 갱신은 리듀서에 의해 이루어짐!
* 리듀서끼리 합성/조합 가능
  * 물론 겁나 큰 리듀서를 만들어도 되긴하지만 FP나 모듈화 장점이 없어짐 => 순수 단일책임 주의하긔
* 자신의 상태 트리에서 맡은 부분을 갱신할 때 필요한 액션만 처리하도록 설계되어있음

* 항상 값을 반환해야 함 (익셉션시에는 현재상태를 반환하게 하자)
* 상태를 불변객체로서 다루어야 함
  * 통채로 업데이트

* 리듀서는 예상가능해야 한다. 사이드이펙트(부수효과)가 나타나면 안됨.
  * 데이터 만들어내기, Api 호출, 비동기처리 동작 등은 리듀서 밖에서 하기
  * 리듀서안에서는 상태를 변경하거나 부수효과를 발생시키면 안됨

## [스토어 (Store, 저장소)](https://ko.redux.js.org/understanding/thinking-in-redux/glossary/#%EC%A0%80%EC%9E%A5%EC%86%8C)

스토어에서는 애플리케이션의 상태 데이터를 저장하고 모든 상태 갱신을 처리한다. 즉, **애플리케이션의 상태 트리를 가지고 있는 객체**이다. 리듀서로 모듈화를 하고 결합을 하므로 **Redux 앱에는 단 하나의 스토어만 존재**해야 함.

```typescript
type Store = {
  dispatch: Dispatch,
  getState: () => State,
  subscribe: (listener: () => void) => () => void,
  replaceReducer: (reducer: Reducer) => void
}
```

* 리덕스 어플리케이션의 상태 데이터를 저장하고 관리한다.

  * 애플리케이션의 상태를 하나의 객체에 담는다.
  * 상태 변경은 리듀서를 통해 진행한다.

* 현재 상태와 액션을 리듀서에 전달해서 상태 갱신을 처리한다. => 디스패치

* 스토어 생성 : `createStore(reducer, initialState)`

  * 필수 : 리듀서 (reducer)
    * 리듀서를 조합하고 합성해서 스토어가 사용할 단일 리듀서를 만들 수 있다 (`combineReducers`)
  * 옵셔널 : 초기 상태 데이터 (initialState)

* `dispatch(action)` : 상태를 바꾸기! 모든 리듀서에 액션이 전달하고 상태를 갱신할 수 있게 한다

  ```javascript
  store.dispatch({
    type: "ADD_ITEM",
    id: "id1"
    title: "item_title"
  })
  
  console.log(store.getState().items[0].id) // "id1"
  ```

* `subscribe(listener)` : 액션이 디스패치 된 경우 notify 받을 수 있는 listener 를 등록하여 '구독'한다

  ```javascript
  const unsubscriber = store.subscribe(() => {
    console.log('item length: ', store.getState().items.length)
  })
  
  unsubscriber(); // 구독 해제
  ```

  * 내 생각) 근데 이게 디스패치 될 때마다 계속 불리니까 좀 신기함. 액션으로 분기 태우긴하겟찌만.. 이벤트 별로 리스너를 달았던 것에 넘 익숙해져버린 내자신ㅠㅠ 파편화에 익숙해졌어. 뭐 필요하면 모듈화시키면 되긴하니까. 리덕스 사상은 최대한 구조를 심플하게 가져가는 거니까

  * 상태 변경을 리슨하고 그 변경을 `redux-store` 키 아래의 `localStorage` 에 저장함

    * 브라우저에서 영속적인 상태 정보를 사용하려면 아래와 같이 구현

      ```javascript
      const store = createStore(
        combineReducers({ reducer, reducer2 }),
        // 로컬스토리지의 redux-store 값을 초기 상태 값으로 셋팅
        (localStorage['redux-store']) ? JSON.parse(localStorage['redux-store'] : {}
      )
      
      // 액션이 디스패치 될 때마다 스토어 상태를 로컬스토리지에 저장하기
      const unsubscriber = store.subscribe(() => {
        localStorage['redux-store'] = JSON.stringify(store.getStore())
      })
      ```

## [액션 생성자 (Action Creator)](https://ko.redux.js.org/understanding/thinking-in-redux/glossary/#%EC%95%A1%EC%85%98-%EC%83%9D%EC%82%B0%EC%9E%90)

말 그대로 액션을 만들어내는 함수다. 하나하나 손으로 입력하긴 좀 그러니깐!

```typescript
type ActionCreator<A, P extends any[] = any[]> = (...args: P) => Action | AsyncAction
```

백문이불여일견..... 예제... 예제를 보자 :

```javascript
const addItem = title => ({
  type: Constants.ADD_ITEM,
  id: uuid.v4(),
  title,
  addedTimestamp: new Date().toString()
})

const updateItemScore = (id, score) => ({
  type: Constants.UPDATE_ITEM_SCORE,
  id, 
  score
})

store.dispatch(addItem("title1"));
store.dispatch(updateItemScore("id1", 100))
```

* 액션 생성과 관련한 세부 구현 사항을 감추고 생성 작업을 추상화함
  * 호출부에서 굳이 알 필요 없는 불필요한 정보는 감추고 필요한 정보만 명시하도록 하여 가독성을 향상시키고 개발자가 논리에만 집중할 수 있게 도와준다.
  * => 액션 생성 로직 캡슐화
  * => 액션 생성과 디스패치 과정을 추상화~ 베리굿
* 내 생각) 걍... 일반 생성 함순데 리덕스에서 걍 친절하게 이름까지 붙여주고 컴포넌트처럼 개념을 만들어줬네.. 진ㅉㅏ 친절해..
* API 통신 등의 비동기 로직을 액션 생성자에다가 집어넣기도 함. (그렇겠지.. 진짜 친절하네)
* 



## [미들웨어 (Middleware)](https://ko.redux.js.org/understanding/thinking-in-redux/glossary/#%EB%AF%B8%EB%93%A4%EC%9B%A8%EC%96%B4)

서로 다른 두 계층이나 두 소프트웨어를 붙여주는 풀 같은 역할

```typescript
type MiddlewareAPI = { dispatch: Dispatch, getState: () => State }
type Middleware = (api: MiddlewareAPI) => (next: Dispatch) => Dispatch
```

디스패치 함수를 결합하여 새로운 디스패치 함수를 반환하는 고차함수.

* 스토어의 디스패치 파이프라인에 작용함
  * 액션을 디스패치하는 과정에서 연쇄 호출되는 일련의 함수들
* 비동기 액션을 동기 액션으로 전환해줌
* [`applyMiddleware(...middleware)`](https://ko.redux.js.org/api/applymiddleware/)

```javascript
import { createStore, combineReducers, applyMiddleware } from 'redux'
import { colors, sotr } from './reducers'
import stateData from 'initialState'

// '함수를 반환하는 함수'를 반환하는 함수
const loggerMiddleware = store => next => action => {
  let result
  console.groupCollapsed("디스패칭", action.type)
  console.log('이전 상태', store.getState())
  console.log('액션', action)
  result = next(action) // action이 dispatch 될 때마다 실행되쥬 => next middleware를 거쳐 reducer까지 action이 전달됨
  console.log('다음 상태', store.getState())	// action이 다 전달되어 갱신된 state가 출력됨
  console.groupEnd()
}

const saverMiddleware = store => next => action => {
  let result = next(action)
  // 업데이트 된 state를 localStorage에 저장
  localStorage['redux-stroe'] = JSON.stringify(store.getState())
  return result
}

const stroeFactory = (initialState=stateData) => 
	applyMiddleware(logger, saver)(createStore)(
  	combineReducers({colors, sort}),
    (localStorage['redux-store']) ?
    	JSON.parse(localStorage['redux-store']) : initialState
  )

// 스토어를 직접 export 하지 않고 팩토리를 export 하기
export default storeFactory
```

```javascript
import storeFactory from "./store"

const store = storeFactory()

store.dispatch(addItem("아이템이름1"))
store.dispatch(addItem("아이템이름2"))
store.dispatch(addItem("아이템이름2"))
```


# 리액트 리덕스 (`react-redux`)

`react-redux` 를 사용하기 전에 스토어를 직접 사용하는 것을 먼저 접해보즈아

## 1. 명시적으로 스토어 전달해보기

스토어를 컴포넌트 트리의 프로퍼티로서 명시적으로 UI에 전달해보자

* 단순하고 컴포넌트의 개수가 적은 앱에서 잘 동작할 것임

```react
// import 구문 생략
const store = storeFactory()
const render = () =>
	ReactDOM.render(
    <App store={store}/>, // App을 렌더링할 때 스토어를 프로퍼티로 전달
    document.getElementById('react-container')
  )
store.subscribe(render) // 스토어가 변경될 때마다 렌더링
render()
```

store를 App에 넘겼다. App의 자식 컴포넌트가 store를 사용해야 하면 자식 컴포넌트에게도 store를 프로퍼티로 전달해줘야한다:

```react
// import 구문 생략

// 프로퍼티에서 가져온 스토어를 모든 자식 컴포넌트에게 명시적으로 전달
const App = ({ store }) =>
	<div className="app">
  	<SortMenu store={store} />
  	<AddColorForm store={store} />
    <ColorList store={store} />
  </div>
export default App
```

각 자식 컴포넌트들은 필요한 액션을 import 하고 이 스토어를 통해 dispatch 하며 활용할 것임 : 

```react
// [AddColorForm 컴포넌트]
// import 구문 생략

const AddColorForm = ({ store }) => {
  let _title, _color
  const submit = e => {
    e.preventDefault()
    store.dispatch(addColor(_title.value, _color.value))
    _title.value = ''
    _color.value = '#000000'
    _title.focus()
  }
  
  // Form이 submit되면 ADD_COLOR 액션을 dispatch
  return (
  	<form className="add-color" onSubmit={submit}>
    	<input ref={input => _title = input}
        type="text" placeholder="색 이름..." required/>
      <input ref={input => _color = input}
        type="color" required/>
      <button>추가</button>
    </form>
  )
}

AddColorForm.propTypes = {
  store: PropTypes.object
}

export default AddColorForm
```

```react
// [ColorList 컴포넌트]
// import 구문 생략

const ColorList = ({store}) => {
  const { colors, sort } = store.getState() // 스토어에서 색 목록 GET
  const sortedColors = [...colors].sort(sortFunction(sort)) // 색 복사본 생성
  return (
    <div clasName="color-list">
    	{
        (colors.length === 0) ? 
	      	<p>텅 비었어요! 색상을 추가하세요!</p> : 
  	    	sortedColors.map(color => 
             <Color key={color.id}
               {...color}
               onRate={(rating) => 
                             store.dispatch(
                             	rateColor(color.id, rating)
                             )
                      }
               onRemove={() => 
                             store.dispatch(
                             	removeColor(color.id)
                             )
                      } />
          )
      }
    </div>
  )
}

ColorList.propTypes = {
  store: PropTypes.object
}

export default ColorList
```

* 컴포넌트 트리가 작은 경우에는 무리 없이 잘 동작함
* 스토어를 자식 컴포넌트에 명시적으로 전달하다보니 생기는 단점들이 있음 : 
  * 코드가 더 길어진다
  * 이해하기 어려워진다 (계속 스토어를 전달해야 하니 기억해둬야함)
  * 해당 스토어에 디펜던시가생겨 자식 컴포넌트들을 다른 앱에 재활용하기 어려워진다
* => 결국 어느 컴포넌트에만 상태를 전달하고 싶은 경우에도 App부터 그 컴포넌트까지 도달하기 위해 연결된 모든 컴포넌트에 스토어를 다 전달해줘야 함 => 전달..전달..전달.. 전달의 향연...

## 2. [컨텍스트](https://ko.reactjs.org/docs/context.html)를 통해 묵시적으로 스토어 전달해보기

React의 컨텍스트를 이용하면 스토어를 '묵시적으로' 전달할 수 있다. 즉, 중간에 연결된 컴포넌트에 일일이 전달해 줄 필요 없이 스토어가 필요한 컴포넌트에 바로 전달할 수 있음 (사실은 그렇게 보여지는 것이겠지만)



우선 App 컴포넌트를 리팩토링해야한다 : 

* 스토어를 구독하고 상태가 바뀔 때 마다 UI를 갱신하도록 한다 => Componet를 상속하여 mount 콜백, render 함수를 구현하자

```react
class App extends Component {
  // 마운트시 스토어 구독
  componentWillMount() {
    this.unsubscribe = store.subscribe(
    	() => this.forceUpdate()	// 상태 업뎃시마다 컴포넌트 트리 업뎃
    )
  }

  // 언마운트시 구독해제
  componentWillUnMount() {
    this.unsubscribe()
  }
  
  render() {
    const { colors, sort } = store.getState()
    const sortedColors = [...colors].sort(sortFunction(sort))
    return (
      	<div className="app">
  				<SortMenu />
			  	<AddColorForm />
			    <ColorList colors={sortedColors} />
			  </div>
    )
  }
  
  // Context 를 정의하는 객체를 반환
  getChildContext() {
    return {
      store: this.props.store	 // 프로퍼티에 정의된 스토어를 컨텍스트에 추가
    }
  }
}

App.propTypes = {
  store: PropTypes.object.isRequired
}

// 컨텍스트 객체 정의
App.childContextTypes = {
  store: PropTypes.object.isRequired
}

export default App
```

* 자식 컴포넌트들(SortMenu, AddColorForm, ColorList)은 모두 컨텍스트를 통해서 스토어에 접근할 수 있다 => `store.getState()` 와 같이 직접접근 가능
* App 컴포넌트에서 스토어를 구독하고 있으므로 `index.js` 안에 있던 스토어 구독구문 삭제 가능

```react
// [AddColorForm 컴포넌트]
// import 구문 생략
														// 컨텍스트는 두번째 인자로 전달됨
const AddColorForm = (props, { store }) => {
  let _title, _color
  const submit = e => {
    e.preventDefault()
    store.dispatch(addColor(_title.value, _color.value))
    _title.value = ''
    _color.value = '#000000'
    _title.focus()
  }
  
  // Form이 submit되면 ADD_COLOR 액션을 dispatch
  return (
  	<form className="add-color" onSubmit={submit}>
    	<input ref={input => _title = input}
        type="text" placeholder="색 이름..." required/>
      <input ref={input => _color = input}
        type="color" required/>
      <button>추가</button>
    </form>
  )
}

// 기존 : propTypes
// contextTypes은 필수로 정의되어야 함
AddColorForm.contextTypes = {
  store: PropTypes.object
}

export default AddColorForm
```

```react
// [Color 컴포넌트]
// import 구문 생략

class Color extends Component {
  render() {
    const { id, title, color, rating, timestamp } = this.props
    const { store } = this.context // 컴포넌트 클래스에서는 이렇게 접근 가능
    render(
      <div>
        <!-- 중략... -->
        <button onClick={() => store.dispatch(removeColor(id))}></button>
      </div>
    )
  }
}

// 필수
Color.contextTypes = {
  store: PropTypes.object
}

Color.propTypes = {
  id: PropTypes.string.isRequired,
  title: PropTypes.string.isRequired,
  color: PropTypes.string.isRequired,
  rating: PropTypes.number
}

Color.defaultProps = {
  rating: 0
}

export default Color
```

* 확실히 이전보다 훨씬 나아지긴 했다!
* 하지만 각 컴포넌트들이 리덕스 스토어와 직접 상호작용하면서 UI를 렌더링 하고 있다
  * => **스토어를 UI 렌더링 컴포넌트와 분리**하여 아키텍처 개선 가능

## 표현 컴포넌트 vs 컨테이너 컴포넌트 (Presentational Component vs Container Component)

* 관련문서 : [Presentational and Container Component](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
  * 한글 번역본 : https://medium.com/@seungha_kim_IT/presentational-and-container-components-%EB%B2%88%EC%97%AD-1b1fb2e36afb

### 표현 컴포넌트 (Presentational Component)

UI를 렌더링하는 컴포넌트. 정말 딱 렌더링만 한다. View에 해당한다.

* 데이터와 분리되어있어 다른 데이터를 활용하는 경우에도 재활용 가능
* 쉽게 교체 가능
* 쉽게 테스트 가능
* 표현 컴포넌트들 합성해서 새로운 UI 도 만듬

### 컨테이너 컴포넌트 (Container Component)

표현 컴포넌트와 데이터를 연결해주는 컴포넌트.

컨텍스트에서 스토어를 꺼내 스토어와의 모든 상호작용을 처리.

표현 컴포넌트의 프로퍼티를 State와 연결 + 콜백 함수 프로퍼티를 Store dispatch에 연결 => 표현 컴포넌트 렌더링

UI 엘리먼트에는 관여하지 않음. 표현 컴포넌트랑 데이터만 연결연결!

### 아까 그 코드들 리팩토링 해보자

* 표현 컴포넌트
  *  `AddColorForm`, `ColorList`, `SortMenu`, `Color`, `StarRating`, `Star` 컴포넌트
* 컨테이너 컴포넌트
  * `SortMenu` 와 데이터를 연결하는 `Menu` 컴포넌트
  * `AddColorForm` 와 데이터를 연결하는 `NewColor`  컴포넌트
  * `ColorList ` 와 데이터를 연결하는 `Colors` 컴포넌트

```react
render() {
  return (
    <div className="app">
      <Menu />
      <NewColor />
      <Colors />
    </div>
  )
}
```

```react
import AddColorForm from './ui/AddColorForm'
import SortMenu from './ui/SortMenu'
import ColorList from './ui/ColorList'
// 모든 액션 생성기를 임포트해서 사용한다 => 모든 리덕스의 기능들이 이 파일안엣 연결되는 중!
import { addColor, sortColors, rateColor, removeColor } from '../actions'
// import 구문 중략

/* Wrapper Components */

// AddColorForm 컴포넌트를 포함하고 그것을 리덕스 스토어와 연결해줌
export const NewColor = (props, { store }) =>
	<AddColorForm onNewColor={(title, color) => store.dispatch(addColor(title, color))} />
NewColor.contextTypes = {
  store: PropTypes.object
}

export const Menu = (props, { store }) =>
	<SortMenu sort={store.getState().sort}
    				onSelect={sortBy => store.dispatch(sortColors(sortBy))} />
Menu.contextTypes = {
  store: PropTypes.object
}

export const Colors = (porps, { store }) => {
  const { colors, sort } = store.getState()
  const sortedColors = [...colors].sort(sortFunction(sort))
  return (
  	<ColorList colors={sortedColors}
      				 onRemove={id => sotre.dispatch(removeColor(id))}
      				 onRate={(id, rating) => store.dispatch(rateColor(id, rating))}
      />
  )
}
Colors.contextTypes = {
  store: PropTypes.object
}
```

* UI와 데이터를 분리하여 그 사이에 둘을 연결해주는 컨테이너를 두는 아키텍처!
  * 굳이 이럴 필요 없는 경우 : 프로젝트 규모가 작은 경우, 프로토타이핑인 경우, 개념 증명하는 용도로 쓰는 경우... 넘 당연한디..?
  * "컨테이너와 표현 컴포넌트를 분리해서 코드 복잡도를 개선할 수 있는 경우에만 그렇게 하라" - 댄 아브라모프 (리덕스 제작자)

## 3. 리액트 리덕스 활용하기

[`react-redux`](https://www.npmjs.com/package/react-redux) 를 활용하면 컨텍스트를 통해 스토어를 전달하는 과정의 복잡함을 덜 수 있다.

* 코드의 복잡도가 낮아짐
* 개발 속도가 빨라짐

### React Redux Provider

### React Redux Connect


# 그 외 기타 리덕스 관련 정보들

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

