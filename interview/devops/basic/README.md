# 데브옵스 기본 면접 질문

## 🎯 개요

**대상**: 0-3년차 DevOps / 인프라 엔지니어, 또는 백엔드 개발자에서 인프라 직군으로 전환하는 지원자
**평가 목표**: Linux·네트워크·컨테이너·CI/CD 등 운영 기본기를 알고 있는지, 실제 트러블슈팅 흐름을 그릴 수 있는지

---

## 1. Linux & Shell 기초

### Q1. 운영 서버에서 "CPU 사용률이 100%다"라는 알림을 받았다고 가정해보세요. 어떤 순서로 원인을 찾아갈 건가요?

- **의도**: 운영 트러블슈팅 흐름과 기본 명령어 숙련도 평가
- **핵심 포인트**:
  - `top` / `htop`으로 점유 프로세스 식별
  - `ps`, `pidstat`, `pstree`로 자식 프로세스까지 추적
  - User CPU vs System CPU vs IOwait 구분
  - 일시적 스파이크인지 지속적인지 모니터링 도구로 확인
- **좋은 답변 예시**:
  - `top` → 가장 높은 PID 확인 → `ps -ef | grep <PID>`로 어떤 명령인지 확인 → 동일 프로세스가 여러 개라면 부모/자식 관계 추적 → 애플리케이션이라면 스레드 덤프, 시스템 호출이라면 `strace`로 깊이 진단
  - 단순 재기동 전에 힙 덤프나 스레드 덤프 같은 증거를 먼저 확보
- **꼬리 질문**:

  **Q. `load average`가 높은데 CPU 사용률은 낮은 경우 어떤 가능성을 의심하나요?**
  - **좋은 답변**: Load average에는 R(실행 중)뿐 아니라 D(디스크 I/O 대기) 상태 프로세스가 포함됩니다. 따라서 CPU는 한가한데 load만 높다면 디스크/네트워크 I/O 병목이거나 락 경쟁을 의심합니다. `iostat -xz 1`, `vmstat 1`, `iotop`으로 I/O 상태를 먼저 확인합니다.
  - **추가 질문**: D 상태(uninterruptible sleep)에 빠진 프로세스를 어떻게 처리하나요?

  **Q. 컨테이너 안에서 `top`을 보면 호스트의 CPU·메모리가 보이는데, 이유와 해결 방법은?**
  - **좋은 답변**: 일반적인 `top`/`free`는 호스트의 `/proc`를 읽기 때문입니다. cgroup 제한값을 보려면 `/sys/fs/cgroup/...`의 cgroup v2 파일을 읽거나 `lxcfs`, JDK라면 `-XX:+UseContainerSupport`(JDK 10+ 기본 ON)을 활용합니다. K8s에서는 `kubectl top pod`로 cgroup 기준 사용량을 봅니다.
  - **추가 질문**: 컨테이너의 메모리 limit을 초과하면 어떤 일이 일어나나요?

---

### Q2. 프로세스가 SIGKILL과 SIGTERM을 받았을 때 동작 차이를 설명해주세요.

- **의도**: 시그널과 graceful shutdown에 대한 이해
- **핵심 포인트**:
  - SIGTERM(15): 종료를 "요청" — 애플리케이션이 핸들러로 정리 작업 수행 가능
  - SIGKILL(9): 즉시 종료 — 핸들러 등록 불가, 자원 정리 기회 없음
  - K8s Pod 종료 시 SIGTERM → `terminationGracePeriodSeconds` → SIGKILL 흐름
- **좋은 답변 예시**:
  - SIGTERM은 가로채서 진행 중인 요청 마무리, 커넥션 풀 정리, 메시지 큐 ack 처리 등을 한 뒤 종료할 수 있습니다
  - SIGKILL은 강제 종료이므로 in-flight 요청이 끊기고, 파일 락이 남거나 트랜잭션이 롤백될 수 있어 가능하면 피해야 합니다
- **꼬리 질문**:

  **Q. Spring Boot 애플리케이션을 K8s에서 graceful하게 종료시키려면 어떤 설정이 필요한가요?**
  - **좋은 답변**: `server.shutdown=graceful`과 `spring.lifecycle.timeout-per-shutdown-phase`로 진행 중인 HTTP 요청을 마저 처리하게 하고, K8s `terminationGracePeriodSeconds`를 그 timeout보다 길게 설정합니다. preStop hook으로 readinessProbe를 먼저 끄는 `sleep`을 둬서 LB가 트래픽을 빼낸 후 종료되도록 합니다.
  - **추가 질문**: preStop hook에서 `sleep`을 두는 이유가 정확히 무엇인가요?

  **Q. `kill -9`를 자주 쓰면 안 되는 이유는?**
  - **좋은 답변**: SIGKILL은 트랜잭션 롤백, 임시 파일 정리, DB 커넥션 close 같은 정리 코드를 건너뜁니다. 운영 중에는 데이터 불일치, 파일 락 잔존, 메시지 중복 처리 같은 부작용을 만들 수 있어 SIGTERM부터 시도하는 것이 원칙입니다.
  - **추가 질문**: SIGTERM을 보냈는데도 종료되지 않는 프로세스가 있다면 어떻게 진단하나요?

---

### Q3. 디스크가 가득 찼다는 알림을 받았는데, `df`로는 90%인데 `du`로 계산하면 50%만 사용 중입니다. 어떤 원인이 있을 수 있나요?

- **의도**: 파일 시스템 동작과 실전 디버깅 경험 평가
- **핵심 포인트**:
  - 삭제됐지만 fd가 열려 있는 파일은 `du`에는 안 보이고 `df`에만 잡힘
  - inode 고갈은 별개 (`df -i`)
  - 로그 로테이션 누락, mount 포인트 가려진 디렉터리
- **좋은 답변 예시**:
  - `lsof | grep deleted` 또는 `lsof +L1`로 삭제됐지만 열려 있는 파일을 찾습니다. 해당 프로세스를 재시작하거나 fd를 닫게 하면 디스크가 즉시 회수됩니다.
  - 로그 파일을 `> /var/log/app.log`로 비우는 트릭은 fd가 살아 있다면 동작하지만, 단순 `rm`은 fd가 닫힐 때까지 공간이 반환되지 않습니다.
- **꼬리 질문**:

  **Q. inode가 부족할 때는 어떻게 진단하나요?**
  - **좋은 답변**: `df -i`로 inode 사용률을 확인하고, `find / -xdev -type f | wc -l` 또는 `find / -xdev -printf '%h\n' | sort | uniq -c | sort -n`으로 작은 파일이 폭증한 디렉터리를 찾습니다. 메일 큐, 세션 디렉터리, 캐시 디렉터리가 흔한 범인입니다.
  - **추가 질문**: inode를 늘리려면 어떻게 해야 하나요?

  **Q. 로그가 디스크를 가득 채우지 않게 하려면 어떤 방법들이 있나요?**
  - **좋은 답변**: `logrotate`로 크기/날짜 기준 로테이션, 컨테이너라면 stdout으로만 출력하고 컨테이너 런타임의 log driver(`json-file max-size`)나 K8s 로그 수집기(Promtail/Fluent Bit)로 외부 저장소에 보냅니다. 운영 중인 노드에 raw 로그가 쌓이지 않게 하는 것이 가장 안전합니다.
  - **추가 질문**: 컨테이너에서 로그를 파일로 쓰면 안 되는 이유는?

