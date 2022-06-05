# 안정성 (Safety)

Kotlin은 다양한 설계 지원을 통해 애플리케이션의 잠재적인 오류를 줄여준다.

Kotlin은 안전한 언어이지만 정말 안전하게 사용하려면 개발자가 이를 뒷받침 해야 한다. 

이를 올바르게 사용하는 방법을 알아보자. 

- 목적 : 오류가 덜 발생하는 코드를 만드는 것!



## 1. 가변성을 제한하라

`var` 키워드를 사용하거나 mutable 객체를 사용하면 상태를 가지게 된다. (변할 수 있음)

변수가 상태를 가지게 되는 경우 해당요소의 동작은 사용방법과 그 이력(history, context)에 의존하게 된다.

시간의 변화에 따라 변하게 되는 상태를 적절하게 관리하는 것은 생각보다 꽤 어렵다 : 

1. 프로그램을 이해하고 디버깅하기 어려워진다 : 상태를 변경하는 부분들의 관계들을 이해해야 함. 상태 변경이 많아지면 추적하기 어려워짐. 이해하기 어렵고 이후 수정하기는 더 어려움. 예상하지 못한 동작이나 오류를 발생시키는 경우가 많아짐.
2. 가변성이 있으면 코드의 실행을 추론하기 어려워진다 : 시점에 따라 값이 달라질 수 있기 때문. 특정 시점에 확인한 값이 이후에 유지된다는 보장이 없음.
3. 동시성 이슈 : 변경이 일어나는 부분들이 많을 수록 충돌이 발생할 수 있음.
4. 테스트하기 어려움 : 모든 상태를 테스트해야하므로 테스트 경우의수가 크게 늘어남.
5. 상태 변경 notify 이슈 : 예를들어 sorted list의 경우 변경이 일어날 때 마다 다시 정렬해줘야 함.

=> 변경 가능한 부분들에 대한 일관성 유지 문제, 복잡성 증가와 관련된 문제가 매우 많음.

### Solution 1 : `val` 키워드를 사용하거나 Immutable 객체를 생성하기

#### 장점 : 

1. 쉬운 예측과 이해 : 한번 정의된 상태가 유지되므로 코드를 이해하기 쉬워진다.
2. 병렬처리에 안전함 : immutable 객체는 공유했을 때에도 충돌이 이뤄지지 않으니...
3. 쉬운 캐싱 : immutable 객체에 대한 참조는 변경되지 않으니..
4. 방어적인 복사본을 만들거나 깊은 복사를 따로 하지 않아도 된다.
5. 다른 객체를 만들 때 활용하기 좋다 : immutable 객체를 활용해서 새로운 mutable 객체를 만들거나...
6. set, map의 key로 사용할 수 있다 : mutable 객체는 사용 불가. 요소에 수정이 일어나면 요소를 찾을 수 없게 되어버리기 때문.



- 읽기 전용 프로퍼티 (`val`) : 값은 변경될 수 있으나 레퍼런스 자체를 변경할 수는 없어 동기화 문제등을 줄일 수 있다. 기본적으로 val을 더 많이 사용한다.

  > [참고] `val` 을 `var` 로 override  할 수 있다.
  >
  > ```kotlin
  > interface Element {
  >     val active: Boolean
  > }
  > 
  > class ActualElement {
  >     override var active: Boolean = false
  > }
  > ```

  - 완전히 변경할 필요가 없다면 `final`을 사용하도록 하자. 

- 가변 컬렉션과 읽기 전용 컬렉션 구분하기

  - 읽기 전용 컬렉션 : 실제로 immutable(불변)한 것은 아님. 외부적으로 immutable하게 보여지게 하여 안정성 획득. 

  - 컬렉션 다운캐스팅 : 읽기 전용 컬렉션을 가변 컬렉션으로 다운 캐스팅하는 행위 => 추상화를 무시하는 행위. 안전하지 않고 예측하지 못한 결과를 초래하게 됨.

    ```kotlin
    val list = listOf(1,2,3)
    
    // don't do that
    if (list is MutableList) {
        list.add(4)
    }
    
    // should do that
    val mutableList = list.toMutableList() // 가변리스트 버전 리턴 (기존 list는 읽기전용 컬렉션으로 유지)
    mutableList.add(4)
    ```

- `data` 클래스의 copy 메서드 활용

  ```kotlin
  data class User(
  	val name: String,
      val surname: String,
  )
  
  var user = User("Yena", "Kim")
  user = user.copy(surname = "Lee")
  print(user) // User(name=Yena, surname=Lee)
  ```

  

### Solution 2 : 변경 가능한 지점을 최소화하기 

상태를 변경할 수 있는 불필요한 방법들은 만들지 않아야 한다. (가변성 제한...)

`var`, `mutable` 객체.. 

mutable 프로퍼티에 읽기전용 컬렉션을 넣어 사용하라. 이렇게하면 세터를 사용하면 되고 이를 private 화 시킬 수 있기도 하다.

```Kotlin
var names by Delegates.observable(listOf<String>()) { _, old, new -> println("Changed! $old to $new") }
names += "aaa" // Changed! [] to ["aaa"]
names += "bbb" // Changed! ["aaa"] to ["aaa", "bbb"]

var announcements = listOf<Announcement>()
	private set

// worst : 변경될 수 있는 두 지점에 대한 동기화를 모두 구현해 줘야 함 (var, mutableList...)
var list = mutableListOf<Int>()
```

### Solution 3 : 변경 가능한 지점을 노출하지 말기

상태를 나타내는 mutable 객체를 외부에 노출하는 것은 굉장히 위험하다.  외부에서 수정이 가능해져버리기 때문...

```kotlin
class UserRepository {
    private val storedUsers: MutableMap<Int, String> = mutableMapOf()
    
    fun loadAll(): MutableMap<Int, String> { return storedUsers }
}

val userRepository = UserRepository()
val storedUsers = userRepository.loadAll()
storedUsers[4] = "Yena" // ?@?@?@!! 외부에서 수정 가능해져버림
```

1. 방어적 복제 사용하기

   ```Kotlin
   class UserHolder {
       private val user: MutableUser()
       
       fun get(): MutableUser {
           return user.copy()
       }
   }
   ```

2. 가변성 제한하기 : 읽기 전용 타입으로 업캐스트

   ```kotlin
   class UserRepository {
       private val storedUsers: MutableMap<Int, String> = mutableMapOf()
       
       fun loadAll(): Map<Int, String> { return storedUsers }
   }
   ```

### 정리

- `var` 보다는 `val`을 사용하자
- mutable 프로퍼티 보다는 immutable 프로퍼티를 사용하자
- mutable 객체와 클래스 보다는 immutable 객체와 클래스를 사용하자.
- 변경이 필요한 대상을 만들어야 한다면 immutable data 클래스로 만들고 copy를 활용하자
- 컬렉션에 상태를 저장해야 한다면, 읽기전용 컬렉션을 사용하자
- 변경 지점을 적절하게 설계하고 불필요한 변경 지점은 만들지 않도록 하자
- mutable 객체를 외부에 노출하지 않는 것이 좋다



## 2. 변수의 스코프를 최소화하라

kotlin scope functions