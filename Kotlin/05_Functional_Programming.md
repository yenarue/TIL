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

## Function types

Function type은 말그대로.. 함수를 저장한 변수의 타입을 뜻한다. 정확하게는 함수의 참조(reference) 정보를 저장한 변수(variable)의 타입이다. 

Function type은 `(argument) -> return type` 과 같은 형태로 정의할 수 있다.

```kotlin
val isEven: (Int) -> Boolean = { i: Int -> i % 2 == 0 }
isEven(42)

// 아래와 같이 타입 추론도 가능
val isEven = { i: Int -> i % 2 == 0 }
```

참고로 C 계열 언어와는 달리, 함수로 선언한 경우에는 변수에 저장할 수는 없다. **오직 Lamdba 만 저장 가능하다.**

```kotlin
fun isEven(i: Int): Boolean = i % 2 == 0
val predicate = isEvent // 컴파일 에러!
```

### Calling lambda directly

람다는 아래와 같이 직접적으로 호출이 가능하다.

```kotlin
{ println("hey!") }()	// 이렇게 호출이 가능하지만 가독성이 떨어지므로
run { println("hey!") }	// 이렇게 호출하는걸 추천한다
```

### SAM (Single Abstract Method) interface

추상 메소드가 딱 하나만 존재하는 interface를 뜻한다. Java8에서는 이러한 interface를 Lambda로 치환해서 사용할 수 있다.

```java
// java
public interface Runnable {
    public abstract void run();
}

void postpone(int delay, Runnable computation) {...}

postpone(1000, () -> { println("yeah!") })
```

Kotlin과 Java를 함께 사용할때는 아래와 같이 바로 치환이 가능하다.

```kotlin
postpone(1000) { println("yeah!") }

// Java 처럼 사용하고싶다면 이렇게
val f = Runnable { println("yeah!") }
postpone(1000, f)
```

### Nullability Function types

Function type도 일반 변수와 같이 Nullabile & Non-nullable 타입 처리가 가능하다.

- `() -> Int?` : Nullable Int 를 리턴하는 함수
- `(() -> Int)?` : Nullable 함수 타입 (이 함수를 저장하는 변수가 Nullable)

## 멤버 참조 (Member references)

Java와 마찬가지로 Kotlin도 Member reference를 할 수 있다.

```kotlin
class Person(val name: String, val age: Int)
people.maxBy { it.age }
people.maxBy(Person::age)
```

Member reference를 활용하면 이미 함수를 선언 및 정의한 경우에도 변수에 저장할 수 있게 된다.

```kotlin
fun isEvent(i: Int): Boolean = i % 2 == 0
val predicate = ::isEven
```

### Bound & Non-bound references

#### Non-bound Reference

```kotlin
class Person(val name: String, val age: Int) {
    fun isOlder(ageLimit: Int) = age > ageLimit
}
val nonBound_agePredicate: (Person, Int) -> Boolean = Person::isOlder
val alice = Person("Alice", 29)
nonBound_agePredicate(alice, 21)	// true
```

#### Bound Reference

```kotlin
val bound_agePredicate: (Person, Int) -> Boolean = alice::isOlder
bound_agePredicate(21)	// true
```

## return from Lambda

- Kotlin에서 `return` 키워드는 `fun` 키워드로 명시된 함수에 대응된다.
- `fun` 키워드로 선언되지 않는 Lambda 식에서 `return` 키워드를 사용하면 **바깥쪽(outer) 함수에 대해 리턴**을 하게된다.

일단 아래의 예시 코드를 확인해 보자. 우리가 구현하려는 메소드는 '리스트를 파라미터로 받아와서 리스트의 아이템 중 0이 아닌 요소만 2개씩 가지는 새로운 리스트를 반환하는 메소드'이다.

```kotlin
fun duplicateNonZero(list: List<Int>): List<Int> {
    return list.flatMap {
        if (it == 0) return listOf()
        listOf(it, it)
    }
}

println(duplicateNonZero(listOf(3, 0, 5)))	// []
```

얼핏보면 정상동작 할 것처럼 보여지지만 결과는 엉뚱하게도 emptyList(`[]`)가 반환되었다.

flatMap의 파라미터인 Lambda 함수의 `return` 키워드를 보면, 마치 하나의 이터레이션에 대해서만 `listOf()` 를 반환할 것 처럼 보여지진다. 하지만 사실은 **이  `return` 키워드는 바깥쪽 함수에 대하여 실행**되게 된다. 그에 따라 결과값은 emptyList (`[]`) 가 되었던 것이다.

사실 이 코드는 설명을 자세히 읽지 않으면 오해하기 쉬운 코드이다. 왜 이렇게 동작하는걸까?

위의 `duplicateNonZero` 함수를 `for` 루프 구문으로 바꿔서 보도록 하자.

