# 복원력 (Resilience) 패턴

복원력 패턴은 분산 시스템에서 장애 상황을 감지하고 대응하여 시스템의 안정성을 유지하기 위한 패턴입니다. 외부 서비스 장애가 전체 시스템으로 전파되는 것을 방지하고, 장애로부터 빠르게 복구할 수 있도록 합니다.

## 패턴 목록

| 패턴 | 설명 | 난이도 | Spring 활용 |
|------|------|--------|-------------|
| [Circuit Breaker](./CircuitBreaker) | 장애 서비스 호출을 차단하여 연쇄 장애 방지 | 중 | Resilience4j @CircuitBreaker |
| [Retry](./Retry) | 일시적 장애에 대한 자동 재시도 | 하 | Spring Retry @Retryable |
| [Bulkhead](./Bulkhead) | 리소스를 격리하여 장애 영향 범위 제한 | 중 | Resilience4j @Bulkhead |

## 핵심 개념

### 장애 격리 (Fault Isolation)
한 부분의 장애가 전체 시스템에 영향을 미치지 않도록 격리합니다.

### 빠른 실패 (Fail Fast)
장애 상황을 빠르게 감지하고 불필요한 대기 시간을 줄입니다.

### 우아한 저하 (Graceful Degradation)
일부 기능이 실패해도 시스템의 핵심 기능은 유지합니다.

## 패턴 선택 가이드

### Circuit Breaker를 선택할 때
- 외부 서비스 호출 시 장애 전파를 방지하고 싶을 때
- 반복적인 실패 시 빠르게 실패하고 싶을 때
- Fallback 로직을 통해 대안을 제공하고 싶을 때

### Retry를 선택할 때
- 네트워크 일시 장애 등 재시도로 해결될 수 있는 상황
- 멱등성이 보장된 작업에서 일시적 실패를 처리할 때
- 백오프 전략으로 서버 부하를 조절하고 싶을 때

### Bulkhead를 선택할 때
- 리소스 소비를 제한하여 시스템을 보호하고 싶을 때
- 느린 서비스가 다른 서비스에 영향을 주지 않게 하고 싶을 때
- 동시 요청 수나 스레드 수를 제한하고 싶을 때

## 패턴 조합

실제 운영 환경에서는 이러한 패턴들을 조합하여 사용합니다:

```
요청 → Bulkhead (격리) → Retry (재시도) → Circuit Breaker (차단) → 외부 서비스
```

## 관련 GoF 패턴

| GoF 패턴 | 관계 |
|----------|------|
| [Proxy](../../structural/Proxy) | Circuit Breaker는 Proxy 패턴으로 서비스 호출을 감싸 구현 |
| [State](../../behavioral/State) | Circuit Breaker의 상태 전이(Closed→Open→Half-Open)는 State 패턴 활용 |
| [Strategy](../../behavioral/Strategy) | Retry의 백오프 전략은 Strategy 패턴으로 교체 가능 |
