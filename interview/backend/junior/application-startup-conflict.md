# 설정 변경 후 애플리케이션이 기동하지 않는다

[이 폴더 목차](README.md) · [전체 진행 방법](../../README.md)

## 1. 빠른 안내

| 평가 역량 | 난이도 | 예상 시간 |
|---|---|---|
| DI · 자동 구성 · Bean 생명주기 | 기본 → 심화 | 기본 10~15분 / 심화 15~20분 |

<a id="basic"></a>

**기본 진행**: 상황과 첫 질문 → 확인 질문에 맞춰 추가 정보 제공 → 답변에 맞는 분기 1~2개 → 평가·경험 연결. 답변을 먼저 듣고 해설은 읽어주지 않습니다.

**심화 진행**: 같은 상황에서 기본 가설을 짧게 확인한 뒤 아래 심화 조건과 표의 `→` 뒤 질문을 선택합니다. 예상 시간은 기본과 심화를 합산한 시간이 아니라 각 진행 경로의 시간입니다. 모든 분기를 묻지 않습니다.

## 2. 지원자에게 읽어줄 상황과 첫 질문

> 공통 라이브러리를 추가한 뒤 개발 서버는 뜨지만 운영 서버는 기동에 실패합니다. 급히 배포를 복구해야 합니다. 어떤 증거를 모으고 변경 범위를 줄이겠습니까?

## 3. 면접관용 추가 정보

JDK 21, Spring Boot 3.5.7입니다. 환경별 설정 파일과 라이브러리 버전이 다릅니다. 실패 로그에는 Bean 후보 충돌이 있습니다. 새 라이브러리는 DataSource Bean과 설정 바인딩 클래스를 제공합니다.

정보가 없는 질문에는 정답을 추측해 주지 말고 지원자의 가정을 명시하게 합니다. 같은 면접에서는 합의한 조건을 유지합니다.

<a id="deep"></a>

**심화에서 추가할 조건**: 충돌을 해결하자 두 서비스의 생성자 순환 의존성이 드러납니다. 초기화가 오래 걸리고 임시 작업 객체의 자원도 반환되지 않습니다.

## 4. 기본 답변 예시와 해설

의존성 그래프·활성 프로파일·설정 우선순위와 조건 평가 보고서를 비교합니다. 자동 구성은 클래스패스와 조건에 따라 적용되므로 사용자 Bean과의 관계를 확인합니다. 무조건 전체 자동 구성을 끄기보다 충돌하는 구성만 제거하거나 명시적으로 선택합니다. 생성자 주입으로 필수 의존성을 드러내고 순환 책임을 분리합니다. 실제 운영 설정을 민감값 없이 재현한 기동 테스트로 검증합니다.

이는 가능한 접근의 예입니다. 다른 해법도 조건·선택 이유·실패 한계·검증이 연결되면 인정합니다.

<details>
<summary>면접관 보충 해설</summary>

Boot 3.5.7의 자동 구성 후보 파일은 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports입니다. @ConfigurationProperties로 묶음 설정을 바인딩·검증하고 @Value는 개별 값에 사용할 수 있습니다. 자동 구성 순서는 before/after 등으로 조절하되 Bean 생성 순서와 동일시하지 않습니다.

초기화 자원 누수 분기에서는 @PostConstruct/InitializingBean과 @PreDestroy/DisposableBean의 호출 목적, request/session 스코프의 수명도 묻습니다. singleton은 컨테이너 내 공유 인스턴스이며 prototype을 한 번 주입받으면 그 참조가 자동으로 교체되지 않습니다. @Lazy는 초기화를 미루므로 장애 발견과 첫 요청 지연을 옮길 수 있습니다.

</details>

기술 확인 출처: [Boot 3.5 자동 구성](https://docs.spring.io/spring-boot/3.5/reference/using/auto-configuration.html), [Framework 6.2 스코프](https://docs.spring.io/spring-framework/reference/6.2/core/beans/factory-scopes.html)

추가 기술 확인 출처: [Boot 3.5.7 자동 구성 후보 파일](https://github.com/spring-projects/spring-boot/blob/v3.5.7/spring-boot-project/spring-boot-docs/src/docs/antora/modules/reference/pages/features/developing-auto-configuration.adoc)

## 5. 답변별 꼬리 질문

`→` 뒤는 앞 질문에 답했을 때 이어갈 한 단계 깊은 질문입니다. 모르는 답변에는 추가 조건을 하나 주고 추론이 어떻게 바뀌는지 확인합니다.

| 지원자 답변 또는 접근 | 다음 질문 | 확인할 핵심과 판단 근거 |
|---|---|---|
| 자동 구성을 제외합니다 | 어떤 조건 때문에 Bean이 생겼나요? → 사용자 Bean 등록 때 모든 자동 구성이 사라지나요? | @SpringBootApplication의 scan/configuration/auto-configuration 역할, ConditionalOnClass/MissingBean과 개별 조건을 구분한다. |
| 필드 주입으로 바꿉니다 | 순환 책임은 해결됐나요? → 단일 생성자와 Lombok 생성자는 어떻게 선택하나요? | 생성자·세터·필드 주입의 필수성/변경 가능성/테스트 비용을 설명한다. RequiredArgsConstructor와 AllArgsConstructor의 필드 범위를 구분한다. |
| Bean을 찾지 못합니다 | 메인 클래스와 스캔 경계는 어디인가요? → Repository와 Service를 나눈 이유는요? | 스테레오타입 역할과 Repository 예외 변환을 설명하고 basePackages를 무분별하게 넓히지 않는다. |
| Lazy 또는 prototype으로 해결합니다 | 문제가 첫 요청으로 미뤄질 뿐인가요? → singleton에 주입된 prototype은 매번 새로 생기나요? | 초기화·소멸과 스코프를 구분한다. prototype 파괴 콜백은 컨테이너가 관리하지 않으며 필요하면 provider로 획득한다. |

## 6. 평가 기준

| 구분 | 관찰할 행동과 설명 |
|---|---|
| 기본 이해 | 로그와 설정 차이로 Bean 충돌을 좁힌다. |
| 깊이 있는 이해 | 설정 바인딩·조건 평가·스코프를 근거로 책임을 분리한다. |
| 추가 확인 필요 | 순환 참조 허용이나 Lazy만 켜고 해결됐다고 말한다. |

기술명 언급 수로 점수를 매기지 않습니다. 답변에서 관찰한 근거, 제공한 힌트, 남은 불확실성을 따로 적습니다. 질문하지 않은 역량은 미평가로 남깁니다.

## 7. 실제 경험으로 연결

- 이 상황과 비슷한 경험에서 제약과 본인의 역할은 무엇이었습니까?
- 직접 내린 결정·수행한 검증과 동료의 기여를 구분해 설명해 주세요.
- 결과를 어떤 수치나 관찰로 확인했으며, 다시 한다면 무엇을 바꾸겠습니까?

직접 경험이 없으면 유사한 학습·실험 경험 또는 이 상황의 검증 계획으로 이어갑니다.
