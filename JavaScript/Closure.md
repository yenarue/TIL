클로저 (Closure)
=====
>'열려있는 것을 닫는다'

## 개요
> 자기 자신이 정의된 환경에서 함수 안에 있는 자유 변수의 식별자 결정을 실행한다.

스킴(Scheme) 언어에 영향을 받아 생긴 기능이다.

'열려있는 것을 닫는다'는 의미의 용어로서 열린 함수를 폐쇄함수로 만드는 그 모든 행위를 클로저라고 부른다.

변수를 은닉하여 지속성을 보장하는 등의 기능을 구현할 수 있음.

## 특징
```js
function makeCounter() {
    let count = 0;
    return counting;

    function counting() {
        return count++;
    }
}

const counter = makeCounter();
counter();  // 0
counter();  // 1
```
### 위의 코드를 해설해보면...
`makeCounter()` 는 `counting()`의 참조를 반환한다. `counting()` 함수는 `makeCounter()`의 지역변수인 `count`를 참조하여 종속성을 가지게 된다. 이 형태를 **클로저**라고 한다.

### 캡슐화
이 경우, `count`에 직접적인 접근은 불가능하지만 `counting()`을 참조하고있는 `counter`변수를 통해 간접적으로 참조할 수 있다. 즉, `count` 프로퍼티에 대한 직접 접근에 대해 외부로부터 은폐하는 효과를 낼 수 있다.

클로저를 **캡슐화된 객체**로 볼 수도 있겠다.

### 팩토리
사실 클로저는 추상적인 개념이지만 클로저를 객체로 볼 경우, `makeCounter()`를 클로저를 생성하는 팩토리 함수로 볼 수도 있다.

## 활용
**'데이터와 데이터를 조작하는 함수를 하나로 묶을 수 있다'** 는 점을 이용하여 클로저를 활용해보자.

### 여러개의 내부 상태와 메서드를 가진 클로저

```js
function Person(name, age) {
    let _name = name;
    let _age = age;
    return {
        getName: function() { return _name; },
        getAge: function() { return _age; },
        setAge: function(age) { _age = age; }
    };
}

const person = Person("Tom", 18);
person.getName();   // "Tom"
person.getAge();    // 18
person.setAge(19);
person.getAge();    // 19
```

### 함수 팩토리
다양한 매개변수를 받는 함수를 여러개 생성할 수 있다.

다음은 다양한 Multiplier를 생성하는 로직이다.

```js
function makeMultiplier(x) {
    return function(y) {
        return x * y;
    }
}

const multi2 = makeMultiplier(2);   // 2를 곱하는 멀티플라이어 생성
const multi10 = makeMultiplier(10); // 10을 곱하는 멀티플라이어 생성
console.log(multi2(3));  // 6
console.log(multi10(3)); // 30
```

### 초기화 기능이 추가된 함수 생성하기
전처리, 초기화 등의 로직을 수행하도록 할 수 있다.

아래는 `maxNumber` 까지의 자연수 중에서 홀수만을 골라 `count`개 만큼의 홀수를 반환하는 함수이다. `maxNumber`까지의 자연수 중에서 홀수를 고르는 로직을 먼저 계산해두도록 하고 있다. (테스트성 로직이라 중복제거는 하지 않았음)
```js
function RandomOddUntil(maxNumber) {
    // 미리 홀수만 뽑아두기
    var oddArray = [];
    for (let i = 1; i <= maxNumber; i++) {
        if (i % 2 !== 0) {
           oddArray.push(i); 
        }
    }

    return function(count) {
        var resultArray = [];
        while (count--) {
            resultArray.push(oddArray[Math.floor(Math.random() * (oddArray.length))]);
        }
        return resultArray;
    }
}

const randomOdd = RandomOddUntil(10);
randomOdd(3); // [7, 9, 5]
```

## 잘못된 활용의 예
### 반복문 안에서 클로저 만들기

```js
for(var i = 0; i < 3; i++) {
    btn[i].onclick = function() {
        console.log(i);
    }
}
```

위의 경우, 어떤 버튼을 클릭하든 `3`이 출력된다. 변수 `i`를 참조하는 클로저가 되었기 때문이다. (`console.log(i)`가 실행되는 순간, `i`는 이미 `3`이 되어있는 상태)

#### 해결방안
* 즉시 실행 함수에 카운터 변수 `i` 를 전달한다.
```js
for(var i = 0; i < 3; i++) (functio(_i) {
    btn[_i].onclick = function() {
        console.log(_i);
    }
})(i);
```

* ES6에서 추가된 `let`을 사용한다. `let`은 블록 유효 범위를 가지고 있기 때문에 해당 블록이 반복 될 때 마다 매번 새롭게 선언되어 `i`값이 대입된다.
```js
for(var i = 0; i < 3; i++) {
    let _i = i;
    btn[_i].onclick = function() {
        console.log(_i);
    }
}
```

## 참고자료
* [[YouTube] Understanding Closures (In Under 10 Minutes!)](https://youtu.be/rBBwrBRoOOY)