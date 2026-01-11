# 프록시 패턴 (Proxy Pattern)

## 정의

프록시 패턴은 다른 객체에 대한 접근을 제어하기 위해 대리자나 자리채움자 역할을 하는 객체를 제공하는 구조 디자인 패턴입니다. 프록시는 원래 객체와 같은 인터페이스를 구현하며, 클라이언트의 요청을 받아 실제 객체에 전달하기 전후에 추가적인 처리를 수행할 수 있습니다.

## 🎯 한눈에 보기

| 항목 | 설명 |
|------|------|
| **핵심** | 실제 객체 대신 대리자가 접근을 제어하고 부가 기능 제공 |
| **비유** | 비서: 사장님에게 직접 연락 대신 비서를 통해 접근 |
| **언제** | 접근 제어, 지연 로딩, 캐싱, 로깅이 필요할 때 |
| **Spring** | `@Transactional`, AOP 프록시, JDK Dynamic Proxy/CGLIB |

> **💡 DB 트랜잭션 처리를 모든 메서드에 직접 작성?**
>
> **❌ Before (직접 트랜잭션 처리)**
> ```java
> public void transfer() {
>     tx.begin();
>     try {
>         // 비즈니스 로직
>         tx.commit();
>     } catch (e) { tx.rollback(); }
> }
> // → 모든 메서드에 반복 코드!
> ```
>
> **✅ After (프록시 패턴 - @Transactional)**
> ```java
> @Transactional
> public void transfer() {
>     // 비즈니스 로직만!
> }
> // → 프록시가 트랜잭션 처리를 대신!
> ```

## 구조 (Structure)

```mermaid
classDiagram
    class Subject {
        <<interface>>
        +request(): void
    }

    class RealSubject {
        +request(): void
    }

    class Proxy {
        -realSubject: RealSubject
        +request(): void
        -checkAccess(): boolean
        -logAccess(): void
    }

    class Client {
        +main(): void
    }

    Subject <|.. RealSubject
    Subject <|.. Proxy
    Proxy --> RealSubject : controls access to
    Client --> Subject : uses

    note for Subject "공통 인터페이스"
    note for RealSubject "실제 비즈니스 로직을 담당"
    note for Proxy "접근 제어 및 부가 기능 제공"
```

## 동작 흐름 (시퀀스 다이어그램)

```mermaid
sequenceDiagram
    participant Client as 클라이언트
    participant Proxy as Proxy<br/>(@Transactional)
    participant Real as RealSubject<br/>(Service)
    participant DB as Database

    Client->>Proxy: transfer() 호출

    Note over Proxy: 전처리 (Before)
    Proxy->>Proxy: 1. 권한 확인
    Proxy->>Proxy: 2. 트랜잭션 시작
    Proxy->>Proxy: 3. 로깅

    Proxy->>Real: transfer() 위임
    Real->>DB: UPDATE 실행
    DB-->>Real: 결과
    Real-->>Proxy: 결과 반환

    Note over Proxy: 후처리 (After)
    Proxy->>Proxy: 4. 트랜잭션 커밋/롤백
    Proxy->>Proxy: 5. 로깅

    Proxy-->>Client: 최종 결과
```

**핵심 포인트:**
- 클라이언트는 Proxy를 RealSubject처럼 사용 (동일한 인터페이스)
- Proxy가 전처리/후처리 담당 (트랜잭션, 로깅, 캐싱 등)
- Spring의 `@Transactional`이 이 방식으로 동작

## 사용 이유

- **접근 제어**: 실제 객체에 대한 접근을 제어하고 권한을 확인할 수 있습니다.
- **지연 초기화**: 실제로 필요할 때까지 무거운 객체의 생성을 지연시킬 수 있습니다.
- **캐싱**: 결과를 캐시하여 성능을 향상시킬 수 있습니다.
- **로깅**: 객체에 대한 접근을 기록하고 모니터링할 수 있습니다.
- **원격 접근**: 원격 객체에 대한 로컬 대리자 역할을 할 수 있습니다.

## 프록시 패턴의 종류

### 1. 가상 프록시 (Virtual Proxy)
무거운 객체의 생성을 지연시키는 프록시

### 2. 보호 프록시 (Protection Proxy)
접근 권한을 제어하는 프록시

### 3. 원격 프록시 (Remote Proxy)
원격 객체에 대한 로컬 대리자

### 4. 캐싱 프록시 (Caching Proxy)
결과를 캐시하여 성능을 향상시키는 프록시

## 적용 상황

프록시 패턴은 다음과 같은 상황에서 특히 유용합니다:

### 1. 지연 로딩 (Lazy Loading)
- **대용량 파일**: 이미지, 비디오 등의 무거운 리소스
- **데이터베이스 연결**: 실제 필요할 때까지 연결 지연
- **복잡한 계산**: 비용이 큰 연산의 지연 실행

### 2. 접근 제어 및 보안
```java
// 나쁜 예: 직접 접근으로 보안 취약
class SecretDocument {
    public String getContent() {
        return "기밀 내용";  // 누구나 접근 가능
    }
}

// 좋은 예: 프록시를 통한 접근 제어
class SecretDocumentProxy implements Document {
    private SecretDocument realDocument;
    private String userRole;

    public String getContent() {
        if (hasAccess()) {
            return getRealDocument().getContent();
        }
        throw new SecurityException("접근 권한 없음");
    }
}
```

