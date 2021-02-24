# 고차 컴포넌트 (HOC, High Order Component)

> "고차 컴포넌트 (HOC, higher-order component)는 컴포넌트 로직을 재사용하기 위한 React의 고급 기술입니다. 고차 컴포넌트는 그 자체로는 React API의 일부분이 아닙니다. 고차 컴포넌트는 React의 컴포넌트적 성격에서 나타나는 패턴입니다." - React.js 공식 문서

리액트 컴포넌트를 인자로 받아 새로운 리액트 컴포넌트를 반환하는 **함수**

보통 인자로 받은 컴포넌트의 상태를 관리하는 컴포넌트이거나

다른 기능을 추가하여 컴포넌트로 감싸 반환함

⇒ 기능 재사용에 좋음



## 이점

**기능을 재활용하고 컴포넌트 상태나 생애주기 관리를 추상화할 수 있는 훌륭한 방법.**

고차 컴포넌트를 사용하면 UI만 담당하는 **상태가 없는 함수형 컴포넌트를 더 많이 만들 수 있다.**

⇒ 예나르 뇌피셜) 순수성이 유지되어 상태가 반드시 필요한 곳에만 상태가 존재할 수 있고 역할이 명확히 구분될 수 있기에 독립성도 강화되기 때문이다



### 크로스 커팅 문제 해결

* 이전에는 mixing 사용 : [mixin considered harmful](https://reactjs-kr.firebaseapp.com/blog/2016/07/13/mixins-considered-harmful.html)



## 구현

* 네이밍 규칙 : prefix  `with` 로 시작

```react
const withRequest = (url) => (WrappedComponent) => {
  return class extends Component {
    state = { data: null }
  
  	async initialize() {
      try {
        const response = await axios.get(url);
        this.setState({
          data: response.data
        });
      } catch (e) {
        console.log(e);
      }
    }
  
  	componentDidMount() {
      this.initialize();
    }
  
    render() {
      return (
        <WrappedComponent {...this.props} data={data}/>
      )
    }
  }
}
```

```react
import withRequest from './withRequest';

class Post extends Component {
  render() {
    const {data} = this.props;
    if (!data) return null;
    return (
      <div>
      	{console.log(this.props.data)}
      </div>
    )
  }
}

export default withReqeust('https://backendurl/posts/1')(Post);
```



* 예나르 뇌피셜) Post 클래스를 상태가 없는 함수형 컴포넌트로 바꾸는게 좋을 것 같다. [KISS](https://en.wikipedia.org/wiki/KISS_principle) 기반..

## 참고자료

* [[리액트 공식문서] 고차 컴포넌트](https://reactjs-kr.firebaseapp.com/docs/higher-order-components.html)

* [[컴포넌트에 날개를 달아줘, 리액트 Higher-order Component (HoC)](https://velopert.com/3537)](https://velopert.com/3537)

* [[위키] Cross-Cutting Concerns](https://ko.wikipedia.org/wiki/%ED%9A%A1%EB%8B%A8_%EA%B4%80%EC%8B%AC%EC%82%AC)

* [[위키] 고차함수 (HOF, High Order Function)](https://ko.wikipedia.org/wiki/%EA%B3%A0%EC%B0%A8_%ED%95%A8%EC%88%98)

  * 순수 수학 개념을 다시한번 되돌아보기. 결국 본질적인 개념과 사상이 다 연결되어있으니까!
  * 하나 이상의 함수를 인수로 취함 & 함수를 리턴
  * `map` 은 대표적인 고차함수~_~

  ```javascript
  // hof
  var callFuncTwice = function(func, initialValue) {
    return func(func(initialValue));
  }
  
  var add3 = function(value) {
    return value + 3;
  }
  
  console.log(callFuncTwice(add3, 2)) // 7
  ```

  