---

### Q4. systemd에서 서비스 하나를 만들고 부팅 시 자동 시작되도록 등록하려면 어떻게 하나요?

- **의도**: Linux 서비스 관리 기본기 평가
- **핵심 포인트**:
  - `/etc/systemd/system/<name>.service` 위치
  - `ExecStart`, `Restart`, `User`, `WorkingDirectory` 등 주요 지시어
  - `systemctl daemon-reload` → `enable` → `start` 흐름
- **좋은 답변 예시**:
  - unit 파일을 만들고 `daemon-reload`로 새로고침한 뒤 `enable`로 부팅 시 시작 등록, `start`로 즉시 기동, `status`/`journalctl -u <name>`으로 로그 확인
  - 비정상 종료 시 자동 재시작이 필요하면 `Restart=on-failure`, `RestartSec=5s`
- **꼬리 질문**:

  **Q. 컨테이너 시대에 systemd를 직접 다룰 일이 줄어든 이유는?**
  - **좋은 답변**: 컨테이너 자체가 단일 프로세스 단위로 격리/재시작을 책임지고, 오케스트레이터(K8s)가 systemd가 하던 라이프사이클 관리·재시작 정책을 대신합니다. 다만 노드 자체(kubelet, containerd, 로그 수집기)는 여전히 systemd 위에 올라가 있어 노드 트러블슈팅 시 필요합니다.
  - **추가 질문**: 컨테이너 안에 systemd를 띄우는 게 안티패턴인 이유는?

---

### Q5. 자주 쓰는 트러블슈팅 명령 5가지만 꼽아주세요. 어떤 상황에 쓰나요?

- **의도**: 실무 경험과 직관 평가
- **핵심 포인트**: 명령을 외우는지가 아니라, "어떤 상황에 무엇을 보는지" 흐름을 가진 사람인지가 핵심
- **좋은 답변 예시**:
  - `top` / `htop` — CPU·메모리 점유 전체 뷰
  - `ss -tnp` (또는 `netstat -tnp`) — 어떤 프로세스가 어떤 포트를 잡고 있는지
  - `journalctl -u <svc> -f` — 시스템 서비스 로그 실시간
  - `tcpdump -ni eth0 port 8080` — 패킷 레벨 검증
  - `strace -p <PID>` — 시스템 콜 추적, "왜 멈춰 있는가"를 보여줌
- **꼬리 질문**:

  **Q. `tcpdump`를 운영 서버에서 떠도 괜찮은가요? 주의할 점은?**
  - **좋은 답변**: 패킷 캡처는 CPU·디스크에 부담이 갈 수 있고, 평문 트래픽이 잡히면 개인정보·토큰이 그대로 노출됩니다. 그래서 `-c 1000` 같은 카운트 제한, `port 8080 and host x.x.x.x` 같은 필터, 짧은 시간 윈도우로 떠야 하며, 캡처 파일은 격리된 장소에 보관하고 분석 후 삭제합니다.
  - **추가 질문**: TLS로 암호화된 트래픽을 어떻게 분석하나요?

---

## 2. 네트워크 기초

### Q6. 브라우저 주소창에 `https://example.com`을 입력하면 패킷이 응답으로 돌아올 때까지 어떤 일이 일어나는지 설명해주세요.

- **의도**: DNS·TCP·TLS·HTTP의 전체 흐름을 머릿속에 갖고 있는지 평가 (자바 면접의 "Spring 요청 흐름"에 대응하는 데브옵스의 도입 질문)
- **핵심 포인트**:
  - DNS 조회 (캐시 → 리졸버 → 권한 네임서버)
  - TCP 3-way handshake → TLS handshake
  - HTTP 요청 → LB → 백엔드 → 응답
  - CDN/캐시 계층, Keep-alive
- **좋은 답변 예시**:
  - 브라우저 → OS DNS 캐시 → `/etc/resolv.conf` 리졸버 → 권한 네임서버에서 A/AAAA 레코드 획득 → TCP 3-way handshake → TLS 핸드셰이크(인증서 검증, 세션 키 교환) → HTTP 요청 → LB가 백엔드 풀로 라우팅 → 응답 → 연결 재사용
- **꼬리 질문**:

  **Q. TLS handshake에서 RSA와 ECDHE의 차이는?**
  - **좋은 답변**: RSA 키 교환은 클라이언트가 pre-master secret을 서버 공개키로 암호화해 전달하는 방식이라, 서버 개인키가 유출되면 과거 패킷도 복호화 가능합니다(전방 비밀성 X). ECDHE는 임시 키 쌍으로 매 세션 비밀을 새로 만들어 전방 비밀성(PFS)을 보장합니다. 현재는 ECDHE가 권장입니다.
  - **추가 질문**: TLS 1.3에서 handshake가 빨라진 이유는?

  **Q. DNS TTL을 짧게 설정하면 어떤 장단점이 있나요?**
  - **좋은 답변**: TTL이 짧으면 IP 변경(LB 교체, 페일오버)이 빠르게 전파되지만 DNS 서버 쿼리량이 늘고, 클라이언트마다 캐시 동작이 달라 일관성이 떨어질 수 있습니다. 운영적으로는 평소 5분~1시간으로 두고 변경 직전에 짧게 줄였다가 다시 늘리는 식으로 운영합니다.
  - **추가 질문**: Java 애플리케이션은 DNS TTL을 기본적으로 얼마나 캐싱하나요?

  **Q. TCP 3-way handshake가 끝나기 전에 데이터를 보내는 방법이 있나요?**
  - **좋은 답변**: TCP Fast Open(TFO)이 있고, TLS 1.3의 0-RTT가 비슷한 역할을 합니다. 다만 0-RTT는 replay 공격에 노출될 수 있어 GET 같은 멱등 요청에만 적용하는 게 일반적입니다.
  - **추가 질문**: 0-RTT를 사용할 때 보안상 어떤 점을 주의해야 하나요?

---

### Q7. L4 로드밸런서와 L7 로드밸런서의 차이를 설명해주세요. 각각 어떤 상황에서 쓰나요?

- **의도**: 네트워크 계층과 LB 동작 원리 이해
- **핵심 포인트**:
  - L4: TCP/UDP 레벨, IP·포트만 보고 분배 (NLB, IPVS, HAProxy TCP mode)
  - L7: HTTP 레벨, URL·헤더·쿠키 보고 분배 (ALB, Nginx, HAProxy HTTP mode)
  - L7은 TLS 종료, 경로 기반 라우팅, 헤더 기반 라우팅 가능
- **좋은 답변 예시**:
  - 단순 TCP 분산(예: DB, gRPC raw), 초고성능이 필요하면 L4
  - URL `/api/*`, `/static/*`처럼 경로별 라우팅이나 헤더 기반 카나리, WAF 결합이 필요하면 L7
