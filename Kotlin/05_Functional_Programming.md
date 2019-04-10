함수형 프로그래밍 (FP, Functional Programming)
====
Kotlin은 함수형 프로그래밍을 일부 지원한다. (100%는 아니다!)

## 람다 (Lambda)

Lambda 는 표현식 (expression), 인스턴스 (instance), 인수 (argument, actual parameter)로서 사용가능한 **익명함수**이다.  간결성이 극대화되고 자율성이 높아지는 이점이 부각되어 최근에는 대부분의 modern language 들에서 람다 표현식을 지원하고 있다. (Java도 Java8 이후부터 도입)

> 사실 C계열 언어에서는 pointer 변수 개념으로 인해 함수의 인스턴스, 인수화 개념 자체가 너무나 당연해서 딱히 명칭을 붙여 부르지도 않았지만 OOP 언어 계열에서는 복잡성을 이유로 pointer 변수 개념을 없앰으로서 이 사용법 자체가 불가능했었다. 나도 사실 처음에 Java를 익힐 때 가장 불편했던 점이 이 부분이었다. 함수가 표현식이 될 수 없어서….. 얼마나 불편했던지… 이제는 넘나 익숙해져서 괜찮지만….ㅋ… SmallTalk 에서 Java로 넘어갈때 엉클밥이 하셨던 말씀이 떠오른다.

Kotlin의 Lambda 형태는 다음과 같다.

```kotlin
{x: Int, y: Int -> x + y}
{x, y -> x + y}
people.filter { it.name == "YENA" }
map.mapValues { entry -> "${entry.value}"}
map.mapValues { (key, value) -> "$value!"}

// Java에서는 _를 쓰면 Warning을 띄우지만 대부분의 FP 계열 언어에서는 사용하지 않는 변수에 대해서는 아예 표시를 하지 않거나 미지수를 표현하는 `_`를 권장한다
map.mapValues { (_, value) -> "$value!"}
```

위와 같이 익명함수를 매우 간단 명료하게 표현할 수 있어 가독성이 올라가는 효과가 있다.

## Collection의 일반적인 연산들 (Common Operations on collections)

Kotlin은 많은 수의 Collection 확장함수를 제공한다. 

일단 일반적인 연산들에 대해 알아보자. 

대중적인 연산인 `filter`, `count`, `flatMap`, `map`,  `reduce` , `first`, `last`는 설명을 생략하도록 하겠다. 레퍼런스는 [여기](<https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index.html>)에 있으니 모르는 연산이 보이거나 어떤 연산들이 존재하는지 궁금하다면 확인해보자.

### `all(predicate): Boolean`

`all()` 은 모든 아이템이 해당 조건에 맞는지 확인한다. 모든 아이템이 조건을 충족하면 `true`, 하나라도 충족하지 못하면 `false` 를 반환한다.

### `any(predicate): Boolean`

`any()` 는 해당 조건에 맞는 아이템이 하나라도 존재하면 `true`, 전혀 없으면 `false` 를 반환한다.

```kotlin
val list = 1..10
list.any { it % 2 == 0 }	// true

val emptyList = emptyList<Int>()
emptyList.any()	// false
```

### `find(predicate): T?`

`find()` 는 해당 조건에 맞는 첫번째 원소를 반환한다. = `firstOrNull()`

### `partition(predicate): Pair<List<T>, List<T>>`

`partition()` 은 해당 조건에 `true` 인 경우와 `false` 인 경우의 아이템 리스트를 각각 만들어 리턴한다.

### `groupBy(keySelector): Map<K, List<T>> `

`grouptBy()` 는 해당 값을 가지는 아이템들을 리스트를 묶어 Map으로 만든다.

```kotlin
val people = listOf("Bob" to 29, "Alice" to 31, "Carol" to 31)
val ageMap = people.groupBy { it.second }
println(ageMap)	// {29=[(Bob, 29)], 31=[(Alice, 31), (Carol, 31)]}
```

### `associateBy(keySelector): Map<K, T> `

`associateBy()` 는 `groupBy` 와 유사하지만 하나의 Key에 하나의 아이템만 담길 수 있게 되어있다. 가장 마지막에 들어온 아이템만 저장된다. (중복 제거)

```kotlin
val people = listOf("Bob" to 29, "Alice" to 31, "Carol" to 31)
val ageMap = people.associateBy { it.second }
println(ageMap)	// {29=(Bob, 29), 31=(Carol, 31)}
```

### `associate(transform: T -> Pair<K, V>): Map<K, V>`

`associate()` 는 지정한 구조의 Pair 구조를 Map 구조로 변환시킨다.

```kotlin
val list = listOf(1, 1, 2, 3)
list.associate { it to it * 10 }	// {1=10, 2=20, 3=30}

// Map으로 반환하다보니 가장 마지막에 들어간 값만 저장된다.
val pairList = listOf(1 to 1, 1 to 2, 2 to 2, 3 to 3)
pairList.associate { (one, two) -> one to (two + 2) } // {1=4, 2=4, 3=5}
```

### `zip(other): List<Pair<T, R>>`

`zip()` 은 2개의 리스트를 결합시킨다.

```kotlin
val list1 = listOf(1, 2, 3)
val list2 = listOf("a", "b", "c")
list1.zip(list2)	// [(1, a), (2, b), (3, c)]
```

### `zipWithNext(): List<Pair<T, T>>`

`zipWithNext()` 는 `zip()` 과는 달리 하나의 리스트를 Pair 리스트의 형태로 만든다. 항상 그 다음번 아이템과 Pair를 만들어내는 방식이다.

```kotlin
val list1 = listOf(1, 2, 3)
list1.zipWithNext()	// [(1, 2), (2, 3)]
```

### `flatten(): List<T>`

`flatten()` 은 리스트안에 리스트가 있는 중첩 리스트를 하나의 리스트로 펼쳐준다. `flatMap`의 flat이 바로 이 flatten이다.

```kotlin
val list = listOf(listOf(1, 2, 3), listOf(6, 7), listOf(4, 5))
println(list.flatten())	// [1, 2, 3, 6, 7, 4, 5]
```



## 참고자료

* 맨날 헷깔리는 [argument 의 번역](<https://groups.google.com/forum/#!topic/scala-korea/P2KKbN-H_-c>)