### 3. 성능 최적화
- **캐싱**: 이전 결과를 저장하여 재사용
- **배치 처리**: 여러 요청을 모아서 일괄 처리
- **압축**: 데이터 압축/해제를 투명하게 처리

## 초급 예제 - 5분 만에 이해하기

유튜브 동영상 지연 로딩으로 프록시 패턴을 이해해봅시다.

```java
// 1. 공통 인터페이스
interface Video {
    void play();
    String getTitle();
}

// 2. 실제 객체 (무거운 동영상 파일)
class RealVideo implements Video {
    private String filename;
    private byte[] data;

    public RealVideo(String filename) {
        this.filename = filename;
        loadFromDisk();  // 생성 시 바로 로드 (무거운 작업)
    }

    private void loadFromDisk() {
        System.out.println("⏳ 로딩 중: " + filename + " (3초 소요)");
        try { Thread.sleep(3000); } catch (Exception e) {}
        this.data = new byte[100000000];  // 100MB 가정
        System.out.println("✅ 로딩 완료: " + filename);
    }

    public void play() {
        System.out.println("▶️ 재생: " + filename);
    }

    public String getTitle() {
        return filename;
    }
}

// 3. 프록시 (지연 로딩 담당)
class VideoProxy implements Video {
    private String filename;
    private RealVideo realVideo;  // 실제 객체는 나중에 생성

    public VideoProxy(String filename) {
        this.filename = filename;
        // 여기서는 RealVideo를 생성하지 않음!
        System.out.println("📋 프록시 생성: " + filename);
    }

    public void play() {
        // 실제로 재생할 때만 로드
        if (realVideo == null) {
            realVideo = new RealVideo(filename);
        }
        realVideo.play();
    }

    public String getTitle() {
        return filename;  // 제목은 프록시만으로 반환 가능
    }
}

// 4. 사용
public class Main {
    public static void main(String[] args) {
        System.out.println("=== 동영상 목록 로드 ===");

        // 프록시 생성 (실제 동영상은 아직 로드 안 됨!)
        Video video1 = new VideoProxy("고양이.mp4");
        Video video2 = new VideoProxy("강아지.mp4");
        Video video3 = new VideoProxy("토끼.mp4");

        System.out.println("\n제목만 보기 (로드 불필요):");
        System.out.println("- " + video1.getTitle());
        System.out.println("- " + video2.getTitle());
        System.out.println("- " + video3.getTitle());

        System.out.println("\n=== 첫 번째 영상만 재생 ===");
        video1.play();  // 이때 비로소 로드됨!

        System.out.println("\n=== 같은 영상 다시 재생 ===");
        video1.play();  // 이미 로드되어 있어서 바로 재생
    }
}
```

**출력:**
```
=== 동영상 목록 로드 ===
📋 프록시 생성: 고양이.mp4
📋 프록시 생성: 강아지.mp4
📋 프록시 생성: 토끼.mp4

제목만 보기 (로드 불필요):
- 고양이.mp4
- 강아지.mp4
- 토끼.mp4

=== 첫 번째 영상만 재생 ===
⏳ 로딩 중: 고양이.mp4 (3초 소요)
✅ 로딩 완료: 고양이.mp4
▶️ 재생: 고양이.mp4

=== 같은 영상 다시 재생 ===
▶️ 재생: 고양이.mp4
```

**핵심 포인트:**
- 3개 영상 중 실제로 로드된 건 1개뿐 (지연 로딩)
- 제목 조회는 프록시만으로 처리 (무거운 로드 불필요)
- 두 번째 재생은 즉시 실행 (이미 로드됨)

---

## Spring Boot 예제

Spring에서 **@Transactional**, **AOP**가 대표적인 프록시 패턴입니다. Spring은 내부적으로 JDK Dynamic Proxy 또는 CGLIB을 사용합니다.

### 프로젝트 구조
```
src/main/java/com/example/payment/
├── service/
│   ├── PaymentService.java           # 인터페이스
│   └── PaymentServiceImpl.java       # 실제 구현
├── proxy/
│   ├── LoggingProxy.java             # 직접 구현한 프록시 (학습용)
│   └── CachingProxy.java
├── aspect/
│   └── PerformanceAspect.java        # AOP 방식 프록시
└── config/
    └── ProxyConfig.java
```

### 1. @Transactional의 동작 원리

```java
// 인터페이스
public interface AccountService {
    void transfer(Long fromId, Long toId, int amount);
}

// 실제 구현 - 비즈니스 로직만 작성
@Service
public class AccountServiceImpl implements AccountService {

    @Transactional  // ← 이것이 프록시를 생성!
    @Override
    public void transfer(Long fromId, Long toId, int amount) {
        // 트랜잭션 처리 코드 없이 비즈니스 로직만!
        accountRepository.withdraw(fromId, amount);
        accountRepository.deposit(toId, amount);
        // 예외 발생 시 자동 롤백
    }
}

// Spring이 내부적으로 생성하는 프록시 (개념적 코드)
class AccountServiceProxy implements AccountService {
    private AccountServiceImpl target;
    private TransactionManager txManager;

    @Override
    public void transfer(Long fromId, Long toId, int amount) {
        txManager.begin();  // 전처리
        try {
            target.transfer(fromId, toId, amount);  // 실제 호출
            txManager.commit();  // 성공 시 커밋
        } catch (Exception e) {
            txManager.rollback();  // 실패 시 롤백
            throw e;
        }
    }
}
```