- **꼬리 질문**:

  **Q. K8s `Service`의 ClusterIP, NodePort, LoadBalancer 타입 차이는?**
  - **좋은 답변**: ClusterIP는 클러스터 내부 가상 IP, NodePort는 모든 노드의 동일 포트로 외부 노출, LoadBalancer는 클라우드 LB와 연계되어 외부에 진짜 IP를 줍니다. LoadBalancer 타입은 내부적으로 NodePort + 클라우드 LB 조합입니다.
  - **추가 질문**: ClusterIP는 어떻게 동작하나요? (kube-proxy iptables/IPVS 모드)

  **Q. `Service`와 `Ingress`의 역할 차이는?**
  - **좋은 답변**: Service는 Pod 묶음에 대한 L4 가상 엔드포인트입니다. Ingress는 그 위에서 호스트/경로 기반 L7 라우팅을 담당하며, Ingress Controller(Nginx, Traefik 등)가 실제 트래픽을 처리합니다.
  - **추가 질문**: Ingress와 Gateway API의 차이는?

---

### Q8. CIDR `10.0.0.0/16`은 몇 개의 IP를 포함하나요? 그리고 같은 VPC 안에 서브넷을 어떻게 자르는 게 일반적인가요?

- **의도**: 서브넷 계산과 VPC 설계 기본기
- **핵심 포인트**:
  - `/16`은 2^(32-16) = 65,536개 IP
  - 예약된 IP (네트워크/브로드캐스트, 클라우드 예약)는 빼고 가용 IP는 더 적음
  - 가용 영역(AZ) 단위로 서브넷을 자르고, public/private을 분리
- **좋은 답변 예시**:
  - VPC `/16` 하나에 AZ당 public `/24` + private `/24` + DB private `/24`처럼 자르는 것이 일반적
  - 마이크로서비스 확장을 고려해 처음부터 넉넉히 잡아두는 것이 좋고, 인프라가 커진 뒤에는 CIDR 변경이 어렵기 때문에 IP 계획을 미리 세웁니다
- **꼬리 질문**:

  **Q. Private 서브넷의 Pod이 인터넷으로 나가려면 어떤 구성이 필요한가요?**
  - **좋은 답변**: NAT Gateway(또는 NAT 인스턴스)를 public 서브넷에 두고, private 서브넷의 라우팅 테이블이 `0.0.0.0/0`을 NAT GW로 향하게 합니다. NAT GW가 outbound IP를 단일 IP로 정리해주기 때문에 외부 시스템의 IP whitelist에도 유리합니다.
  - **추가 질문**: NAT Gateway 비용이 부담될 때 대안은?

---

### Q9. 사용자가 "사이트가 느리다"고 합니다. 네트워크 관점에서 어떤 지표를 확인하나요?

- **의도**: 네트워크 성능 지표 이해도
- **핵심 포인트**:
  - DNS 조회 시간, TCP 연결 시간, TLS handshake 시간, TTFB, 콘텐츠 다운로드 시간
  - 클라이언트 측은 브라우저 dev tools, 서버 측은 ALB/Nginx 로그의 단계별 시간
- **좋은 답변 예시**:
  - 브라우저 dev tools의 waterfall에서 어느 단계가 긴지 보고, 서버 측 로그(`$request_time`, `$upstream_response_time`)로 LB ↔ 백엔드 구간 시간을 분리합니다
  - 외부 모니터링(Synthetic monitoring)으로 지역별 응답 시간을 함께 봅니다
- **꼬리 질문**:

  **Q. TTFB(Time To First Byte)가 길면 어떤 가능성을 의심하나요?**
  - **좋은 답변**: 백엔드 응답 자체가 늦거나(DB 쿼리, 외부 API 호출, GC), LB ↔ 백엔드 네트워크가 느리거나, 백엔드의 동시 처리 한계에 걸려 큐잉되는 경우를 의심합니다. APM과 LB 로그를 함께 보면 어디서 시간을 쓰는지 끊어서 볼 수 있습니다.
  - **추가 질문**: TCP Retransmission이 자주 일어나면 어떤 영향이 있나요?

---

## 3. Git & 브랜치 전략

### Q10. GitFlow와 Trunk-Based Development의 차이를 설명하고, 어떤 상황에 어느 쪽이 적합한가요?

- **의도**: 브랜치 전략에 대한 이해와 의사결정 능력
- **핵심 포인트**:
  - GitFlow: `develop`, `release`, `hotfix` 등 장기 브랜치 → 리리스 주기가 긴 프로덕트에 유리
  - Trunk-Based: `main` 한 줄, 짧은 feature branch + feature flag → 잦은 배포·CD에 유리
  - 팀 규모, 배포 빈도, QA 모델에 따라 선택
- **좋은 답변 예시**:
  - 모바일 앱·패키지처럼 명시적 릴리스 주기가 있으면 GitFlow가 자연스럽고, 매일 여러 번 배포하는 SaaS는 Trunk-Based + feature flag가 CD와 잘 맞습니다
- **꼬리 질문**:

  **Q. Trunk-Based에서 미완성 기능을 main에 머지해도 되나요?**
  - **좋은 답변**: 됩니다. 단 사용자에게 노출되지 않도록 feature flag로 가립니다. 코드가 분기되어 머지가 어려워지는 비용보다, 한 줄에서 통합 빌드/테스트를 도는 이득이 크다는 철학입니다.
  - **추가 질문**: feature flag를 도입하면 코드는 어떻게 관리하나요? 오래된 flag를 청소하는 방법은?

  **Q. PR(MR)을 작게 유지해야 하는 이유는?**
  - **좋은 답변**: 리뷰 품질이 PR 크기에 반비례합니다. 작을수록 리뷰어가 모든 라인을 보게 되고, 충돌이 줄고, 롤백 단위가 작아져 운영 리스크가 줄어듭니다. 통상 300줄 이하를 권장합니다.
  - **추가 질문**: 어쩔 수 없이 커진 PR은 어떻게 다루나요?

---

### Q11. `git rebase`와 `git merge`의 차이를 설명하고, 팀에서는 어떤 정책을 쓰는 게 좋다고 생각하나요?

- **의도**: Git 내부 동작 이해와 협업 정책 감각
- **핵심 포인트**:
  - merge는 히스토리 보존, rebase는 히스토리 선형화
  - 공유 브랜치에 rebase 후 force push는 다른 사람의 작업을 덮어쓸 수 있어 위험
  - "main에서 feature 브랜치 따올 때는 rebase, feature에서 main으로 합칠 때는 머지 커밋"이 흔한 절충안
- **좋은 답변 예시**:
  - 개인 브랜치는 rebase로 깨끗하게 정리, 공유 브랜치는 squash merge 또는 merge commit으로 PR 단위 추적성을 남깁니다
- **꼬리 질문**:

  **Q. Squash merge의 장단점은?**
  - **좋은 답변**: 장점은 main 히스토리가 PR 단위로 깔끔해지고 `git bisect`가 쉬워진다는 점입니다. 단점은 중간 커밋의 의도가 사라져서 디버깅 시 세부 변경을 추적하기 어렵다는 것입니다. PR 본문에 상세한 컨텍스트를 남기는 문화가 함께 가야 합니다.
  - **추가 질문**: `git bisect`를 사용해 본 적이 있나요?