```kotlin
fun duplicateNonZero(list: List<Int>): List<Int> {
    var result: MutableList<Int> = mutableListOf()
    for (i in list) {
        if (i == 0) return listOf()
        result.addAll(listOf(i, i))
    }
    return result
}
```

`for` 구문 안에 있는 `return` 키워드는 당연히 함수의 반환 키워드임을 알 수 있다. 이 루프문을 Functional Method 로 치환시켜보면 왜 `return` 키워드가 outer function의 리턴값으로 인식되었는지 알 수 있을 것이다.

그렇다면 처음 우리가 의도했던대로 동작하게 하려면 어떻게 해야할까? (Lambda 식에 대한 리턴)

### Lambda 리턴시키기

#### 1. Label 이용하기

Label을 이용하여 간단하게 해결가능하다.

```kotlin
// 1. 반환시킬 메소드명과 동일한 라벨을 사용하는 방법
list.flatMap {
    if (it == 0) return@flatMap listOf<Int>()
    listOf(it, it)
}

// 2. alias 를 붙여주는 방법
list.flatMap hey@{
    if (it == 0) return@hey listOf<Int>()
    listOf(it, it)
}
```

Label을 붙이면 Java의 `continue` 와 동일하게 동작하게 된다.

그럼 이제 이 방법을 이용해서 처음의 `duplicateNonZero` 함수를 올바르게 수정해보자.

```kotlin
fun duplicateNonZero(list: List<Int>): List<Int> {
    return list.flatMap {
        if (it == 0) return@flatMap listOf<Int>()
        listOf(it, it)
    }
}
println(duplicateNonZero(listOf(3, 0, 5)))	// [3, 3, 5, 5]
```

#### 2. Local Function 이용하기

outer function에 대하여 리턴시킨다는 점을 이용해서 내부 함수를 하나 더 구현하는 방법이다.

```kotlin
fun duplicateNonZeroWithLocalFunction(list: List<Int>): List<Int> {
    fun duplicateNonZeroElement(e: Int): List<Int> {
        if (e == 0) return listOf()
        return listOf(e, e)
    }
    return list.flatMap(::duplicateNonZeroElement)
}
println(duplicateNonZero(listOf(3, 0, 5)))	// [3, 3, 5, 5]
```

#### 3. 익명 함수(Anonymous function) 이용하기

익명 함수를 명시적으로 정의해서 해당 함수에 대한 리턴임을 명확히 하는 방법이다.

```kotlin
fun duplicateNonZero(list: List<Int>): List<Int> {
    return list.flatMap(fun (e): List<Int> {
        if (e == 0) return listOf()
        return listOf(e, e)
    })
}
println(duplicateNonZero(listOf(3, 0, 5)))	// [3, 3, 5, 5]
```

Lambda 도 익명함수인데 왜 다르게 동작할까?

- Lambda와 익명함수는 java bytecode 레벨에서는 동일하지만, Kotlin의 '`fun` 키워드 기준 return 방식'으로인해  return 에 대한 대응지점만 달라진다.

#### 4. `return` 을 사용하지 않기

`return` 키워드를 사용하지 않고 컨텍스트 컨트롤을 하는 방법이다.

```kotlin
fun duplicateNonZero(list: List<Int>): List<Int> {
    return list.flatMap {
        if (it == 0)
            listOf()
        else 
        	listOf(it, it)
    }
}
println(duplicateNonZero(listOf(3, 0, 5)))	// [3, 3, 5, 5]
```

## 코틀린은 함수형 언어일까? (Is Kotlin is functional language?)

Kotlin은 순수한 함수형 언어는 아니다. OOP, Structural, Functional Programming 등 다양한 패러다임의 아이디어를 조합하여 만들어진 언어이다.

각 패러다임의 영향을 받은 Kotlin의 특징들은 아래와 같다:

- OOP : class, interface
- FP : lambda, 불변성(immutability), 고차 함수(high-order function)

> 여기서 계속 practical purpose (실용적 관점/목적)이라는 이야기가 나오는데 프로그래밍 관점에서 실용적이라는 것은 뭘까..

'무슨 형태의 언어인가'를 고민하기보다는 '어떤 스타일의 프로그래밍이 가능한 언어인가'를 고민하는 것. Kotlin은 함수형 언어는 아니지만 **함수형 스타일의 프로그래밍이 가능한 언어**이다.

## 참고자료

* 맨날 헷깔리는 [argument 의 번역](<https://groups.google.com/forum/#!topic/scala-korea/P2KKbN-H_-c>)
* [Kotlin SAM(Single Abstract Method)을 사용하는 방법](<https://thdev.tech/kotlin/androiddev/2017/10/07/Kotlin-SAM/>)