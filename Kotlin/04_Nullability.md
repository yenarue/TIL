Nullability
====

Null 발생 여부를 명확하게 한다.

## Nullable types

### Billion Dollar Mistake

`null` 이라는 개념을 만들어낸 Tony Hoare 는 이 발명을 Billion dollar mistake 라고 불렀다. Null Point Exception 문제들은 고치기도 어렵고 발견하기도 어렵기 때문이다.

* 이를 위한 현대적인 접근 : **Null Point Exception을 런타임 에러가 아니라 컴파일 에러로 만들자** => 좀 더 빠르게 알아낼 수 있도록 하자!

### Nullable type : `?`

```kotlin
val nonNull: String = "always not null"
val nonNull2: String = null // can't compile
val nullable: String? = null
val nullable2: String? = "can be null or non-null"

nonNull.length	// ok
nullable.length	// compile failed - null이 될 가능성이 있기 때문
```

#### Safe access: `?`

변수가 null이 아닐 때는 `.` 연산을 진행하고, null 일때는 null을 리턴한다.

```kotlin
val s: String?
s?.length // Int? 타입이 리턴됨

// 아래의 코드와 완전히 동일함
// if (s != null) {
//     s.length
// }
```

#### Elvis operator : `?:`

물론 null 일 경우에 null이 아닌 디폴트 값을 설정하는 것도 간단하게 가능하다!

```kotlin
val length: Int = if (s != null) s.length else 0
val length: Int = s?.length ?: 0
```

이 연산자는 Groovy 에서 아이디어를 얻어 가져온 것이다.

When using safe access, you can use so called Elvis separator and provide the default value that will be used when your expression is null. The following scheme illustrates how Elvis operator works. The result is either the left expression if it's not null, or the right expression if the left expression is null. Why score it like this? You can guess it if you read the operator as an emoticon. Elvis operator isn't a quarter invention. This name comes from Groovy. The whole bunch of nullability operators, including Elvis operator, comes from Groovy. They are proven to be very useful for working with expressions that can be null.

#### Control-flow analysis

흐름상 분명히 null이 아닌 경우에는 nullable 변수인 경우에도 `?` 연산자 없이 `.` 연산이 가능해진다.

```kotlin
val s: String?
if (s == null) return
s.length
```


### Not-null assertion : `!!`

`!!` 연산자를 사용하면 절대 `null`이 아님을 보장한다는 뜻으로 쓰인다. 이 경우에는 `!!` 를 지정한 변수가 `null` 이면 NPE를 발생시킨다.

```kotlin
val s: String?
s!!.lenghth

// 아래처럼 null이 아니라는 흐름 파악이 가능하면 !!를 안써도 된다.
val s1: String?
s1!!
s1.length
```

#### Bad examples

`!!` 은 논리적으로 확실하게 `null` 일 가능성이 없다고 판단될 때에만 사용해야 한다. 

아래와 같이 작성하면 Billion dollar mistake가 재현되는 꼴이다. 지양하자.

```kotlin
person.company!!.address!!.country
```

NPE가 발생 가능하다는 것은 어찌보면 Java와 비슷한 꼴이 아니냐는 생각이 들 수 있다. 하지만 Java에 비해 **NPE 발생가능 지점을 명시적으로 보여준다**는 점이 `!!` 의 장점이라고 볼 수 있다.

### Under the hood

#### Nullable types != Optional

Nullable 타입을 반환하는 함수를 Java로 변환하면 Optional 클래스가 아니라 Java annotation 형태로 변환된다. .

```kotlin
// Kotlin
fun foo(): String = "foo"
fun boo(): String? = "boo"
```

```java
// Java
@NotNull
public static final String foo() {
    return "foo";
}

@Nullable
public static final String boo() {
    return "boo";
}
```

#### Optional 클래스 대신 Nullable 타입을 사용함으로써 얻는 장점들

1. Kotlin에는 아예 Optional 클래스 자체가 존재하지 않는데, Null 처리를 위해 Wrapper 객체를 사용하던 Optional 클래스를 사용하지 않으므로 이에 따라 런타임 시에 추가적인 성능 오버헤드는 발생하지 않으면서도 Null 문제를 해결할 수 있게 된다.
2. Nullable 타입은 컴파일 타임에 Null 여부를 체크할 수 있기 때문에 안전성이 높다 볼 수 있다.

## Safe casts : `as?`

### Type cast (Unsafe cast) : `as`

명시적으로 캐스팅 할때는 `as`를 쓰면 된다. `is` 를 이용하면 smart cast가 되어 암시적 캐스팅이 가능하다.