---

### Q12. 실수로 비밀번호나 API 키를 커밋해서 origin에 push했습니다. 어떻게 대응하나요?

- **의도**: 보안 사고 대응 능력
- **핵심 포인트**:
  - 1순위는 키 무효화(rotate), 그다음이 히스토리 정리
  - `git reset`이나 `git rebase`만으로는 원격 히스토리에 흔적이 남음
  - `git filter-repo`, BFG Repo-Cleaner로 정리 후 force push
- **좋은 답변 예시**:
  - 먼저 해당 키/토큰을 즉시 폐기(rotate)하고, 영향 범위 점검(액세스 로그 확인) → 히스토리에서 제거 → 모든 클론에 알리고 다시 받게 함 → 사후로 pre-commit hook(`detect-secrets`, `gitleaks`)로 재발 방지
- **꼬리 질문**:

  **Q. pre-commit hook으로 secret 누출을 어떻게 막나요?**
  - **좋은 답변**: `pre-commit` 프레임워크에 `gitleaks`/`detect-secrets`/`trufflehog`를 등록하면 패턴 매칭으로 커밋 차단이 가능합니다. CI 파이프라인에도 같은 스캔을 넣어서 hook을 우회한 경우도 잡습니다.
  - **추가 질문**: 정규식 기반 스캐너의 false positive는 어떻게 관리하나요?

---

## 4. CI/CD 파이프라인 기본

### Q13. CI와 CD의 차이를 설명하고, 두 개념이 왜 분리되어 있는지 말씀해주세요.

- **의도**: 기본 개념과 파이프라인 설계 감각
- **핵심 포인트**:
  - CI: 코드 통합 단계의 자동화(빌드/테스트/정적분석)
  - Continuous Delivery: 언제든 배포 가능한 상태로 유지(수동 승인 후 배포)
  - Continuous Deployment: 통과한 변경을 자동으로 운영까지 배포
  - 분리되어야 운영 위험과 개발 속도의 균형을 잡을 수 있음
- **좋은 답변 예시**:
  - CI는 "코드가 항상 빌드/테스트를 통과하는 상태"를 보장하는 것이고, CD는 "그 코드를 운영까지 자동으로 보내는 것"입니다. 둘을 분리해야 운영 배포 직전에 수동 게이트를 둘 수 있습니다.
- **꼬리 질문**:

  **Q. Jenkins, GitHub Actions, GitLab CI를 비교해주세요.**
  - **좋은 답변**: Jenkins는 자유도 높은 self-hosted, 플러그인 생태계가 크지만 관리 부담이 큽니다. GitHub Actions는 GitHub 통합이 매끄럽고 Marketplace가 풍부하며 매니지드. GitLab CI는 코드 저장소와 CI/CD/Registry/Pages가 한 곳에 묶여 있어 통합성이 좋습니다. 팀이 어디서 코드를 관리하는지에 따라 자연스럽게 결정되는 경우가 많습니다.
  - **추가 질문**: self-hosted runner는 언제 필요한가요?

  **Q. 파이프라인 단계는 어떻게 나누는 게 좋다고 생각하나요?**
  - **좋은 답변**: 빠른 단계부터 배치합니다. lint → unit test → build → integration test → security scan → 배포. 빠른 피드백을 우선하고, 비싼 검증은 뒤로 미뤄 실패 시 전체 시간을 줄입니다. 병렬화 가능한 단계는 적극 병렬화합니다.
  - **추가 질문**: 테스트가 느려져서 CI 실행 시간이 30분이 넘는다면 어떻게 줄이나요?

---

### Q14. CI 파이프라인에서 시크릿(DB 비밀번호, API 키)을 안전하게 다루는 방법은?

- **의도**: 보안 의식과 시크릿 관리 기본기
- **핵심 포인트**:
  - 코드/이미지에 시크릿 절대 포함 금지
  - CI 도구의 secret store(GitHub Secrets, GitLab Variables)
  - 운영 시크릿은 Vault, AWS Secrets Manager, SealedSecrets 등 외부 저장소
  - 로그에 마스킹 처리되는지 확인
- **좋은 답변 예시**:
  - CI 단계에서는 비밀을 환경 변수로 주입하되, 로그 출력 시 자동 마스킹되는지 확인합니다. 운영 환경의 시크릿은 K8s Secret 자체가 아니라 외부 시크릿 매니저에서 주입되도록(External Secrets Operator) 구성합니다.
- **꼬리 질문**:

  **Q. K8s Secret이 base64 인코딩일 뿐 암호화가 아닌데, 어떻게 안전하게 쓰나요?**
  - **좋은 답변**: etcd 암호화를 활성화하고, RBAC로 Secret 접근 권한을 제한하며, 가능하면 SealedSecrets/External Secrets/Sealed-Secrets처럼 Git에 넣어도 안전한 형태를 씁니다. 노드에 평문으로 마운트되는 점은 변하지 않으므로 노드 자체의 보안도 같이 챙겨야 합니다.
  - **추가 질문**: Secret을 환경 변수로 주입하는 것과 파일로 마운트하는 것의 차이는?

---

### Q15. 빌드된 아티팩트(이미지, 패키지)는 어디에, 어떻게 보관하나요?

- **의도**: 아티팩트 관리와 추적성
- **핵심 포인트**:
  - 컨테이너 이미지는 이미지 레지스트리(ECR, GCR, Harbor)
  - 빌드 아티팩트는 Nexus, Artifactory
  - 태깅 정책 (`latest` 지양, 커밋 SHA 또는 시맨틱 버전)
  - 보존 정책 (오래된 이미지 자동 삭제)
- **좋은 답변 예시**:
  - 이미지 태그는 항상 immutable해야 합니다. `latest`는 디버깅 시 어떤 빌드인지 식별 불가능하므로 운영에 쓰면 안 되고, 보통 `<git-sha>` 또는 `<version>-<build>` 형식으로 태그를 박습니다.
- **꼬리 질문**:

  **Q. 이미지 태그를 `latest`로 쓰면 안 되는 이유는?**
  - **좋은 답변**: 롤백 시 어떤 이미지를 다시 띄워야 하는지 식별 불가능하고, K8s가 `imagePullPolicy: Always`가 아니면 캐시된 다른 이미지를 쓸 수 있어 의도와 다른 버전이 배포될 수 있습니다. 또 사고 조사 시 "그때의 latest가 무엇이었는가"를 알 수 없습니다.
  - **추가 질문**: 이미지 보존 정책은 어떻게 잡나요? 오래된 이미지를 자동 삭제할 때 주의점은?

  **Q. Multi-arch 이미지(amd64/arm64)는 어떻게 빌드하나요?**
  - **좋은 답변**: `docker buildx`로 QEMU를 통해 cross 빌드하거나, 각 아키텍처별 네이티브 러너에서 빌드 후 manifest list로 합칩니다. M1 Mac 개발과 ARM 인스턴스(Graviton 등) 운영이 늘면서 표준이 되고 있습니다.
  - **추가 질문**: 빌드 시간이 너무 길어지면 어떻게 줄이나요?

