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

