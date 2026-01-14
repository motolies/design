# 확장성 패턴 (Scaling Patterns)

확장성 패턴은 대규모 시스템에서 성능과 용량을 확장하기 위한 패턴입니다. 읽기 성능 최적화, 데이터 분산 저장을 다룹니다.

## 패턴 목록

| 패턴 | 설명 | 난이도 | 구현 |
|------|------|--------|------|
| [Materialized View](./MaterializedView) | 쿼리 결과를 물리적으로 저장 | ⭐⭐ | DB View, CQRS 읽기 모델 |
| [Sharding](./Sharding) | 데이터를 수평 분할하여 분산 저장 | ⭐⭐⭐ | ShardingSphere, Vitess |

## 핵심 개념

### 수직 확장 vs 수평 확장
- **수직 확장 (Scale Up)**: 서버 성능 향상 (CPU, 메모리)
- **수평 확장 (Scale Out)**: 서버 수 증가

### 읽기 확장 vs 쓰기 확장
- **읽기 확장**: Read Replica, Materialized View
- **쓰기 확장**: Sharding, Partitioning

## 패턴 선택 가이드

### 복잡한 조회 성능 개선
→ **Materialized View**로 미리 계산된 결과 저장

### 대용량 데이터 처리
→ **Sharding**으로 데이터 분산

## Sharding 전략

| 전략 | 설명 | 장점 | 단점 |
|------|------|------|------|
| Range | 값 범위로 분할 | 범위 쿼리 효율적 | 핫스팟 가능 |
| Hash | 해시값으로 분할 | 균등 분배 | 범위 쿼리 어려움 |
| Directory | 매핑 테이블 사용 | 유연함 | 매핑 테이블 병목 |

## 관련 패턴

| 패턴 | 관계 |
|------|------|
| [CQRS](../distributed/CQRS) | Materialized View는 읽기 모델로 활용 |
| [Repository](../data-access/Repository) | Sharding은 Repository 내부에서 처리 |
