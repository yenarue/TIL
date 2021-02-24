# One Way Data Flow

이건 사실 대부분의 유저와 가까운 프레임워크(Web Frontend, Mobile Native(ios, android 등..), Native Apps(Window, Mac 등...))라면 동일한 맥락을 가지게 되는 데이터 흐름 관리법이다.

결국 협업 측면에서와 프로젝트 규모가 커졌을 때의 복잡성 및 예측불가능성을 최소화하기 위함이 가장 큰 목적이라 생각한다. 그래야 비즈니스 로직에 더욱 집중할 수 있게되고 다른 팀원과의 컨센서스 통일 및 모듈끼리의 커뮤니케이션이 용이해지기 때문이다.



관련하여 간단하게 잘 설명한 포스팅이 보이길래 일단 스크랩

https://dev.to/lukeshiru/one-way-data-flow-why-47fk

(중략)



## 왜 Two-way data flow는 안된다는 거죠?

짧게 이야기하자면, 유지 비용이 너무 크기 때문이다.

Two-way 예시에서 One-way 예시에서보다 `InputUsername` 및 `InputPassword`  에 대한 사용법이 더 간단한 것 처럼 보일 수 있겠지만 실제로는 Two-way 방식이 이러한 단순성을 대가로 하여 아래와 같은 이슈들을 불러일으키게 된다 : 

* `LoginPage` 의 상태가 여러 위치에서 업데이트 될 수 있는 상태다 => 상태의 변경을 추적하고 예측하기 어려워진다
* `InputUsername` 및 `InputPassword` 을 LoginPage 외에서 사용하기 어려워진다

