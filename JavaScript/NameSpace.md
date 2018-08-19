이름 공간 (name space)
=====
변수 이름과 함수 이름을 한 곳에 모아 이름 충돌을 방지한다.

## 개요
다음과 같은 상황에서 변수, 함수의 네이밍이 겹쳐 프로그램이 오작동하는 경우가 생길 수 있다.

* 라이브러리를 사용할 때
* 규모가 큰 프로그램을 만들때
* 여러 사람이 한 프로그램을 개발할 때

변수나 함수의 네이밍이 겹쳐서 선언되더라도 자바스크립트 엔진은 해당 변수나 함수를 끌어올려 단 하나만 생성한다. 그렇기 때문에 딱히 오류표시도 제대로 되지않고 그저 오작동을 하는 경우가 생길 수 있다.

## 이름공간은 어떻게 만들까
해당 변수에 특정 유효범위를 적용하면 위의 문제를 해결할 수 있을 것이다.

### 객체를 이름 공간을 활용하자
Java나 C++의 경우에는 기본적으로 네임스페이스(이름공간) 기능을 제공한다. 자바스크립트는 공식적으로 이런 기능을 제공하지는 않지만 객체를 선언하여 비슷하게 만들 수 있다.

```js
// myApp이라는 이름 공간을 만들어보자.
const myApp = myApp || {};  // myApp이 있으면 기존 인스턴스 사용, 없으면 새로 생성.

myApp.name = "yena";
myApp.showName = function(){ console.log(this.name); };
```

### 함수를 이름공간으로 활용하자
함수 내에서 선언된 변수는 함수 내에서는 유효하다. 이 성질을 이용하여 이름공간처럼 사용할 수 있다. 이 경우엔 즉시 실행 함수를 사용하는 경우가 많다.

```js
const x = 'global x';
(function() {
    const x = 'local x';
    const y = 'local y';
})();
console.log(x); // global x
console.log(y); // ReferenceError: y is not defined
```

## 모듈패턴
즉시 실행 함수를 응용하여 모듈을 정의할 수 있다. 

```js
const Module = Module || {};

(function(_Module) {
    /**** private ****/
    var name = "NoName";

    function getName() {
        return name;
    }
    /*****************/

    /***** public *****/    // 아래의 메소드들로 인해 이 모듈은 클로저가 된다.
    _Module.showName = function() {
        console.log(getName());
    };

    _Module.setName = function(_name) {
        name = _name;
    };
    /*****************/
})(Module);

Module.setName("Yena");
Module.showName();  // Yena
```