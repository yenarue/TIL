Conventions
====

Kotlin은 Convention 과 확장함수를 통해 기존의 클래스에 대해서 기능을 확장시키기 수월하다. 또한, 편의성 기능과 클래스 핵심 기능을 분리하여 코드 가독성을 높힐 수가 있다.

## 연산자 오버로딩 (Operator overloading)

### 산수 연산자

+, -, *, /, % 연산을 오버로딩 할 수 있다.

```kotlin
operator fun Point.plus(other: Point): Point {
  return Point(other.x + x, other.y + y)
}

Point(1, 2) + Point(2, 3)
```

### 단항 연산자 (Unary operations)

++, --, -, +, ! 등의 단항 연산자를 오버로딩 할 수 있다.

### 대입 연산자 (Assignment operations)

a += b, a = b 등의 대입 연산자를 오버로딩 할 수 있다.

### 주의사항

* 새로운 연산자를 개발할 수는 없다
* 연산자 우선순위를 수정할 수는 없다
* 오버로딩이 허용된 연산자에 대응되는 메소드를 정의하는 방식으로만 구현된다
* 연산자는 일반적으로 합의된 의미내에서 사용되어야 하며 남용되면 안된다. (예 : plus는 ''합치는'' 의미를 가지는 연산이어야 한다. plus에 빼거나 나누거나 곱하는 의미을 넣으면 안된다;;;)

## Kotlin의 Conventions 들

### 비교 (Comparisons)

`compareTo()` 메소드에 따라 `<`, `<=` 과 같은 비교 연산이 이루어진다.

```kotlin
private operator fun Point.compareTo(other: Point): Int { ... }
```

### 동일함 체크 (Equality check)

`equals()` 메소드에 따라 `==`, `===` 연산이 이루어진다. Kotlin 에서는 `null` 에 대한 체크도 가능하다.

```kotlin
s1 == s2 			// s1.equals(s2)
null == "abc"	// false
null == null 	// true
```

### 인덱스 접근 (Accessing elements by index) : `[]`

`map[key]` 나 `arr[1]` 과 같은 인덱스 접근이 가능하다. Kotlin에서는 2개 이상의 인덱스 접근도 가능하다.

```kotlin
x[a, b] 			// = x.get(a, b)
x[a, b] = c 	// x.set(a, b, c)
```

###  `in` Convention

Collection에 특정 아이템이 존재하는지 확인하는 `in` 은 `contains()` 와 매칭된다

### `rangeTo` (`..`)

`start..end`, `start.rangeTo(end)` 와 같이 범위를 나타낸다

### `iterator`

Kotlin에서는 for 반복문에 사용가능하도록 만들기 위한  `iterator` 컨벤션이 존재한다. 

```kotlin
operator fun CharSequence.iterator(): CharIterator
for (c in "abc") { }
```

Kotlin의 String이 이터레이팅 될 수 있는 이유는 String의 확장 함수로 iterator 연산자를 구현했기 때문이다.

### 비구조화 선언 (destruction declaration)

람다와 루프문에서 사용가능했던 `val (a, b) = p` 와 같은 표현은 사실상 `val a = p.component1()` 과 `val b = p.component2()` 에 대응된다. 이는 `component()` 메소드를 생성해야 사용 가능하다.

data class 는 이 component()를 자동으로 구현해준다:

```kotlin
data class Contact(val name: String, val email: String, val phoneNumber: String) {
//  fun component1() = name
//  fun component2() = email
//  fun component3() = phoneNumber
}

// underscore(_)로 선언된 변수는 사용하지 않는 것으로 판단하여 컴파일러가 생성하지 않도록 번역한다.
val (name, _, phoneNumber) = contact
```