---

## 5. Docker & 컨테이너

### Q16. 컨테이너와 VM의 차이를 설명해주세요.

- **의도**: 가상화 계층에 대한 기본 이해
- **핵심 포인트**:
  - VM은 하이퍼바이저 위에 게스트 OS 전체를 띄움 → 격리는 강하지만 무겁고 부팅 느림
  - 컨테이너는 호스트 커널을 공유하고 namespace/cgroup으로 격리 → 가볍고 빠름
  - 컨테이너는 OS 격리가 아니라 프로세스 격리
- **좋은 답변 예시**:
  - 컨테이너는 결국 호스트 위의 프로세스이고, namespace(pid, net, mnt, uts, ipc, user)로 시야를, cgroup으로 자원 사용을 제한합니다. 그래서 부팅이 1초 미만이고 이미지가 수십 MB로 작은 대신, 커널 취약점이 있으면 격리가 깨질 수 있습니다.
- **꼬리 질문**:

  **Q. namespace와 cgroup의 역할 차이는?**
  - **좋은 답변**: namespace는 "무엇을 볼 수 있는가"(파일시스템, PID, 네트워크 인터페이스 등)를 격리하고, cgroup은 "얼마나 쓸 수 있는가"(CPU, 메모리, I/O)를 제한합니다. 두 개가 함께 있어야 우리가 아는 컨테이너가 됩니다.
  - **추가 질문**: rootless 컨테이너는 어떻게 동작하나요?

  **Q. Docker와 containerd, runc의 관계는?**
  - **좋은 답변**: runc는 OCI 스펙대로 실제 컨테이너를 만드는 저수준 런타임이고, containerd는 그 위에서 이미지 관리·라이프사이클을 다루는 고수준 런타임입니다. Docker는 사용자 친화 CLI/API 레이어로 내부적으로 containerd → runc를 호출합니다. K8s는 1.24부터 dockershim을 제거하고 containerd/CRI-O와 직접 통신합니다.
  - **추가 질문**: K8s가 dockershim을 제거한 배경은?

---

### Q17. Dockerfile을 작성할 때 이미지 크기를 줄이고 빌드 시간을 단축하는 방법은?

- **의도**: 이미지 최적화 경험
- **핵심 포인트**:
  - 멀티 스테이지 빌드로 빌드 도구와 런타임 분리
  - 작은 베이스 이미지(distroless, alpine)
  - 레이어 캐시를 활용하는 명령 순서 (의존성 설치를 앞쪽에)
  - `.dockerignore`로 불필요한 컨텍스트 제외
- **좋은 답변 예시**:
  - 의존성 설치 단계와 소스 복사 단계를 분리해서 코드만 바뀌었을 때 의존성 캐시가 살아 있도록 합니다. 빌드 산출물만 최종 이미지에 복사하면 JDK 같은 빌드 도구가 운영 이미지에 들어가지 않습니다.
- **꼬리 질문**:

  **Q. Alpine 이미지가 작지만 가끔 문제가 되는 이유는?**
  - **좋은 답변**: Alpine은 musl libc를 쓰는데, glibc를 가정한 일부 바이너리(예: 일부 JNI 라이브러리, Python wheel)가 호환되지 않을 수 있습니다. 그래서 `gcompat`를 추가하거나 `debian-slim`을 선택하기도 합니다. DNS resolver 동작도 미묘하게 달라 컨테이너 안에서 DNS 이슈를 만들 수 있습니다.
  - **추가 질문**: distroless 이미지는 어떤 장단점이 있나요?

  **Q. 이미지에서 보안 취약점을 찾으려면 어떻게 하나요?**
  - **좋은 답변**: Trivy, Grype, Snyk 같은 스캐너를 CI에 넣어 CVE를 검출합니다. 베이스 이미지 버전을 정기적으로 올리는 자동화(Renovate 등)도 함께 운영합니다.
  - **추가 질문**: 스캔에서 잡힌 CVE를 모두 막을 수는 없는데, 우선순위는 어떻게 정하나요?

---

### Q18. 컨테이너 안에서 데이터를 영구 저장하려면 어떻게 해야 하나요?

- **의도**: 볼륨과 상태 관리 이해
- **핵심 포인트**:
  - 컨테이너 라이프사이클과 데이터 라이프사이클 분리
  - 볼륨 종류: bind mount, named volume, tmpfs
  - K8s에서는 PV/PVC, StorageClass
- **좋은 답변 예시**:
  - 컨테이너는 일회용으로 보고 상태는 외부에 둔다는 원칙으로 시작합니다. 로컬 개발은 bind mount, 운영은 K8s PV/PVC를 통해 EBS, EFS, S3 등 외부 스토리지로 연결합니다.
- **꼬리 질문**:

  **Q. 컨테이너에 DB를 띄워서 운영하는 게 권장되지 않는 이유는?**
  - **좋은 답변**: 상태가 있는 서비스라 노드 장애·재스케줄 시 데이터 일관성·성능 보장이 어렵고, 백업·HA·튜닝 운영 부담이 매니지드 DB 대비 큽니다. K8s StatefulSet + operator(예: Crunchy PG)로 가능하긴 하지만, 매니지드(RDS, Cloud SQL)로 빼는 게 일반적인 선택입니다.
  - **추가 질문**: 그럼에도 K8s에서 stateful 워크로드를 운영해야 한다면 무엇을 챙겨야 하나요?

---

### Q19. 컨테이너 안에서 PID 1로 자식 프로세스를 띄울 때 주의할 점은?

- **의도**: 시그널과 좀비 프로세스에 대한 깊이
- **핵심 포인트**:
  - PID 1은 시그널 기본 핸들러가 없어 SIGTERM 전달이 안 될 수 있음
  - 자식 프로세스가 죽으면 PID 1이 reap해야 좀비 안 생김
  - `tini`, `dumb-init` 같은 init 시스템을 ENTRYPOINT에 두는 패턴
- **좋은 답변 예시**:
  - 쉘 스크립트를 ENTRYPOINT로 두면 SIGTERM이 자식 앱까지 전파되지 않아 graceful shutdown이 깨질 수 있습니다. `exec` 키워드를 쓰거나 `tini` 같은 init을 두면 PID 1 문제를 회피할 수 있습니다.
- **꼬리 질문**:

  **Q. 컨테이너 안에서 cron을 돌리는 게 안티패턴이라는 이야기를 들어본 적 있나요?**
  - **좋은 답변**: 컨테이너는 단일 책임 원칙으로 운영하는 게 좋고, cron은 K8s `CronJob`이나 외부 스케줄러(Argo Workflows 등)로 빼는 게 일반적입니다. 컨테이너 내부 cron은 로그·실패 알림·재시도 정책을 컨테이너 안에서 다시 만들어야 해서 불리합니다.
  - **추가 질문**: K8s CronJob에서 동시 실행을 막으려면 어떻게 하나요?

---

## 6. Kubernetes 기초

