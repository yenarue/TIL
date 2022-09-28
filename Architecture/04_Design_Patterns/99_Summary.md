Summary
===



# Design Patterns

**Designing for Change**

특정한 방향으로 확장될 것이라는 예상이 될 때 변화에 적응하기 쉬운 디자인을 제공하기 위함

변화에 적응하기 쉽도록 대비책을 만든다.

## 종류

### Strategy Pattern

### Observer Pattern

### Template Method Pattern

### Iterator Pattern

### State Pattern

- 다른 패턴들과의 비교 

  - State Pattern 
    - State Machine에서 State가 전이되므로 상황(context)에 따라 State가 계속 변함
    - Encapsulate state-based behavior and delegate behavior to the current state

  - Strategy Pattern
    - 객체가 사용하는 전략이 하나로 유지됨
    - Encapsulate interchangeable behaviors and use delegation to decide which behavior to use

  - Template Method Pattern
    - Subclasses decide how to implement steps in an algorithm

### Mediator Pattern

- 다른 패턴들과의 비교
  - Observer
    - 재사용이 쉽다
    - 커뮤니케이션 플로우를 이해하는 입장에서는 어렵다
  - Mediator : Observer의 반대
    - 재사용이 어렵다
    - 커뮤니케이션 플로우를 이해하기 쉽다

### Factory Method & Abstract Factory Pattern

- Factory Method로 쉽게 시작하고 확장해라
  - Product Family가 필요하다 -> Abstract Factory
  - 주어진(존재하는) Product를 복사해야한다 -> Prototype
  - Product를 생성하는 과정을 복사하고싶다 -> Builder





## 카테고라이징

### Creating an Object by Specifying a Class Explicitly

명시적으로 특정 클래스의 오브젝트를 생성하는 경우
=> new를 명시적으로 하지않고도 만들 수 있도록 하기

- **Design Patterns : Abstract Factory, Factory Method, Builder**

### Dependence on Hardware and Software Platform

- platform을 바꿀 때 영향을 미칠 수 있음
- native platform이 바뀌었을 때 유지하기 어려움

=> 변화가 생기더라도 abstract  계층에는 영향이 가지 않도록 처리

- **Design Patrerns : Bridge, Abstract Factory**

### Alogorithmic Dependencies

특정 알고리즘에 디펜던시가 생기는 경우, 알고리즘이 변했을 때 오브젝트가 변할 수 있음.

=> 알고리즘의 변화는 독립적으로 이루어지도록 해야 함

- **Design Patterns : Builder, Iterator, Strategy, Template Method**
  - Builder는 생성패턴이기도 하지만 알고리즘의 부품을 만들어 디펜던시를 해결해줌
  - Iterator : iteration order 가 변경되어도 사용할 수 있도록 (SRP)
  - Strategy : 알고리즘 통채로 구현
  - Template Method : 알고리즘의 일부를 subclass에서 구현
    - Factory Method 패턴은 Template Method를 사용하고 있음 (특별한 형태)

### Extending Functionality by Subclassing

Subclass를 이용해서 기능을 확장하는 경우, 세분화 및 변종들이 만들어지며 컴파일타임에 모든 클래스가 존재해야하므로 폭발현상이 일어남

- subclassing can lead to explosion of subclasses. (조합만큼 늘어남..)

=> object composition and delegation provide flexible alternatives

- **Design Patterns : Bridge, Composite, Decorator, Strategy**
  - Bridge : 개념적인 계층의 클래스와 구현에 대한 클래스를 한꺼번에 만드니 폭발하니까
  - Decorator : 커피와 커피에 넣을 수 있는 콤비네이션 클래스를 다 만드면 폭발하니까 
    => 재료를 클래스화해서 런타임에 사용자가 원하는 조합을 만들어 쓸 수 있도록
    => 기존 클래스를 변경하지 않아도 됨
  - Stategy : 처음엔 상속으로 다 해결하려하다가 폭발했었지
  - Composite : 
    => Component 형태로 공통 부모를 가지고 있으니 굿

### Inability to Alter Classes Conveniently

기존 클래스를 고치기 어려운 상황 (나에게 코드가 없거나 잘 돌아가고 있던 기존 클래스를 고치고 싶지 않을때)

- **Design Patterns : Adapter, Decorator**
  - Adapter
    - Class Adapter
    - Object Adapter
  - Decorator : 기존에 있던 클래스를 변경하지 않고 그 클래스를 둘러싸는 조합 형태로 구현

### 디자인패턴의 한계

- 디자인패턴을 적용한다고해서 전반적인 아키텍처가 좋아지는 건 아니다
  - 디자인패턴은 그냥 마이크로 아키텍처이기 때문
- 창의성이 여전히 필요하다
  - 모든 설계 결정이 패턴으로 커버되지는 않는다
  - 일부패턴이 비슷해보일 수 있음. 하지만 의도와 문제가 다름.
    - composition & delegation -> State, Strategt, Bridge...
- 패턴을 적용하는 연습이 필요하다
  - 다양한 문제와 솔루션을 접해보자



## 잘못된 패턴 적용

- Overenthusism
  - 패턴을 오용하거나 남용하거나
    - 패턴을 적용하여 더 복잡해질 수 있다
  - 딱 필요한만큼만 유연해야 한다.
  - 가장 간단한 디자인이 가장 좋은 디자인이다

- Overly dense application





abcde, acbde, acdbe, cdabe, cadbe, cabde
