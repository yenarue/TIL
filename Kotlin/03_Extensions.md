확장 (Extensions)
====

Kotlin은 프로젝트 내에서 객체의 함수나 프로퍼티를 임의로 확장하여 정의할 수 있는 확장(extension) 기능을 제공한다.

- 함수, 프로퍼티 확장이 가능하다
- 직접적인 객체 확장의 방법을 제공한다 : 유틸리티 클래스를 더 이상 만들 필요가 없다!
- Generic을 통해 객체의 타입 처리가 가능하다 
- Extension이 적용될 범위(Scope) 를 지정할 수 있다 : 기본적으로 Extension을 선언한 곳에서만 사용 가능하다. 물론 import를 하면 다른 곳에서도 사용 가능하다.

## 확장 함수 (Extension Function)

클래스를 확장하는 것. 원래부터 클래스에 정의되어 있던 함수인 것 처럼 사용할 수 있다.

js의 프로토타입이나 swift의 extension과 비슷.

* 리시버 (Receiver) : 확장 대상. 확장될 클래스를 뜻한다.

```kotlin
fun String.lastChar() = this.get(this.length - 1) // this는 생략가능

import com.example.util.*
val c: Char = "abc".lastchar()

// for Android
fun Context.showToast(text: String) {
    // 정의한다~
}
showToast("test!")
```

Extension Function을 구현한 경우, Java에서는 이를 어떻게 호출할까?

```java
char c = StringExtesionKt.lastChar("abc");
```

위와 같이 별도의 보조 클래스(separate auxiliary class)가 생성되고 **static 메소드**로 해석되어 호출하게 된다. static 메소드로 해석되기 때문에 클래스의 멤버에 대한 접근이 제한된다.

기존의 `this` 는 첫번째 argument로 대체된다. 기존의 argument들은 차례로 그 다음 arguemnt로 설정된다.

## Standard Library

No Kotlin SDK!

Kotlin Standard Library = Java Standard Library + Kotlin Extensions

Kotlin에서 기본적으로 추가된 Extension Function들을 소개해보겠다.

- List의 `joinToString` : 요소들을 스트링으로 합쳐줌

- Array, List의 `getOrNull(index)` : 요소가 있으면 리턴, 없으면 Null (OutOfIndex 에러 없음)

- List의 `withIndex()` : 이터레이팅을 할 때 Index도 함께 보내주기

- `infix` : 정의한 함수를 중위 연산자 (+나 -같은)로 만든다

  ```kotlin
  infix fun Int.plus(x: Int): Int {
      return this + x
  }
  
  val plus = 4 plus 2
  print(plus)		// 6
  ```

- `until(to)`  : Int의 infix function. `IntRange`를 반환하여 범위를 나타낸다.

- `to` : infix function 으로서 `Pair` 로 만들어준다.

  ```kotlin
  "zero" to 0 // = Pair("zero", 0)
  ```

- `Char` 클래스의 Extensions : Kotlin은 원시타입(primitive type)과 Wrapper 타입을 구분하지 않고 모두 Wrapper 타입으로 처리한다. Kotlin에서는 char형 변수를 위해 추가된 Extension function들이 몇가지 존재하는데 대표적으로는 `isLetter()`, `isLetterOrDigit()` 이 있다. 

- `String` 클래스의 Extensions

  - `"""` : """로 감싸진 문자열은 indent와 new line, special characters 등 이 모두 그대로 출력된다. 보통 indent는 가독성을 높히기만을 위해 넣곤 하는데 그 경우에는 `trimMargin()` 나 `trimIndent()` 를 사용하여 원치않는 indent를 제하면 된다.

    ```kotlin
    val wow = """wow,
    	|this is a new line..!""".trimMargin()
    val other = """wow2,
    	#this is a new line..!2""".trimMargin('#')
    val indent = """wow3,
    	#this is a new line..!3""".trimIndent()
    ```

  - 정규표현식 : `toRegex()` 를 이용해서 쉽게 만들 수 있음

    ```kotlin
    val regex = "\\d{4}\\.\\d{2}\\.\\d{2}".toRegex()
    regex.match("2019.04.03")	// true
    regex.match("2019.4.3")		// false
    
    // """와 조합하면 매우 편리하다!
    val regexWithTripleQuoted = """\d{4}\.\d{2}\.\d{2}""".toRegex()
    ```

  - 숫자로 변환하기 : `toInt()`, `toDouble()`, `toIntOrNull()`등의 메소드로 쉽게 변환 가능

## Calling Extensions

Kotiln의 확장 함수(Extension Functions)들은 재정의(override) 될 수 없다. 확장 함수들은 Java 로 변환시 static function 으로 변환 되기 때문이다. (정확히는 확장함수들은 Top level에서 정의되기 때문에 static으로 변환되는 것!)

아래의 예제를 참고하여 헷갈리지 않도록 하자:

```kotlin
// Kotlin 에서는 기본적으로 상속이 불가능하도록 정의된다.
// 상속을 해야할 때에는 open 키워드를 사용하여 상속이 가능하게 만들자!
open class Parent
class Child: Parent()