### Q20. `kubectl apply -f deployment.yaml` 한 줄로 Pod이 뜨기까지 K8s 내부에서 어떤 일이 일어나는지 아는 범위에서 설명해주세요.

- **의도**: K8s 컨트롤 플레인의 동작을 머릿속에 그릴 수 있는지 평가 (basic의 도입 질문)
- **핵심 포인트**:
  - kubectl → API Server (인증·인가·admission webhook → etcd 저장)
  - Deployment Controller → ReplicaSet → Pod 오브젝트 생성
  - Scheduler가 Pod을 노드에 배치
  - kubelet이 컨테이너 런타임(containerd)에 컨테이너 생성 요청
  - kube-proxy가 Service 라우팅 갱신
- **좋은 답변 예시**:
  - 매니페스트가 API Server를 통과하면서 admission controller로 검증/변형되고, etcd에 desired state로 저장됩니다. Deployment 컨트롤러가 그 변화를 감지해 ReplicaSet/Pod 오브젝트를 만들고, 스케줄러가 노드를 골라 Pod에 nodeName을 박아 넣습니다. 해당 노드의 kubelet이 컨테이너 런타임을 호출해 실제 컨테이너를 띄우고, readiness가 통과되면 Service의 endpoint에 포함되어 트래픽을 받습니다.
- **꼬리 질문**:

  **Q. Pod이 `Pending` 상태로 멈춰 있을 때 어떤 순서로 진단하나요?**
  - **좋은 답변**: `kubectl describe pod <name>`의 Events를 먼저 봅니다. "0/X nodes are available" 메시지면 스케줄링 실패라 노드 자원 부족, taint 미허용, nodeSelector/affinity 미스매치 등을 확인합니다. 이미지 풀 단계라면 ImagePullBackOff 메시지가 보이고, PVC 바인딩 대기일 수도 있습니다.
  - **추가 질문**: "0/X nodes are available, X Insufficient memory"가 떴을 때 무엇을 봐야 하나요?

  **Q. Deployment와 ReplicaSet의 관계는?**
  - **좋은 답변**: Deployment가 상위 추상화이고 실제 Pod 복제는 ReplicaSet이 관리합니다. Deployment 이미지를 바꾸면 새 ReplicaSet이 생기고, 기존 ReplicaSet은 보존되어 롤백 시 그 ReplicaSet으로 되돌립니다.
  - **추가 질문**: `kubectl rollout undo`는 어떻게 동작하나요?

---

### Q21. `Service`와 `Endpoint`의 관계, 그리고 트래픽이 Pod까지 어떻게 도달하는지 설명해주세요.

- **의도**: K8s 네트워킹 내부 동작 이해
- **핵심 포인트**:
  - Service는 Pod selector로 묶이는 가상 IP
  - Endpoint(EndpointSlice)는 selector에 매칭된 Pod IP 목록
  - kube-proxy가 iptables 또는 IPVS 룰을 노드에 깔아 ClusterIP로 들어온 트래픽을 실제 Pod IP로 분배
- **좋은 답변 예시**:
  - Service ClusterIP는 실재하는 인터페이스가 아니라 kube-proxy가 iptables/IPVS로 만들어둔 가상 주소입니다. Pod이 떴다 죽었다 할 때 EndpointSlice가 갱신되고 kube-proxy가 룰을 업데이트해서 트래픽이 살아 있는 Pod으로만 갑니다.
- **꼬리 질문**:

  **Q. Headless Service(`clusterIP: None`)는 언제 쓰나요?**
  - **좋은 답변**: 클라이언트가 Pod IP를 직접 알아야 하는 경우(예: StatefulSet의 각 인스턴스에 직접 접속, Cassandra·Kafka 같은 클러스터링) DNS가 ClusterIP 대신 Pod IP들의 A 레코드를 그대로 반환합니다.
  - **추가 질문**: StatefulSet에서 Pod 이름은 어떻게 안정적으로 유지되나요?

  **Q. Service의 `externalTrafficPolicy: Local`은 어떤 효과를 주나요?**
  - **좋은 답변**: NodePort/LoadBalancer로 들어온 트래픽을 그 노드에 있는 Pod으로만 전달합니다. 클라이언트 IP를 보존할 수 있지만, Pod이 없는 노드에 트래픽이 들어오면 drop됩니다. 헬스체크와 같이 신중히 써야 합니다.
  - **추가 질문**: `Cluster`와 `Local`의 트래픽 분포 차이는?

---

### Q22. ConfigMap과 Secret의 차이는 무엇이고, 애플리케이션에 어떻게 주입하나요?

- **의도**: 설정 분리 원칙과 운영 패턴
- **핵심 포인트**:
  - ConfigMap: 비밀이 아닌 설정값
  - Secret: 비밀 데이터, base64로 인코딩(암호화 아님), etcd 암호화 옵션 필요
  - 환경 변수 주입 vs 파일 마운트
- **좋은 답변 예시**:
  - ConfigMap은 환경 변수 또는 파일로 주입할 수 있고, Secret도 동일하게 주입 가능하지만 가능하면 파일 마운트가 좋습니다(환경 변수는 자식 프로세스나 코어 덤프로 노출 위험). 설정이 바뀌면 Pod을 재시작하거나 reloader 같은 도구로 자동 롤링.
- **꼬리 질문**:

  **Q. ConfigMap을 바꿨는데 Pod이 새 값을 못 읽는 경우가 있나요?**
  - **좋은 답변**: 환경 변수로 주입한 경우는 Pod 재시작 전까지 옛 값 그대로입니다. 파일 마운트는 kubelet이 주기적으로 갱신하지만 애플리케이션이 파일을 다시 읽도록 reload 시그널을 받아야 합니다. 재시작이 필요한 변경은 `stakater/reloader`로 자동화하기도 합니다.
  - **추가 질문**: 운영에서 ConfigMap 변경으로 사고가 났던 경험이 있나요?

---

### Q23. Liveness Probe와 Readiness Probe의 차이는 무엇이고, 잘못 설정하면 어떤 사고가 날 수 있나요?

- **의도**: 헬스체크 설계와 운영 안정성
- **핵심 포인트**:
  - Liveness 실패 → 컨테이너 재시작
  - Readiness 실패 → Service endpoint에서 제외(트래픽 차단), 컨테이너는 살아 있음
  - Startup probe로 초기 부팅 시간이 긴 앱 보호
- **좋은 답변 예시**:
  - Liveness를 너무 공격적으로 잡으면 부팅 중인 앱을 무한 재시작하는 사고가 납니다. Startup probe로 초기 기간을 보호하거나 `initialDelaySeconds`를 충분히 주고, Liveness는 정말 "프로세스가 죽은 것과 다름없는 상태"에서만 실패하도록 보수적으로 잡습니다.
- **꼬리 질문**:

  **Q. Readiness probe만 두고 Liveness probe를 안 쓰는 전략도 있다고 들었는데, 어떻게 생각하세요?**
  - **좋은 답변**: 합리적인 입장입니다. 회복 가능한 상태인지 애플리케이션 스스로 판단하기 어렵고, 잘못된 liveness가 무한 재시작 루프를 만드는 사고를 키우기 쉬워서, 정말 명확한 deadlock 감지 같은 경우가 아니면 Liveness를 빼는 팀도 많습니다.
  - **추가 질문**: 그 대신 노드 레벨 모니터링·알람으로 보완해야 할 것은 무엇인가요?

