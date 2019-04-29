Lambda with Receiver (extension lambda)
====

Lambda with receiver 는 Kotlin의 강력한 기능 중 하나이다. 이 기능은 Extension Function과 Lambda에 대한 아이디어의 결합(union)으로 볼 수 있다.

> Extension Function & Lambda = Lambda with receiver

## `with()` 를 통해 알아보는 Lambda with receiver

```kotlin
val sb = StringBuilder()
sb.appendIn("Alphabet: ")
for (c in 'a'..'z') {
  sb.append(c)
}
sb.toString()
```

위의 코드에서 **`sb` 에 대한 연산이 반복적으로 이루어졌음**을 알 수 있다. Kotlin 표준 라이브러리에서는 이를 좀 더 **간결**하게 표현할 수 있는 `with()` 함수를 제공한다.

```kotlin
val sb = StringBuilder()
with (sb) {
  // 이 구문의 주체(this)는 sb이다.
  // 이것이 아래의 StringBuilder 함수를 그대로 호출할 수 있는 이유이다.
  appendln("Alphabet: ")
  for (c in 'a'..'z') {
    append(c)
  }
  toString()
}
```

`with` 의 구성 요소들을 하나씩 살펴보자:

- with : 함수
- lambda : `wtih` 의 두번째 인수(argument)
- `this` : Lambda 내의 implicit receiver 

`with` 의 실제구현은 아래와 같다:

```kotlin
inline fun <T, R> with(
	receiver: T,
  block: T.() -> R
): R = receiver.block()
```

Lambda를 인자로 받기때문에 inline으로 정의하는 것이 퍼포먼스 개선에 좋고 판단하여 lambda with receiver는 대부분 inline으로 구현되어있다. **특정 멤버(receiver)를 비번하게 사용하는 로직**일수록 lambda with receiver를 사용하는 것이 **간결성**과 **가독성**을 높힐 수 있는 방법이다.

## Extension function vs lambda with receiver

확장함수와 리시버를 가진 람다는 아래와 같은 표로 매칭될 수 있다:

| regular function   | regular lambda       |
| ------------------ | -------------------- |
| extension function | lambda with receiver |

```kotlin
val isEven: (Int) -> Boolean = { it % 2 == 0 }
val isOdd: Int.() -> Boolean = { this % 2 == 1 }
isEven(0)	// calling as regular function
1.isOdd()	// calling as extension function
```

## Lambdas with receiver을 사용하는 예시

### `buildString`

Lambda with receiver 로 구현된 예시 중 하나이다. 위에서 `with(sb)` 으로 구현했던 내용은 `buildString` 을 이용해서도 구현 가능하다.

```kotlin
val s: String = buildString {
  appendln("Alphabet: ")
  for (c in 'a'..'z') {
    append(c)
  }
}

inline fun buildString(
  builderAction: StringBuilder.() -> Unit
): String {
  val stringBuilder = StringBuilder()
  stringBuilder.builderAction()
  return stringBuilder.toString()
}
```

### HTML Builders

```kotlin
html {
  table {
    for (product in products) {
      tr {
        td { text(product.description) }
        td { text(product.price) }
        td { text(product.popularity) }
      }
    }
  }
}
```

### Gradle Build Script in Kotlin

Gradle 빌드 스크립트도 Kotlin 으로 작성가능하다.

```kotlin
plugins {
  application
  kotlin("jvm") version "1.3.10"
}

application {
  mainClassName = "sample.HelloWorldKt"
}

dependencies {
  compile(kotlin("stdlib"))
}

repositories {
  jcenter()
}
```

## More useful library functions

```kotlin
inline fun <T, R> with(receiver: T, block: T.() -> R): R { return receiver.block() }
inline fun <T, R> T.run(block: T.() -> R): R { return this.block() }
inline fun <T, R> T.let(block: (T) -> R): R { return block(this) }
inline fun <T> T.apply(block: T.() -> Unit): T { this.block(); return this; }
inline fun <T> T.also(block: (T) -> Unit): T { block(this); return this; }
```

위의 함수들은 아래 3가지 항목에 있어 차이점이 존재하며 사용 목적이 달라진다.

* 함수가 객체를 reciver로(확장함수 처럼) 받는지, 함수의 argument로 받는지
* `block`이 일반 lambda인지 lambda with receiver 인지
* 함수의 반환값이 객체인지, lambda인지


### `run`

`run` 은 `with` 과 매우 비슷하지만 `with` 보다 좀 더 확장된 버전이다. `run` 은 **객체를 receiver로 받기 때문에 `?` 연산자를 통해 안전한 접근(safe access)이 가능**하다!

```kotlin
inline fun <T, R> T.run(block: T.() -> R): R { return this.block() }

val windowOrNull = windowById["main"]
windowOrNull?.run {
  width = 300
  height = 200
  isVisible = true
}
```

### `apply`

`apply` 는 객체를 receiver로 받고 해당 객체의 타입을 반환하는 함수이다. **해당 객체의 타입을 반환하기 때문에 추가적인 연산을 체이닝 형태로 수행 할 수 있다**.

```kotlin
inline fun <T> T.apply(block: T.() -> Unit): T { this.block(); return this; }

val mainWindow = windowById["main"]?.apply {
  width = 300
  height = 200
  isVisible = true
} ?: return
```

### `also` : regular argument instead of `this`

