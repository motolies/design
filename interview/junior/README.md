# 주니어 개발자 면접 질문

## 🎯 개요

**대상**: 0-3년차 Java/Spring 개발자
**평가 목표**: 기본 개념 이해, 실무 적용 능력, 학습 의지

---

## 1. Spring Framework 기초

### Q1. URL로 요청이 들어올 때부터 응답이 나갈 때까지 Spring의 동작을 아는 범위에서 전부 설명해주세요.

- **의도**: Spring MVC의 요청 처리 흐름에 대한 전반적인 이해도 평가
- **핵심 포인트**:
  - DispatcherServlet의 역할
  - HandlerMapping → HandlerAdapter → Controller → ViewResolver 흐름
  - Filter와 Interceptor의 위치
- **좋은 답변 예시**:
  - 클라이언트 요청 → Tomcat → Filter → DispatcherServlet → HandlerMapping → HandlerAdapter → Controller → Service → Repository → 응답 반환
  - 각 단계에서 어떤 처리가 일어나는지 설명
- **꼬리 질문**:

  **Q. Filter와 Interceptor의 차이점은 무엇인가요?**
  - **좋은 답변**: Filter는 서블릿 컨테이너(Tomcat) 레벨에서 동작하여 Spring Context 외부에 있고, Interceptor는 Spring Context 내부의 DispatcherServlet 이후에 동작합니다. Filter는 모든 요청에 적용되고, Interceptor는 특정 핸들러에만 적용 가능합니다. Filter에서는 Spring Bean 주입이 어렵지만(DelegatingFilterProxy 필요), Interceptor는 Spring Bean으로 등록되어 다른 Bean 주입이 쉽습니다.
  - **추가 질문**: 어떤 경우에 Filter를 사용하고 어떤 경우에 Interceptor를 사용하나요?

  **Q. `@RestController`와 `@Controller`의 차이는?**
  - **좋은 답변**: `@RestController` = `@Controller` + `@ResponseBody`입니다. `@Controller`는 View를 반환하고, `@RestController`는 객체를 JSON/XML로 직렬화하여 HTTP Response Body에 직접 반환합니다. REST API 개발 시 `@RestController`를 사용합니다.
  - **추가 질문**: `@Controller`에서 JSON을 반환하려면 어떻게 해야 하나요?

---

### Q2. Spring Security가 추가되면 위 흐름이 어떻게 달라지나요?

- **의도**: Spring Security의 기본 동작 원리 이해도 평가
- **핵심 포인트**:
  - SecurityFilterChain의 위치 (Filter 레벨)
  - 인증(Authentication)과 인가(Authorization)의 차이
  - SecurityContext의 역할
- **좋은 답변 예시**:
  - Filter Chain에 Security Filter들이 추가됨
  - UsernamePasswordAuthenticationFilter, BasicAuthenticationFilter 등
  - 인증 성공 시 SecurityContext에 Authentication 객체 저장
- **꼬리 질문**:

  **Q. 인증과 인가의 차이를 설명해주세요**
  - **좋은 답변**: 인증(Authentication)은 "누구인가?"를 확인하는 것으로 로그인 과정입니다. 인가(Authorization)는 "무엇을 할 수 있는가?"를 확인하는 것으로 권한 검사입니다. 인증이 먼저 이루어지고, 그 다음 인가가 수행됩니다. 예를 들어 로그인(인증) 후 관리자 페이지 접근 권한 체크(인가)가 이루어집니다.
  - **추가 질문**: Spring Security에서 인가를 처리하는 방법에는 어떤 것들이 있나요?

  **Q. Session 기반 인증과 Token 기반 인증의 차이는?**
  - **좋은 답변**: Session 기반은 서버에 세션 정보를 저장(Stateful)하고, Token 기반은 서버에 상태를 저장하지 않습니다(Stateless). Session은 서버 확장 시 세션 동기화 문제가 있고, Token은 서버 확장이 쉽습니다. Session은 탈취 시 서버에서 무효화 가능하지만, Token(JWT)은 만료까지 무효화가 어렵습니다.
  - **추가 질문**: Session 클러스터링을 해결하는 방법에는 어떤 것들이 있나요?

