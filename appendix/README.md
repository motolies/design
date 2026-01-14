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

### 📦 캐싱 패턴 (Caching)

데이터 캐싱으로 성능을 향상시키기 위한 패턴들입니다.

| 패턴 | 설명 | 난이도 |
|------|------|--------|
| [Cache-Aside](./caching/CacheAside/README.md) | 캐시 미스 시 DB 조회 후 캐시에 저장 | ⭐ |
| [Read-Through](./caching/ReadThrough/README.md) | 캐시가 DB 조회를 대신 처리 | ⭐⭐ |
| [Write-Through](./caching/WriteThrough/README.md) | DB와 캐시를 동시에 업데이트 | ⭐⭐ |
| [Write-Behind](./caching/WriteBehind/README.md) | 캐시 먼저 쓰고 DB는 비동기 저장 | ⭐⭐ |

### 🔐 API 보안 패턴 (API Security)

API 보호와 트래픽 관리를 위한 패턴들입니다.

| 패턴 | 설명 | 난이도 |
|------|------|--------|
| [Token Bucket](./api-security/TokenBucket/README.md) | 토큰 기반 요청 속도 제한 | ⭐⭐ |
| [Leaky Bucket](./api-security/LeakyBucket/README.md) | 고정 속도로 요청 처리 | ⭐⭐ |
| [Idempotency](./api-security/Idempotency/README.md) | 동일 요청의 중복 처리 방지 | ⭐⭐ |
| [BFF](./api-security/BFF/README.md) | 클라이언트별 전용 백엔드 | ⭐⭐ |

### 📨 메시징 패턴 (Messaging)

비동기 메시지 기반 통신을 위한 패턴들입니다.

| 패턴 | 설명 | 난이도 |
|------|------|--------|
| [Outbox](./messaging/Outbox/README.md) | 이벤트를 DB에 저장 후 발행 보장 | ⭐⭐ |
| [Inbox](./messaging/Inbox/README.md) | 수신 메시지 중복 처리 방지 | ⭐⭐ |
| [Competing Consumers](./messaging/CompetingConsumers/README.md) | 다중 소비자로 메시지 병렬 처리 | ⭐⭐ |

### 🚀 배포 패턴 (Deployment)

안전하고 효율적인 배포를 위한 패턴들입니다.

| 패턴 | 설명 | 난이도 |
|------|------|--------|
| [Blue-Green](./deployment/BlueGreen/README.md) | 두 환경 준비 후 즉시 전환 | ⭐⭐ |
| [Canary](./deployment/Canary/README.md) | 일부 사용자에게 먼저 배포 후 확대 | ⭐⭐ |
| [Rolling](./deployment/Rolling/README.md) | 서버를 순차적으로 교체 | ⭐ |
| [Feature Toggle](./deployment/FeatureToggle/README.md) | 런타임에 기능 활성화/비활성화 | ⭐⭐ |

### 🔗 일관성 패턴 (Consistency)

분산 시스템에서 데이터 일관성을 보장하기 위한 패턴들입니다.

| 패턴 | 설명 | 난이도 |
|------|------|--------|
| [Two-Phase Commit](./consistency/TwoPhaseCommit/README.md) | 분산 트랜잭션의 원자성 보장 | ⭐⭐⭐ |
| [TCC](./consistency/TCC/README.md) | Try-Confirm-Cancel 보상 트랜잭션 | ⭐⭐⭐ |
| [Eventual Consistency](./consistency/EventualConsistency/README.md) | 최종적 일관성 보장 | ⭐⭐⭐ |

### 📈 확장성 패턴 (Scaling)

대용량 데이터와 트래픽 처리를 위한 패턴들입니다.

| 패턴 | 설명 | 난이도 |
|------|------|--------|
| [Materialized View](./scaling/MaterializedView/README.md) | 쿼리 결과를 미리 계산하여 저장 | ⭐⭐ |
| [Sharding](./scaling/Sharding/README.md) | 데이터를 여러 DB에 수평 분할 | ⭐⭐⭐ |

---

## 난이도 가이드

| 표시 | 설명 | 대상 |
|------|------|------|
| ⭐ | 초급 | 신입~1년차 |
| ⭐⭐ | 중급 | 2~4년차 |
| ⭐⭐⭐ | 상급 | 5년차 이상 |

---

## 카테고리별 요약

| 카테고리 | 패턴 수 | 핵심 키워드 |
|----------|---------|------------|
| 동시성 | 2개 | 스레드 안전, 성능 |
| 데이터 접근 | 4개 | DB 추상화, 트랜잭션 |
| 복원력 | 3개 | 장애 대응, 안정성 |
| 분산 시스템 | 3개 | MSA, 이벤트 기반 |
| 코드 품질 | 1개 | null 안전성 |
| 캐싱 | 4개 | 성능 최적화 |
| API 보안 | 4개 | Rate Limiting, 멱등성 |
| 메시징 | 3개 | 비동기, 이벤트 발행 |
| 배포 | 4개 | 무중단, CI/CD |
| 일관성 | 3개 | 분산 트랜잭션 |
| 확장성 | 2개 | 대용량 처리 |
| **합계** | **33개** | |

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
| Cache-Aside | Proxy |
| BFF | Facade, Adapter |
| Feature Toggle | Strategy |

---

## 학습 순서 추천

### 1단계: 필수 (모든 개발자)
1. **DTO/DAO** - 모든 프로젝트의 기본 구조
2. **Repository** - Spring Data JPA의 핵심
3. **Null Object** - 코드 안정성 향상
4. **Cache-Aside** - 기본 캐싱 전략

### 2단계: 권장 (중급 진입)
5. **Double Checked Locking** - Singleton 심화
6. **Retry** - 외부 API 연동 필수
7. **Circuit Breaker** - MSA 환경 필수
8. **Idempotency** - API 안정성
9. **Rolling** - 기본 배포 전략

### 3단계: 심화 (시니어)
10. **Object Pool** - 성능 최적화
11. **Unit of Work** - JPA 내부 이해
12. **Specification** - 복잡한 조회 조건
13. **Bulkhead** - 시스템 안정성
14. **Outbox/Inbox** - 메시징 신뢰성
15. **Blue-Green/Canary** - 안전한 배포

### 4단계: 고급 (아키텍트)
16. **CQRS** - 대규모 시스템
17. **Event Sourcing** - 이벤트 기반 아키텍처
18. **Saga** - 분산 트랜잭션
19. **Two-Phase Commit/TCC** - 일관성 보장
20. **Sharding** - 대용량 데이터 처리