### 2. 직접 구현하는 프록시 예제 (학습용)

```java
// 결제 서비스 인터페이스
public interface PaymentService {
    PaymentResult processPayment(PaymentRequest request);
    PaymentStatus getStatus(String paymentId);
}

// 실제 결제 서비스
@Slf4j
public class RealPaymentService implements PaymentService {

    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        log.info("💳 실제 결제 처리: {}원", request.getAmount());
        // PG사 API 호출 등 실제 로직
        return new PaymentResult(true, "PAY_" + System.currentTimeMillis());
    }

    @Override
    public PaymentStatus getStatus(String paymentId) {
        log.info("🔍 결제 상태 조회: {}", paymentId);
        return new PaymentStatus(paymentId, "COMPLETED");
    }
}

// 로깅 프록시
@Slf4j
public class LoggingPaymentProxy implements PaymentService {

    private final PaymentService target;

    public LoggingPaymentProxy(PaymentService target) {
        this.target = target;
    }

    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        long startTime = System.currentTimeMillis();
        log.info("📝 [START] 결제 요청 - 금액: {}원, 사용자: {}",
                request.getAmount(), request.getUserId());

        try {
            PaymentResult result = target.processPayment(request);  // 위임

            long duration = System.currentTimeMillis() - startTime;
            log.info("📝 [END] 결제 완료 - ID: {}, 소요시간: {}ms",
                    result.getPaymentId(), duration);

            return result;
        } catch (Exception e) {
            log.error("📝 [ERROR] 결제 실패 - 원인: {}", e.getMessage());
            throw e;
        }
    }

    @Override
    public PaymentStatus getStatus(String paymentId) {
        log.info("📝 상태 조회 요청: {}", paymentId);
        return target.getStatus(paymentId);
    }
}

// 캐싱 프록시
public class CachingPaymentProxy implements PaymentService {

    private final PaymentService target;
    private final Map<String, PaymentStatus> cache = new ConcurrentHashMap<>();

    public CachingPaymentProxy(PaymentService target) {
        this.target = target;
    }

    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        return target.processPayment(request);  // 결제는 캐싱 안 함
    }

    @Override
    public PaymentStatus getStatus(String paymentId) {
        // 캐시 확인
        if (cache.containsKey(paymentId)) {
            System.out.println("💾 캐시 히트: " + paymentId);
            return cache.get(paymentId);
        }

        // 캐시 미스 - 실제 조회
        System.out.println("💾 캐시 미스: " + paymentId);
        PaymentStatus status = target.getStatus(paymentId);

        // 완료된 결제만 캐싱 (상태가 변하지 않으므로)
        if ("COMPLETED".equals(status.getStatus())) {
            cache.put(paymentId, status);
        }

        return status;
    }
}
```

### 3. Spring 설정 - 프록시 체인 구성

```java
@Configuration
public class ProxyConfig {

    /**
     * 프록시 체인: 로깅 → 캐싱 → 실제 서비스
     */
    @Bean
    public PaymentService paymentService() {
        // 1. 실제 서비스 (가장 안쪽)
        PaymentService realService = new RealPaymentService();

        // 2. 캐싱 프록시로 감싸기
        PaymentService cachingProxy = new CachingPaymentProxy(realService);

        // 3. 로깅 프록시로 감싸기 (가장 바깥)
        PaymentService loggingProxy = new LoggingPaymentProxy(cachingProxy);

        return loggingProxy;
    }
}
```

### 4. Spring AOP로 프록시 구현 (실무 권장)

```java
// AOP 방식 - 어노테이션 정의
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {}

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Cacheable {
    String key() default "";
}

// AOP Aspect
@Aspect
@Component
@Slf4j
public class PaymentAspects {

    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        String methodName = joinPoint.getSignature().getName();

        log.info("📝 [START] {}", methodName);

        try {
            Object result = joinPoint.proceed();

            long duration = System.currentTimeMillis() - start;
            log.info("📝 [END] {} - {}ms", methodName, duration);

            return result;
        } catch (Exception e) {
            log.error("📝 [ERROR] {} - {}", methodName, e.getMessage());
            throw e;
        }
    }
}

// 서비스에서 사용
@Service
public class PaymentServiceImpl implements PaymentService {

    @LogExecutionTime
    @Transactional
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        // 비즈니스 로직만 작성
        // 로깅, 트랜잭션은 프록시(AOP)가 처리
        return paymentGateway.process(request);
    }
}
```

### 5. JDK Dynamic Proxy vs CGLIB

