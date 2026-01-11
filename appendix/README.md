# 부록: 실무 패턴 (Appendix: Practical Patterns)

GoF 23개 디자인 패턴 외에 실무에서 자주 사용되는 패턴들을 정리한 부록입니다.

## 패턴 목록

### 🔄 동시성 패턴 (Concurrency)

멀티스레드 환경에서 안전하고 효율적인 코드를 작성하기 위한 패턴들입니다.

| 패턴 | 설명 | 난이도 |
|------|------|--------|
| [Double Checked Locking](./concurrency/DoubleCheckedLocking/README.md) | 지연 초기화 시 성능과 스레드 안전성 확보 | ⭐⭐ |
| [Object Pool](./concurrency/ObjectPool/README.md) | 비용이 큰 객체를 재사용하여 성능 향상 | ⭐⭐ |

### 💾 데이터 접근 패턴 (Data Access)

데이터베이스와 도메인 로직 사이의 깔끔한 분리를 위한 패턴들입니다.

| 패턴 | 설명 | 난이도 |
|------|------|--------|
| [Repository](./data-access/Repository/README.md) | 도메인 객체 컬렉션처럼 데이터 접근 추상화 | ⭐ |
| [DTO/DAO](./data-access/DtoDao/README.md) | 계층 간 데이터 전송과 데이터베이스 접근 분리 | ⭐ |
| [Unit of Work](./data-access/UnitOfWork/README.md) | 트랜잭션 내 변경사항 추적 및 일괄 커밋 | ⭐⭐ |
| [Specification](./data-access/Specification/README.md) | 비즈니스 규칙을 재사용 가능한 객체로 캡슐화 | ⭐⭐ |

### 🛡️ 복원력 패턴 (Resilience)

분산 시스템에서 장애에 대응하고 시스템 안정성을 확보하기 위한 패턴들입니다.

| 패턴 | 설명 | 난이도 |
|------|------|--------|
| [Circuit Breaker](./resilience/CircuitBreaker/README.md) | 장애 서비스 호출 차단으로 연쇄 장애 방지 | ⭐⭐ |
| [Retry](./resilience/Retry/README.md) | 일시적 장애 시 자동 재시도로 복원력 확보 | ⭐ |
| [Bulkhead](./resilience/Bulkhead/README.md) | 리소스 격리로 장애 전파 방지 | ⭐⭐ |

### 🌐 분산 시스템 패턴 (Distributed Systems)

마이크로서비스와 대규모 분산 시스템을 위한 아키텍처 패턴들입니다.

| 패턴 | 설명 | 난이도 |
|------|------|--------|
| [CQRS](./distributed/CQRS/README.md) | 명령(쓰기)과 조회(읽기) 모델 분리 | ⭐⭐⭐ |
| [Event Sourcing](./distributed/EventSourcing/README.md) | 상태 대신 이벤트를 저장하여 완전한 이력 보존 | ⭐⭐⭐ |
| [Saga](./distributed/Saga/README.md) | 분산 트랜잭션을 로컬 트랜잭션 체인으로 관리 | ⭐⭐⭐ |

### ✨ 코드 품질 패턴 (Code Quality)

더 안전하고 깔끔한 코드를 작성하기 위한 패턴들입니다.

| 패턴 | 설명 | 난이도 |
|------|------|--------|
| [Null Object](./quality/NullObject/README.md) | null 체크를 없애고 NullPointerException 방지 | ⭐ |

---

## 난이도 가이드

| 표시 | 설명 | 대상 |
|------|------|------|
| ⭐ | 초급 | 신입~1년차 |
| ⭐⭐ | 중급 | 2~4년차 |
| ⭐⭐⭐ | 상급 | 5년차 이상 |

---

## GoF 패턴과의 관계

| 부록 패턴 | 관련 GoF 패턴 |
|----------|--------------|
| Double Checked Locking | Singleton |
| Object Pool | Flyweight, Prototype |
| Repository | Facade, Factory |
| Null Object | Strategy, Proxy |
| Circuit Breaker | State, Proxy |
| CQRS | Command, Observer |
| Event Sourcing | Memento, Observer |
| Saga | Command, Chain of Responsibility |

---

## 학습 순서 추천

### 1단계: 필수 (모든 개발자)
1. **DTO/DAO** - 모든 프로젝트의 기본 구조
2. **Repository** - Spring Data JPA의 핵심
3. **Null Object** - 코드 안정성 향상

### 2단계: 권장 (중급 진입)
4. **Double Checked Locking** - Singleton 심화
5. **Retry** - 외부 API 연동 필수
6. **Circuit Breaker** - MSA 환경 필수

### 3단계: 심화 (아키텍트 지향)
7. **Object Pool** - 성능 최적화
8. **Unit of Work** - JPA 내부 이해
9. **Specification** - 복잡한 조회 조건
10. **Bulkhead** - 시스템 안정성

### 4단계: 고급 (시스템 설계)
11. **CQRS** - 대규모 시스템
12. **Event Sourcing** - 이벤트 기반 아키텍처
13. **Saga** - 분산 트랜잭션
