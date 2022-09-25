Overview
====

- Book
  - Head First Design Patterns
  - Agile Software Development by Robert C. Martin
  - Design Patterns: Elements of Reusable Object-Oriented Software by GoF



# Design Pattern

## Benefit of patterns

- Reusable design : 더욱 유연한 디자인. 재사용 가능한 디자인. 확장이 쉬운 디자인.
- Communication language : 개발자들끼리 커뮤니케이션 원활해짐.



## Three Part Rule

"A **Solution** to a **problem** in a **context**"

- context : the situation in which the pattern applies. This should be a **recurring** situation
- problem : context내에서 성취하고자하는 목표
- solution : **what you’re after**: a general design that anyone can apply which resolves the goal and the set of constraints



## Category of GoF Patterns

|            | Creational                                                   | Structural                                                   | Behavioral                                                   |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Class**  | **Factory Method**                                           | **Adapter**                                                  | Interpreter<br />**Template**                                |
| **Object** | **Abstract Factory**<br />**Builder**<br />Prototype<br />**Singleton** | **Adapter**<br />**Bridge**<br />**Composite**<br />**Decorator**<br />Facade<br />Flyweight<br />Proxy | CoR<br />Command<br />**Iterator**<br />**Mediator**<br />Memento<br />**Observer**<br />**State**<br />**Strategy**<br />Visitor |



## Key Features

- 패턴의 이름 (Pattern name)
- 의도 (Intent)
- 문제 (Problem)
- 해결방법 (Solution)
- 패턴에 참여하고있는 객체들 (Participants and collaborators)
- 단점 (Consequences)
- 구현방법(예시) (Implementation)
- 일반적인 구조 (Generic structure) : UML 등을 활용하여 표현



## Levels of Pattern

- Architectural Pattern : 시스템 전체가 대상 
  - 예 : MVC, MVVM 등...
- **Design Pattern** : subsystem, component과 그들의 relationship가 대상
  - 전체 시스템을 대상으로하지 않음. micro-architecture가 대상
  - 예 : Observer Pattern 등...
- Coding Pattern : 프로그래밍 언어에 디펜던시가 있는 로우레벨의 패턴
  - 예 : Coding conventions



# Quiz

- 다음 중 잘못된 설명은 ?

1. 설계 작업에서 설계패턴의 이름을 통해 보다 명확하게 의사전달을 할 수 있다.
2. 설계패턴은 설계 시 자주 반복되는 문제에 대한 해결책을 담고 있다.
3. **아키텍처 패턴은 주로 컴포넌트 내부의 설계에 사용되며, 설계패턴은 시스템의 전체 구조를 결정하는데 사용된다.**
4. 코딩 패턴은 특정한 프로그래밍 언어의 특징에 종속적일 수 있다.



- 다음 중 생성(creational) 패턴이 아닌 것은 ?

1. Factory Method Pattern
2. Abstract Factory Pattern
3. Singleton Pattern
4. **State Pattern** => 행위(Behavioral) 패턴