```java
// JDK Dynamic Proxy - 인터페이스 기반
public class DynamicProxyExample {

    public static PaymentService createProxy(PaymentService target) {
        return (PaymentService) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                new Class[]{PaymentService.class},
                (proxy, method, args) -> {
                    System.out.println("Before: " + method.getName());
                    Object result = method.invoke(target, args);
                    System.out.println("After: " + method.getName());
                    return result;
                }
        );
    }
}

// Spring 설정에서 프록시 방식 선택
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = false)  // JDK Proxy (인터페이스 필요)
// @EnableAspectJAutoProxy(proxyTargetClass = true)  // CGLIB (인터페이스 불필요)
public class AopConfig {}
```

### Spring에서 프록시가 사용되는 곳

| 어노테이션 | 프록시 기능 | 설명 |
|-----------|------------|------|
| `@Transactional` | 트랜잭션 관리 | 시작/커밋/롤백 자동 처리 |
| `@Cacheable` | 캐싱 | 결과 캐싱, 중복 호출 방지 |
| `@Async` | 비동기 실행 | 별도 스레드에서 실행 |
| `@Retryable` | 재시도 | 실패 시 자동 재시도 |
| `@Secured` | 보안 | 권한 검사 |

---

## 실생활 예제 - 이미지 로더 및 캐싱 시스템

대용량 이미지의 지연 로딩과 캐싱 기능을 제공하는 시스템을 프록시 패턴으로 구현해보겠습니다.