---

### Q3. Spring Boot의 자동 구성(Auto-configuration) 기능이 어떻게 동작하는지 설명해주세요.

- **의도**: Spring Boot의 핵심 기능에 대한 이해도 평가
- **핵심 포인트**:
  - `@SpringBootApplication` 어노테이션의 구성
  - `@EnableAutoConfiguration`의 역할
  - `spring.factories` (또는 AutoConfiguration.imports) 파일
  - `@Conditional` 어노테이션 시리즈
- **좋은 답변 예시**:
  - 클래스패스에 있는 라이브러리를 감지하여 자동으로 Bean 등록
  - `@ConditionalOnClass`, `@ConditionalOnMissingBean` 등으로 조건부 설정
- **꼬리 질문**:

  **Q. 자동 설정을 비활성화하거나 커스터마이징하는 방법은?**
  - **좋은 답변**: `@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})`로 특정 자동 설정을 제외할 수 있습니다. `application.yml`에서 `spring.autoconfigure.exclude`로도 가능합니다. 커스터마이징은 같은 타입의 Bean을 직접 정의하면 `@ConditionalOnMissingBean`으로 인해 자동 설정이 적용되지 않습니다.
  - **추가 질문**: 자동 설정 클래스의 적용 순서를 제어하려면 어떻게 하나요?

  **Q. `@ConfigurationProperties`는 어떻게 사용하나요?**
  - **좋은 답변**: 외부 설정(application.yml)을 타입 안전하게 Java 객체로 바인딩합니다. `@ConfigurationProperties(prefix = "app.config")`로 prefix를 지정하고, `@EnableConfigurationProperties` 또는 `@ConfigurationPropertiesScan`으로 활성화합니다. `@Value`와 달리 계층적 설정을 객체로 관리할 수 있어 복잡한 설정에 유리합니다.
  - **추가 질문**: `@Value`와 `@ConfigurationProperties`는 어떤 상황에서 각각 사용하나요?

---

### Q4. Spring에서 의존성 주입(DI) 방법에는 어떤 것들이 있나요?

- **의도**: DI의 기본 개념과 방법에 대한 이해도 평가
- **핵심 포인트**:
  - 생성자 주입, 세터 주입, 필드 주입
  - 각 방식의 장단점
  - 생성자 주입을 권장하는 이유
- **좋은 답변 예시**:
  - 생성자 주입: 불변성 보장, 테스트 용이, 순환 참조 감지
  - 필드 주입: 간편하지만 테스트 어려움, 불변성 보장 안됨
- **꼬리 질문**:

  **Q. `@Autowired`를 생략해도 되는 경우는 언제인가요?**
  - **좋은 답변**: 생성자가 하나뿐일 때 `@Autowired`를 생략할 수 있습니다. Spring 4.3부터 단일 생성자에 대해 자동으로 의존성 주입이 적용됩니다. Lombok의 `@RequiredArgsConstructor`를 사용하면 final 필드에 대한 생성자를 자동 생성하므로 편리합니다.
  - **추가 질문**: `@RequiredArgsConstructor`와 `@AllArgsConstructor`의 차이는?

  **Q. 순환 참조(Circular Dependency)가 발생하면 어떻게 해결하나요?**
  - **좋은 답변**: 순환 참조는 설계 문제를 나타내므로 근본적으로 구조를 개선해야 합니다. 해결 방법으로는: 1) 공통 로직을 별도 클래스로 분리, 2) 인터페이스를 통한 의존성 역전, 3) `@Lazy` 어노테이션으로 지연 초기화 (임시 방편), 4) Setter 주입 사용 (권장하지 않음). Spring Boot 2.6+부터는 기본적으로 순환 참조가 금지되어 애플리케이션이 시작되지 않습니다.
  - **추가 질문**: 순환 참조가 왜 나쁜 설계인가요?

---

### Q5. `@Component`, `@Service`, `@Repository`, `@Controller`의 차이점은 무엇인가요?

