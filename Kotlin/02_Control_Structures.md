Control Structures
====

Kotlin의 제어문에 대해서 알아보자!

## Conditionals: `if`, `when`

Kotlin에서 `if`, `when` 는 구문(statement)이 아니라 **표현식(expression)** 이다. 즉, 변수에 바로 값을 할당할 수 있다. 그렇다보니 Kotlin에는 삼항연산자(ternary operator)가 존재하지 않는다.

```kotlin
val max = if (a > b) a else b
```

### `when`

`when` 은 Java의 `switch` 구문을 대체할 수 있다. Java의 `case` 구문은 Kotlin에서는 브랜치(branch)라고 불리게 된다. 그리고 Kotlin에서는 Java의 `break` 구문은 사용하지 않아도 된다.

```kotlin
enum class Color {
    BLUE, ORANGE, RED
}

fun getDescription(color: Color): String = when(color) {
    BLUE -> "cold"
    ORANGE -> "mild"
    RED -> "hot"
    else -> "nothing"
}
```

하나의 브랜치(Java에서는 `case` 구문)에서 여러가지 값을 체크하고 싶을 때는 간단하게 쉼표(`,`)로 처리하면 된다.

```kotlin
fun respondToInput(input: String) = when(input) {
    "y", "yes" -> "I'm glad you agree"
    "n", "no" -> "Sorry to hear that"
    else -> "I don't understand you"
}
```

각각의 브랜치에도 임의의 표현식을 사용할 수 있다.

```kotlin
fun mix(c1: Color, c2: Color) =
	when(setOf(c1, c2)) {
   		setOf(RED, YELLOW) -> ORANGE
	    setOf(YELLOW, BLUE) -> GREEN
	    setOf(BLUE, VIOLET) -> INDIGO
	    else -> throw Exception("Dirty color")
	}
```

#### 타입 체크하기

Kotlin의 `is` 는 Java의 `instanceof` 에 대응된다 :-D

`is` 로 체크하고나면 스마트 캐스팅(smart cast)이 되어 체크된 클래스로 자동 캐스팅된다.

```kotlin
// Cat extends Pet
// Dog extends Pet
when(pet) {
    is Cat -> pet.meow()	// smart casts
    is Dog -> pet.woof()
}
```

#### Capturing `when` subject in a variable

`when` 의 argument에 변수를 할당해서 사용할 수 있다.

```kotlin
when(val pet = getMyFavoritePet()) {
    is Cat -> pet.meow()
    is Dog -> pet.woof()
}
```

#### `when` without argument

`when` 에 argument를 주지 않고 브랜치에 그냥 조건문을 넣어도 된다.

```kotlin
fun updateWeather(degrees: Int) {
    val (description, color) = when {
        degrees < 5 -> "cold" to BLUE
        degrees < 23 -> "mild" to ORANGE
        else -> "hot" to RED
    }
}
```



## Loops

Kotlin의 반복문에서는 list, map, String 을 iterate 할 수 있다.

* 반복문의 종류 :  `while`, `do-while`, `for`

### 종류

- List 이터레이팅

  ```kotlin
  val list = listOf("a", "b", "c")
  
  // for (s: String in list) 의 형태로도 사용가능
  for(s in list) {
      print(s)
  }
  
  // Iterating with index
  for((index, element) in list.withIndex()) {
      println("$index: $element")
  }
  ```

- map도 `in` 연산자로 이터레이팅을 할 수 있다.

  ```kotlin
  val map = mapOf (1 to "one", 2 to "two", 3 to "three")
  for((key, value) in map) {
      println("$key = $value")
  }
  ```

- Iterating over range (`..`, `until`)

  ```kotlin
  // 1이상 9이하의 범위 형성
  for (i in 1..9) {
      print(i) // 123456789
  }
  
  // 1이상 9미만의 범위 형성
  for (i in 1 until 9) {
      print(i) // 12345678
  }
  ```

- Iterating with a step : 이터레이팅의 단위를 설정

  ```kotlin
  // 9부터 1까지 2개씩 뛰어넘으면서
  for (i in 9 downTo 1 step 2) {
      print(i) // 97531
  }
  ```