```java
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.io.*;

// 이미지 인터페이스 (Subject)
interface Image {
    void display();
    void resize(int width, int height);
    void rotate(int degrees);
    ImageMetadata getMetadata();
    byte[] getRawData();
    String getFormat();
    boolean isLoaded();
}

// 이미지 메타데이터
class ImageMetadata {
    private String filename;
    private String format;
    private int width;
    private int height;
    private long fileSize;
    private LocalDateTime lastModified;
    private String colorSpace;

    public ImageMetadata(String filename, String format, int width, int height, long fileSize) {
        this.filename = filename;
        this.format = format;
        this.width = width;
        this.height = height;
        this.fileSize = fileSize;
        this.lastModified = LocalDateTime.now();
        this.colorSpace = "RGB";
    }

    // getter 메서드들
    public String getFilename() { return filename; }
    public String getFormat() { return format; }
    public int getWidth() { return width; }
    public int getHeight() { return height; }
    public long getFileSize() { return fileSize; }
    public LocalDateTime getLastModified() { return lastModified; }
    public String getColorSpace() { return colorSpace; }

    public void setDimensions(int width, int height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public String toString() {
        return String.format("%s (%s) - %dx%d, %s, %.2f MB",
                filename, format, width, height, colorSpace, fileSize / (1024.0 * 1024));
    }
}

// 실제 이미지 클래스 (RealSubject)
class RealImage implements Image {
    private String filename;
    private ImageMetadata metadata;
    private byte[] imageData;
    private boolean loaded;
    private LocalDateTime loadTime;
    private int currentWidth;
    private int currentHeight;
    private int rotation;

    public RealImage(String filename) {
        this.filename = filename;
        this.loaded = false;
        this.rotation = 0;

        // 메타데이터만 먼저 로드 (실제 이미지 데이터는 지연 로드)
        loadMetadata();
        this.currentWidth = metadata.getWidth();
        this.currentHeight = metadata.getHeight();
    }

    private void loadMetadata() {
        // 파일 시스템에서 메타데이터 읽기 시뮬레이션
        String format = getFileExtension(filename);
        int width = 1920;  // 기본값
        int height = 1080;
        long fileSize = (long) (Math.random() * 10000000 + 1000000); // 1-10MB

        // 파일 타입에 따른 크기 조정
        switch (format.toLowerCase()) {
            case "jpg", "jpeg" -> {
                width = (int) (Math.random() * 2000 + 1000);
                height = (int) (Math.random() * 2000 + 1000);
            }
            case "png" -> {
                width = (int) (Math.random() * 1500 + 800);
                height = (int) (Math.random() * 1500 + 800);
            }
            case "gif" -> {
                width = (int) (Math.random() * 800 + 400);
                height = (int) (Math.random() * 800 + 400);
                fileSize = fileSize / 2; // GIF는 보통 더 작음
            }
        }

        this.metadata = new ImageMetadata(filename, format, width, height, fileSize);
        System.out.println("📋 메타데이터 로드 완료: " + filename);
    }

    private String getFileExtension(String filename) {
        int lastDot = filename.lastIndexOf('.');
        return lastDot > 0 ? filename.substring(lastDot + 1) : "unknown";
    }

    private void loadImageData() {
        if (loaded) return;

        System.out.println("⏳ 실제 이미지 데이터 로드 중: " + filename);

        // 대용량 이미지 로드 시뮬레이션
        try {
            Thread.sleep((long) (Math.random() * 2000 + 500)); // 0.5-2.5초 지연
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        // 가상의 이미지 데이터 생성
        this.imageData = new byte[(int) metadata.getFileSize()];
        Arrays.fill(imageData, (byte) 1); // 더미 데이터

        this.loaded = true;
        this.loadTime = LocalDateTime.now();

        System.out.println("✅ 이미지 로드 완료: " + filename +
                " (" + formatFileSize(metadata.getFileSize()) + ")");
    }

    private String formatFileSize(long bytes) {
        if (bytes < 1024) return bytes + " B";
        if (bytes < 1024 * 1024) return String.format("%.1f KB", bytes / 1024.0);
        return String.format("%.1f MB", bytes / (1024.0 * 1024));
    }

    @Override
    public void display() {
        loadImageData(); // 실제 데이터가 필요할 때 로드

        System.out.println("🖼️ 이미지 표시: " + filename);
        System.out.println("   크기: " + currentWidth + "x" + currentHeight);
        System.out.println("   회전: " + rotation + "도");
        System.out.println("   로드 시간: " +
                (loadTime != null ? loadTime.format(DateTimeFormatter.ofPattern("HH:mm:ss")) : "N/A"));
    }

    @Override
    public void resize(int width, int height) {
        System.out.println("🔄 이미지 리사이즈: " + filename + " → " + width + "x" + height);
        this.currentWidth = width;
        this.currentHeight = height;

        // 메타데이터 업데이트
        metadata.setDimensions(width, height);
    }

    @Override
    public void rotate(int degrees) {
        System.out.println("↻ 이미지 회전: " + filename + " → " + degrees + "도");
        this.rotation = (rotation + degrees) % 360;

        // 90도 배수 회전 시 가로세로 치환
        if (degrees % 90 == 0 && degrees % 180 != 0) {
            int temp = currentWidth;
            currentWidth = currentHeight;
            currentHeight = temp;
        }
    }

    @Override
    public ImageMetadata getMetadata() {
        return metadata;
    }

    @Override
    public byte[] getRawData() {
        loadImageData();
        return imageData.clone();
    }

    @Override
    public String getFormat() {
        return metadata.getFormat();
    }

    @Override
    public boolean isLoaded() {
        return loaded;
    }
}

// 이미지 프록시 (가상 프록시 + 캐싱 프록시)
class ImageProxy implements Image {
    private String filename;
    private RealImage realImage;
    private ImageMetadata cachedMetadata;
    private static Map<String, byte[]> imageCache = new ConcurrentHashMap<>();
    private static Map<String, ImageMetadata> metadataCache = new ConcurrentHashMap<>();
    private static List<String> accessLog = new ArrayList<>();

    // 접근 제어를 위한 사용자 권한
    private static Set<String> authorizedUsers = new HashSet<>();
    private String currentUser;

    static {
        // 기본 권한 사용자 설정
        authorizedUsers.add("admin");
        authorizedUsers.add("user1");
        authorizedUsers.add("designer");
    }

    public ImageProxy(String filename, String user) {
        this.filename = filename;
        this.currentUser = user;

        // 캐시에서 메타데이터 확인
        this.cachedMetadata = metadataCache.get(filename);

        logAccess("프록시 생성");
    }

    private boolean checkAccess() {
        if (!authorizedUsers.contains(currentUser)) {
            System.out.println("❌ 접근 거부: " + currentUser + "는 이미지 접근 권한이 없습니다.");
            return false;
        }
        return true;
    }

    private void logAccess(String operation) {
        String timestamp = LocalDateTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss"));
        String logEntry = String.format("[%s] %s - %s (%s)", timestamp, operation, filename, currentUser);
        accessLog.add(logEntry);
    }

    private RealImage getRealImage() {
        if (realImage == null) {
            // 캐시에서 확인
            if (imageCache.containsKey(filename)) {
                System.out.println("💾 캐시에서 이미지 로드: " + filename);
                logAccess("캐시 히트");
            } else {
                logAccess("실제 객체 생성");
            }

            realImage = new RealImage(filename);

            // 메타데이터 캐싱
            if (cachedMetadata == null) {
                cachedMetadata = realImage.getMetadata();
                metadataCache.put(filename, cachedMetadata);
            }
        }
        return realImage;
    }

    @Override
    public void display() {
        if (!checkAccess()) return;

        logAccess("표시");

        // 캐시 확인 후 실제 이미지 표시
        if (imageCache.containsKey(filename)) {
            System.out.println("💾 캐시된 이미지 표시: " + filename);
        }

        getRealImage().display();

        // 이미지 데이터를 캐시에 저장
        if (realImage.isLoaded() && !imageCache.containsKey(filename)) {
            imageCache.put(filename, realImage.getRawData());
            System.out.println("💾 이미지 캐시에 저장: " + filename);
        }
    }

    @Override
    public void resize(int width, int height) {
        if (!checkAccess()) return;

        logAccess("리사이즈 " + width + "x" + height);
        getRealImage().resize(width, height);

        // 캐시 무효화 (크기가 변경되었으므로)
        imageCache.remove(filename);
        System.out.println("💾 캐시 무효화: " + filename);
    }

    @Override
    public void rotate(int degrees) {
        if (!checkAccess()) return;

        logAccess("회전 " + degrees + "도");
        getRealImage().rotate(degrees);

        // 캐시 무효화 (회전되었으므로)
        imageCache.remove(filename);
        System.out.println("💾 캐시 무효화: " + filename);
    }

    @Override
    public ImageMetadata getMetadata() {
        if (!checkAccess()) return null;

        // 메타데이터는 캐시에서 빠르게 반환
        if (cachedMetadata != null) {
            logAccess("메타데이터 (캐시)");
            return cachedMetadata;
        }

        logAccess("메타데이터");
        return getRealImage().getMetadata();
    }

    @Override
    public byte[] getRawData() {
        if (!checkAccess()) return null;

        logAccess("원본 데이터 요청");

        // 캐시에서 확인
        if (imageCache.containsKey(filename)) {
            System.out.println("💾 캐시에서 원본 데이터 반환: " + filename);
            return imageCache.get(filename).clone();
        }

        byte[] data = getRealImage().getRawData();
        imageCache.put(filename, data.clone());
        return data;
    }

    @Override
    public String getFormat() {
        return cachedMetadata != null ? cachedMetadata.getFormat() : getRealImage().getFormat();
    }

    @Override
    public boolean isLoaded() {
        return realImage != null && realImage.isLoaded();
    }

    // 정적 메서드들 (캐시 및 로그 관리)
    public static void displayCacheStatistics() {
        System.out.println("\n📊 캐시 통계");
        System.out.println("=".repeat(30));
        System.out.println("이미지 캐시 크기: " + imageCache.size());
        System.out.println("메타데이터 캐시 크기: " + metadataCache.size());

        if (!imageCache.isEmpty()) {
            System.out.println("\n캐시된 이미지:");
            for (String filename : imageCache.keySet()) {
                byte[] data = imageCache.get(filename);
                System.out.println("  - " + filename + " (" + formatBytes(data.length) + ")");
            }
        }
    }

    public static void displayAccessLog() {
        System.out.println("\n📜 접근 로그 (최근 20건)");
        System.out.println("=".repeat(50));

        int start = Math.max(0, accessLog.size() - 20);
        for (int i = start; i < accessLog.size(); i++) {
            System.out.println(accessLog.get(i));
        }
    }

    public static void clearCache() {
        imageCache.clear();
        metadataCache.clear();
        System.out.println("💾 모든 캐시가 클리어되었습니다.");
    }

    public static void addAuthorizedUser(String user) {
        authorizedUsers.add(user);
        System.out.println("👤 사용자 권한 추가: " + user);
    }

    public static void removeAuthorizedUser(String user) {
        authorizedUsers.remove(user);
        System.out.println("👤 사용자 권한 제거: " + user);
    }

    private static String formatBytes(long bytes) {
        if (bytes < 1024) return bytes + " B";
        if (bytes < 1024 * 1024) return String.format("%.1f KB", bytes / 1024.0);
        return String.format("%.1f MB", bytes / (1024.0 * 1024));
    }
}

// 이미지 갤러리 매니저
class ImageGalleryManager {
    private Map<String, Image> imageProxies;
    private String currentUser;

    public ImageGalleryManager(String user) {
        this.imageProxies = new HashMap<>();
        this.currentUser = user;
    }

    public void loadImage(String filename) {
        if (!imageProxies.containsKey(filename)) {
            imageProxies.put(filename, new ImageProxy(filename, currentUser));
            System.out.println("📸 이미지 프록시 생성: " + filename);
        }
    }

    public void displayImage(String filename) {
        Image image = imageProxies.get(filename);
        if (image != null) {
            image.display();
        } else {
            System.out.println("❌ 이미지를 찾을 수 없습니다: " + filename);
        }
    }

    public void resizeImage(String filename, int width, int height) {
        Image image = imageProxies.get(filename);
        if (image != null) {
            image.resize(width, height);
        } else {
            System.out.println("❌ 이미지를 찾을 수 없습니다: " + filename);
        }
    }

    public void rotateImage(String filename, int degrees) {
        Image image = imageProxies.get(filename);
        if (image != null) {
            image.rotate(degrees);
        } else {
            System.out.println("❌ 이미지를 찾을 수 없습니다: " + filename);
        }
    }

    public void showImageInfo(String filename) {
        Image image = imageProxies.get(filename);
        if (image != null) {
            ImageMetadata metadata = image.getMetadata();
            if (metadata != null) {
                System.out.println("ℹ️ 이미지 정보: " + metadata);
                System.out.println("   로드 상태: " + (image.isLoaded() ? "로드됨" : "지연 로드"));
            }
        } else {
            System.out.println("❌ 이미지를 찾을 수 없습니다: " + filename);
        }
    }

    public void listImages() {
        System.out.println("\n📁 로드된 이미지 목록");
        System.out.println("=".repeat(30));

        if (imageProxies.isEmpty()) {
            System.out.println("로드된 이미지가 없습니다.");
            return;
        }

        for (Map.Entry<String, Image> entry : imageProxies.entrySet()) {
            String filename = entry.getKey();
            Image image = entry.getValue();
            String status = image.isLoaded() ? "✅ 로드됨" : "⏳ 지연 로드";
            System.out.println("📸 " + filename + " - " + status);
        }
    }
}

// 이미지 프록시 시스템 데모
public class ImageProxyDemo {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("🖼️ 이미지 프록시 및 캐싱 시스템");
        System.out.println("=".repeat(50));

        // 1. 갤러리 매니저 생성 (권한 있는 사용자)
        ImageGalleryManager gallery = new ImageGalleryManager("admin");

        // 2. 이미지 프록시들 생성 (실제 로드는 아직 안됨)
        gallery.loadImage("vacation_photo.jpg");
        gallery.loadImage("wedding_picture.png");
        gallery.loadImage("family_portrait.jpg");
        gallery.loadImage("landscape.png");

        Thread.sleep(500);

        // 3. 이미지 정보 확인 (메타데이터만 로드됨)
        System.out.println("\n📋 이미지 정보 확인 (지연 로드 상태)");
        gallery.showImageInfo("vacation_photo.jpg");
        gallery.showImageInfo("wedding_picture.png");

        Thread.sleep(1000);

        // 4. 실제 이미지 표시 (이때 실제 데이터 로드)
        System.out.println("\n🖼️ 이미지 표시 테스트");
        gallery.displayImage("vacation_photo.jpg");
        Thread.sleep(500);

        gallery.displayImage("wedding_picture.png");
        Thread.sleep(500);

        // 5. 캐시 효과 테스트 (두 번째 접근은 빠름)
        System.out.println("\n💾 캐시 효과 테스트");
        gallery.displayImage("vacation_photo.jpg"); // 캐시에서 빠르게 로드
        Thread.sleep(500);

        // 6. 이미지 편집 (캐시 무효화 확인)
        System.out.println("\n✏️ 이미지 편집 테스트");
        gallery.resizeImage("vacation_photo.jpg", 800, 600);
        gallery.rotateImage("wedding_picture.png", 90);
        Thread.sleep(500);

        // 7. 권한 없는 사용자 테스트
        System.out.println("\n🔒 권한 테스트");
        ImageGalleryManager restrictedGallery = new ImageGalleryManager("unauthorized_user");
        restrictedGallery.loadImage("secret_document.jpg");
        restrictedGallery.displayImage("secret_document.jpg");
        Thread.sleep(500);

        // 8. 사용자 권한 추가 및 재시도
        ImageProxy.addAuthorizedUser("unauthorized_user");
        restrictedGallery.displayImage("secret_document.jpg");
        Thread.sleep(500);

        // 9. 갤러리 상태 확인
        gallery.listImages();

        // 10. 캐시 통계 및 접근 로그
        ImageProxy.displayCacheStatistics();
        ImageProxy.displayAccessLog();

        // 11. 캐시 클리어 테스트
        System.out.println("\n🗑️ 캐시 클리어 테스트");
        ImageProxy.clearCache();

        // 12. 캐시 클리어 후 재접근
        System.out.println("\n🔄 캐시 클리어 후 재접근");
        gallery.displayImage("vacation_photo.jpg"); // 다시 실제 로드 필요

        System.out.println("\n🎯 이미지 프록시 시스템 데모 완료!");
    }
}
```