// 확장함수를 정의하자
fun Parent.foo() = "parent"
fun Child.foo() = "child"

fun main() {
    val parent: Parent = Child()
    println(parent.foo())	// parent
}
```

> 사실 Kotlin의 상속 및 재정의(override)의 철학에도 Java와 약간의 차이가 존재한다. `open` 키워드가 그것을 나타내는 키워드인데… 이는 추후에 상속부분에서 살펴보도록하고 일단 지금은 확장 함수에 대해 집중해보도록 하자 😊

### Member vs Extension

멤버함수와 확장함수를 비교해보자!

#### 1. override

아래와 같이 동일한 이름으로 멤버함수와 확장함수를 정의하고 call을 해보면 어떤 함수가 불릴까?

```kotlin
class Something {
    fun foo() = 1
}
fun Something.foo() = 2

Something().foo()
```

정답은 `1` 이다. 

* override : 멤버함수 > 확장함수

이유는 확장함수가 Java로 어떻게 변환되는지를 기억해보면 당연하다는 것을 알 수 있을 것이다.

위와 같이 멤버함수와 동일한 네이밍의 확장함수를 작성하게되면 혼란을 방지하기위해 경고메세지를 띄우게되니 실수하는 일이 없도록 주의하자. (`Warning: Extension is shadowed by a member`)

#### 2. overload

그렇다면 overload의 경우는 어떨까? 이것도 Java로의 변환을 떠올려보면 답을 쉽게 찾을 수 있다.

```kotlin
class Something {
    fun foo() = "member"
}
fun Something.foo(i: Int) = "extension($i)"

Something().foo(2)	// extension(2)
```

overload는 사실상 아예 다른 함수이기 때문에 위 두개는 완전히 다른 함수다. 그렇기 때문에 static 함수라고해도 `foo()` 와 `foo(i)` 는 전혀 다른 함수이기 때문에 `foo(2)` 를 호출하면 `foo(i)` 가 호출되게 된다.

* overload : 멤버함수 < 확장함수

## 확장함수의 중요성 (Importance of extensions)

### Java와의 상호운용성 증대

기존 클래스에 대한 확장함수를 정의할 수 있게 됨으로서, Java 라이브러리의 객체에 대해 추가 기능들을 자유롭게, 본인의 프로젝트에 맞춰, 구현할 수 있다.

### 컴파일 속도 개선

기존 Java 코드를 그대로 사용하며 추가할 부분을 확장함수로 처리하도록 했기 때문에 효율적인 컴파일이 가능해졌다.

### 확장함수를 사용하면 좋은 경우는?

Java 기본 클래스나 외부 라이브러리 등 직접적인 수정이 불가능한 클래스의 추가 동작을 구현해야 할 때 (대부분의 프로젝트에서 추가로 구현해서 사용하던 유틸 클래스의 경우처럼..) 매우 유용하다. 자동완성 기능에 포함되기도하고 다른 개발자가 해당 클래스에 대해 이해하는데에도 도움이 된다.

* 다른 개발자가 해당 클래스에 대해 이해하는데 도움이 된다?

  > 예를 들어 Class Board이라는 약간 광범위하게 해석될 수 있는 객체를 만든다고 했을 때 여기에 대한 함수를을 확장함수로 정의해두므로써 (ex: fun Board.resetGame() ) 위에 이미지처럼 다른 개발자도 해당 객체에 커서 올려놓았을때 확장함수를 보면서 이 Board객체가 게임보드 객체구나.. 식으로 이해를 도울 수 있다는 뜻입니다.
  >
  > 물론 유틸 메소드 형태로 정의해두는 것도 방법이지만 그 경우는 그 개발자가 그 파일을 확인해야만 알 수 있으므로 그에 비해서는 좀 더 실용적인 측면이 있는 것 같습니다.
  >
  > -김병묵(스터디진행자)



## 참고자료

* [Kotlin Extension은 어떻게 동작하는가 Part1](<https://medium.com/til-kotlin-ko/kotlin%EC%9D%98-extension%EC%9D%80-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%8F%99%EC%9E%91%ED%95%98%EB%8A%94%EA%B0%80-part-1-7badafa7524a>)
* [Kotlin Extension은 어떻게 동작하는가 Part2](<https://medium.com/til-kotlin-ko/kotlin%EC%9D%98-extension%EC%9D%80-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%8F%99%EC%9E%91%ED%95%98%EB%8A%94%EA%B0%80-part-2-fb52bb20bc9e>)
* [Kotlin - open 키워드와 상속]()
* [RxJava + Kotlin Extensions](<https://stackoverflow.com/questions/54492039/kotlin-extension-function-on-subsbcribing-to-rxjavas-flowable-data>)
* [Exceptions handling in Kotlin - are you doing it right?](<http://www.douevencode.com/articles/2018-09/kotlin-error-handling/>)

