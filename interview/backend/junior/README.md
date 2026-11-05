# 백엔드 기본 상황 선택

[전체 진행 방법](../../README.md) · [평가 주제 대조표](../../coverage.md)

상황 1개를 골라 지원자의 답변에 맞는 분기 1–2개만 진행합니다. 기본은 10–15분, 심화는 15–20분이며 경력 연수 대신 관찰한 설명과 검증 근거로 평가합니다. 같은 상황의 기본·심화는 한 문서에서 조건을 달리합니다.

| 상황 제목 | 평가 역량 | 난이도 | 예상 시간 |
|---|---|---|---|
| [쿠폰 100장 중 101건 발급](coupon-over-issuance.md#basic) | 동시성 · 트랜잭션 · 불변식 · 검증 | 기본 ([심화 경로](coupon-over-issuance.md#deep)) | 10–15분 |
| [주문 실패 뒤 일부 데이터만 남는다](partial-order-save.md#basic) | Spring AOP · 트랜잭션 · 예외 · 테스트 | 기본 ([심화 경로](partial-order-save.md#deep)) | 10–15분 |
| [로그인했는데 다른 사용자의 주문이 보인다](access-denied-after-login.md#basic) | 요청 흐름 · 인증과 인가 · 웹 보안 | 기본 ([심화 경로](access-denied-after-login.md#deep)) | 10–15분 |
| [설정 변경 후 애플리케이션이 기동하지 않는다](application-startup-conflict.md#basic) | DI · 자동 구성 · Bean 생명주기 | 기본 ([심화 경로](application-startup-conflict.md#deep)) | 10–15분 |
| [주문 항목 수정과 삭제 결과가 예상과 다르다](order-model-integrity.md#basic) | JPA 매핑 · 동일성 · 컬렉션 · 데이터 모델 | 기본 ([심화 경로](order-model-integrity.md#deep)) | 10–15분 |
| [장시간 운영한 JVM의 지연과 메모리 사용이 늘어난다](../senior/jvm-latency-and-upgrade.md#basic) | JVM · GC · JDK 이전 · 동시 실행 | 기본 ([심화 경로](../senior/jvm-latency-and-upgrade.md#deep)) | 10–15분 |
| [결제는 끝났는데 주문 상태가 늦거나 중복된다](../senior/duplicate-payment-events.md#basic) | 분산 정합성 · 메시지 · 아키텍처 선택 | 기본 ([심화 경로](../senior/duplicate-payment-events.md#deep)) | 10–15분 |
| [리뷰 의견 충돌로 배포가 지연된다](../../common/review-and-mentoring.md#basic) | 코드 리뷰 · Git · 멘토링 · 의사결정 | 기본 ([심화 경로](../../common/review-and-mentoring.md#deep)) | 10–15분 |

첫 상황에서는 관찰 사실과 가정을 구분하는지, 두 번째에서는 선택의 한계와 실패 시 복구·검증을 설명하는지 확인하세요. 공통 질문도 별도 지식 시험처럼 추가하지 말고 실제 경험으로 이어갑니다.
