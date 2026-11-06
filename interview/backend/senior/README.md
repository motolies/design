# 백엔드 심화 상황 선택

[전체 진행 방법](../../README.md) · [평가 주제 대조표](../../coverage.md)

상황 1개를 골라 지원자의 답변에 맞는 분기 1–2개만 진행합니다. 기본은 10–15분, 심화는 15–20분이며 경력 연수 대신 관찰한 설명과 검증 근거로 평가합니다. 같은 상황의 기본·심화는 한 문서에서 조건을 달리합니다.

| 상황 제목 | 평가 역량 | 난이도 | 예상 시간 |
|---|---|---|---|
| [쿠폰 100장 중 101건 발급](../junior/coupon-over-issuance.md#deep) | 동시성 · 트랜잭션 · 불변식 · 검증 | 심화 ([기본 경로](../junior/coupon-over-issuance.md#basic)) | 15–20분 |
| [주문 실패 뒤 일부 데이터만 남는다](../junior/partial-order-save.md#deep) | Spring AOP · 트랜잭션 · 예외 · 테스트 | 심화 ([기본 경로](../junior/partial-order-save.md#basic)) | 15–20분 |
| [로그인했는데 다른 사용자의 주문이 보인다](../junior/access-denied-after-login.md#deep) | 요청 흐름 · 인증과 인가 · 웹 보안 | 심화 ([기본 경로](../junior/access-denied-after-login.md#basic)) | 15–20분 |
| [설정 변경 후 애플리케이션이 기동하지 않는다](../junior/application-startup-conflict.md#deep) | DI · 자동 구성 · Bean 생명주기 | 심화 ([기본 경로](../junior/application-startup-conflict.md#basic)) | 15–20분 |
| [하나의 택배사를 사용하던 시스템에 새 배송사를 추가해야 한다](../junior/shipping-carrier-extension.md#deep) | 책임 분리 · 디자인 패턴 · 확장성 · 회귀 검증 | 심화 ([기본 경로](../junior/shipping-carrier-extension.md#basic)) | 15–20분 |
| [주문 목록 조회가 데이터 증가 후 느려진다](order-query-regression.md#deep) | JPA 조회 · 캐시 · 부하 분석 | 심화 ([기본 경로](order-query-regression.md#basic)) | 15–20분 |
| [장시간 운영한 JVM의 지연과 메모리 사용이 늘어난다](jvm-latency-and-upgrade.md#deep) | JVM · GC · JDK 이전 · 동시 실행 | 심화 ([기본 경로](jvm-latency-and-upgrade.md#basic)) | 15–20분 |
| [결제는 끝났는데 주문 상태가 늦거나 중복된다](duplicate-payment-events.md#deep) | 분산 정합성 · 메시지 · 아키텍처 선택 | 심화 ([기본 경로](duplicate-payment-events.md#basic)) | 15–20분 |
| [배포 중 진행 중인 요청이 끊긴다](../../devops/basic/deployment-request-failures.md#deep) | 배포 전략 · 프로세스 종료 · 롤백 | 심화 ([기본 경로](../../devops/basic/deployment-request-failures.md#basic)) | 15–20분 |
| [새벽 장애의 담당팀과 운영 권한이 불명확하다](../../devops/considerations/on-call-ownership.md#deep) | 온콜 · 책임 경계 · 사고 소통 | 심화 ([기본 경로](../../devops/considerations/on-call-ownership.md#basic)) | 15–20분 |
| [리뷰 의견 충돌로 배포가 지연된다](../../common/review-and-mentoring.md#deep) | 코드 리뷰 · Git · 멘토링 · 의사결정 | 심화 ([기본 경로](../../common/review-and-mentoring.md#basic)) | 15–20분 |
| [직접 구현과 외부 솔루션 중 선택해야 한다](../../common/build-or-buy.md#deep) | 설계 선택 · 실험 · 기술 의사결정 | 심화 ([기본 경로](../../common/build-or-buy.md#basic)) | 15–20분 |

첫 상황에서는 관찰 사실과 가정을 구분하는지, 두 번째에서는 선택의 한계와 실패 시 복구·검증을 설명하는지 확인하세요. 공통 질문도 별도 지식 시험처럼 추가하지 말고 실제 경험으로 이어갑니다.
