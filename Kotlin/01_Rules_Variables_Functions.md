문법
====
코틀린의 문법적 특징에 대해서 알아보자!

## Variables

### 종류

* `val` : value. **read-only** (assigned-once, not immutable). 

* `var` : variable. **mutable**

Kotlin에서는 변수의 디폴트 형태가 `val` 이다. 왜냐하면 '변경되는 것'보다 '변경되지 않을 것으로 보장되는 것'이 사이드 이펙트가 발생할 가능성이 적어 좀 더 안전하기 때문이다.

또한, 코드가 Functional style 에 가까워지도록 유도하기 때문이다. 코드 흐름에 디펜던시를 가지지 않고 순수해지기 때문에 병렬적인 분리가 가능해져 함수형에 가까워진다는 뜻이다. 매우 맞는말!

### 타입 추론 (Local type inference)

type specification is verbose! 문맥 흐름에 따라 타입이 결정나기 때문에 명쾌함!

```kotlin
val greeting = "Hi"	// 알아서 String으로 추론함
var number = 0		// 알아서 Int로 추론함
```

Kotlin은 정적 타입 언어이지만 타입 추론을 지원하여 동적 타입 언어의 이점(코드의 단순화)을 가져갈 수 있게 되었다. 하지만 결국엔 정적 타입 언어이기 때문에 그 추론된 변수에 다른 타입의 값을 재할당하면 컴파일 에러가 발생한다.

즉, 특정 변수의 타입이 문맥상 명쾌(clear)하지 않다는 생각이 든다면 꼭 타입을 명확하게 적어주는 것이 안전하다. 즉, 개발이 지속되면서 오해가 생기게 되면 혼란이 오고 사이드 이펙트가 생길 가능성이 있으므로 **타입을 명시하는 것을 더 권장**한다.

### 두가지 타입의 리스트(`List`)

* Mutable List : 변경가능한 리스트

```kotlin
val mutableList = mutableListOf("Java")
mutableList.add("Kotlin")
```

* Read-only List : 변경불가능한 읽기 전용의 리스트 [#](<https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/index.html>)

```kotlin
val readOnlyList = listOf("Java")
readOnlyList.add("Kotlin") // compile error! 읽기 전용 리스트라 add 메소드가 없음
```



## Functions

Parameter의 타입은 반드시 지정해주어야 한다.

```kotlin
// Block Body
fun max(a: Int, b: Int): Int = {
    return if (a > b) a else b
}

// Expression Body
fun max(a: Int, b: Int): Int = if (a > b) a else b

// Expression Body - 리턴값 타입추론 가능 (헷깔릴 것 같다면 비권장)
fun max(a: Int, b: Int) = if (a > b) a else b
```

### Unit

Java의 `void` 와 유사(완전히 동일한건 아닌걸까?). 이 경우는 거의 생략해서 쓴다고 한다. 

```kotlin
							// : Unit 이 생략 되어있다
fun displayMax(a: Int, b: Int) {
    println(max(a, b))
}
```

### Functions everywhere

* Top-level function

  ```kotlin
  fun topLevel() = 1
  ```

  자바에서는 해당 파일명을 클래스로 가지는 static function 으로 변환되기때문에 아래와 같이 호출 가능하다.

  ```kotlin
  // MyFile.kt
  package intro
  fun foo() = 0
  ```

  ```java
  // UsingFoo.java
  import intro.MyFileKt
  
  public class UsingFoo {
  	public static void main(String[] args) {
  		MyFileKt.foo();
  	}
  }
  ```

* Member function

  ```kotlin
  class A {
      fun member() = 2
  }
  ```

* Local function

  ```kotlin
  fun other() {
      fun local() = 3
  }
  ```


### @file:JvmName()

changes the JVM name of the class containing top-level functions

```kotlin
// MyFile.kt
@file:JvmName("Util")
package intro
fun foo() = 0
```

```java
// UsingFoo.java
import intro.MyFileKt
  
public class UsingFoo {
	public static void main(String[] args) {
  		Util.foo();
  	}
}
```

## Named & default arguments

가독성, 명시성 향상

```kotlin
                                    // default arguments
fun displaySeparator(character: Char = '*', size: Int = 10) {
    repeat(size) {
        print(character)
    }
}

displaySeparator('#', 5) 	// #####
displaySeparator() 			// **********
displaySeparator(3, "5") 	// 컴파일 되지 않음
// named arguments
displaySeparator(size = 5)	// *****
displaySeparator(size=3, character="5") // 555
```

Java에서는 위와 같은 usage를 가지도록 구현하려면 오버로딩을 해야한다. Kotlin 에서는 Name & Default Arguments 를 통해서 효율적으로 다양한 경우의 수를 만들어 낼 수 있다.

### @JvmOverloads

이 기능을 Java 에서도 사용하기 위해서는 아래와 같이 처리해주어야 한다.

```kotlin
@JvmOverloads
fun sum(a: Int = 0, b: Int = 0, c: Int = 0)
```

```java
// 나머지는 디폴트 값이 사용된다
sum(1);
```

