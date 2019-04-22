Inline Functions
====

`inline` 키워드로 함수를 선언하면 해당 함수는 인라인 함수가 된다. 인라인 함수는 호출된 로직에, 함수의 본문이 inline 되는 함수를 뜻한다. 실제로 함수를 호출하는 것이 아니라 함수의 본문 자체가 호출된 로직에 그대로 붙여넣어진다고 생각하면 된다. (바이트코드로 컴파일 시 해당 본문의 바이트코드가 붙여넣어진다)

## 인라인 함수의 종류

Kotlin에서 제공하는 기본 인라인 함수들은 다음과 같다. (아래 함수들의 실제 구현을 살펴보면 `inline` 키워드와 함께 구현되어 있음을 확인할 수 있다.)

### `run`

코드 블럭(람다)를 실행시키고 마지막 표현식을 결과 값으로서 반환하는 함수이다.

```kotlin
val foo = run {
  println("Calculating foo...")
  "foo"
}
```

### `let`

인수(argument)가 non-null 임을 보장하는 함수이다. 만약 Nullable argument를 받게되면 컴파일 에러가 발생한다.

```kotlin
if (email != null) sendEmailTo(email)
email?.let { e -> sendEmailTo(e) }
email?.let { sendEmailTo(it) }
```

```kotlin
val user = session.user
if (user is FacebookUser) {
  println(user.accountId)
}

// ===>
(session.user as? FacebookUser)?.let { println(it.accountId) }
```

### `takeIf`

해당 조건문(predicate)을 만족하면 해당 개체(object)를 반환하고 만족하지 않으면 `null` 을 반환한다.

```kotlin
issue.takeIf { it.status == FIXED }
age.takeIf { it > 18 }
issues.filter { it.status == OPEN }
		.takeIf(List<Issue>::isNotEmpty)
		?.let { println("There're some open issues") }
```

### `takeUnless`

`takeIf` 와는 반대로 해당 조건문을 만족하지 않으면 해당 개체를 반환하고, 아니면 `null` 을 반환한다.

### `repeat`

해당 구문을 특정 횟수 반복한다.

```kotlin
repeat(10) {
  println("Welcome!")
}
```

## 인라인 함수의 강점 (The power of inline)

inline vs non-line

### 1. 오버헤드가 줄어든다

함수를 인라인으로 구현하지 않고 일반 함수로 구현한 경우에는 해당 함수 호출 시, Function Call Interface나 해당 Lambda에 대응되는 익명 클래스 개체(anonymous class object)가 생성되기 때문에 그에 따른 오버헤드가 생길 수 있다.

```kotlin
fun myRun (f: () -> Unit) = f()

fun main() {
  myRun { println("Hey") }	// class ExampleKt$main$1
}
```

하지만 함수를 인라인으로 구현하게되면 함수 호출의 형태가 아니라, 호출된 논리 흐름의 일부로 동작할 수 있게 된다.

```kotlin
// 실제로 구현되어있는 인라인 함수 run
inline fun <R> run(block: () -> R): R = block()

val name = "Kotlin"
run { println("Hi, $name!") }

// 위 코드가 바이트코드 단에서는 아래와 같이 코드가 생성된다. (람다 구문이 인라인 코드로 들어간다)
val name = "Kotlin"
println("Hi, $name!")
```

### 2. 간결성 + 가독성 향상

`takeUnless` , `takeIf`  함수와 같이 반복되는 구문을 줄여 조건문에만 집중할 수 있도록 함으로써 간결성과 가독성을 증가시킬 수 있다

```kotlin
// 실제로 구현되어있는 인라인 함수 takeUnless
inline fun <T> T.takeUnless(predicate: (T) -> Boolean): T? =
	if (!predicate(this)) this else null
val result = number.takeUnless { it > 10 }

// 위 코드가 바이트코드 단에서는 아래와 같이 코드가 생성된다. (람다 구문이 인라인 코드로 들어간다)
val result = if (!(number > 10)) number else null
```

`synchronised`  함수의 경우에도 보일러 플레이트 구문은 줄이고 성능은 유지하는 효과를 볼 수 있다.

```kotlin
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
  lock.lock()
  try {
    return action()
  } finally {
		lock.unlock()
  }
}

fun foo(lock: Lock) {
  synchronized(lock) {
    println("Action")
  }
}

// 위 코드가 바이트코드 단에서는 아래와 같이 코드가 생성된다. (람다 구문이 인라인 코드로 들어간다)
fun foo(lock:Lock) {
  lock.lock()
  try {
    println("Action")
  } finally {
    lock.unlock()
  }
}
```

참고) `withLock` 함수는 Lock 변수의 확장함수이자 인라인 함수로서 해당 변수로 Lock을 걸고 synchronized 시키는 함수이다.

```kotlin 
inline fun <T> Lock.withLock(action: () -> T): T {
  lock()
  try {
    return action()
  } finally {
    unlock()
  }
}
```

`use` 함수의 경우 인스턴스 관리를 퍼포먼스 오버헤드 없이 해줌으로서 자원 관리를 용이하게 할 수 있다.

```kotlin
// Java
String readFirstLine(String path) {
	try (BufferedReader br = new BufferedReader(new FileReader(path))) {
  	return br.readLine();
  } catch (IOException e) {
    // something...
  }
}

// Kotlin
fun readFirstLine(path: String): String {
  BufferReader(FileReader(path)).use { br -> return br.readLine() }
}
```

### 주의사항

기본적으로 꼭 필요한 경우가 아니라면 함수를 인라인 함수로 정의하지 말아야한다.

#### 1. `@kotlin.internal.InlineOnly`

Java에서 Kotlin의 인라인 함수를 호출하게되면 호출은 가능하지만 인라인 처리가 되지는 않는다. (ex. `filter`)

`@InlineOnly` 어노테이션을 이용하여 Java 코드에서 해당 인라인 함수에 접근할 수 없도록 할 수 있다. 즉, Kotlin 에서만 사용 가능하도록 가시성을 제한하는 어노테이션이다.

#### 2. 인라인 함수의 본문의 크기는 작게 유지하자

인라인 함수는 결과적으로 함수의 본문 전체가 그대로 붙여넣어지기 때문에 프로젝트의 크기가 커질 수 있다. 그렇기 때문에, 혹시 인라인 함수를 구현하게 된다면, 작은 크기의 함수만 인라인 함수로 변경하는 것이 좋을 것이다.

