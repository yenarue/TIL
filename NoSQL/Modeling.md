NoSQL/Modeling
=====


| RDBMS | NoSQL |
|-------|-------|
| 개체 모델 지향 | 쿼리 결과 지향 |
| 테이블 설계 -> 쿼리 | 도메인 모델 -> 쿼리 결과 유추 -> 테이블 정의 |
| 정규화 | 비정규화 |
|       | Document 단위로 조회하므로 IO부담이 큼. => IO 호출을 최소화하는 설계 필요 |

* 읽을때 Join하는 것이 아니라 작성할때 Join한다. 즉, 조회시 Join이 아니라 설계시 부터 Join이 되어있어야 한다는 뜻.

# 모델링을 하기 전에 파악해야 할 특성
## DeNormalization (비정규화)
비정규화 = 데이터 중복 허용.
* 쿼리의 효율성을 위해 데이터를 정규화 하지 않고 의도적으로 중복된 데이터를 저장한다.

## Secondary Index (보조 인덱스)

# 참고자료
* [NoSQL 데이터 모델링 개념](http://flowarc.tistory.com/entry/NoSQL-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%AA%A8%EB%8D%B8%EB%A7%81-%EA%B0%9C%EB%85%90)
* [NoSQL 데이터 모델링 개요 - 모델링 절차 및 사고 흐름이 나열되어 있어 이해하기 편함](http://interconnection.tistory.com/58?category=623648)
* [NoSQL 데이터 모델링 패턴](http://interconnection.tistory.com/59?category=623648)
* [MongoDB - NoSQL 모델링 강의 (2~3강)](https://www.youtube.com/watch?v=Dvi_TsBMFJo)
* [VELOPERT - [MongoDB] 강좌 1편: 소개, 설치 및 데이터 모델링](https://velopert.com/436)