- **의도**: 스테레오타입 어노테이션에 대한 이해도 평가
- **핵심 포인트**:
  - 모두 `@Component`를 상속한 어노테이션
  - 각 어노테이션의 역할과 의미론적 차이
  - `@Repository`의 예외 변환 기능
- **좋은 답변 예시**:
  - `@Service`: 비즈니스 로직 계층
  - `@Repository`: 데이터 접근 계층, 예외를 DataAccessException으로 변환
  - `@Controller`: 웹 요청 처리 계층
- **꼬리 질문**:

  **Q. 컴포넌트 스캔은 어떻게 동작하나요?**
  - **좋은 답변**: `@SpringBootApplication`에 포함된 `@ComponentScan`이 해당 패키지와 하위 패키지를 스캔합니다. `@Component` 및 이를 상속한 어노테이션이 붙은 클래스를 찾아 Bean으로 등록합니다. `basePackages` 속성으로 스캔 범위를 지정할 수 있고, `excludeFilters`로 특정 클래스를 제외할 수 있습니다.
  - **추가 질문**: 메인 클래스의 위치가 컴포넌트 스캔에 어떤 영향을 주나요?

---

## 2. JPA 기초

### Q6. JPA를 사용해 보셨나요? JPA의 장점과 단점을 설명해주세요.

- **의도**: JPA 사용 경험과 기본 개념 이해도 평가
- **핵심 포인트**:
  - 객체-관계 매핑(ORM)의 개념
  - 생산성 향상, DB 독립성
  - N+1 문제, 학습 곡선
- **좋은 답변 예시**:
  - 장점: SQL 작성 감소, 객체 중심 개발, 캐싱, DB 벤더 독립
  - 단점: 복잡한 쿼리 작성 어려움, 성능 튜닝 필요, 학습 비용
- **꼬리 질문**:

  **Q. MyBatis와 비교했을 때 어떤 상황에서 JPA가 더 적합한가요?**
  - **좋은 답변**: JPA는 단순 CRUD가 많고 객체 중심 설계가 필요한 경우, 도메인 모델이 복잡한 경우에 적합합니다. MyBatis는 복잡한 SQL이나 레거시 DB와의 연동, 통계/리포트성 쿼리가 많은 경우, SQL 튜닝이 중요한 경우에 적합합니다. 둘을 함께 사용하는 경우도 많습니다.
  - **추가 질문**: JPA에서 복잡한 쿼리는 어떻게 처리하나요?

---

### Q7. JPA에서 복합키를 사용해 보셨나요? `@IdClass`와 `@EmbeddedId`의 차이점은?

- **의도**: JPA 엔티티 매핑에 대한 심화 이해도 평가
- **핵심 포인트**:
  - 복합키 사용 시나리오
  - 두 방식의 구현 차이
  - 각 방식의 장단점
- **좋은 답변 예시**:
  - `@IdClass`: 엔티티에 키 필드를 직접 선언, JPQL에서 필드 직접 접근
  - `@EmbeddedId`: 별도의 키 클래스를 `@Embeddable`로 만들어 사용, 객체지향적
  - 선호도: `@EmbeddedId`가 더 객체지향적이지만, 간단한 경우 `@IdClass`도 사용

```java
// @IdClass 방식
@Entity
@IdClass(OrderItemId.class)
public class OrderItem {
    @Id
    private Long orderId;
    @Id
    private Long itemId;
}

// @EmbeddedId 방식
@Entity
public class OrderItem {
    @EmbeddedId
    private OrderItemId id;
}
```

- **꼬리 질문**:

  **Q. 복합키 사용 시 equals()와 hashCode()는 왜 구현해야 하나요?**
  - **좋은 답변**: JPA는 엔티티의 동일성을 식별자로 판단합니다. 복합키 클래스가 `equals()`와 `hashCode()`를 구현하지 않으면 영속성 컨텍스트에서 동일한 엔티티를 찾지 못하거나, HashSet/HashMap에서 제대로 동작하지 않습니다. 복합키 클래스는 반드시 `Serializable`을 구현하고, 기본 생성자가 있어야 합니다.
  - **추가 질문**: 복합키 대신 대리키(Surrogate Key)를 사용하는 것이 더 좋은 경우는 언제인가요?