- Iterating over `String` : 스트링도 char 형의 반복문을 만들어 낼 수 있다.

  ```kotlin
  for (ch in "abc") {
      print(ch + 1) // bcd
  }
  ```



## `in`: checks and ranges

Kotlin의 키워드 `in` 은 아래의 두가지의 경우에 사용될 수 있다.

### 1. 특정 범위를 순환할 때 (iteration)

 a~z까지 하나씩 꺼내서 i에 넣고 본다 ( `for (i in 'a'..'z') {…}`)

### 2. 해당 범위내에 특정 항목이 포함되어 있는지 체크할 때 (check for belonging) 

변수 c가 a~z까지의 문자에 속하는지 (`c in 'a'..'z'`)

* 속하지 않는지 보고 싶으면 : `!in`

* `when` 의 브랜치에서 사용될 때 :

  ```kotlin
  fun recognize(c: Char) = when(c) {
      in '0'..'9' -> "It's a digit!"
      in 'a'..'z', in 'A'..'Z' -> "It's a letter!"
      else -> "I don't know..."
  }
  ```

두가지 경우에서 모두 '범위' 라는 개념이 나왔다. '범위'란 무엇인지 알아보자.

### 범위 (Range)

Kotlin 의 독특한 자료구조로서 '1부터 9까지'와 같은 연속적인 값들의 집합이다.

```kotlin
1..9				// IntRagne
1 until 10			// IntRange
'a'..'z'			// CharRange
"ab".."az"			// ClosedRange<String>
startDate..endDate	// ClosedRange<MyDate>
```

#### 문자열

문자열의 경우에는 사전적인 순서로 비교 (compared lexicographically)가 이루어진다.

```kotlin
// 아래 3줄의 코드는 모두 동일함
"ball" in "a".."k"
"a" <= "ball" && "ball" <= "k"
"a".compareTo("ball") <= 0 && "ball".compareTo("k") <= 0

// 사전적 순서로 비교가 이루어져 포함여부가 결정된다.
"ball" in "a".."k"	// true (a < b < k)
"zoo" in "a".."k"	// false (a < k < z)
```

####커스텀 타입

그렇다면 커스텀 타입의 경우에는 어떻게 범위가 만들어지는 걸까? 문자열의 경우에서 알 수 있었듯이, 범위 비교는 `compareTo()` 메소드에 종속적이다. 즉, `compareTo`를 오버라이드하여 원하는 비교 방법을 작성하면 그에 맞게 범위가 형성된다.

```kotlin
class MyDate: Comparable<MyDate>

// 아래 3줄의 코드는 모두 동일함
if (myDate.compareTo(startDate) >= 0 && myDate.compareTo(endDate) <= 0) {...}
if (myDate >= startDate && myDate <= endDate) {...}
if (myDate in startDate..endDate) {...}
```

#### 컬렉션

컬렉션의 범위는 단순하다. 컬렉션 순서대로 범위가 결정난다.

```kotlin
// 아래 2줄의 코드는 모두 동일함
if (element in list) {...}
if (list.contains(element)) {...}
```



## Exceptions

> No difference between checked and unchecked exceptions

Kotlin에서는 `throw` 도 표현식(expression)이다. (헐?)

```kotlin
val percentage = if (number in 0..100) number
	else throw IllegalArgumentException("A percentage value must be between 0 and 100: $number")
```

`try` 도 표현식이다.

```kotlin
val number = try {
    Integer.parseInt(string)
} catch(e: NumberFormatException) {
    null
}
```

### @Throws

명시적으로 해당 함수가 특정 예외를 던질 것이라고 명시하는 어노테이션. 즉, **Checked Exception**을 위한 어노테이션이다.

```kotlin
@Throws(IOException::class)	// 위로 던져주겠다고 명시함
fun foo() {
    throw IOException()	// 여기서 발생해도
}

//in Java
try {
    Demo.foo();
} catch (IOException e) {
    // ...
}
```

그런데 사실 Kotlin에서는 **기본적으로 Kotlin에서는 Checked Exception를 따로 검사하지 않는다.** 그래서 따로 `try-catch` 구문을 작성해주지 않아도 정상적으로 컴파일된다.

