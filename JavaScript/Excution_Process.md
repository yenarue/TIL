프로그램의 평가와 실행과정
=====

## 실행 문맥 (`ExecutionContext`)
실행 가능한 코드가 실제로 실행되고 관리되는 영역. 실행에 필요한 모든 정보를 컴포넌트 여러개로 나눠 관리함.

```pseudocode
ExecutionContext = {
    LexicalEnvironment: {},
    VariableEnvironment: {},
    ThisBinding: null
}
```
### 렉시컬 환경 타입 컴포넌트

* 렉시컬 환경 (`LexicalEnvironment`) 컴포넌트
* 변수 환경 (`VariableEnvironment`) 컴포넌트

위의 두 컴포넌트는 타입이 같고 내부 값이 같음.

### 디스 바인딩 (`ThisBinding`) 컴포넌트
함수를 호출한 객체의 참조를 저장. 해당 실행 컨텍스트의 `this`.

## 렉시컬 환경 컴포넌트의 구성

자바스크립트 엔진이 자바스크립트 코드를 실행하기 위해 자원을 모아 둠.
함수나 블록의 유효범위 안에 있는 식별자와 값들이 저장됨.

* 환경 레코드 (Environment Record)
  * 유효 범위 안에 포함된 식별자를 기록하고 실행하는 영역
  * 유효 범위 안의 식별자와 값을 `식별자 : 값` 형식으로 바인드하여 환경레코드에 기록

* 외부 렉시컬 환경 참조 (Outer Lexical Environment Reference) 컴포넌트
  * 해당 함수가 속한 곳의 렉시컬 환경 컴포넌트의 참조가 저장됨.
  * 외부의 변수를 참조해야 할때 외부 렉시컬 환경 참조를 따라 검색함.

```pseudocode
LexicalEnvironment: {
    EnvironmentRecord: {},
    OuterLexicalEnvironmentReference: {}
}
```

### 환경 레코드
렉시컬 환경 안의 식별자와 그 식별자가 가리키는 값의 묶음이 저장되는 영역.

특정 함수 안에 선언된 지역 변수와 중첩 함수의 참조는 **그 함수가 속한 환경의 환경레코드의 프로퍼티로 저장**된다.

#### 구성

* 선언적 환경 레코드 (Declarative Environment Record)
  * 실제로 식별자와 결과값이 저장되는 영역 (`키 : 밸류` 형태로 저장)

* 객체 환경 레코드 (Object Environment Record)
  * 실행 컨텍스트 외부에 별도로 저장된 객체의 참조에서부터 데이터를 읽거나 씀.
  * **전역 객체, `with`문의 렉시컬환경 처럼 별도의 객체에 저장된 데이터**는 복사하여 저장하지 않고 **참조 자체를 가져와서** 객체 환경 레코드의 `bindObject` 라는 프로퍼티에 바인드 한다.

```pseudocode
EnvironmentRecord: {
    DeclarativeEnvironmentRecord: {},
    ObjectEnvironmentRecord: {}
}
```

## 전역 환경과 전역 객체
자바스크립트 인터프리터는 시작하자마자 렉시컬 환경 타입의 전역 환경을 생성하고 객체 환경 레코드에 전역 객체의 참조를 대입한다. 

### 웹 브라우저의 자바스크립트 실행 환경
웹 페이지를 읽어들인 후 전역 환경을 생성한다.

```pseudocode
// 전역 환경
GlobalEnvironment = {
    // `Winodw` 객체가 전역 객체이므로 객체환경레코드의 `bindObject`에 할당된다.
    ObjectEnvironmentRecord: {
        bindObject: window
    },
    // 최상단이므로 외부렉시컬환경참조에 null을 넣는다.
    OuterLexicalEnvironmentReference: null
}

// 전역 실행 컨텍스트
ExecutionContext = {
    LexicalEnvironment: GlobalEnvironment,
    // 전역 실행 컨텍스트의 `this`는 `Window`가 된다.
    ThisBinding: window,
}
```

* `Window` 객체 : 일반 전역 객체의 프로퍼티와 클라이언트 측 자바스크립트에서 사용할 수 있는 다양한 프로퍼티가 정의되어 있음.

## 전역변수와 전역함수
최상위 레벨에 작성한 전역변수/전역함수를 전역환경의 객체환경레코드에 프로퍼티로 기록한다.

즉, 전역변수는 **전역 객체의 프로퍼티** 혹은 **전역 객체의 실행문맥에 들어있는 객체 환경 레코드의 프로퍼티** 이다.
