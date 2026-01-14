# 분산 시스템 (Distributed Systems) 패턴

분산 시스템 패턴은 여러 서비스가 협력하는 마이크로서비스 아키텍처에서 데이터 일관성과 트랜잭션을 관리하기 위한 패턴입니다. 분산 환경의 복잡성을 해결하고 확장 가능한 시스템을 구축하는 데 도움을 줍니다.

## 패턴 목록

| 패턴 | 설명 | 난이도 | Spring 활용 |
|------|------|--------|-------------|
| [CQRS](./CQRS) | 명령(쓰기)과 조회(읽기) 모델을 분리 | 상 | 별도 Command/Query Repository |
| [Event Sourcing](./EventSourcing) | 상태 대신 이벤트 시퀀스로 데이터 저장 | 상 | Spring ApplicationEvent, Axon |
| [Saga](./Saga) | 분산 트랜잭션을 보상 트랜잭션으로 관리 | 상 | Spring State Machine |

## 핵심 개념

### 최종 일관성 (Eventual Consistency)
분산 환경에서 즉각적 일관성 대신 시간이 지나면 일관성이 보장되는 모델입니다.

### 이벤트 기반 아키텍처 (Event-Driven Architecture)
서비스 간 통신을 이벤트를 통해 비동기적으로 처리합니다.

### 보상 트랜잭션 (Compensating Transaction)
분산 환경에서 롤백이 불가능할 때 역작업을 통해 일관성을 복구합니다.

## 패턴 선택 가이드

### CQRS를 선택할 때
- 읽기와 쓰기의 요구사항이 매우 다를 때
- 읽기 성능을 극대화해야 할 때
- 복잡한 도메인에서 쓰기 모델을 단순화하고 싶을 때

### Event Sourcing을 선택할 때
- 모든 상태 변경 이력이 필요할 때 (감사, 분석)
- 시간 여행(특정 시점 상태 복원)이 필요할 때
- 이벤트 기반 아키텍처를 구현할 때

### Saga를 선택할 때
- 여러 서비스에 걸친 비즈니스 트랜잭션이 필요할 때
- 2PC(Two-Phase Commit)가 적합하지 않은 환경에서
- 장기 실행 트랜잭션을 관리해야 할 때

## 패턴 조합

CQRS와 Event Sourcing은 자주 함께 사용됩니다:

```
명령 → Event Sourcing (이벤트 저장) → 이벤트 발행 → CQRS 읽기 모델 업데이트
                                          ↓
                                    Saga (다른 서비스 연동)
```

## 주의사항

이 패턴들은 복잡도가 높습니다:
- **시작은 단순하게**: 모놀리식에서 시작하여 필요할 때 도입
- **명확한 필요성**: 분산 트랜잭션이 정말 필요한지 검토
- **운영 복잡도**: 디버깅, 모니터링, 장애 대응이 복잡해짐

## 관련 GoF 패턴

| GoF 패턴 | 관계 |
|----------|------|
| [Observer](../../behavioral/Observer) | Event Sourcing과 CQRS의 이벤트 발행/구독 메커니즘 |
| [Command](../../behavioral/Command) | CQRS의 Command는 Command 패턴의 확장 |
| [State](../../behavioral/State) | Saga의 상태 관리는 State 패턴 활용 가능 |
