# 캐싱 패턴 (Caching Patterns)

캐싱 패턴은 데이터 접근 성능을 향상시키고 시스템 부하를 줄이기 위한 패턴입니다. 적절한 캐싱 전략은 응답 시간을 크게 개선할 수 있습니다.

## 패턴 목록

| 패턴 | 설명 | 난이도 | Spring 활용 |
|------|------|--------|-------------|
| [Cache-Aside](./CacheAside) | 캐시 미스 시 DB 조회 후 캐시 저장 | ⭐ | `@Cacheable` |
| [Read-Through](./ReadThrough) | 캐시가 DB 접근을 담당 | ⭐⭐ | CacheLoader |
| [Write-Through](./WriteThrough) | DB와 캐시 동시 업데이트 | ⭐⭐ | `@CachePut` |
| [Write-Behind](./WriteBehind) | 캐시 먼저, 비동기 DB 저장 | ⭐⭐ | `@Async` + Queue |

## 핵심 개념

### 캐시 히트 vs 캐시 미스
- **Cache Hit**: 캐시에 데이터가 존재하여 빠른 응답
- **Cache Miss**: 캐시에 없어 원본 저장소에서 조회

### 캐시 무효화 전략
- **TTL (Time To Live)**: 시간 기반 자동 만료
- **Event-Driven**: 데이터 변경 이벤트로 무효화
- **Manual**: 명시적 삭제 (`@CacheEvict`)

## 패턴 선택 가이드

### 읽기 중심 워크로드
→ **Cache-Aside** 또는 **Read-Through** 사용

### 쓰기와 일관성이 중요한 경우
→ **Write-Through** 사용

### 높은 쓰기 성능이 필요한 경우
→ **Write-Behind** 사용 (일관성 트레이드오프)

## 관련 GoF 패턴

| GoF 패턴 | 관계 |
|----------|------|
| [Proxy](../../structural/Proxy) | 캐시는 원본 데이터의 프록시 역할 |
| [Flyweight](../../structural/Flyweight) | 캐시된 객체 공유로 메모리 절약 |