**실행 결과 예시:**
```
🖼️ 이미지 프록시 및 캐싱 시스템
==================================================
📸 이미지 프록시 생성: vacation_photo.jpg
📋 메타데이터 로드 완료: vacation_photo.jpg
📸 이미지 프록시 생성: wedding_picture.png
📋 메타데이터 로드 완료: wedding_picture.png
📸 이미지 프록시 생성: family_portrait.jpg
📋 메타데이터 로드 완료: family_portrait.jpg
📸 이미지 프록시 생성: landscape.png
📋 메타데이터 로드 완료: landscape.png

📋 이미지 정보 확인 (지연 로드 상태)
ℹ️ 이미지 정보: vacation_photo.jpg (jpg) - 1456x1789, RGB, 7.23 MB
   로드 상태: 지연 로드
ℹ️ 이미지 정보: wedding_picture.png (png) - 1123x967, RGB, 4.56 MB
   로드 상태: 지연 로드

🖼️ 이미지 표시 테스트
⏳ 실제 이미지 데이터 로드 중: vacation_photo.jpg
✅ 이미지 로드 완료: vacation_photo.jpg (7.2 MB)
🖼️ 이미지 표시: vacation_photo.jpg
   크기: 1456x1789
   회전: 0도
   로드 시간: 14:30:25
💾 이미지 캐시에 저장: vacation_photo.jpg

💾 캐시 효과 테스트
💾 캐시된 이미지 표시: vacation_photo.jpg
🖼️ 이미지 표시: vacation_photo.jpg
   크기: 1456x1789
   회전: 0도
   로드 시간: 14:30:25
```

