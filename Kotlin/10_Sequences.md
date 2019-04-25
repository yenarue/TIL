Sequences
====

## Collections vs Sequences

### Collections : Eager Evaluation

`filter`, `map`, `find`, `groupBy` 등의 Collections 확장 함수들은 `inline` 으로 정의되어있어 익명 클래스 객체 생성이 필요없기 때문에 관련 퍼포먼스 오베헤드가 줄어들 수 있다.

하지만, 또 또 다른 관점에서는 퍼포먼스 측면 문제가 발생한다. 이러한 확장 함수들은 매 실행 시 마다 그 함수가 실행된 결과를 가지는 Collections 를 반환한다. 그렇기 때문에 확장 함수들을 체이닝하여 호출하게 되면 체이닝 한 횟수만큼 Collections가 생성된다. 그로인해 **최종 결과물을 도출할 때 까지 불필요한 중간결과물을 생성**한다는 비효율성이 나타난다.

### Sequences : Lazy Evaluation

Kotlin의 Sequence 는 Java8의 Stream 에 대응되는 개념이다. 즉, Lazy 관점으로 연산을 접근한다. 중간 결과물을 만들지 않고 쌓아두다가 마지막 'action' 연산시에 수행되는 것을 뜻한다.

> **Lazy Execution vs Lazy Evaluation ?** 

Collection에 `asSequence()` 메소드를 호출시킴으로서 아주 간단하게 Sequence로 전환시킬 수 있다.

```kotlin
val list = listOf(1, 2, 3)
val maxOddSquare = list.asSequence()
			.map { it * it }				// koltin.sequences.TransformingSequence
			.filter { it % 2 == 1 } // koltin.sequences.FilteringSequence
			.max()									// 9
```

### 비교에 대한 결론

* Collections : intermediate collections are created on chained calls
* Sequences : lambdas are not inlined

## Horizontal Evaluation vs Vertical Evaluation

Collections의 Eager Evaluation의 경우 Horizontal Evaluation 과 유사하다. 즉, 각 함수들의 실행이 수평적으로 이루어져 중간 결과값이 하나씩 나타나는 것이다. 

Sequences의 Lazy Evaluation의 경우 Vertical Evaluation 과 유사하다. 즉, 각 함수의 실행이 수직적으로 이루어져 중간 결과물이 만들어지지 않는 것이다. 실행이 수직적이라는 것은 오퍼레이션을 리스트 단위로 수행하여 중간 결과물을 내는 것과는 달리, 오러페이션 집합이 각 요소별로 수행된다는 뜻이다.

```kotlin
fun m(i: Int): Int { 
  print("m$i ")
  return i 
}

fun f(i: Int): Boolean { 
  print("f$i ")
  return i % 2 == 0
}

val list = listOf(1, 2, 3, 4) 
list.map(::m).filter(::f)  //m1 m2 m3 m4 f1 f2 f3 f4
list.asSequence().map(::m).filter(::f).toList() //m1 f1 m2 f2 m3 f3 m4 f4
list.asSequence().map(::m).filter(::f)	// 아무것도 출력되지 않는다
```

## 연산자 순서의 중요성

함수형 프로그래밍에서는 연산자의 순서를 어떻게 배치하느냐에 따라 같은 결과도 퍼포먼스가 달라질 수 있다.

```kotlin
list.asSequence().map(::m).filter(::f).toList()	// [2, 4]
// 출력 : m1 m2 m3 m4 f1 f2 f3 f4

list.asSequence().filter(::f).map(::m).toList()	// [2, 4]
// 출력 : f1 f2 m2 f3 f4 m4
```

위의 두 코드는 둘 다 결과 값은 `[2, 4]` 로 동일한 값을 보이지만 실제 연산은 `filter` 를 먼저 수행한 쪽이 더 적게 일어났다.

## Creating Sequences

Kotlin 에서 Sequence는 `Sequence` 라는 인터페이스로 구현되어있다.

```kotlin
interface Sequence<out T> {
  operator fun iterator(): Iterator<T>
}
```

### 중간 연산(Intermediate operations)와 종료 연산(terminal operation)

Kotlin Sequence의 함수들은 중간연산과 종료연산로 나뉜다.

#### 중간연산 (Intermediate operations)

Intermediate operation은 쌓이기만 할 뿐 실제로 수행되지 않는다. 실제로는 Sequence를 반환할 뿐이다. 이렇게 쌓여있던 중간연산자들은 종료연산자가 호출되었을 때 비로소 실행된다.

```kotlin
fun <T> Sequence<T>.filter(predicate: (T) -> Boolean): Sequence<T>
fun <T, R> Sequence<T>.map(transform: (T) -> R): Sequence<R>
```

#### 종료 연산 (Terminal operations)

Terminal operation은 즉시 Lambda를 호출하여 연산을 수행한다. 그렇기 때문에 `inline` 함수로 구현되어있다. 반환 값도 결과 값을 반환하도록 되어있다.

```kotlin
inline fun <T> Sequence<T>.any(predicate: (T) -> Boolean): Boolean
inline fun <T> Sequence<T>.find(predicate: (T) -> Boolean): T?
```

### Sequence 생성하기

컬렉션을 시퀀스로 전환하는 `asSequence()` 메소드외에도 시퀀스를 생성할 수 있는 메소드들이 존재한다.

* `generateSequence(nextFunction: () -> T?)` 

* `generateSequence(seed: T?, nextFunction: () -> T?)`  : 초기값 (seed)를 지정해줄 수 있다.

* `yield` : 개발자가 원하는 특정한 규칙에 따라 커스텀 시퀀스를 만들어 낼 수 있다.

  ```kotlin
  // infinite integer
  val numbers = sequence {
    var x = 0
    while (true) {
      yield(x++)	// return은 그대로 종료지만, yield는 numbers 큐에 하나씩 넣어지는 느낌이라 이해하면 된다.
    }
  }
  number.take(5).toList()	// [0, 1, 2, 3, 4]
  ```

  `yieldAll(list)`, `yieldAll(sequence)` 등도 사용 가능하다.

## Library Functions







## 참고자료

* [코틀린 입문 스터디 **(15) Sequences**](https://medium.com/@kbm1378/코틀린-입문-스터디-15-sequences-52cfca1805c8)