`also` 는 `apply` 와 유사하다. 하지만 `also` 는 **`block` 을 일반 Lambda로 받는다. 그렇기 때문에 꼭 객체에  `it` 으로 접근해주어야 한다. **

```kotlin
inline fun <T> T.also(block: (T) -> Unit): T { block(this); return this; }

windowById["main"]?.apply {
  width = 300
  height = 200
  isVisible = true
}?.also {
  showWindow(it)
}

// 아래와 같이 객체의 이름을 지정해서 좀 더 명시적으로도 사용할 수 있다.
//}?.also {
//  window -> showWindow(window)
//}
```

Lambda with the receiver 는 객체에 대한 멤버참조를 하는 경우  `this` 참조를 생략할 수 있으므로 유용하다. 하지만 객체가 특정 함수의 인수(argument)로 전달되어야 하는 경우에는 일반 Lambda 가 더 유용하다.

> 왜지? 그냥 Lambda with the receiver에서 this를 써서 수행해도 되는게 아닌가? 되긴하는데 지양하는 건가? 
>
> => 물론 문법적으로는 가능하지만 그렇게 구현하는 것을 지양하는 것이다. 사실 this는 생략 가능하도록 되어있기 때문에 this를 넘긴다는 것 자체가 추상화된 것을 넘기는 느낌이므로 지양한다는 의미를 추가했다고 볼 수 있다. 그에비해 it으로 명시된 표현의 경우에는 생략이 불가능하고 실체가 존재하는 것으로 느껴지기 때문에 it으로 접근하는 경우에만 인수로서 사용하는 것을 지향한다고 볼 수 있다.
>
> 이는 스터디 진행자분께서 공유해주신 아래의 표를 보면 더욱 그 미묘한 차이를 알 수 있다:
>
> ![](./images/kotlin_lambda_with_receiver_table.png)
>
> > - 관련하여 정리한 내용 참고 부탁드립니다. 일단은 정리해서 말씀드리면 사실 큰 차이가 없는데, 그럼에도 굳이 구분을 한 이유는 각 함수의 네이밍에 기반해서 특정한 목적과 상황일때는 가능하면 그 함수를 쓰도록 권장해서 코드의 가독성을 높이기 위함입니다.
> > - 예를 들어 apply는 적용하다라는 뜻이니, 무언가 작업을 해서 할당하는데 쓰는 것, also는 또한이라는 뜻이니 재할당은 없지만 무언가 관련있는 작업을 같이할 때 쓰기, run은 수행하다라는 뜻이어서 함수를 정의할 때 쓰기.. 등등입니다. 관련하여 위에 이미지 읽어봐주시면 좋을 것 같아요 :slightly_smiling_face:
> > - 그리고 아래 2개 글도 읽어보시면 이해하시는데 더 도움이 될 것 같습니다.
> >    - https://medium.com/@limgyumin/코틀린-의-apply-with-let-also-run-은-언제-사용하는가-4a517292df29
> >    - https://kotlinlang.org/docs/reference/coding-conventions.html (edited)
> >
> > 즉, 질문 주신 내용처럼 인수로 전달되는 경우에 this를 사용해도 문제는 없습니다. 하지만 기본적으로 Kotlin은 그동안 쭉 지향해왔던 바를 보면 축약된 문법을 통해 생산성을 높여주고 코드의 간결성과 가독성을 높여주는 방향을 지향했습니다. (코틀린 철학도 다시한번 나오는 부분이구요) 이에 따라 사용해도 문제는 없지만 위에 공유드린 이미지 처럼 각 상황에 적합한 문법을 사용하도록 하여 가독성을 높이고자 하는 측면이라 볼 수 있습니다. 한편 이 경우 주의할점은 팀에서 이러한 문법을 사용할 때 apply/run/whith/also/let에 대해 모든 팀원이 정확한 의미의 이해와 합의하에 쓴다면 상당히 코드 가독성을 높일 수 있는 기회가 되지만, 반대로 이 부분에 대한 합의없이 각자 자기 마음대로 쓰게 된다면 오히려 코드가 더 헷갈리고 가독성이 떨어지는 문제가 있을 수 있습니다. 이러한 점이 축약된 문법의 양면의 칼같은 부분이라 주의하여 사용하시는 것이 좋습니다.
> >
> > 저 5개를 이해하고 있는 사람 입장에서는 예컨데 apply가 변수할당전 프로퍼티 초기화 목적을 암시하기 위해 썼나.. 라고 생각하며 보지만 그렇게 안썼으면 오히려 헷갈리죠..! (그렇게 안쓴다고 에러가 뜨는 것도 나이구요) 그래서 이 부분에 대한 이해와 또 실무에 사용시 팀내에 충분한 공유가 필요합니다 :slightly_smiling_face:

### 각 함수에 대한 정리 표

|                         | { .. this .. } | { .. it .. } |
| ----------------------- | -------------- | ------------ |
| return result of lambda | `with`, `run`  | `let`        |
| return receiver         | `apply`        | `also`       |

![](https://cdn-images-1.medium.com/max/1600/1*0FysXdiFRPqeVFGFFo6iVg.png)
[표의 출처](<https://medium.com/@kbm1378/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%84%B0%EB%94%94-16-lambda-with-receiver-e265044206f9>)



## 참고자료

* [코틀린 입문 스터디 **(16) Lambda with Receiver**](<https://medium.com/@kbm1378/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%84%B0%EB%94%94-16-lambda-with-receiver-e265044206f9>)