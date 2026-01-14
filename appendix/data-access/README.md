# 데이터 접근 (Data Access) 패턴

데이터 접근 패턴은 애플리케이션과 데이터 저장소 사이의 상호작용을 추상화하고 관리하기 위한 패턴입니다. 비즈니스 로직과 데이터 접근 로직을 분리하여 유지보수성과 테스트 용이성을 높입니다.

## 패턴 목록

| 패턴 | 설명 | 난이도 | Spring 활용 |
|------|------|--------|-------------|
| [Repository](./Repository) | 데이터 접근 로직을 컬렉션처럼 추상화 | 하 | Spring Data JPA |
| [DTO/DAO](./DtoDao) | 계층 간 데이터 전송과 접근 분리 | 하 | @Entity, MapStruct |
| [Unit of Work](./UnitOfWork) | 트랜잭션 내 변경사항을 추적하고 일괄 처리 | 중 | @Transactional, EntityManager |
| [Specification](./Specification) | 비즈니스 규칙을 재사용 가능한 객체로 캡슐화 | 중 | JPA Specification |

## 핵심 개념

### 계층 분리 (Layer Separation)
프레젠테이션, 비즈니스, 데이터 접근 계층을 명확히 분리합니다.

### 영속성 무지 (Persistence Ignorance)
도메인 모델이 특정 데이터 저장 기술에 의존하지 않도록 합니다.

### 트랜잭션 관리 (Transaction Management)
데이터 일관성을 보장하기 위한 작업 단위를 관리합니다.

## 패턴 선택 가이드

### Repository를 선택할 때
- 도메인 객체에 대한 CRUD 작업을 추상화하고 싶을 때
- 데이터 접근 로직을 중앙화하고 싶을 때
- 테스트 시 데이터 계층을 쉽게 모킹하고 싶을 때

### DTO/DAO를 선택할 때
- 계층 간 데이터 전송 객체가 필요할 때
- 엔티티를 직접 노출하지 않고 API 응답을 구성할 때
- 여러 엔티티의 데이터를 조합한 뷰가 필요할 때

### Unit of Work를 선택할 때
- 여러 엔티티의 변경을 하나의 트랜잭션으로 처리할 때
- 변경사항 추적과 지연 저장이 필요할 때
- 동시성 제어가 중요한 시스템에서

### Specification을 선택할 때
- 복잡한 검색 조건을 동적으로 조합해야 할 때
- 비즈니스 규칙을 재사용 가능하게 만들고 싶을 때
- 쿼리 로직을 도메인 언어로 표현하고 싶을 때

## 관련 GoF 패턴

| GoF 패턴 | 관계 |
|----------|------|
| [Strategy](../../behavioral/Strategy) | Specification은 Strategy 패턴의 특수한 형태 |
| [Facade](../../structural/Facade) | Repository는 데이터 접근의 단순화된 인터페이스 제공 |
| [Composite](../../structural/Composite) | Specification의 조합은 Composite 패턴 활용 |
