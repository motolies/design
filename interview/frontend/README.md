# 프론트엔드 상황형 면접 가이드

[전체 진행 방법](../README.md) · [공통 상황](../common/README.md) · [평가 주제 대조표](../coverage.md)

난이도는 연차로 고정하지 않습니다. 기본 상황에서 사실 확인과 원리 설명을 듣고, 심화에서는 실패·부하·조직 제약을 추가합니다.

| 진입 목차 | 평가 역량 | 난이도 | 예상 시간 |
|---|---|---|---|
| [프론트엔드 기본](basic/README.md) | 해당 목차에서 상황별 역량 선택 | 기본·심화 선택 | 상황당 10~15 / 15~20분 |
| [프론트엔드 심화](advanced/README.md) | 해당 목차에서 상황별 역량 선택 | 심화 | 상황당 15~20분 |
| [프론트엔드 협업·운영](considerations/README.md) | 해당 목차에서 상황별 역량 선택 | 기본·심화 선택 | 상황당 10~15 / 15~20분 |

## 바로 고를 상황

| 상황 제목 | 평가 역량 | 난이도 | 예상 시간 |
|---|---|---|---|
| [빠르게 검색하면 이전 결과가 마지막에 보인다](basic/search-result-race.md#basic) | JavaScript 비동기 · 클로저 · Effect | 기본 → [심화](basic/search-result-race.md#deep) | 10~15 / 15~20분 |
| [정렬 후 입력값이 다른 행으로 이동한다](basic/editable-list-state.md#basic) | React 동일성 · 상태 소유권 · 폼 | 기본 → [심화](basic/editable-list-state.md#deep) | 10~15 / 15~20분 |
| [모바일 결제 창이 잘리고 키보드로 닫을 수 없다](basic/mobile-dialog-layout.md#basic) | CSS 레이아웃 · 접근성 · 협업 | 기본 → [심화](basic/mobile-dialog-layout.md#deep) | 10~15 / 15~20분 |
| [검색 입력과 첫 화면 표시가 느리다](advanced/input-rendering-lag.md#basic) | 이벤트 루프 · 렌더링 · 성능 측정 | 기본 → [심화](advanced/input-rendering-lag.md#deep) | 10~15 / 15~20분 |
| [저장 성공 뒤 목록과 상세의 값이 다르다](advanced/stale-product-cache.md#basic) | 서버 상태 · 캐시 계층 · 렌더링 모델 | 기본 → [심화](advanced/stale-product-cache.md#deep) | 10~15 / 15~20분 |
| [배포 후 로그인과 주문 화면 통신이 실패한다](advanced/login-and-contract-failure.md#basic) | 인증 · 브라우저 보안 · BFF · 계약 검증 | 기본 → [심화](advanced/login-and-contract-failure.md#deep) | 10~15 / 15~20분 |
| [새 버전 배포 뒤 일부 사용자에게 흰 화면이 보인다](advanced/white-screen-release.md#basic) | 배포 · 캐시 · 관측성 · 사고 대응 | 기본 → [심화](advanced/white-screen-release.md#deep) | 10~15 / 15~20분 |
| [공통 UI 패키지 패치 후 여러 앱이 깨진다](considerations/shared-package-release.md#basic) | 디자인 시스템 · 패키지 호환성 · 빌드 | 기본 → [심화](considerations/shared-package-release.md#deep) | 10~15 / 15~20분 |
| [테스트는 자주 실패하지만 결제 버그는 놓친다](considerations/flaky-checkout-tests.md#basic) | 테스트 전략 · 회귀 검증 · 멘토링 | 기본 → [심화](considerations/flaky-checkout-tests.md#deep) | 10~15 / 15~20분 |

공통 진행 기준은 최상위 목차를 따릅니다. 인터뷰 중에는 해설을 읽어주지 말고 지원자의 질문에 맞춰 추가 정보를 공개하세요.
