# 메시징 패턴 (Messaging Patterns)

메시징 패턴은 분산 시스템에서 신뢰성 있는 메시지 발행과 처리를 보장하기 위한 패턴입니다. 메시지 손실 방지, 중복 처리 방지, 처리량 증가를 다룹니다.

## 패턴 목록

| 패턴 | 설명 | 난이도 | Spring 활용 |
|------|------|--------|-------------|
| [Outbox](./Outbox) | 이벤트를 DB에 저장 후 발행 | ⭐⭐ | `@TransactionalEventListener` |
| [Inbox](./Inbox) | 수신 메시지 중복 처리 방지 | ⭐⭐ | Redis/DB 중복 체크 |
| [Competing Consumers](./CompetingConsumers) | 다중 소비자 메시지 분산 처리 | ⭐⭐ | Kafka Consumer Group |

## 핵심 개념

### 메시지 신뢰성
- **At-Most-Once**: 최대 1번 전달 (손실 가능)
- **At-Least-Once**: 최소 1번 전달 (중복 가능)
- **Exactly-Once**: 정확히 1번 전달 (Outbox + Inbox)

### 메시지 순서
- 전역 순서 vs 파티션 순서
- 순서 보장이 필요하면 파티션 키 설계 중요

## 패턴 선택 가이드

### 메시지 손실 방지 (발행 측)
→ **Outbox** 패턴으로 트랜잭션과 함께 저장

### 중복 처리 방지 (소비 측)
→ **Inbox** 패턴으로 멱등성 보장

### 처리량 증가
→ **Competing Consumers** 패턴으로 다중 소비자 배치

## 관련 GoF 패턴

| GoF 패턴 | 관계 |
|----------|------|
| [Observer](../../behavioral/Observer) | 메시징은 분산 Observer 패턴 |
| [Command](../../behavioral/Command) | 메시지는 명령 객체와 유사 |
