UP - Elaboration - OOD
====

In iterative development, **a transition from requirements/OOA to design/implementation occur in each iteration**

- Sequence Diagram
- Class Diagram



# Logical Architecture & UML Package DIagrams





## Static Object Modeling

- Class Diagram
- Object Diagram (스냅샷이라 다이나믹이라고 볼 수도 있겠지만... 스태틱에 더 가까워보임)



## Dynamic Object Modeling

- Interaction diagrams (Sequence Diagram)
- Statechart Diagram
- Activity Diagram



---



# Quiz

- 다음의 Object Design에 대한 설명 중 올바르지 않은 것은?

  1. Static Models은 Class나 Package를 정의하는데 도움이 되며, Class Diagram, Package Diagram 등이 해당된다.
  2. Dynamic Model은 Method나 로직 등을 정의하는데 도움이 되며, Interaction Diagram, Statechart Diagram, Acitivity Diagram 등이 해당된다.
  3. **Class Diagram으로부터 Skeleton Code를 생성하고 구현 (Implementation) 을 시작하므로, Static Model이 가장 중요한 Object Model이다.** => Interactive한 부분들은  Dynamic Diagram도 봐야하기 때문에... 
  4. UP 기반의 OOAD에서는, 현 Iteration에서 개발할 Use Case에 대해서, Interaction Diagrams과 Class Diagram을 반복적 (Iterative) 이고 점증적 (Incremental) 으로 작성한다.

  

- UML Interaction Diagrams에 대한 다음의 설명 중 올바른 것은?

  1. **Interaction Diagram은 4가지 다이어그램에 대한 통칭이며, 실제로 그릴 수 있는 Diagram은 아니다.**
  2. Sequence Diagram은 Communication Diagram 보다 더 Expressive Power가 강력하다.
  3. Communication Diagram은 Sequence Diagram 보다 더 Expressive Power가 강력하다.
  4. Interaction Overview Diagram은 더 큰 시나리오를 하나의 Interaction Diagram 으로 그리는 방법이다 => 여러가지의 Interaction Diagram 종류를 엮어둔 것

  

- UML Object Model에는 Static Model과 Dynamic Model 두 종류가 있습니다. 각각을 대표하는 UML Diagram은 무엇인가요? 또 이 두 모델이 서로 어떤 관계를 가지면서 어떻게 사용되는지 설명하세요.
  - **State Model : Class Diagram**
  - **Dynamic Model : Interaction Diagram (Seqeunce Diagram)**
  - **관계 : Static한 Diagram과 Dynamic Digram이 서로 유기적인 관계를 가짐. Domain Model을 통해 Class Diagram을 그리고 UseCase 모델을 통해 Sequence Diagram을 그리고 반영하고...**