## 다른 구조 패턴과의 비교

| 패턴 | 목적 | 주요 특징 |
|------|------|-----------|
| **Proxy** | 접근 제어 및 부가 기능 | 같은 인터페이스, 투명한 접근 |
| **Adapter** | 인터페이스 변환 | 호환되지 않는 인터페이스 연결 |
| **Decorator** | 기능 추가 | 런타임에 동적으로 기능 확장 |
| **Facade** | 복잡성 단순화 | 여러 객체를 하나의 간단한 인터페이스로 |

## 기본 예제 코드 (Java)

```java
// Subject 인터페이스
interface Subject {
    void request();
}

// RealSubject 클래스
class RealSubject implements Subject {
    @Override
    public void request() {
        System.out.println("RealSubject: Handling request");
    }
}

// Proxy 클래스
class Proxy implements Subject {
    private RealSubject realSubject;
    private String clientId;

    public Proxy(String clientId) {
        this.clientId = clientId;
    }

    @Override
    public void request() {
        if (checkAccess()) {
            if (realSubject == null) {
                realSubject = new RealSubject();
            }

            logAccess();
            realSubject.request();
        }
    }

    private boolean checkAccess() {
        System.out.println("Proxy: Checking access for " + clientId);
        return "authorized".equals(clientId);
    }

    private void logAccess() {
        System.out.println("Proxy: Logging access for " + clientId);
    }
}

// 사용 예시
public class ProxyPatternDemo {
    public static void main(String[] args) {
        System.out.println("권한 있는 클라이언트:");
        Subject proxy1 = new Proxy("authorized");
        proxy1.request();

        System.out.println("\n권한 없는 클라이언트:");
        Subject proxy2 = new Proxy("unauthorized");
        proxy2.request();
    }
}
```

