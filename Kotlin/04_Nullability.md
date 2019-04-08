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

#### Safe access

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

물론 null 일 경우에 null이 아닌 디폴트 값을 설정하는 것도 간단하게 가능하다

```kotlin
val length: Int = if (s != null) s.length else 0
val length: Int = s?.length ?: 0
```

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

Nullable 타입을 반환하는 함수를 Java로 변환하면 Optional 클래스가 아니라 Java annotation 형태로 변환된다. 이에 따라 성능 오버헤드는 발생하지 않으면서도 Null 문제를 해결할 수 있다.

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