```kotlin
// as 를 이용한 명시적 캐스팅
val s = any as String

// is 를 이용한 암시적 캐스팅
if (any is String) {
    any.toUppserCase()
}
```

언뜻보면 꽤 안전해보이지만 `as` 사용시에 주의해야할 점이 있다.

`Int` 와 `Int?` 는 Non-nullable 타입과 Nullable 타입으로서 다른 타입처럼 보여진다. 그렇다면 아래 코드는 무엇을 리턴할까?

```kotlin
val s: Int = 1
println(s as Int?)	// 1
```

다른 타입이라 생각됨에도 불구하고 캐스팅되어 1을 리턴한다. 이는 `Int?` 이 Annotation 방식으로 구현되어있기 때문이다. (윗 단락의 'Under the hood' 부분 참고) 본질적으로는 같은 타입이며 Null이 될 수 있는지의 여부를 그저 Annotation로 처리한 것일 뿐이다.

### Safe cast : `as?`

`as?` 를 사용하면 좀 더 안전한 캐스팅이 가능하다. 캐스팅이 성공하면 해당 타입을, 성공하지 않으면 null 을 리턴한다.

```kotlin
foo as? Type
// foo가 Type이면 캐스팅이 됨
// foo가 Type이 아니면 null 반환
// 아래의 조건 분기문과 동일
if (foo is Type) foo else null

// 아래와 같이 사용하는 예를 확인해보면 왜 안전한지 감이 바로 올 것임!
(any as? String)?.toUpperCase()
```

## Nullability의 중요성 (importance of nullability)

Kotlin 타입 시스템의 Nullability는 Kotlin의 가장 중요한 기능이다.

NULL 자체는 저장된 값이 없다는 것을 표현하기 위한, 꼭 필요한 개념이지만 Runtime 상에서 발생하는 Null Point Exception을 예상하기 어렵다는 점 때문에 안티패턴으로도 치부되고 있다.

### 안티 패턴을 적합한 패턴으로! (anti-pattern to legitimate pattern)

Kotlin에서는 Nullable/Non-nullable 타입과 `?`, `?:`, `!!`, `as?` 등의 오퍼레이터들, smart casts 등의 기능들을 통해 NULL의 문제들을 효과적으로 통제하고 추적할 수 있다. 

그리고 NULL을 일급 객체 (First Class Citizen)로 다룰 수 있다.

### 서브타이핑 (SubTyping)

Java8의 Optional 보다 Kotlin이 선택한 Annotation 방식이 더 좋은 이유는 무엇일까? 성능 오버헤드가 적다는 이유외에도 런타임 상에서 Nullable 타입과 Non-Nullable 타입이 동일한 타입으로 동작하여 변수 할당에 유연성을 가져갈 수 있다는 점도 그 이유이다! :-D

* Non-Nullable 타입과 Nullable 타입은 서브타이핑이다!

> *서브클래스가 슈퍼클래스를 대체할 수 있는 경우 이를 서브타이핑이라고 한다. 서브클래스가 슈퍼클래스를 대체할 수 없는 경우에는 서브클래싱이라고 한다. 서브타이핑은 설계의 유연성이 목표인 반면 서브클래싱은 코드의 중복 제거와 재사용이 목적이다. 흔히 서브타이핑을 인터페이스 상속(interface inheritance)이라고 하고, 서브클래싱을 구현 상속(implementation inheritance)이라고 한다.*



## 참고자료

* [코틀린 입문 스터디 (7) Nullability](<https://medium.com/@kbm1378/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%84%B0%EB%94%94-7-nullability-77d92220aad2>)
* [[Kotlin] Kotlin 키워드 및 연산자 해부 Part1](<https://medium.com/@joongwon/kotlin-kotlin-%ED%82%A4%EC%9B%8C%EB%93%9C-%EB%B0%8F-%EC%97%B0%EC%82%B0%EC%9E%90-%ED%95%B4%EB%B6%80-1-hard-keywords-3062f5fe2d11>)
* [Kotlin Nullable Type vs. Java Optional](<https://medium.com/@fatihcoskun/kotlin-nullable-types-vs-java-optional-988c50853692>)
  * [번역글 - 코틀린 Nullable 타입 vs. 자바 Optional](<https://medium.com/@limgyumin/%EC%BD%94%ED%8B%80%EB%A6%B0-nullable-%ED%83%80%EC%9E%85-vs-%EC%9E%90%EB%B0%94-optional-e698adc6d617>)