ES6 에서 추가된 데이터 타입
=====
Symbol, Template literals

## 심벌 (Symbol)
자기자신을 제외한 그 어떤 값과도 다른 유일무이한 값. ES6에서 추가된 원시 값.

### 기본 사용법
```javascript
var sym1 = Symbol();
var sym2 = Sysmbol();

sym1 == sym2 // false

var HEART = Symbol("하트");
console.log(HEART); // Symbol(하트)
```

### 상태를 나타내는 상수를 대체하자
유일무이하기 때문에 상수로 구분하던 상태값을 대체할 수 있다.

```javascript
var NONE = 0;
var BLACK = -1;
var WHITE = 1;

// 아래와 같이 대체
var NONE = Symbol("none");
var BLACK = Symbol("black");
var WHITE = Symbol("white");
```

### 태깅? 키연결?
심볼에 문자열 키를 추가할 수 있다. 이 경우엔 전역 레지스트리에 저장된다.

```javascript
var sym1 = Symbol.for("club");
var sym2 = Symbol.for("club");
sym1 == sym2 // true

// keyFor 로 지정한 키를 가져올 수 있다.
var sym3 = Symbol.for("club");
var sym4 = Symbol("club");
console.log(Symbol.keyFor(sym3));   // club
console.log(Symbol.keyFor(sym4));   // undefined
```

## 템플릿 리터럴 (Template Literals)

### 보간표현식
템플릿 리터럴 안에 플레이스 홀더를 넣을 수 있다.

```javascript
var a = 2, b = 3;
console.log(`${a} + ${b} = ${a+b}`);
var now = new Date();
console.log(`${now.getMonth()+1} 월`);
```