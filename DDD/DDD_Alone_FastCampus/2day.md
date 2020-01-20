## Spring

### Anemic Domain Model

'Service에 비즈니스 로직을 넣어야 한다'고 생각하며 코드를 짜다보면 이런 문제를 마주하게 된다.

* 빈약한 도메인 모델
* 모델에 기능이 없다. (getter/setter 등은 기능이 아니니깐)
* 과도한 서비스 레이어의 사용을 부추기게 됨
  * 뚱뚱한 서비스 클래스가 나타난다~~
* 실제로 이 구조를 서버사이드 아키텍처로 선보이기도 했었다.

```java
public class Box {
    private int height;
    private int width;

    public int getHeight() {
        return height;
    }

    public void setHeight(final int height) {
        this.height = height;
    }

    public int getWidth() {
        return width;
    }

    public void setWidth(final int width) {
        this.width = width;
    }
}
```

> 실제로 우리 프로젝트에서 개발을 위와 같은 구조로 구현하면서 늘 고민했던 문제... 그래서 작년 중반쯤 부터 조금씩 모델에 도메인 비즈니스 로직을 넣고 있었음.

## Big Service Layer

- 이런 형태의 거대한 서비스 레이어(big service layer) 형태는 객체지향의 설계원칙에 맞지 않을 뿐더러 도메인로직을 여러 곳에 산재하게 만들 뿐더러 코드의 중복과 오브젝트의 재활용성을 극히 떨어뜨리게 한다.
- 초기 스프링의 예제나 스프링 개발자들에 의해서 쓰여진 스프링 서적의 샘플 코드도 이런 구조를 그대로 사용했다는 것은 이런 모델이 얼마나 자연스럽게 개발자들에게 받아들여지고 사용되어져 왔는지 짐작하게 해준다.



## Application Layer의 구현

* Domain Model을 모든 계층에 사용하는 것 : Domain Model Everywhere
* DTO와 Domain Model을 나눠서 사용 : Pure Domain Model

### DTO (Data Transfer Object)

데이터 구조를 담는 객체(사실은 구조체)

> "Value Object와는 다르다." - 마틴 파울러
>
> 완전 데이터와 겟터셋터만 있는 것 (기존에 우리가 vo로 쓰던 그 구조)

### 1) Domain Model Everywhere

도메인 모델을 모든 계층에서 사용하는 것

* 'DTO는 안티패턴이다!!!!' 라고 생각하는 사람들이 주로 쓰는 모델
* 프로젝트 초반에 매우 편하다

### 2) Pure Domain Model

클라이언트와 도메인 모델 간의 분리 (DTO를 만들어서 Domain과 분리)

* 애플리케이션 내부의 요구사항 : Domain
* 애플리케이션 외부의 요구사항 : DTO

* 이렇게 만들다보면 엄청 많은 DTO가 생길 것임
  * 항상 DTO를 만드는 것은 실용적이지 않음
  * DTO 중에 Value Object로 바꿀만한 것이 뭐가 있을지 고민해야 한다!

```java
public class PostUpdateDto {
    String title;
    String body;

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getBody() {
        return body;
    }

    public void setBody(String body) {
        this.body = body;
    }
}

public class Post {
    Long id;
    String title;
    String body;
    
    public void update(String title, String body) {
        this.title = title;
        this.body = body;
    }
}
```

* PostUpdateDto 를 Value Object로 바꿀수는 없을까?

  * ```java
    // ValueObject
    public class Content {
      String title;
      String body;
    }
    
    // Entity
    public class Post {
    	Long id;
      Content content;
      public void update(Content content) {
            this.content = content;
      }
    }
    ```

  * 음? 그러면 Value Object 가 외부에 노출되는 것 아닌가요?!

    * 넵 그렇습니다. 이거슨 트레이드 오프....어떤 구조가 본인들에게 더 나을지 생각해보고 고르시오! 장점만 있는 것은 없음!!!

### 인터페이스는 어느 시점에 생성할ㄲㅏ?

외부와 소통할 때는 인터페이스가 나와야 한다 - 강사님 주관

그러나 응용 계층에서 꼭 인터페이스가 나올 필요는 없다. 트레이드 오프를 잘 따져서 쓰레드 홀드를 찾자. 아래 링크에는 관련 그래프도 나옴ㅋㅋ

[안정된 의존관계 원칙과 안정된 추상화 원칙에 대하여 - 우아한형제들 기술 블로그](http://woowabros.github.io/study/2018/03/05/sdp-sap.html)



## 의존성 역전의 원칙

DIP 관련 이야기... 예전에 개념잡을 때 봤던 포스팅을 다시 찾아보자!

## 육각형, 양파, 클린 아키텍처

- 육각형 아키텍처(hexagonal architecture), 양파 아키텍처(onion architecture) 및 클린 아키텍처(clean architecture)의 기본 규칙

![](https://herbertograca.files.wordpress.com/2018/11/100-explicit-architecture-svg.png)

![](https://blog.coderifleman.com/images/2017/the-clean-architecture/the-clean-architecture.jpg)

https://raindrop.dooray.com/share/posts/-Fny78YURpKKPjMUgpaD6w