---

### Q24. Pod이 `CrashLoopBackOff` 상태입니다. 어떤 순서로 디버깅하나요?

- **의도**: 실전 트러블슈팅 절차
- **핵심 포인트**:
  - `kubectl describe pod` → Events, exit code
  - `kubectl logs <pod> --previous`로 직전 인스턴스 로그 확인
  - 이미지 잘못, env/Secret 누락, 의존 서비스(DB) 연결 실패가 흔한 원인
- **좋은 답변 예시**:
  - Events로 ImagePullBackOff/OOMKilled/Error 같은 원인 단서를 먼저 보고, `--previous`로 종료 직전 로그를 봅니다. exit code 137이면 OOMKilled, 1·2면 애플리케이션 비정상 종료. Init container가 막혀 있는 경우도 흔합니다.
- **꼬리 질문**:

  **Q. exit code 137과 143의 차이는?**
  - **좋은 답변**: 137은 SIGKILL(주로 OOMKill), 143은 SIGTERM 종료입니다. 137이 나오면 메모리 limit이 부족하거나, 노드 메모리 압박으로 evict된 경우입니다.
  - **추가 질문**: OOMKilled가 났을 때 메모리를 더 주는 것 외에 어떤 접근이 가능한가요?

---

## 7. 모니터링 / 로그 기초

### Q25. 모니터링에서 "메트릭, 로그, 트레이스"가 어떻게 다른지 설명해주세요.

- **의도**: 관측성(Observability) 3개 축에 대한 이해
- **핵심 포인트**:
  - 메트릭: 시계열 숫자, 집계에 적합, 카디널리티 제한
  - 로그: 텍스트 이벤트, 풍부한 컨텍스트, 검색 비용 큼
  - 트레이스: 분산 시스템에서 한 요청의 흐름, 서비스 간 호출 추적
  - 세 가지를 traceId로 연결하면 "지표 이상 감지 → 트레이스로 좁히기 → 로그로 자세히 보기"가 가능
- **좋은 답변 예시**:
  - 메트릭으로 "지금 무엇이 이상한가"를 보고, 트레이스로 "어떤 요청 경로에서"를 좁히고, 로그로 "왜"를 봅니다. 셋이 traceId로 연결돼야 진단 시간이 짧아집니다.
- **꼬리 질문**:

  **Q. RED/USE 메서드를 들어본 적 있나요?**
  - **좋은 답변**: RED는 요청 단위 지표(Rate, Errors, Duration)로 서비스 헬스 보기에 좋고, USE는 자원 단위(Utilization, Saturation, Errors)로 인프라 헬스 보기에 좋습니다. 보통 서비스는 RED, 노드/디스크/네트워크는 USE로 대시보드를 짭니다.
  - **추가 질문**: P99 latency를 보는 이유와 평균 latency만 보면 안 되는 이유는?

  **Q. 로그를 보기 좋은 형태로 남기려면 어떤 원칙을 따라야 하나요?**
  - **좋은 답변**: 구조화 로그(JSON)로 남기고, traceId/userId 같은 컨텍스트를 모든 라인에 포함시킵니다. 로그 레벨을 일관되게 사용하고, 메시지에는 가변값을 직접 넣지 않고 필드로 분리해서 검색·집계가 가능하게 합니다.
  - **추가 질문**: 로그에 PII(개인정보)가 들어가지 않게 어떻게 막나요?

---

### Q26. Prometheus는 어떻게 메트릭을 수집하나요? (Pull vs Push)

- **의도**: Prometheus 동작 원리 이해
- **핵심 포인트**:
  - 기본은 Pull: Prometheus가 `/metrics` 엔드포인트를 주기적으로 스크랩
  - Push가 필요한 경우(짧은 job): Pushgateway
  - 서비스 디스커버리로 K8s/Consul 등에서 타깃 자동 발견
- **좋은 답변 예시**:
  - Pull 모델은 타깃이 살아 있는지를 자체적으로 알 수 있고, 타깃 측 구현이 단순합니다. Push 모델은 단명 job 같은 경우에만 Pushgateway로 우회합니다.
- **꼬리 질문**:

  **Q. PromQL의 `rate()`와 `increase()` 차이는?**
  - **좋은 답변**: `rate(metric[5m])`은 5분 윈도우의 초당 평균 증가율, `increase(metric[5m])`은 같은 윈도우의 총 증가량(`rate * 윈도우`)입니다. counter가 reset되어도 보정합니다. 알람용으로는 `rate`가 일반적이고, "지난 1시간 동안 몇 번 일어났나"는 `increase`가 자연스럽습니다.
  - **추가 질문**: counter와 gauge의 차이는?

  **Q. 라벨을 너무 많이 붙이면 어떤 문제가 생기나요? (카디널리티)**
  - **좋은 답변**: 메트릭 + 라벨 조합 수만큼 시계열이 만들어집니다. userId나 traceId 같은 고유값을 라벨로 넣으면 시계열이 무한히 늘어나 메모리·디스크가 폭발합니다. 카디널리티가 높은 차원은 로그/트레이스로 옮기고, 메트릭에는 enum 가능한 차원만 둡니다.
  - **추가 질문**: 운영 중인 Prometheus가 메모리 부족으로 죽는다면 어디부터 보나요?

---

### Q27. Grafana 대시보드를 만들 때 일반적으로 어떤 패널 구성을 하나요?

- **의도**: 운영 대시보드 설계 감각
- **핵심 포인트**:
  - 한 화면에 RED 지표(요청량, 에러율, 지연)
  - 의존 자원(DB, 캐시) 지연/에러
  - 인프라 USE 지표(CPU, 메모리, 디스크)
  - 변수(Variable)로 환경/서비스 전환 가능하게
- **좋은 답변 예시**:
  - "한눈에 서비스 상태가 보이는" 대시보드를 목표로 합니다. 위에는 RED(요청량·에러율·P99) 큰 패널, 아래에 의존성과 인프라. 너무 많은 패널은 오히려 보지 않게 되므로 SLO에 해당하는 지표 위주로 정리합니다.
- **꼬리 질문**:

  **Q. 알람은 어디서 정의하는 게 좋다고 생각하세요? Grafana? Prometheus Alertmanager?**
  - **좋은 답변**: 알람 룰은 Prometheus(또는 Mimir/Thanos)에 두고, Alertmanager가 라우팅/그루핑/사일런스를 담당하는 구조가 표준입니다. Grafana 알람은 대시보드 패널 기준이라 데이터 소스가 여러 개일 때 유연하지만, 운영 룰은 Git에서 코드로 관리되는 Prometheus 룰 쪽이 추적하기 좋습니다.
  - **추가 질문**: 알람 룰을 코드(GitOps)로 관리하는 이점은?

---

### Q28. 운영에서 traceId로 로그를 추적해본 경험을 설명해주세요.