```kotlin
@Throws(IOException::class) // 위로 던져주겠다고(Checked Exception) 선언했지만
fun foo() {
    throw IOException()
}

// 걍 컴파일 됨
foo();
```

Checked Exception을 강제했던 Java와는 사뭇 다른 결과이다. Kotlin이 이러한 철학을 가져가게 된 데에는 사실 오래된 이유가 있다. (관련 링크 : [Java에서 Checked Exception은 언제 써야 하는가?](http://egloos.zum.com/benelog/v/1901121))

간단하게 설명하자면, Java에서 초반에 강점으로 내세웠던 Checked Exception 개념이 코드 품질 하락 등의 이유로 '실패한 개념'인 것으로 결론나면서 Kotlin에서는 이를 지양하는 철학을 가> 져가게 된 것이다.

> Java의 Checked Exception은 오용을 낳아 불필요한 코드 블럭이 생겨 가독성을 해치고, 예외가 발생한 즉시 나타나지 않고 위로 올라가고 나서야 나타나고 다뤄지다보니 처리하기 애매하거나 새로운 예외를 만들어서 던져버리는 경우가 많았는데 Kotlin에서는 아예 Checked Exception을 처리하지 않도록 한 점이 맘에 든다! :-D

참고로, `@Throws` 어노테이션을 사용하지 않고 try-catch 구문을 쓰게되면 **호출하는 구문에서 컴파일 에러가 발생**한다(Java에서는 예외를 던지는 곳에서 에러 발생함: try-catch 구문을 넣던지 위로 올리든지 하라는 에러). Kotlin의 Checked Exception에 대한 의견이 어떤지를 확실하게 엿볼 수 있다 :-)ㅋ

```kotlin
fun boo() {
    throw IOException()	// 여기서 발생하고 위로 올라가지 않기 때문
}

// in Java
// 컴파일에러 - 예외가 발생하지 않는데 왜 try-catch를 썼냐는 에러가 뜸
try {
    Demo.boo()
} catch (IOException e) {
    // ...
}
```

### [번외] Java의 Checked Exception에 대한 논란

김병묵(스터디진행자) 

> 혹시나 “Checked Exception 강제가 개발자들에게 방어적으로 코딩하도록 한다는 장점이 있지 않는가?” 라는 생각을 하실 수 도 있는데, 이 부분은 특정한 언어 설계에 대한 효용과 실익 비교에 따른 선택이라 볼 수 있습니다. 관련하여 좀 더 자세히 알아보고 싶으신 분들은 아래 자료들을 읽어보시길 추천합니다.

1. 로버트 C. 마틴  <클린 코드>의 7장 오류 처리 부분. 요약하면 checked exception은 안정적인 소프트웨어 제작을 위한 요소로 도입된 아이디어지만 이에 따른 비용이 발생하는데, 하위 단계에서 checked exception과 관련한 코드를 변경하면 상위 단계 메서드 선언부를 전부 고쳐야 해서 수정이 어려워지는데 이러한 의존성의 비용이 이익보다 크다는 것입니다.
     ( https://ko.wikipedia.org/wiki/개방-폐쇄_원칙 )

2. Checked Exception에 대한 논쟁에 대한 자료 모음 http://egloos.zum.com/benelog/v/1901121



## 참고자료

* [코틀린 입문 스터디 (4) Control Structures](<https://medium.com/@kbm1378/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%84%B0%EB%94%94-4-control-structures-e8e9af298ebd>)
* [Kotlin 키워드 및 연산자 해부 Part1](<https://medium.com/@joongwon/kotlin-kotlin-%ED%82%A4%EC%9B%8C%EB%93%9C-%EB%B0%8F-%EC%97%B0%EC%82%B0%EC%9E%90-%ED%95%B4%EB%B6%80-1-hard-keywords-3062f5fe2d11>)
* [Java에서 Checked Exception은 언제 써야 하는가?](http://egloos.zum.com/benelog/v/1901121)
* [Java Checked Exceptions : Good or Bad?](<https://programming.guide/java/checked-exceptions-good-or-bad.html>)

