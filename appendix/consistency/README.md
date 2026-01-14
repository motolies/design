# 데이터 일관성 패턴 (Data Consistency Patterns)

데이터 일관성 패턴은 분산 시스템에서 여러 서비스 간 데이터 일관성을 보장하기 위한 패턴입니다. 트랜잭션 관리, 보상 처리, 최종 일관성을 다룹니다.

## 패턴 목록

| 패턴 | 설명 | 난이도 | 구현 |
|------|------|--------|------|
| [Two-Phase Commit](./TwoPhaseCommit) | 2단계 분산 트랜잭션 | ⭐⭐⭐ | JTA, Atomikos |
| [TCC](./TCC) | Try-Confirm-Cancel 보상 트랜잭션 | ⭐⭐⭐ | Seata, 수동 구현 |
| [Eventual Consistency](./EventualConsistency) | 시간 경과 후 일관성 보장 | ⭐⭐⭐ | Event + Saga |

## 핵심 개념

### 일관성 수준
- **강한 일관성 (Strong)**: 모든 노드가 동시에 동일한 데이터
- **최종 일관성 (Eventual)**: 시간이 지나면 동일한 데이터

### CAP 정리
분산 시스템에서 Consistency, Availability, Partition Tolerance 중 2개만 선택 가능

## 패턴 선택 가이드

### 강한 일관성 필수 (레거시, 단일 DB)
→ **2PC** 사용 (성능 트레이드오프)

### MSA에서 장기 트랜잭션
→ **TCC** 또는 **Saga** 사용

### MSA 기본 전략
→ **Eventual Consistency** + Outbox/Inbox

## 패턴 비교

| 패턴 | 일관성 | 성능 | 복잡도 | 사용 시점 |
|------|--------|------|--------|----------|
| 2PC | 강함 | 낮음 | 중간 | 레거시, 단일 DB |
| TCC | 중간 | 중간 | 높음 | MSA 장기 트랜잭션 |
| Saga | 최종 | 높음 | 중간 | MSA 기본 |
| Eventual | 최종 | 높음 | 낮음 | 이벤트 기반 |

## 관련 패턴

| 패턴 | 관계 |
|------|------|
| [Saga](../distributed/Saga) | Eventual Consistency 구현 방법 |
| [Outbox](../messaging/Outbox) | 신뢰성 있는 이벤트 발행 |