- **의도**: 분산 추적 실전 감각
- **핵심 포인트**:
  - traceId 발급(엔트리 포인트에서) → 모든 서비스가 전파(W3C `traceparent` 헤더)
  - 로그에 traceId 필드 포함
  - Grafana에서 메트릭 → Loki traceId 검색 → Tempo/Jaeger 트레이스로 점프
- **좋은 답변 예시**:
  - 사용자가 신고한 단일 요청 traceId로 시작해서, API → 인증 → 결제 → DB까지 어느 단계가 느렸는지를 트레이스로 보고, 같은 traceId로 각 서비스 로그를 한 데 모아 원인을 좁혔습니다.
- **꼬리 질문**:

  **Q. 모든 요청에 분산 트레이스를 켜면 부담이 큰데, 어떻게 다루나요?**
  - **좋은 답변**: 샘플링을 씁니다. 헤드 샘플링(1% 같은 비율)이 가장 흔하고, 에러 발생 시는 무조건 샘플링하는 tail-based sampling도 있습니다. 운영 비용과 진단 가치의 균형을 보는 일입니다.
  - **추가 질문**: tail-based sampling은 어떻게 구현되나요?

---

## 8. 배포 전략 기초

### Q29. Rolling, Blue/Green, Canary 배포의 차이를 설명해주세요.

- **의도**: 배포 전략 비교와 선택 감각
- **핵심 포인트**:
  - Rolling: 점진적 교체 (K8s 기본), 자원 적게 쓰지만 두 버전이 동시 존재
  - Blue/Green: 두 환경 운영 → LB 스위치, 빠른 롤백, 자원 두 배
  - Canary: 새 버전에 일부 트래픽만 → 검증 후 점진 확대, 가장 안전하지만 라우팅 인프라 필요
- **좋은 답변 예시**:
  - 단순 stateless 서비스는 Rolling으로 충분합니다. 큰 위험을 가진 변경(스키마 동반 변경 등)은 Blue/Green이 롤백이 빠르고, 사용자 영향을 단계적으로 확인하고 싶으면 Canary가 가장 안전합니다.
- **꼬리 질문**:

  **Q. K8s Deployment의 `maxUnavailable`과 `maxSurge`는 각각 어떤 의미인가요?**
  - **좋은 답변**: 롤링 업데이트 중 동시에 내릴 수 있는 Pod 수(`maxUnavailable`)와 일시적으로 늘릴 수 있는 Pod 수(`maxSurge`)를 정합니다. 가용성을 최우선으로 하면 unavailable=0, surge=1 같이 설정하고, 자원이 빠듯하면 unavailable=1로 두기도 합니다.
  - **추가 질문**: Pod이 한 개뿐인 Deployment에서 rolling update가 어떻게 동작하나요?

  **Q. Canary 배포에서 트래픽을 5% → 25% → 100%로 늘릴 때 무엇을 기준으로 다음 단계로 갈지 판단하나요?**
  - **좋은 답변**: 에러율과 latency가 기존 버전 대비 악화되지 않는지, 비즈니스 지표(전환율 등)가 떨어지지 않는지를 자동으로 비교합니다. Argo Rollouts/Flagger 같은 도구가 메트릭 분석(analysis template)으로 자동 promote/rollback을 처리합니다.
  - **추가 질문**: 트래픽 비율은 어떤 계층에서 조절하나요? Ingress? Service Mesh?

---

### Q30. 배포 후 문제가 발생했을 때 롤백 절차는 어떻게 설계해야 하나요?

- **의도**: 롤백 전략과 운영 성숙도
- **핵심 포인트**:
  - 코드 롤백 vs 데이터/스키마 롤백 분리 (스키마는 backward compatible이 원칙)
  - 자동 롤백 기준 정의 (헬스체크, 에러율)
  - 사람의 결정이 필요한 케이스와 자동화의 경계
- **좋은 답변 예시**:
  - 가장 중요한 원칙은 "DB 마이그레이션은 항상 이전 코드와 호환되게" 만들어 두는 것입니다. 코드만 되돌려도 안전하게 복귀할 수 있습니다. K8s `kubectl rollout undo`로 빠른 롤백, GitOps 환경에서는 Git revert로 동일 효과를 냅니다.
- **꼬리 질문**:

  **Q. "DB 컬럼 삭제" 같은 비가역 변경을 안전하게 배포하려면 어떻게 하나요?**
  - **좋은 답변**: 한 번에 삭제하지 않고 단계로 나눕니다. ① 새 코드에서 해당 컬럼을 더 이상 사용하지 않게 배포 → ② 모든 인스턴스가 새 코드인지 확인 → ③ 며칠 관찰 → ④ 컬럼 삭제. 도중에 문제가 생겨도 되돌릴 곳이 있도록 합니다.
  - **추가 질문**: 이 패턴을 expand-contract 패턴이라고도 부르는데, 들어본 적 있나요?

  **Q. 배포 직후 모니터링해야 할 핵심 지표 3가지만 꼽는다면?**
  - **좋은 답변**: 에러율(특히 5xx와 4xx 급증), P99 latency, 핵심 비즈니스 지표(주문/결제 같은 도메인 KPI)입니다. 인프라 지표는 그다음입니다. 사용자 관점에서 문제가 보이는 지표를 가장 앞에 둡니다.
  - **추가 질문**: 비즈니스 지표가 떨어지지만 시스템 지표는 모두 정상인 경우 어떻게 진단하나요?

---

### Q31. 무중단 배포에서 in-flight 요청이 끊기지 않게 하려면 무엇이 필요한가요?

- **의도**: 종료 흐름의 디테일
- **핵심 포인트**:
  - preStop hook → readiness off → 트래픽 차단 후 진행 중 요청 마무리 → SIGTERM
  - `terminationGracePeriodSeconds`를 애플리케이션 graceful timeout보다 길게
  - LB의 connection drain 시간 고려
- **좋은 답변 예시**:
  - K8s가 SIGTERM을 보내기 직전에 readinessProbe가 먼저 실패하도록 preStop에 짧은 sleep을 두는 패턴이 일반적입니다. LB가 endpoint 갱신과 동시에 connection drain을 하는 동안 진행 중 요청을 마저 처리한 뒤 종료됩니다.
- **꼬리 질문**:

  **Q. WebSocket·gRPC 같은 long-lived 커넥션은 무중단 배포 시 어떻게 처리하나요?**
  - **좋은 답변**: 서버가 GOAWAY/close 프레임으로 클라이언트에게 재연결을 요청하고, 클라이언트는 재연결 정책으로 다른 인스턴스에 붙습니다. 단순 HTTP보다 graceful 시간이 더 길게 필요하고, 클라이언트 측 재시도 로직과 함께 설계해야 합니다.
  - **추가 질문**: 진행 중 큰 파일 업로드처럼 시간이 오래 걸리는 요청은 어떻게 다루나요?

---

## 🔗 바로가기

- [데브옵스 가이드 인덱스](../README.md)
- [데브옵스 심화 질문](../advanced/README.md)
- [데브옵스 고려사항 질문](../considerations/README.md)
- [전체 인터뷰 인덱스](../../README.md)
