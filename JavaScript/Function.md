함수 (Functionm)
====
[실행흐름](./Excution_Process.md)까지 살펴보고나니 자바스크립트는 함수형 프로그래밍이 가능할 수 밖에 없는 구조구나 싶다.

## 정의
```javascript
// 함수 선언문
function square(x) { return x*x; }

// 함수 리터럴
var square = function(x) { return x*x; }

// Function 생성자
var square = new Function("x", "return x*x;")

// 애로우 표현식
var square = x => x*x;
```

## 중첩함수 (Nested Function)
외부 함수의 최상위 레벨에만 중첩함수 작성 가능. 즉, if문, while문 등의 블록 안에서는 작성 불가능.

```js
function aaa(x) {
    bbb();

    function bbb() {
        console.log(x);
    } 
}

aaa(1);
// bbb는 여기서 호출 불가능
```

## 즉시 실행 함수
정의와 동시에 실행시킨다. 전역 유효 범위를 오염시키지 않는(?) 이름 공간 (namespace)를 생성할 때 사용한다.

```js
(function(){})();
(function(){}());

// 이름도 붙일 수 있음
(function name(a) {
    name(); // 함수 내에서만 유효
})('test');
```

## 파라미터
자바스크립트에서는 함수 호출 시 파라미터 생략이 가능하다. (헐)

```js
function sum(a, b) {
    b = b || 0;     // 초기값을 지정해두면 좋다
    return a + b;
}

sum(2);    // 2
```

### 가변 길이 파라미터 리스트 (Arguments)
모든 함수는 `Arguments` 클래스의 인스턴스인 `arguments` 변수를 갖는다. 
`Arguments`는 유사 배열 객체 이다.

```js
function myConcat(separator) {
    let s = ""
    for (let i = 1; i < arguments.length; i++) {
        s += arguments[i];
        if (i < arguments.length - 1) {
            s += separator;
        }
    }
    return s;
}

myConcat("/", "apple", "oragne", "peach");  // apple/orange/peach
```


