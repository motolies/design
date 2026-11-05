# DevOps 기본 상황 선택

[전체 진행 방법](../../README.md) · [평가 주제 대조표](../../coverage.md)

상황 1개를 골라 지원자의 답변에 맞는 분기 1~2개만 진행합니다. 기본은 10~15분, 심화는 15~20분이며 경력 연수 대신 관찰한 설명과 검증 근거로 평가합니다. 같은 상황의 기본·심화는 한 문서에서 조건을 달리합니다.

| 상황 제목 | 평가 역량 | 난이도 | 예상 시간 |
|---|---|---|---|
| [서버 자원 경보와 재시작이 반복된다](resource-pressure.md#basic) | Linux · 컨테이너 · 증거 기반 진단 | 기본 ([심화 경로](resource-pressure.md#deep)) | 10~15분 |
| [일부 지역에서만 사이트 연결이 느리거나 실패한다](network-path-failure.md#basic) | DNS · TLS · 라우팅 · Kubernetes 네트워크 | 기본 ([심화 경로](network-path-failure.md#deep)) | 10~15분 |
| [배포 중 진행 중인 요청이 끊긴다](deployment-request-failures.md#basic) | 배포 전략 · 프로세스 종료 · 롤백 | 기본 ([심화 경로](deployment-request-failures.md#deep)) | 10~15분 |
| [같은 배포 버전인데 환경별 동작이 다르다](configuration-and-artifact-drift.md#basic) | CI/CD · 이미지 · 설정 · 공급망 | 기본 ([심화 경로](configuration-and-artifact-drift.md#deep)) | 10~15분 |
| [Pod를 늘려도 대기열과 지연이 줄지 않는다](../advanced/scaling-backlog.md#basic) | 스케줄링 · 오토스케일링 · 부하 시험 | 기본 ([심화 경로](../advanced/scaling-backlog.md#deep)) | 10~15분 |
| [관측 비용은 늘었는데 사용자 장애를 놓친다](../advanced/observability-cost-and-blind-spots.md#basic) | SLI/SLO · 메트릭 · 로그 · 트레이스 | 기본 ([심화 경로](../advanced/observability-cost-and-blind-spots.md#deep)) | 10~15분 |
| [멀티 AZ인데 한 영역 장애로 서비스가 멈춘다](../advanced/zone-outage-recovery.md#basic) | 가용성 · 저장소 · 재해 복구 · 카오스 실험 | 기본 ([심화 경로](../advanced/zone-outage-recovery.md#deep)) | 10~15분 |
| [리뷰 의견 충돌로 배포가 지연된다](../../common/review-and-mentoring.md#basic) | 코드 리뷰 · Git · 멘토링 · 의사결정 | 기본 ([심화 경로](../../common/review-and-mentoring.md#deep)) | 10~15분 |

첫 상황에서는 관찰 사실과 가정을 구분하는지, 두 번째에서는 선택의 한계와 실패 시 복구·검증을 설명하는지 확인하세요. 공통 질문도 별도 지식 시험처럼 추가하지 말고 실제 경험으로 이어갑니다.
