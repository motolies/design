# API 보안 패턴 (API Security Patterns)

API 보안 패턴은 서비스의 안정성과 보안을 확보하기 위한 패턴입니다. Rate Limiting, 중복 요청 방지, 클라이언트별 최적화 등을 다룹니다.

## 패턴 목록

| 패턴 | 설명 | 난이도 | Spring 활용 |
|------|------|--------|-------------|
| [Token Bucket](./TokenBucket) | 토큰 기반 Rate Limiting | ⭐⭐ | Bucket4j, Resilience4j |
| [Leaky Bucket](./LeakyBucket) | 고정 속도 요청 처리 | ⭐⭐ | ScheduledExecutor |
| [Idempotency](./Idempotency) | 중복 요청 동일 결과 보장 | ⭐⭐ | Redis + Key 헤더 |
| [BFF](./BFF) | 클라이언트별 전용 백엔드 | ⭐⭐ | Spring Cloud Gateway |

## 핵심 개념

### Rate Limiting
API 호출 빈도를 제한하여 서비스 남용 방지 및 시스템 보호

### Idempotency (멱등성)
동일한 요청을 여러 번 실행해도 결과가 같음을 보장

### BFF (Backend for Frontend)
웹, 모바일 등 클라이언트 특성에 맞는 전용 API 제공

## 패턴 선택 가이드

### 트래픽 급증 방지
→ **Token Bucket** (버스트 허용) 또는 **Leaky Bucket** (균등 처리)

### 결제/송금 등 중요 API
→ **Idempotency** 패턴 필수

### 다양한 클라이언트 지원
→ **BFF** 패턴으로 클라이언트별 최적화

## 관련 GoF 패턴

| GoF 패턴 | 관계 |
|----------|------|
| [Proxy](../../structural/Proxy) | Rate Limiter는 API의 프록시 역할 |
| [Facade](../../structural/Facade) | BFF는 MSA의 Facade |
