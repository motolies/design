# 상황형 면접 가이드

상황을 제시하고 지원자의 답변에 따라 다음 질문을 선택하는 면접관용 가이드입니다. 해법 이름보다 **원리·선택 근거·한계·검증 방법**을 평가합니다. 장애 진단뿐 아니라 설계, 협업, 운영과 우선순위 선택을 다룹니다.

## 직군별 진입

| 목차 | 평가 역량 | 난이도 | 예상 시간 |
|---|---|---|---|
| [백엔드](backend/README.md) | Java/Spring·JPA·DB·분산 시스템 | 기본·심화 | 상황당 10~15 / 15~20분 |
| [프론트엔드](frontend/README.md) | JS/CSS·React/Next.js·성능·운영 | 기본·심화·협업 | 상황당 10~15 / 15~20분 |
| [DevOps / SRE](devops/README.md) | Linux·네트워크·Kubernetes·IaC·운영 판단 | 기본·심화·협업 | 상황당 10~15 / 15~20분 |
| [공통](common/README.md) | 우선순위·리뷰·레거시·AI·기술 선택 | 기본·심화 | 상황당 10~15 / 15~20분 |

## 진행 방법

1. 직무에 맞는 상황 2개와 공통 상황 1개를 미리 고릅니다. 모든 분기를 질문하지 않습니다.
2. 문서의 **상황과 첫 질문만** 읽고 먼저 지원자의 답변과 확인 질문을 듣습니다. 원인이나 해결책을 암시하지 않습니다.
3. 확인 질문에 맞춰 **면접관용 추가 정보**를 제공합니다. 아직 정하지 않은 조건은 가정을 합의하고 기록합니다.
4. 답변별 표에서 관련 분기 1~2개를 고릅니다. `→` 뒤 질문은 한 단계 깊게 확인할 때 사용합니다. 심화 조건은 기본 추론 뒤에만 추가합니다.
5. 평가 기준에 관찰한 근거를 기록하고 실제 경험의 역할·행동·결과·회고로 연결합니다. 경험이 없으면 검증 계획이나 학습 경험으로 대신합니다.

기본 경로는 10~15분, 심화 경로는 15~20분입니다. 심화 시간에는 기본 가설 확인이 포함되며 두 시간을 합산하지 않습니다. 간단한 도입 5분 → 기술 상황 2개 20~40분 → 공통 상황 10~15분 → 지원자 질문 5분으로 40~65분을 구성할 수 있습니다.

면접관용 답변은 가능한 예시입니다. 합의한 조건을 만족하는 다른 접근도 인정합니다. 막히면 조건 하나를 힌트로 제공하고 추론이 어떻게 변하는지 봅니다. 지원자마다 제공한 힌트와 조건 차이를 기록합니다.

## 공통 평가 기준

| 평가 축 | 기본 이해 | 깊이 있는 이해 | 추가 확인 필요 |
|---|---|---|---|
| 사실 확인 | 현상·제약을 질문한다 | 반증할 관측과 가설을 분리한다 | 원인을 바로 단정한다 |
| 원리 | 실행 순서와 책임 범위를 설명한다 | 실패 경계와 보장 범위를 설명한다 | 기술명만 나열한다 |
| 선택 | 요구에 맞는 해결책을 제안한다 | 대안·비용·운영 한계를 비교한다 | 한 도구를 항상 정답으로 본다 |
| 검증 | 재현과 결과 확인을 제안한다 | 부분 실패·부하·회귀·중단 조건을 검증한다 | 적용하면 해결된다고 가정한다 |
| 협업·경험 | 본인 역할과 행동을 구분한다 | 결과 근거·실패·회고를 설명한다 | 팀 성과만 본인 기여로 설명한다 |

질문하지 않은 항목은 미평가로 남깁니다. 수치가 없다는 이유만으로 낮게 평가하지 말고 관찰 근거와 측정의 한계를 구분하는지 봅니다. 연차·도구 경험 수·암기 속도만으로 수준을 결정하지 않습니다.

| 기록 항목 | 면접관 기록 |
|---|---|
| 선택 상황·진행 경로 | |
| 지원자 가정·제공 정보·힌트 | |
| 실제 답변과 판단 근거 | |
| 기본 이해 / 깊이 / 추가 확인 / 미평가 | |
| 경험에서 본인 기여·결과·회고 | |

## 권장 질문 조합

아래 시간은 상황 진행 합계이며 도입과 지원자 질문 시간은 별도입니다.

| 목적 | 추천 조합 | 예상 시간 |
|---|---|---|
| 백엔드 기본 | [쿠폰 상한 기본](backend/junior/coupon-over-issuance.md#basic) + [부분 저장 기본](backend/junior/partial-order-save.md#basic) + [리뷰](common/review-and-mentoring.md#basic) | 30~45분 |
| 백엔드 심화 | [쿠폰 심화](backend/junior/coupon-over-issuance.md#deep) + [중복 이벤트 심화](backend/senior/duplicate-payment-events.md#deep) + [기술 선택](common/build-or-buy.md#basic) | 40~55분 |
| 프론트엔드 기본 | [검색 역전](frontend/basic/search-result-race.md#basic) + [모바일·접근성](frontend/basic/mobile-dialog-layout.md#basic) + [일정과 품질](common/deadline-and-quality.md#basic) | 30~45분 |
| 프론트엔드 심화 | [캐시 심화](frontend/advanced/stale-product-cache.md#deep) + [배포 오류 심화](frontend/advanced/white-screen-release.md#deep) + [패키지 운영](frontend/considerations/shared-package-release.md#basic) | 40~55분 |
| DevOps 기본 | [자원 부족](devops/basic/resource-pressure.md#basic) + [배포 요청 실패](devops/basic/deployment-request-failures.md#basic) + [온콜](devops/considerations/on-call-ownership.md#basic) | 30~45분 |
| DevOps 심화 | [스케일링 심화](devops/advanced/scaling-backlog.md#deep) + [IaC 복구 심화](devops/advanced/infrastructure-state-conflict.md#deep) + [비용 판단](devops/considerations/cost-and-change-budget.md#basic) | 40~55분 |
| 설계·AI 활용 중심 | [직접 구현과 도입](common/build-or-buy.md#basic) + [AI 코드 검증](common/ai-generated-change-review.md#deep) + [LLM 품질](common/llm-feature-quality.md#basic) | 35~50분 |

## 기술 기준과 유지보수

Java 설명은 JDK 21, Spring 관련 설명은 **Spring Boot 3.5.7**을 기준으로 합니다. Spring Boot 4.1은 향후 고려사항이며 현재 예제 기준이 아닙니다. 버전 의존 설명은 각 문서의 공식 출처와 함께 확인합니다. Boot 3.5.7의 세부 기본값은 해당 릴리스 태그를 확인했으며, 3.5/6.2 계열의 온라인 참조 문서는 이후 패치 내용도 표시할 수 있습니다. 버전이 고정되지 않은 Next.js·Kubernetes 등의 기본값은 지원자가 환경과 설정을 먼저 확인하도록 합니다.

[기존 평가 주제 대조표](coverage.md)에서 개편 전 204개 질문의 이동 위치를 확인할 수 있습니다. 문서 수정 시 상황의 기본·심화 조건이 충돌하지 않는지, 첫 질문이 해법을 노출하지 않는지, 분기 질문이 실제 답변과 연결되는지 확인합니다. 상대 링크·앵커·표·접기 영역을 점검하고 각 직군에서 하나씩 읽기 → 정보 공개 → 분기 → 평가 순서로 사용해 봅니다.