## 장점

- **접근 제어**: 실제 객체에 대한 접근을 세밀하게 제어할 수 있습니다.
- **지연 초기화**: 무거운 객체의 생성을 실제 필요할 때까지 지연시킬 수 있습니다.
- **캐싱**: 결과를 캐시하여 성능을 향상시킬 수 있습니다.
- **투명성**: 클라이언트는 프록시와 실제 객체를 구별하지 않고 사용할 수 있습니다.
- **부가 기능**: 로깅, 모니터링, 보안 등의 부가 기능을 쉽게 추가할 수 있습니다.

## 단점

- **복잡성 증가**: 추가적인 간접 계층으로 인해 코드가 복잡해질 수 있습니다.
- **성능 오버헤드**: 프록시를 통한 간접 호출로 인한 약간의 성능 저하가 있을 수 있습니다.
- **응답 지연**: 지연 초기화로 인해 첫 번째 접근 시 응답이 늦을 수 있습니다.
- **메모리 사용**: 캐싱 프록시의 경우 추가적인 메모리가 필요할 수 있습니다.

## 관련 패턴

### Proxy vs Decorator vs Adapter 비교

| 패턴 | 목적 | 인터페이스 | 사용 시점 |
|------|------|-----------|----------|
| **Proxy** | 접근 **제어** | 동일 | 지연 로딩, 캐싱, 권한 체크 |
| **Decorator** | 기능 **추가** | 동일 | 런타임에 기능 조합 |
| **Adapter** | 인터페이스 **변환** | 다름 | 호환성 맞추기 |

```java
// Proxy: 접근 제어 (하나만 감싸기)
PaymentService proxy = new SecurityProxy(realService);
proxy.pay();  // 권한 체크 → 실제 결제

// Decorator: 기능 추가 (겹쳐서 사용 가능)
Coffee coffee = new Espresso();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);

// Adapter: 인터페이스 변환
PaymentPort adapter = new TossPayAdapter(tossClient);
// TossClient 인터페이스를 PaymentPort에 맞춤
```

### Proxy의 종류별 비교

| 종류 | 목적 | 예시 |
|------|------|------|
| **가상 프록시** | 지연 로딩 | 대용량 이미지 lazy load |
| **보호 프록시** | 접근 제어 | 권한 체크 후 접근 허용 |
| **원격 프록시** | 원격 호출 | RMI, REST API 클라이언트 |
| **캐싱 프록시** | 결과 캐싱 | `@Cacheable` |
| **로깅 프록시** | 로깅/모니터링 | AOP 로깅 |

### Spring에서의 프록시 선택

```java
// 인터페이스가 있으면 → JDK Dynamic Proxy
interface PaymentService { }
class PaymentServiceImpl implements PaymentService { }
// Spring이 JDK Proxy 사용

// 인터페이스가 없으면 → CGLIB
class PaymentService {  // 인터페이스 없음
    @Transactional
    public void pay() { }
}
// Spring이 CGLIB 사용 (상속으로 프록시 생성)

// 강제로 CGLIB 사용
@EnableAspectJAutoProxy(proxyTargetClass = true)
```

### 실무 선택 가이드

| 상황 | 적합한 패턴 |
|------|------------|
| 객체 접근 전 권한/캐시/로깅 처리 | **Proxy** |
| 기능을 동적으로 조합해서 추가 | **Decorator** |
| 외부 API를 내 인터페이스에 맞추기 | **Adapter** |
| Spring에서 횡단 관심사 처리 | **Proxy (AOP)** |
| 무거운 객체 지연 로딩 | **Proxy (Virtual)** |

### 주의: @Transactional 프록시의 함정

```java
@Service
public class OrderService {

    @Transactional
    public void createOrder() {
        // 트랜잭션 시작
        saveOrder();
        sendNotification();  // 내부 호출 - 프록시 안 탐!
    }

    @Transactional(propagation = REQUIRES_NEW)
    public void sendNotification() {
        // 새 트랜잭션으로 시작하고 싶었지만...
        // 내부 호출이라 프록시를 거치지 않음!
    }
}

// 해결책: 자기 자신을 주입받거나 별도 서비스로 분리
@Service
@RequiredArgsConstructor
public class OrderService {
    private final NotificationService notificationService;  // 별도 서비스

    @Transactional
    public void createOrder() {
        saveOrder();
        notificationService.send();  // 외부 호출 - 프록시 정상 작동!
    }
}
```