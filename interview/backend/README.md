# 백엔드 상황형 면접 가이드

[전체 진행 방법](../README.md) · [공통 상황](../common/README.md) · [평가 주제 대조표](../coverage.md)

난이도는 연차로 고정하지 않습니다. 기본 상황에서 사실 확인과 원리 설명을 듣고, 심화에서는 실패·부하·조직 제약을 추가합니다.

Java는 JDK 21, Spring은 **Spring Boot 3.5.7**을 기준으로 합니다. 4.1은 향후 검토 대상으로만 다루며 현재 답변 기준에 섞지 않습니다.

| 진입 목차 | 평가 역량 | 난이도 | 예상 시간 |
|---|---|---|---|
| [백엔드 기본](junior/README.md) | 해당 목차에서 상황별 역량 선택 | 기본·심화 선택 | 상황당 10–15 / 15–20분 |
| [백엔드 심화](senior/README.md) | 해당 목차에서 상황별 역량 선택 | 심화 | 상황당 15–20분 |

## 바로 고를 상황

| 상황 제목 | 평가 역량 | 난이도 | 예상 시간 |
|---|---|---|---|
| [쿠폰 100장 중 101건 발급](junior/coupon-over-issuance.md#basic) | 동시성 · 트랜잭션 · 불변식 · 검증 | 기본 → [심화](junior/coupon-over-issuance.md#deep) | 10–15 / 15–20분 |
| [주문 실패 뒤 일부 데이터만 남는다](junior/partial-order-save.md#basic) | Spring AOP · 트랜잭션 · 예외 · 테스트 | 기본 → [심화](junior/partial-order-save.md#deep) | 10–15 / 15–20분 |
| [로그인했는데 다른 사용자의 주문이 보인다](junior/access-denied-after-login.md#basic) | 요청 흐름 · 인증과 인가 · 웹 보안 | 기본 → [심화](junior/access-denied-after-login.md#deep) | 10–15 / 15–20분 |
| [설정 변경 후 애플리케이션이 기동하지 않는다](junior/application-startup-conflict.md#basic) | DI · 자동 구성 · Bean 생명주기 | 기본 → [심화](junior/application-startup-conflict.md#deep) | 10–15 / 15–20분 |
| [주문 항목 수정과 삭제 결과가 예상과 다르다](junior/order-model-integrity.md#basic) | JPA 매핑 · 동일성 · 컬렉션 · 데이터 모델 | 기본 → [심화](junior/order-model-integrity.md#deep) | 10–15 / 15–20분 |
| [주문 목록 조회가 데이터 증가 후 느려진다](senior/order-query-regression.md#basic) | JPA 조회 · 캐시 · 부하 분석 | 기본 → [심화](senior/order-query-regression.md#deep) | 10–15 / 15–20분 |
| [장시간 운영한 JVM의 지연과 메모리 사용이 늘어난다](senior/jvm-latency-and-upgrade.md#basic) | JVM · GC · JDK 이전 · 동시 실행 | 기본 → [심화](senior/jvm-latency-and-upgrade.md#deep) | 10–15 / 15–20분 |
| [결제는 끝났는데 주문 상태가 늦거나 중복된다](senior/duplicate-payment-events.md#basic) | 분산 정합성 · 메시지 · 아키텍처 선택 | 기본 → [심화](senior/duplicate-payment-events.md#deep) | 10–15 / 15–20분 |

공통 진행 기준은 최상위 목차를 따릅니다. 인터뷰 중에는 해설을 읽어주지 말고 지원자의 질문에 맞춰 추가 정보를 공개하세요.
