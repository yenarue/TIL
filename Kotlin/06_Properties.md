Properties
====

Kotlin에서는 클래스내에 정보를 실제로 담고있는 변수에 직접적으로 접근하는 것을 허용하지 않고 언제나 accessor(접근자)를 통해 간접적으로 접근하도록 한다. Java에서는 이를 문법적으로 강제하고 있지 않아 개발자가 직접 구현해주었어야 했지만 Kotlin에서는 언어적인 기능으로 **Properties** 를 추가하여 간결성을 향상시켰다.

## Accessor

Java로 데이터 클래스를 관리할 일이 있었다면 아래와 같은 코드를 굉장히 많이 짜봤을 것이다. 아래는 전형적인 데이터 클래스에서 field accessor (getter, setter)를 구현한 코드이다.

```java
// java
public class JavaClass {
  private int foo = 0;
  public int getFoo() {
    return foo;
  }
  public void setFoo(int foo) {
    this.foo = foo;
  }
}
```

이를 Kotlin으로 구현해보면 아래와 같다:

```kotlin
// Kotlin
class KotlinClass {
	var foo = 0
}
```

위의 Java 코드와 Kotlin 코드는 완벽히 동일한 코드이다. Kotlin 에서는 accessor 를 굳이 개발자가 직접 구현하지 않아도, *아래의 규칙에 따라*, 내부적으로 구현된다.

- `val` : getter
- `var` : getter + setter

예를 들어, `var contact: Contact` 의 경우에는 getter와 setter가 내부적으로 생성된다. 그 이후 `contact.address` 를 호출하면 getter가, `contact.address = "Korea"` 를 호출하면 setter 가 호출된다.

```kotlin
var contact: Contact
// java : contact.getAddress();
contact.address
// java : contact.setAddress("Korea");
contact.address = "Korea"
```

마치 Java의 유명한 라이브러리인 lombok 을 사용한 듯한 간결화된 코드를 사용할 수 있다.

### Custom Accessor

아래와 같이 프로퍼티를 field 없이(without backing field) 정의하여 getter나 setter를 커스텀할 수 있다. 이렇게 정의하면 불필요한 메모리 할당이 줄어들고 해당 정보가 필요한 경우에만 연산을 수행시킬 수 있어 효율성이 증가한다.

```kotlin
class Rectangle(val height: Int, val width: Int) {
  val isSquare: Boolean
  	get() {
    	return height == width
  	}
}

val rectagle = Rectangle(2, 3)
// 일반 프로퍼티처럼 바로 접근이 가능하다
rectagle.isSuare // false
```

### Lambda 와의 비교

Lambda 는 변수에 할당될 때 최초 한번만 실행된다. 변수에는 결과 값만 저장된다.

```kotlin
val fooLambda = run {
  println("fooLambda")
  123
}

fun main() {
	println("main start!")
	println("$fooLambda $fooLambda")
}

// fooLambda
// main start!
// 123 123
```

그에비해 getter의 경우에는 호출 시마다 새로 불린다.

```kotlin
val fooGetter: Int
	get() {
  	println("fooGetter")
    return 123
	}

fun main() {
  println("main start!")
  println("$fooGetter $fooGetter")
}

// main start!
// fooGetter
// fooGetter
// 123 123
```

### `field` : only use inside accessors

Kotlin에서는 accessor를 통해서만 field에 대한 접근이 가능하도록 하여 직접적으로 접근할 수 없게 하였다. 하지만 Accessor 내부에서는 `field` 라는 키워드로 해당 field에 직접 접근할 수 있다. 실제로 이를 Java 코드로 변환시켜보면 `this.state = value` 로 접근하고 있음을 확인할 수 있다.

```kotlin
var state = false
	set(value) {
  	field = value	// 여기서 field는 state를 뜻한다.
	}
```

### No backing field

경우에 따라서는 아예 backing field 없이 대리자(delegate)용도의 필드를 만들어 사용할 수도 있다.

```kotlin
class StateLogger {
  private var boolState = false
  
  // 여기서 state는 getter, setter에서 전혀 사용되고 있지 않음.
  // boolState의 상태에 따라 결과값이 달라지고, set시에도 boolState를 수정함 => state는 boolState의 공개된 대리자 역할을 한다
  var state: State
  	get() = if (boolState) ON else OFF
	  set(value: State) {
    	boolState = value == ON
  	}
}
```

### Visibility of accessors

accessors에 대해 아래와 같이 접근권한을 설정하면 해당 **클래스 외부에서는 해당 접근자(accessor)를 사용할 수 없게** 처리된다.

```kotlin
var counter: Int = 0
	private set
```