---

### Q8. CreateDate, UpdateDate 같은 반복적인 컬럼들은 JPA에서 어떻게 처리하셨나요?

- **의도**: JPA Auditing 기능 이해도 평가
- **핵심 포인트**:
  - `@MappedSuperclass`와 상속
  - `@EnableJpaAuditing`과 `@EntityListeners`
  - `@CreatedDate`, `@LastModifiedDate`, `@CreatedBy`, `@LastModifiedBy`
- **좋은 답변 예시**:
  - BaseEntity 클래스를 만들어 공통 필드 관리
  - `@EnableJpaAuditing` 설정
  - `@EntityListeners(AuditingEntityListener.class)` 적용

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {
    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

- **꼬리 질문**:

  **Q. DefaultValue를 어떻게 처리하셨나요? (`@ColumnDefault`, `@PrePersist` 등)**
  - **좋은 답변**: `@ColumnDefault("0")`은 DDL 생성 시 DB에 기본값을 설정하지만 JPA가 insert 시 해당 컬럼을 명시하면 적용되지 않습니다. `@PrePersist` 콜백 메서드에서 null 체크 후 기본값을 설정하는 것이 더 확실합니다. 또는 엔티티 필드 선언 시 Java 기본값을 직접 할당(`private int count = 0;`)하는 방법도 있습니다.
  - **추가 질문**: `@CreatedBy`, `@LastModifiedBy`를 사용하려면 어떤 추가 설정이 필요한가요?

---

### Q9. 엔티티의 연관관계 매핑에서 `@ManyToOne`과 `@OneToMany`의 차이점과 주의사항은?

- **의도**: 연관관계 매핑에 대한 기본 이해도 평가
- **핵심 포인트**:
  - 단방향 vs 양방향
  - 연관관계의 주인
  - Lazy Loading과 Eager Loading
- **좋은 답변 예시**:
  - 외래키가 있는 쪽이 연관관계의 주인
  - `@ManyToOne`은 기본 EAGER, `@OneToMany`는 기본 LAZY
  - 양방향 연관관계에서 편의 메서드 작성 필요
- **꼬리 질문**:

  **Q. `mappedBy` 속성은 언제 사용하나요?**
  - **좋은 답변**: 양방향 연관관계에서 연관관계의 주인이 아닌 쪽(거울)에서 사용합니다. `@OneToMany(mappedBy = "team")`처럼 반대편 엔티티의 필드명을 지정합니다. `mappedBy`가 있는 쪽은 읽기 전용이며, 외래키 관리를 하지 않습니다.
  - **추가 질문**: 양방향 연관관계에서 편의 메서드는 왜 필요하고, 어느 쪽에 작성하나요?

  **Q. `cascade` 옵션은 어떤 것들이 있나요?**
  - **좋은 답변**: `PERSIST`(저장), `REMOVE`(삭제), `MERGE`(병합), `REFRESH`(새로고침), `DETACH`(준영속), `ALL`(전부)이 있습니다. 부모 엔티티 작업 시 자식에게 전파됩니다. `CascadeType.REMOVE`는 주의가 필요하며, 고아 객체 제거(`orphanRemoval = true`)와 함께 사용되는 경우가 많습니다.
  - **추가 질문**: `cascade = REMOVE`와 `orphanRemoval = true`의 차이는?

---

## 3. Java 기초

### Q10. `equals()`와 `hashCode()`를 함께 재정의해야 하는 이유는 무엇인가요?

- **의도**: Java 기본 개념에 대한 이해도 평가
- **핵심 포인트**:
  - Object 클래스의 equals()와 hashCode() 계약
  - HashMap, HashSet 등 해시 기반 컬렉션에서의 동작
- **좋은 답변 예시**:
  - equals()가 true인 두 객체는 반드시 같은 hashCode()를 가져야 함
  - hashCode()만 같고 equals()가 false일 수는 있음
  - HashMap에서 버킷을 찾을 때 hashCode() 사용, 충돌 시 equals()로 비교
- **꼬리 질문**:

  **Q. Lombok의 `@EqualsAndHashCode`를 사용할 때 주의할 점은?**
  - **좋은 답변**: 엔티티에 사용할 때 연관관계 필드를 포함하면 순환 참조로 StackOverflowError가 발생할 수 있습니다. `@EqualsAndHashCode(of = "id")`처럼 특정 필드만 지정하거나, `@EqualsAndHashCode.Exclude`로 제외해야 합니다. 또한 JPA 프록시 객체와 실제 객체 비교 시 문제가 생길 수 있어, `id`만 비교하거나 `instanceof` 대신 `getClass()` 사용을 고려해야 합니다.
  - **추가 질문**: JPA 엔티티에서 equals()를 어떻게 구현하는 것이 좋을까요?

---

### Q11. Java 컬렉션 프레임워크에서 `ArrayList`와 `LinkedList`의 차이점은?

- **의도**: 자료구조 기본 이해도 평가
- **핵심 포인트**:
  - 내부 구현 방식 (배열 vs 노드 연결)
  - 시간 복잡도 차이 (접근, 삽입, 삭제)
  - 사용 시나리오
- **좋은 답변 예시**:
  - ArrayList: 인덱스 접근 O(1), 중간 삽입/삭제 O(n)
  - LinkedList: 인덱스 접근 O(n), 앞/뒤 삽입/삭제 O(1)
  - 대부분의 경우 ArrayList가 캐시 효율성 때문에 더 빠름
- **꼬리 질문**:

  **Q. HashMap과 TreeMap의 차이는?**
  - **좋은 답변**: HashMap은 해시 테이블 기반으로 O(1) 접근이 가능하고 순서를 보장하지 않습니다. TreeMap은 Red-Black Tree 기반으로 O(log n) 접근이며 키가 정렬된 상태로 유지됩니다. 정렬이 필요하면 TreeMap, 빠른 접근이 필요하면 HashMap을 사용합니다. 삽입 순서가 필요하면 LinkedHashMap을 사용합니다.
  - **추가 질문**: HashMap의 초기 용량(capacity)과 로드 팩터(load factor)는 무엇인가요?

  **Q. ConcurrentHashMap은 어떤 상황에서 사용하나요?**
  - **좋은 답변**: 멀티스레드 환경에서 동시 접근이 필요할 때 사용합니다. Hashtable과 달리 전체 Map이 아닌 버킷 단위로 락을 걸어(세그먼트 락 또는 노드 락) 동시성과 성능을 모두 확보합니다. 읽기는 락 없이 가능하고, 쓰기만 해당 버킷에 락을 겁니다.
  - **추가 질문**: `Collections.synchronizedMap()`과 `ConcurrentHashMap`의 차이는?

---

### Q12. 제네릭(Generics)을 사용하는 이유와 장점은 무엇인가요?

- **의도**: 타입 안정성에 대한 이해도 평가
- **핵심 포인트**:
  - 컴파일 타임 타입 체크
  - 타입 캐스팅 불필요
  - 코드 재사용성
- **좋은 답변 예시**:
  - 컴파일 시점에 타입 오류 발견 가능
  - 불필요한 형변환 제거
  - 타입 안전한 코드 작성 가능
- **꼬리 질문**:

  **Q. 와일드카드 `?`와 `<? extends T>`, `<? super T>`의 차이는?**
  - **좋은 답변**: `?`는 모든 타입을 허용합니다. `<? extends T>`는 T와 T의 하위 타입만 허용(상한 경계)하며 읽기 전용에 적합합니다. `<? super T>`는 T와 T의 상위 타입만 허용(하한 경계)하며 쓰기에 적합합니다. PECS 원칙: Producer-Extends, Consumer-Super. `List<? extends Number>`는 Number를 꺼내 읽을 수 있고, `List<? super Integer>`는 Integer를 넣을 수 있습니다.
  - **추가 질문**: 제네릭의 타입 소거(Type Erasure)란 무엇인가요?

---

### Q13. Java에서 `String`, `StringBuilder`, `StringBuffer`의 차이점은?

- **의도**: 문자열 처리에 대한 기본 이해도 평가
- **핵심 포인트**:
  - 불변성 (String vs StringBuilder/StringBuffer)
  - 스레드 안전성 (StringBuffer vs StringBuilder)
  - 성능 차이
- **좋은 답변 예시**:
  - String: 불변, 문자열 연산 시 새 객체 생성
  - StringBuilder: 가변, 단일 스레드 환경에서 사용, 빠름
  - StringBuffer: 가변, 멀티 스레드 환경에서 안전, synchronized
- **꼬리 질문**:

  **Q. String Pool이란 무엇인가요?**
  - **좋은 답변**: JVM Heap 영역의 특별한 공간으로, 문자열 리터럴을 저장합니다. `"hello"`와 같이 리터럴로 생성한 문자열은 Pool에 저장되고 재사용됩니다. `new String("hello")`는 Pool을 사용하지 않고 새 객체를 생성합니다. `intern()` 메서드로 Pool에 등록하거나 Pool의 문자열을 참조할 수 있습니다.
  - **추가 질문**: `String s = "hello"`와 `String s = new String("hello")`의 차이는?

  **Q. `+` 연산자로 문자열을 연결하면 내부적으로 어떻게 동작하나요?**
  - **좋은 답변**: Java 5 이전에는 매번 새로운 String 객체를 생성했습니다. Java 5부터는 컴파일러가 `StringBuilder`로 최적화합니다. 하지만 반복문 안에서 `+` 연산을 사용하면 매번 새로운 StringBuilder가 생성되어 비효율적입니다. 반복문에서는 명시적으로 StringBuilder를 사용하는 것이 좋습니다. Java 9+에서는 `invokedynamic`으로 더 최적화되었습니다.
  - **추가 질문**: 반복문에서 문자열 연결 시 StringBuilder를 어떻게 사용하나요?

---

## 4. 인증/보안

### Q14. JWT(JSON Web Token)의 장점과 단점을 설명해주세요.

- **의도**: 토큰 기반 인증에 대한 이해도 평가
- **핵심 포인트**:
  - JWT의 구조 (Header, Payload, Signature)
  - Stateless의 의미와 장단점
  - 보안 고려사항
- **좋은 답변 예시**:
  - **장점**: 서버 무상태(Stateless), 확장성, 다양한 클라이언트 지원, 자체 정보 포함
  - **단점**: 토큰 크기, 탈취 시 만료까지 무효화 어려움, Payload 암호화 안됨
- **꼬리 질문**:

  **Q. Access Token과 Refresh Token을 분리하는 이유는?**
  - **좋은 답변**: Access Token은 짧은 만료 시간(예: 15분)으로 탈취 시 피해를 최소화합니다. Refresh Token은 긴 만료 시간(예: 7일)으로 사용자 경험을 유지합니다. Access Token 만료 시 Refresh Token으로 새 Access Token을 발급받습니다. Refresh Token은 서버에 저장하여 필요시 무효화할 수 있습니다.
  - **추가 질문**: Refresh Token Rotation이란 무엇인가요?

  **Q. JWT를 어디에 저장하는 것이 좋을까요? (LocalStorage vs Cookie)**
  - **좋은 답변**: LocalStorage는 XSS 공격에 취약합니다. Cookie는 HttpOnly 설정으로 JavaScript 접근을 막을 수 있어 XSS에 안전하지만, CSRF 공격에 취약합니다. 권장 방식은 HttpOnly, Secure, SameSite 속성이 설정된 Cookie에 저장하고, CSRF 토큰을 함께 사용하는 것입니다. 또는 Access Token은 메모리에, Refresh Token은 HttpOnly Cookie에 저장하는 방식도 있습니다.
  - **추가 질문**: XSS와 CSRF 공격의 차이는 무엇인가요?

---

### Q15. MSA(Microservices Architecture)의 장점과 단점을 설명해주세요.

- **의도**: 아키텍처에 대한 기본 이해도 평가
- **핵심 포인트**:
  - 모놀리식과의 비교
  - 장점: 독립 배포, 확장성, 기술 다양성
  - 단점: 복잡성, 네트워크 오버헤드, 데이터 일관성
- **좋은 답변 예시**:
  - **장점**: 서비스별 독립 배포/확장, 장애 격리, 팀별 자율성
  - **단점**: 분산 시스템 복잡성, 서비스 간 통신 오버헤드, 트랜잭션 관리 어려움
- **꼬리 질문**:

  **Q. 모놀리식 아키텍처가 더 적합한 경우는 언제인가요?**
  - **좋은 답변**: 초기 스타트업이나 소규모 프로젝트, 팀 규모가 작은 경우(피자 두 판 규칙 이하), 도메인 경계가 명확하지 않은 경우, 빠른 MVP 개발이 필요한 경우에 모놀리식이 적합합니다. MSA는 오버헤드가 크므로 비즈니스가 안정화된 후 점진적으로 전환하는 것이 좋습니다.
  - **추가 질문**: 모놀리식에서 MSA로 전환할 때 어떤 전략을 사용하나요?

  **Q. 서비스 간 통신 방식에는 어떤 것들이 있나요?**
  - **좋은 답변**: 동기 방식으로는 REST API, gRPC가 있고, 비동기 방식으로는 메시지 큐(Kafka, RabbitMQ)가 있습니다. REST는 간단하고 널리 사용되며, gRPC는 성능이 좋고 양방향 스트리밍을 지원합니다. 메시지 큐는 서비스 간 결합도를 낮추고 부하를 분산시킵니다. 이벤트 기반 아키텍처에서는 메시지 큐를 주로 사용합니다.
  - **추가 질문**: 동기 통신과 비동기 통신은 각각 어떤 상황에서 사용하나요?

---

## 5. 개발 도구 및 협업

### Q16. 개발 도구와 협업 도구는 어떤 것들을 사용해 보셨나요?

- **의도**: 실무 환경 적응력 평가
- **핵심 포인트**:
  - IDE (IntelliJ, Eclipse 등)
  - 버전 관리 (Git)
  - 협업 도구 (Jira, Confluence, Slack 등)
  - CI/CD 경험
- **좋은 답변 예시**:
  - IntelliJ IDEA 사용, 단축키 활용, 플러그인 활용 경험
  - Git 브랜치 전략 (Git Flow, GitHub Flow)
  - Jira로 스프린트 관리, Confluence로 문서화
- **꼬리 질문**:

  **Q. Git에서 rebase와 merge의 차이는?**
  - **좋은 답변**: `merge`는 두 브랜치의 히스토리를 합치면서 새로운 커밋(merge commit)을 만들고, 히스토리가 보존됩니다. `rebase`는 커밋들을 다른 브랜치 위로 재배치하여 히스토리가 선형(linear)으로 깔끔해집니다. 공유된 브랜치(main, develop)에서는 rebase를 피하고, feature 브랜치에서 최신 main을 반영할 때 rebase를 사용하기도 합니다. `rebase`는 히스토리를 변경하므로 주의가 필요합니다.
  - **추가 질문**: `git rebase -i`(interactive rebase)는 어떤 용도로 사용하나요?

  **Q. 충돌(Conflict) 해결 경험이 있나요?**
  - **좋은 답변**: 같은 파일의 같은 라인을 여러 명이 수정하면 충돌이 발생합니다. `<<<<<<<`, `=======`, `>>>>>>>`로 표시된 부분에서 원하는 코드를 선택하고 마커를 제거합니다. IDE의 merge 도구를 사용하면 시각적으로 쉽게 해결할 수 있습니다. 충돌 해결 후 테스트를 돌려 코드가 정상 동작하는지 확인해야 합니다.
  - **추가 질문**: 충돌을 미리 방지하기 위한 방법은 무엇인가요?

---

### Q17. 코드 리뷰 경험이 있나요? 코드 리뷰에서 중요하게 보는 점은 무엇인가요?

- **의도**: 협업 경험과 코드 품질에 대한 인식 평가
- **핵심 포인트**:
  - 코드 리뷰의 목적 이해
  - 리뷰 시 체크하는 항목들
  - 피드백 주고받는 방법
- **좋은 답변 예시**:
  - 가독성, 로직의 정확성, 성능 고려
  - 테스트 코드 작성 여부
  - 건설적인 피드백 제공
- **꼬리 질문**:

  **Q. 리뷰에서 의견 충돌이 있을 때 어떻게 해결하나요?**
  - **좋은 답변**: 먼저 상대방의 의견을 끝까지 듣고 이해하려고 합니다. 감정이 아닌 근거(코드 컨벤션, 성능 데이터, 공식 문서 등)를 바탕으로 논의합니다. 합의가 안 되면 제3자(시니어, 테크 리드) 의견을 구하거나 팀 컨벤션을 참고합니다. 최종적으로 팀의 결정을 따르고, "다 맞고 틀린 게 없을 수도 있다"는 열린 자세가 중요합니다.
  - **추가 질문**: 코드 리뷰를 요청받았을 때 어떤 순서로 리뷰하나요?

---

## 6. 추가 질문 (선택)

### Q18. REST API 설계 원칙에 대해 설명해주세요.

- **의도**: API 설계 능력 평가
- **핵심 포인트**:
  - URI 설계 규칙 (명사 사용, 계층 구조)
  - HTTP 메서드의 의미 (GET, POST, PUT, DELETE, PATCH)
  - 상태 코드 활용
- **좋은 답변 예시**:
  - 리소스 중심 URI 설계
  - GET은 조회, POST는 생성, PUT은 전체 수정, PATCH는 부분 수정, DELETE는 삭제
  - 적절한 HTTP 상태 코드 반환 (200, 201, 400, 404, 500 등)

---

### Q19. 테스트 코드 작성 경험이 있나요? JUnit을 사용해 보셨나요?

- **의도**: 테스트에 대한 기본 이해도 평가
- **핵심 포인트**:
  - 단위 테스트의 개념
  - JUnit 기본 사용법
  - Mock 객체 사용
- **좋은 답변 예시**:
  - `@Test`, `@BeforeEach`, `@AfterEach` 어노테이션 사용
  - Assertions 메서드 (assertEquals, assertTrue 등)
  - Mockito를 사용한 의존성 모킹
- **꼬리 질문**:

  **Q. 통합 테스트와 단위 테스트의 차이는?**
  - **좋은 답변**: 단위 테스트는 하나의 클래스나 메서드를 독립적으로 테스트하며, 의존성은 Mock으로 대체합니다. 빠르고 격리되어 있습니다. 통합 테스트는 여러 컴포넌트가 함께 동작하는지 테스트하며, 실제 DB, 외부 API 등을 사용합니다. Spring Boot에서는 `@SpringBootTest`로 통합 테스트를, `@WebMvcTest`, `@DataJpaTest` 등으로 슬라이스 테스트를 합니다.
  - **추가 질문**: Mock과 Stub의 차이는 무엇인가요?

  **Q. 테스트 커버리지를 어느 정도 유지하는 것이 좋을까요?**
  - **좋은 답변**: 숫자보다 중요한 건 핵심 비즈니스 로직이 테스트되는지입니다. 일반적으로 70-80% 정도를 목표로 하되, 100%를 맹목적으로 추구할 필요는 없습니다. getter/setter, 단순 위임 메서드보다 복잡한 로직, 엣지 케이스, 버그가 자주 발생하는 부분을 우선 테스트합니다. 커버리지 도구(JaCoCo)로 측정하고 CI에서 최소 커버리지를 강제하기도 합니다.
  - **추가 질문**: 테스트하기 어려운 코드(외부 API 호출, 현재 시간 등)는 어떻게 테스트하나요?

---

## 📋 평가 체크리스트

| 영역 | 체크 포인트 |
|------|------------|
| Spring | 요청 흐름 이해, DI 개념, 어노테이션 역할 |
| JPA | 기본 매핑, Auditing, 연관관계 |
| Java | 기본 문법, 컬렉션, 문자열 처리 |
| 인증/보안 | JWT, 기본 보안 개념 |
| 개발 도구 | Git, IDE, 협업 도구 경험 |
| 태도 | 학습 의지, 협업 자세, 문제 해결 태도 |
