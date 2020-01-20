나홀로 DDD Workshop
====

###### 패스트캠퍼스 2days Workshop

[강의자료](https://raindrop.dooray.com/share/posts/-Fny78YURpKKPjMUgpaD6w)

* [PokeAPI](https://pokeapi.co/)를 이용하여 간단한 포켓몬 게임을 만들어볼거임

## 개요

- 중요한 것은 패턴 자체가 아니라 **비즈니스 문제에 맞게 코드를 구성하고 동일한 비즈니스 용어(유비쿼터스 언어)를 사용**하는 것.

* 소프트웨어의 존재 가치에 대하여

### AS-IS

* 데이터 종속적인 애플리케이션
* 데이터베이스 위주의 개발
  * 데이터베이스 모델링을 더 중요시
  * 데이터베이스 스키마가 정해지면 모든 코드가 정해짐
* 모델링과 개발과의 불일치 발생
  * 불필요한 해석 과정 야기
  * 잘못된 소프트웨어를 만드는 원인
* 개발자들은 자신의 개발력을 훈련할 수 있는 정량적 문제를 좋아한다.
  * 답이 존재하고 확실하고 템플릿적 훈련이 가능하니까ㅋㅋㅋ

## TO-BE : 도메인 주도 설계 (DDD, Domain-Driven Design)

도메인 모델의 적용범위를 구현까지 확장한다. => 도메인 지식을 구현 코드에 반영!

전략적인 설계인 동시에 [전술적인 설계](https://en.wikipedia.org/wiki/Domain-driven_design#Building_blocks)를 적용하는 모델링 도구

* [DDD-Lite](https://ehsanghanbari.com/blog/post/41/whats-the-ddd-lite) : 전술적 설계만 사용하는 것
  * 이번 워크샵에서는 전술적 설계만 일단 진행할 것임. 전략적 설계도 중요하지만 이 것을 이해하는 데에는 좀 더 깊은 고찰과 많은 시간이 필요함.

* 참고자료 : http://redutan.github.io/2018/04/27/IDDD-chapter01

### 도메인 (Domain)

소프트웨어로 해결하고자 하는 문제 영역

> *언어, OOP, Effective Java 등의 기술공부 외에 도메인 지식도 잘 쌓고 있는가?*
>
> - 도메인 지식이 곧 **적절한** 확장성을 가진 코드로 이어진다.

### 도메인 모델

특정 도메인을 **개념적으로 표현**한 것

- 도메인 모델을 사용하면 여러 관계자들이 동일한 모습으로 도메인을 이해하고 도메인 지식을 공유하는 데 도움이 된다.
- 모델의 각 구성 요소는 특정 도메인을 한정할 때 비로소 의미가 완전해지기 때문에, 각 하위 도메인마다 별도로 모델을 만들어야 한다.

## 도메인 주도 설계 아키텍처

![](https://www.petrikainulainen.net/wp-content/uploads/spring-web-app-architecture.png)

* Service Layer에 비즈니스 로직을 몰아둠으로서 생기는 문제들이 많다
  * 로직의 중복
  * 모델과의 디펜던시가 강해짐
  * 코드가 분산되어 응집력이 낮아짐
* Domain Layer로 응집력을 높히자

### 4 Layers

![](https://dzone.com/storage/temp/4277164-layered-architecture-overview.png)

#### Presentation Layer (or UI Layer)

사용자의 요청을 받아 Application Layer에 전달 & Application Layer의 처리 결과를 다시 사용자에게 보여주는 역할

#### Application Layer

시스템이 사용자에게 제공해야 할 기능을 구현

* 로직을 직접 처리한다기보다는 Domain Layer에 로직 수행을 위임
* 도메인 객체 간의 실행 흐름을 제어
* 트랜잭션 처리 (커밋 단위)
* 도메인 영역에서 발생한 이벤트 핸들링

```java
public Result doSomeFunc(SomeReq req) {
    // 1. 리포지터리에서 애그리거트를 구한다.
    SomeAgg agg = someAggRepository.findById(req.getId());
    checkNull(agg);
    // 2. 애그리거트의 도메인 기능을 실행한다.
    agg.doFunc(req.getValue());
    // 3. 결과를 리턴한다.
    return createSuccessResult(agg);
}
```

#### Domain Layer

도메인 모델 구현

#### Infrastructure Layer

구현 기술에 대한 것을 다룸



## 도메인 주도 설계의 기본 요소

### Entity & Value Object

![](https://docs.microsoft.com/ko-kr/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/media/image9.png)

* 도출한 모델은 크게 Entity와 Value Object로 구분된다.
* 데이터와 함께 도메인 기능을 제공

> 자바빈즈 스타일 get, set NoNo

### 1. Entity

#### VO와의 차이점 

* 각 Entity마다 고유한 식별자를 가진다

### 도메인 모델에 set 메서드 지양하기

대표적인 기능적 메소드 set 메서드

1. 도메인의 핵심 개념이나 의도를 코드에서 사라지게 한다.

다양한 매개변수가 마구 들어올 수 있는 set 메서드는 가독성이 떨어지고 그저 기능적인 메소드임

```java
changeShippingInfo() vs setShippingInfo()
completePayment() vs setOrderState()
```

위와 같이 어떤 상태를 set한다는 기능적 단위보다는 그 상태로 변경되는 행동을 정의

> (주의) set메소드가 무조건 안좋다는 뜻이 아님. 남용을 하지 말라는 것임.

2. 생성자의 의미를 퇴색시킨다

생성 시점에 필요한 Attribute들까지도 set 메소드에 의존하게 만들 수 있는 여지가 있음.

### 2. Value Object

Immutable

primitive value Wrapper Class 와 비슷한 개념으로 생각하면 좋음.

모든 Attribute들이 식별자였어!

메모리를 차지한다는 두려움 vs 불변 객체가 주는 이점

> 응용프로그램을 위한 모던 언어들이 대부분 불변을 디폴트 타입으로 가져가는 이유를 잘 생각해보면 답이 나온다.

애플리케이션 내부의 요구사항 (like `String`)

#### DTO (Data Transfer Obejct)

애플리케이션 외부의 요구사항

* Entity를 그대로 넘겨주는 것이 아니라 DTO로 변환해서 넘겨주는 이유?
  * 넘어가는 과정 속에서 값의 상태가 변하거나 잃게되는 사태를 방지하기 위해
  * 이 관점에서 Value Object는 불변이기 때문에 전혀 문제가 되지 않음. 매우 자유롭게 왔다갔다 한다.

### 3. Aggregate

관련 도메인 객체들을 하나로 묶은 군집

* Entity 와 Value Object 가 같은 Lifecycle를 가지도록 관리할 수 있음
* Aggregate Root
* 객체는 단 하나의 군집에만 속한다.
* 캡슐화 굿

![](https://docs.microsoft.com/ko-kr/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/media/image10.png)

* Order Aggregate
  * Order
    * ID : Global ID
  * OrderItem : 정말 이 아이가 Entity 여야만 하는가를 고민해봐야함!
    * 부모인 Order가 식별됨에 따라 이 아이템은 단순 Index로 식별될 수 있는게 아닐까?
    * ID : Local ID
    * 로컬 ID의 경우는 위와 같은 사례로 인해 굳이 엔티티 일 필요가 없을 수 있다. (무조건 엔티티가 아닌건 아님)
    * 잘 생각해보고 정해야 한다
  * Address

#### Aggreate Root

#### Aggregate Reference

* 직접참조

* **간접참조 : 오용을 방지하기 위해 간접참조를 쓰는 것이 더 낫다.**

  * 직접참조의 디펜던시를 끊어야 한다.

* Spring & JDBC는 직접참조를 방지할 수 있게끔 유도한다.

  > mongoDB도 그런 것 같아

### 4. Repository

엔티티나 밸류가 요구사항에서 도출되는 도메인 모델이라면 **리포지터리는 구현을 위한 도메인 모델**

** 주의 : 테이블 단위가 아니다*

VO와의 차이점 : VO는 좀 더 DB와 디펜던시가 강하고 그를 표현하는 오브젝트라면 Repository는 애플리케이션 내부에서 사용하는 데이터를 표현하는 오브젝트. 컬렉션의 의미에 더 가깝기 때문에 해외 레퍼런스를 확인해보면 인스턴스 이름을 복수형으로 쓰기도 한다.

> JPA Entity 와 DDD Entity가 개념적으로는 동일할 수 있어도 용례는 다를 수 있다.
>
> Entity라는 것의 개념 자체는 식별자를 가지는 객체인데... 예를 들어, 식별자로서 id와 seq를 개념적으로 나눠서 사용하는 개발팀의 경우에는 seq를 사용하기 위해 실제로는 엔티티가 아닐지라도 `@Entity` 가 붙을 수 있는 것.

### 5. Domain Service

특정 로직이 1가지 Aggregate에서 해결되는 것이 아닌 여러가지의 Aggregate가 필요한 기능의 경우 어떻게 처리를 해야할까? 

* **상태 없이 기능만** 가지고 있다.

#### Application Service vs Domain Service

아래의 특징을 가지면 도메인 서비스라고 분류하는 것이 좋다. 

* 특정 기능이 응용 서비스인지 도메인 서비스인지 감을 잡기 어려울 때는 **해당 로직이 애그리거트의 상태를 변경**하거나 **애그리거트의 상태 값을 계산**하는지 검사해 보면 된다. => 그러하면 도메인 서비스
* 유효성 체크 로직
* 버저닝 여부 (or 위키에 관리를 해야겠다는 생각이 드는가)
* 대외비인가(!??!ㅋㅋㅋㅋ)

이러한 특징을 가지고 있어서인지 도메인 서비스는 대부분 절차지향적인 코드로 표현되게 되더라...

* 좀 더 핵심 비즈니스 로직을 다른 개발자들이 쉽게 확인하기 위함

### 6. Factory

어떤 객체를 생성하는 일이 복잡하다면 팩토리를 이용하여 캡슐화 할 수 있다.

### 7. Module

=패키지

Java9에서의 멀티모듈프로젝트에서의 모듈과는 다르다! 모듈과는!...

* 계층 우선 vs 기능 우선

|                                                              |                                              |
| ------------------------------------------------------------ | -------------------------------------------- |
| +-- application<br />+-- domain <br />+-- infra <br />+-- ui | +-- item <br />+-- pokemon <br />+-- trainer |



## Modeling

### 유비쿼터스 언어

* 도메인에서 사용하는 용어를 코드에 반영해야 한다
  * 그렇지 않으면 개발자에게 코드의 의미를 해석해야 하는 부담을 준다.
* 코드는 가장 최전방에 있다. 정합성이 언제나 지켜지는 곳. 
  * 그렇기 때문에 유비쿼터스 언어야말로 코드레벨에서 꼭 지켜져야 하는 것
  * 기획서, 회의록은 정합성을 보장할 수 없음.
  * 그런데 코드는 개발자만 보잖아? => 그래서 개발자들끼리 이해하는 용어로만 구성되는 경우가 많음 => 이러면 안된다!!!!
* 용어가 정의 될 때마다 **용어 사전**에 이를 기록하고 명확하게 정의 함으로써 추후 또는 다른 사람들도 공통된 언어를 사용할 수 있도록 한다.
  * 용어 사전의 필요성/중요성은 팀 전체에 전파되고 인지되어 있어야 한다!
* 하나의 유비쿼터스 언어는 해당하는 Bounded Context 내에서만 의미가 있다.
  * 맥락이 달라지만 같은 단어도 다른 의미가 될 수 있다.

### 효과적인 모델링

* 사용자와 개발자는 동일한 언어로 이야기 하는가?
* 해당 언어가 애플리케이션에서 수행해야 할 내용에 관한 논의를 이끌어갈 만큼 풍성한가?
* 표현해야 할 것을 더 쉽게 말하는 방법을 찾아낸 다음 그러한 새로운 아이디어를 다이어그램과 코드에 적용한다!

> 1) Routing Service에 출발지, 목적지, 도착 시간을 전달하면 화물이 멈춰야 할 지점을 찾고, 그것을 데이터베이스에 삽입한다.
>
> 2) 출발지, 목적지 등등 이것들을 모두 Routing Service에 넣으면 필요한 것이 모두 담긴 Itinerary를 돌려받는다.
>
> 3) Routing Service는 Route Specification을 만족하는 Itinerary를 찾는다.
>
> 세부 사항은 용어 사전에 기록하며 업데이트 한다. 즉, 용어에 대한 참조 방향을 만들어서 동일한 문장/단어를 사용하게 되는 것!

### 하나의 팀, 하나의 언어

> 영업팀에게는 너무 추상적이라서요.

> 영업팀은 객체를 이해하지 못해요.

> 영업팀이 쓰는 용어로 된 요구 사항을 만들어야 해요.

- 영업팀도 해당 모델을 이해하지 못한다면 모델이 뭔가 잘못된 것이다.

### Bounded Context

구분되는 경계를 갖는 컨텍스트를 DDD에서는 BOUNDED CONTEXT라고 부른다.

![](https://martinfowler.com/bliki/images/boundedContext/sketch.png)

* 같은 용어일지라도 도메인에 따라 의미가 달라질 수 있다.
  * 즉, 한 개의 모델로 모든 하위 도메인을 표현하려는 시도는 올바르지 않을 수 있음. 애초에 불가능하기도 함.
* 유비쿼터스 언어는 각 Context에서만 유효한 것.
* 각 Context를 정말 잘 나누면 각 Context가 MSA의 요소들과 매칭되는 것
  * 그래서 `잘 나눈 모노리스, 10 MSA 안부럽다!` 라는 말이 나오는 것
* 지금 당장 `Customer` 가 동일한 필드를 가지고 동일해보여도 비즈니스가 진행되다보며는 달라질 가능성이 있다는 것을 명심하고 고려해야 하는 것. (각 컨텍스트가 다르니 비즈니스는 충분히 다르게 진행될 수 있음)
* 용어의 "이름" 보다는 "의미"가 달라지는 지점이 발견되었다!!! => 그 지점에서 컨텍스트를 나눠야 함
* 하나의 Bounded Context는 하나의 팀에 할당되는 것이 낫다.
  * 용어의 혼란.... 
  * 하나의 팀에서는 여러개의 Bounded Context를 가질 수 있지만 하나의 Bounded Context는 꼭 하나의 팀에 할당되어야 한다
  * 각 Bounded Context는 각각의 개발 환경을 가질 수 있다.

### Context Map

상호 교류하는 시스템의 목록을 제공한다. 팀 내 의사소통의 촉매 역할을 한다.

![](https://www.safaribooksonline.com/library/view/implementing-domain-driven-design/9780133039900/graphics/03fig01.jpg)

U : UpStream / D : DownStream

### 프로젝트와 조직 관계

- 파트너십(Partnership) : 두 CONTEXT가 하나의 트랜잭션으로 묶여 있다.
- 공유 커널(Shared kernel) : 상호 의존하는 공유 모델을 관리한다.
  - `common` , `general`, `core` 등과 같은 패키지...ㅋㅋㅋ
- 고객-공급자(Customer-Supplier Development) : 업스트림(서버:공급자), 다운스트림(클라이언트:고객)로 단방향으로 의존한다.
- 순응주의자(Conformist) : 업스트림(서버)이 모든 것을 제어한다.
- 오픈 호스트 서비스(Open Host Service) : REST/API, RPC, Socket
- 분리된 방법(Seprate Ways) : 의존 없음
- 큰 진흙공(Big ball of mud) : 안티 패턴

### DDD vs OOP

* OOP는 상속이나 재활용성을 위해서 공통된 데이터를 공유하는 것을 중요시 한다.
* DDD는 도메인 분리를 중요시 한다.
  * DDD도 OOP에 근간하지만 OOP 원칙을 정면으로 위배하는 패턴들도 많다.

> *유용한 소프트웨어를 개발하고 싶다면 도메인에 귀를 기울여라. - Eric Evans*



## 그 외

### [이벤트 스토밍](https://en.wikipedia.org/wiki/Event_storming)

모든 이해관계자들이 모여 포스트잇을 붙여가며 우리가 만드는게 무엇인지 구체화하는 과정/워크샵

포스트잇을 그룹핑하면서 컨텍스트를 정의한다.

철저하게 비즈니스에만 초점을 맞춘다. (데이터베이스, 코드 등의 용어를 쓰지 않기)

* 도메인지식에 대한 상향 평준화가 목적

* Tools : miro
* [Event Storming](https://www.eventstorming.com/)
* [[SlideShare] Event Storming](https://www.slideshare.net/odsji/event-storming-128038708)







## 강의들으면서 서칭한 자료들

* Mongo db DDD 관점 설계 관련 : https://stackoverflow.com/questions/44287874/ddd-how-to-store-aggregates-in-nosql-databases

* 이규원님 DDD 관련 아티클
  * https://gyuwon.github.io/blog/2019/12/07/is-your-system-based-on-ddd.html
  * 