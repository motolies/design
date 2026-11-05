# DevOps / SRE 상황형 면접 가이드

[전체 진행 방법](../README.md) · [공통 상황](../common/README.md) · [평가 주제 대조표](../coverage.md)

난이도는 연차로 고정하지 않습니다. 기본 상황에서 사실 확인과 원리 설명을 듣고, 심화에서는 실패·부하·조직 제약을 추가합니다.

| 진입 목차 | 평가 역량 | 난이도 | 예상 시간 |
|---|---|---|---|
| [DevOps 기본](basic/README.md) | 해당 목차에서 상황별 역량 선택 | 기본·심화 선택 | 상황당 10–15 / 15–20분 |
| [DevOps 심화](advanced/README.md) | 해당 목차에서 상황별 역량 선택 | 심화 | 상황당 15–20분 |
| [DevOps 협업·운영](considerations/README.md) | 해당 목차에서 상황별 역량 선택 | 기본·심화 선택 | 상황당 10–15 / 15–20분 |

## 바로 고를 상황

| 상황 제목 | 평가 역량 | 난이도 | 예상 시간 |
|---|---|---|---|
| [서버 자원 경보와 재시작이 반복된다](basic/resource-pressure.md#basic) | Linux · 컨테이너 · 증거 기반 진단 | 기본 → [심화](basic/resource-pressure.md#deep) | 10–15 / 15–20분 |
| [일부 지역에서만 사이트 연결이 느리거나 실패한다](basic/network-path-failure.md#basic) | DNS · TLS · 라우팅 · Kubernetes 네트워크 | 기본 → [심화](basic/network-path-failure.md#deep) | 10–15 / 15–20분 |
| [배포 중 진행 중인 요청이 끊긴다](basic/deployment-request-failures.md#basic) | 배포 전략 · 프로세스 종료 · 롤백 | 기본 → [심화](basic/deployment-request-failures.md#deep) | 10–15 / 15–20분 |
| [같은 배포 버전인데 환경별 동작이 다르다](basic/configuration-and-artifact-drift.md#basic) | CI/CD · 이미지 · 설정 · 공급망 | 기본 → [심화](basic/configuration-and-artifact-drift.md#deep) | 10–15 / 15–20분 |
| [Pod를 늘려도 대기열과 지연이 줄지 않는다](advanced/scaling-backlog.md#basic) | 스케줄링 · 오토스케일링 · 부하 시험 | 기본 → [심화](advanced/scaling-backlog.md#deep) | 10–15 / 15–20분 |
| [긴급 수정 뒤 IaC가 운영 자원을 되돌리려 한다](advanced/infrastructure-state-conflict.md#basic) | Terraform · GitOps · 변경 통제 | 기본 → [심화](advanced/infrastructure-state-conflict.md#deep) | 10–15 / 15–20분 |
| [관측 비용은 늘었는데 사용자 장애를 놓친다](advanced/observability-cost-and-blind-spots.md#basic) | SLI/SLO · 메트릭 · 로그 · 트레이스 | 기본 → [심화](advanced/observability-cost-and-blind-spots.md#deep) | 10–15 / 15–20분 |
| [멀티 AZ인데 한 영역 장애로 서비스가 멈춘다](advanced/zone-outage-recovery.md#basic) | 가용성 · 저장소 · 재해 복구 · 카오스 실험 | 기본 → [심화](advanced/zone-outage-recovery.md#deep) | 10–15 / 15–20분 |
| [새벽 장애의 담당팀과 운영 권한이 불명확하다](considerations/on-call-ownership.md#basic) | 온콜 · 책임 경계 · 사고 소통 | 기본 → [심화](considerations/on-call-ownership.md#deep) | 10–15 / 15–20분 |
| [비용 절감 요구와 배포 위험이 충돌한다](considerations/cost-and-change-budget.md#basic) | 비용 · 변경 관리 · 자동화 우선순위 | 기본 → [심화](considerations/cost-and-change-budget.md#deep) | 10–15 / 15–20분 |

공통 진행 기준은 최상위 목차를 따릅니다. 인터뷰 중에는 해설을 읽어주지 말고 지원자의 질문에 맞춰 추가 정보를 공개하세요.
