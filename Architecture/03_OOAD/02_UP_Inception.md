UP - Inception
====



# Inception

Go/Stop을 결정하는 a short initial step : 

- 이 프로젝트에 어떤 비전과 비즈니스 케이스가 있는가?
- Feasible? 유용한가?
- 사는게 나은가? 만드는게 나은가?
- 비용은 어떻게 되는가?



Inception 단계에서 대부분의 Requirement를 찾기는 하지만 넓게 찾고 깊게는 찾지 않는다.

- Brief 레벨 정도로 대부분의 Requirement 를 찾는다 => 얕고 넓게
- Casual 레벨 부터는 Elaboration 단계에서 진행



## Artifacts

- Vision and Business Case
- **Use-Case Model**
- **Supplementary Specification**
- Glossary
- Risk List & Risk Management Plan
- Prototypes and proof-of-concepts
  - proof-of-concepts : 새로운 기술들을 적절히 써보는 것
- Iteration Plan
- Phase Plan & Software Development Plan
- **Development Case**



## UML

Inception 단계에서는 많은 수의 UML 다이어그램을 그릴 필요가 없다.

필요한 만큼만 그리면 된다.

Elaboration 단계부터 UML을 다양하게 그리게 된다.



# Evolutionary Requirements

## Requirements

**Capabilities** and **conditions** to which **the system must conform**

- Requirement analysis : 고객과 팀 멤버들이 정말로 무엇을 원하는지 찾고 커뮤니케이션하고 조직화한다.

- UP에서는 Requirement가 반복적으로 스킬풀하게 분석된다.
- 스킬풀 하다는 것의 의미는?
  - **고객과 함께** 유즈케이스를 작성한다.
  - 개발자와 고객이 포함된 **Requirement Workshop**을 진행한다. => Elaboration
  - 고객에게 각 이터레이션별로 **결과물의 데모**를 보여주고 피드백을 받는다.

### Requirements의 종류와 카테고리

- FURPS
  - 기능 
    - Functional
  - 비기능
    - Usability
    - Reliability
    - Performance
    - Supportability

## Requirement는 어떻게 조직화되는가..

- Use-case Model : Functional (behavioral) requirements (기능 요구사항)
- Supplementary Specification : all non-functional requirements (비기능 요구사항)
- Glossary : noteworthy terms를 정의한다
- Vision : 프로젝트의 거다란 아이디어를 빠르게 익힐 수 있는 짧은 실행가능한 오버뷰...
- Business Rules : 소프트웨어 프로젝트에서 요구사항과 정책을 정의...



### Use Cases

Inception 단계에서 시작해서 Elaboration 단계에서 수정되고 끝난다.



### Supplementary Specification

Captures and identifies **other kinds of requirements**, such as reports, documentation, packaging, supportability, licensing, and so forth

- URPS + quality attributes



# Quiz

- 다음 중 Inception 단계에서 수행되는 활동 또는 결과물이 아닌 것은?
  1. **Client, Architect 등 모든 과제 관련자들이 참가하는 Requirements Workshop을 개최한다.** => Elaboration Phase에서 진행한다
  2. 대부분의 Use-Cases가 Brief 포맷으로 작성된다.
  3. 대부분의 Architecturally Risky Requirements가 찾아진다.
  4. 필요한 경우 Technical proof-of-concept을 위한 Prototype을 만든다. => 보통은 Elaboration 단계에서 Prototype을 만들긴하지만 Inception 단계에서 proof-of-concept 를 만들어보고 판단하기도 한다.



- 다음의 요구사항 (Requirements) 에 대한 설명 중 올바르지 않은 것은 무엇인가요?

  1. 요구사항은 일반적으로 크게 기능 요구사항과 비기능 요구사항으로 구분할 수 있다.
  2. 비기능 요구사항은 시스템의 품질 (Quality) 에 큰 영향을 미치므로 Quality Attributes/Requirements 라고도 한다.
  3. **시스템의 SW Architecture에 영향을 크게 미치는 요구사항은 기능 요구사항이다.** => 비기능 요구사항도 SW Architecture에 영향을 미친다.
  4. UP에서 일반적으로 기능 요구사항은 Use-Case Model로, 비기능 요구사항은 Supplementary Specification으로 작성된다.

  

- Use-Case에 대한 다음의 설명 중 올바르지 않은 것은?
  1. Use-Case는 요구사항을 체계적으로 분석하는 방법이다.
  2. **Use-Case는 비(非) OOAD 개발 시에 사용하면 효과가 상당히 반감된다.**
  3. Use-Case는 모든 OOAD 개발방법론에서 사용되는 중요한 시작점 (Starting-Point) 중의 하나이다.
  4. Use-Case를 이용한 다양한 요구공학 (Requirements Engineering) 방법론이 있으나, UP에서는 가장 간단한 수준으로만 Use-Case를 활용한다.
