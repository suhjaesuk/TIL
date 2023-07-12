# Redis 정의

- Remote Dictionary Server
- Storage: 데이터 저장소 (데이터 관점)
- Database: 전통적인 DBMS의 역할을 수행 (영속성 관점)
- Middleware: 어플리케이션이 이용할 수 있는 유용한 기능을 제공하는 소프트웨어

# Redis의 활용

- 서버를 개발하며 Redis를 사용하지 않는 기업을 찾기 어려움
- Session Store
- Cache
- Limit Rater
- Job Queue

# In-Memory DB

## DB, DBMS란?

- 데이터를 읽고 쓸 수 있는 기능을 제공하는 소프트웨어
- 어플리케이션이 데이터 저장을 간단히 처리할 수 있도록 해줌
- 관심사의 분리, 계층화

## In-Memory DB

- 데이터를 디스크에 저장하지 않음
- 휘발성인 RAM에 저장
- 빠른 속도

## Memory와 Disk의 속도 차이

|  | Read | Write | 비교 |
| --- | --- | --- | --- |
| HDD | 130 MB/s | 120 MB/s |  |
| SSD | 500 MB/s | 450 MB/s | HDD보다 3~4배 빠름 |
| RAM | 20,000 MB/s | 20,000 MB/s | SSD보다 40배 빠름 |

## 빠른 속도와 휘발성의 절충

- 용도에 맞게 DB와 Redis를 혼합해서 사용
- 주로 캐시 저장소로 이용
- Redis의 영속성 확보 (백업 등)

# Key-Value Store

## 데이터 저장소의 구조

- 프로그램 언어에서의 데이터 구조 (Array, List, Map…)
- DB의 데이터 모델 관점에서의 구조 (네트워크 모델, 계층형 모델 (Tree), 관계형 모델…)

## Key-Value Store

- 특정 값을 Key로 해서 그와 연관된 데이터를 Value로 저장 (Map과 같음)
- 가장 단순한 데이터 저장 방식
- 단순한 만큼 빠르고 성능이 좋음

## Key-Value 구조의 장점과 단점

### 장점

- 단순성에서 오는 쉬운 구현과 사용성
- Hash를 이용해 값을 바로 읽으므로 속도가 빠름 (추가 연산이 필요 없음)
- 분산 환경에서의 수평적 확장성

### 단점

- Key를 통해서만 값을 읽을 수 있음
- 범위 검색 등의 복잡한 질의가 불가능

## Key-Value Store의 활용

- 언어의 자료구조 (Java의 HashMap 등)
- NoSQL DB
- 단순한 구조의 데이터로 높은 성능과 확장성이 필요할 때 사용

# NoSQL

## Redis는 DBMS인가?

- 데이터를 다루는 인터페이스를 제공하므로 DBMS의 성격이 있음
- 기본적으로 영속성을 위한 DB는 아님
- 영속성을 지원 (백업)
- DBMS보다는 빠른 캐시의 성격으로 대표됨

## DBMS로서의 Redis

- Redis의 영속화 수단을 이용해 DBMS처럼 이용
- 일반 RDB 수준의 안정성을 얻기 위해선 속도를 희생해야 함

## NoSQL

- Key-Value Store
- 다양한 자료구조를 지원한다는 점에서 차별화 됨
- 원하는 수준의 영속성을 구성할 수 있음
- In-Memory 솔루션이라는 점에서 오는 특징을 활용할 때 가장 효율적임