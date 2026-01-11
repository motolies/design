# 팩토리 메서드 패턴 (Factory Method Pattern)

## 정의

팩토리 메서드 패턴은 객체를 생성하기 위한 인터페이스를 정의하지만, 어떤 클래스의 인스턴스를 생성할지에 대한 결정은 서브클래스에서 내리도록 하는 생성 디자인 패턴입니다. 즉, 객체 생성 과정을 서브클래스에 위임하는 것입니다.

## 🎯 한눈에 보기

| 항목 | 설명 |
|------|------|
| **핵심** | 객체 생성을 서브클래스에 위임하여 결합도 감소 |
| **비유** | 피자 가게: "피자 주세요" → 지점마다 다른 스타일 피자 제공 |
| **언제** | 생성할 객체 타입이 런타임에 결정될 때 |
| **Spring** | `FactoryBean`, `@Configuration` + `@Bean` |

> **💡 알림 서비스 생성할 때...**
>
> **❌ Before (직접 생성)**
> ```java
> if (type == "email") { new EmailSender() }
> else if (type == "sms") { new SmsSender() }
> else if (type == "push") { new PushSender() }
> // → 새 알림 타입 추가마다 코드 수정 필요!
> ```
>
> **✅ After (팩토리 메서드)**
> ```java
> NotificationFactory factory = getFactory(type);
> Notification notification = factory.create();
> // → 새 타입 = 새 Factory 클래스 추가만!
> ```

## 구조 (Structure)

```mermaid
classDiagram
    class Creator {
        <<abstract>>
        +factoryMethod(): Product
        +someOperation(): void
    }

    class ConcreteCreatorA {
        +factoryMethod(): Product
    }

    class ConcreteCreatorB {
        +factoryMethod(): Product
    }

    class Product {
        <<interface>>
        +use(): void
    }

    class ConcreteProductA {
        +use(): void
    }

    class ConcreteProductB {
        +use(): void
    }

    Creator <|-- ConcreteCreatorA
    Creator <|-- ConcreteCreatorB
    Product <|.. ConcreteProductA
    Product <|.. ConcreteProductB
    ConcreteCreatorA --> ConcreteProductA : creates
    ConcreteCreatorB --> ConcreteProductB : creates
    Creator ..> Product : uses
```

## 동작 흐름 (시퀀스 다이어그램)

```mermaid
sequenceDiagram
    participant Client as 클라이언트
    participant Creator as Creator<br/>(NotificationFactory)
    participant Product as Product<br/>(Notification)

    Note over Client,Product: 1. 팩토리 선택
    Client->>Creator: EmailNotificationFactory 선택

    Note over Client,Product: 2. 팩토리 메서드 호출
    Client->>Creator: create()
    Creator->>Product: new EmailNotification()
    Creator-->>Client: EmailNotification 반환

    Note over Client,Product: 3. 제품 사용
    Client->>Product: send()
    Product-->>Client: "이메일 발송 완료"
```

## 사용 이유

- **결합도 감소**: 객체를 생성하는 코드와 사용하는 코드를 분리하여 결합도를 낮출 수 있습니다. 클라이언트 코드는 구체적인 클래스 이름 대신 인터페이스에만 의존하게 됩니다.
- **유연성 및 확장성 증가**: 새로운 종류의 객체를 추가해야 할 때, 기존 팩토리 코드를 수정하는 대신 새로운 서브클래스를 추가하여 팩토리 메서드를 오버라이드하면 되므로 시스템의 확장성이 향상됩니다.

## 적용 상황

팩토리 메서드 패턴은 다음과 같은 상황에서 특히 유용합니다:

### 1. 런타임에 객체 타입이 결정되는 경우
- **게임 개발**: 플레이어가 선택한 캐릭터 클래스에 따라 다른 캐릭터 객체 생성
- **UI 컴포넌트**: 운영체제에 따라 다른 스타일의 버튼, 윈도우 생성
- **데이터베이스 연결**: 설정에 따라 MySQL, PostgreSQL, Oracle 등 다른 DB 드라이버 생성

### 2. 라이브러리나 프레임워크 개발 시
```java
// 나쁜 예: 직접 구현체에 의존
PaymentProcessor processor = new CreditCardProcessor();

// 좋은 예: 팩토리 메서드 사용
PaymentProcessorFactory factory = getFactory(paymentType);
PaymentProcessor processor = factory.createProcessor();
```

## 초급 예제 - 5분 만에 이해하기

가장 간단한 알림 서비스 생성으로 팩토리 메서드를 이해해봅시다.

```java
// 1단계: 제품 인터페이스 (모든 알림의 공통 규약)
interface Notification {
    void send(String message);
}

// 2단계: 구체적인 제품들
class EmailNotification implements Notification {
    public void send(String message) {
        System.out.println("📧 이메일 발송: " + message);
    }
}

class SmsNotification implements Notification {
    public void send(String message) {
        System.out.println("📱 SMS 발송: " + message);
    }
}

class PushNotification implements Notification {
    public void send(String message) {
        System.out.println("🔔 푸시 알림: " + message);
    }
}

// 3단계: 팩토리 (생성을 담당)
abstract class NotificationFactory {
    // 팩토리 메서드 - 서브클래스가 구현
    public abstract Notification createNotification();

    // 공통 로직
    public void notify(String message) {
        Notification notification = createNotification();
        notification.send(message);
    }
}

// 4단계: 구체적인 팩토리들
class EmailFactory extends NotificationFactory {
    public Notification createNotification() {
        return new EmailNotification();
    }
}

class SmsFactory extends NotificationFactory {
    public Notification createNotification() {
        return new SmsNotification();
    }
}

class PushFactory extends NotificationFactory {
    public Notification createNotification() {
        return new PushNotification();
    }
}

// 5단계: 사용
public class Main {
    public static void main(String[] args) {
        // 이메일로 알림
        NotificationFactory factory = new EmailFactory();
        factory.notify("주문이 완료되었습니다.");  // 📧 이메일 발송: 주문이 완료되었습니다.

        // SMS로 알림 (팩토리만 교체!)
        factory = new SmsFactory();
        factory.notify("배송이 시작되었습니다.");  // 📱 SMS 발송: 배송이 시작되었습니다.
    }
}
```

**핵심 포인트:**
- `Notification` 인터페이스로 모든 알림 타입 통일
- 각 Factory가 어떤 Notification을 생성할지 결정
- 클라이언트는 Factory만 알면 됨 (구체적인 클래스 몰라도 OK)
- 새 알림 타입 추가 = 새 클래스 2개만 추가 (Product + Factory)

---

## Spring Boot 예제

실무에서 Spring Boot와 함께 팩토리 메서드 패턴을 사용하는 방법입니다.

### 프로젝트 구조
```
src/main/java/com/example/notification/
├── domain/
│   ├── Notification.java           # 제품 인터페이스
│   ├── EmailNotification.java
│   ├── SmsNotification.java
│   └── PushNotification.java
├── factory/
│   ├── NotificationFactory.java    # 팩토리 인터페이스
│   ├── EmailFactory.java
│   ├── SmsFactory.java
│   └── PushFactory.java
├── service/
│   └── NotificationService.java
└── controller/
    └── NotificationController.java
```

### 1. 제품 인터페이스와 구현체

```java
public interface Notification {
    void send(String to, String message);
    String getType();
}

@Component
public class EmailNotification implements Notification {

    @Override
    public void send(String to, String message) {
        // 실제로는 JavaMailSender 등 사용
        log.info("📧 이메일 발송 - 수신: {}, 내용: {}", to, message);
    }

    @Override
    public String getType() {
        return "email";
    }
}

@Component
public class SmsNotification implements Notification {

    @Override
    public void send(String to, String message) {
        // 실제로는 SMS API 연동
        log.info("📱 SMS 발송 - 수신: {}, 내용: {}", to, message);
    }

    @Override
    public String getType() {
        return "sms";
    }
}

@Component
public class PushNotification implements Notification {

    @Override
    public void send(String to, String message) {
        // 실제로는 FCM, APNs 등 연동
        log.info("🔔 푸시 발송 - 수신: {}, 내용: {}", to, message);
    }

    @Override
    public String getType() {
        return "push";
    }
}
```

### 2. 팩토리 인터페이스와 구현체

```java
public interface NotificationFactory {
    Notification createNotification();
    String getSupportedType();
}

@Component
public class EmailFactory implements NotificationFactory {

    @Override
    public Notification createNotification() {
        return new EmailNotification();
    }

    @Override
    public String getSupportedType() {
        return "email";
    }
}

@Component
public class SmsFactory implements NotificationFactory {

    @Override
    public Notification createNotification() {
        return new SmsNotification();
    }

    @Override
    public String getSupportedType() {
        return "sms";
    }
}

@Component
public class PushFactory implements NotificationFactory {

    @Override
    public Notification createNotification() {
        return new PushNotification();
    }

    @Override
    public String getSupportedType() {
        return "push";
    }
}
```

### 3. 서비스 (팩토리 자동 주입)

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class NotificationService {

    // Spring이 모든 NotificationFactory 구현체를 List로 주입
    private final List<NotificationFactory> factories;

    // 타입별 팩토리 맵 (초기화 시 구성)
    private Map<String, NotificationFactory> factoryMap;

    @PostConstruct
    public void init() {
        factoryMap = factories.stream()
            .collect(Collectors.toMap(
                NotificationFactory::getSupportedType,
                factory -> factory
            ));
        log.info("등록된 알림 팩토리: {}", factoryMap.keySet());
    }

    public void send(String type, String to, String message) {
        NotificationFactory factory = factoryMap.get(type);

        if (factory == null) {
            throw new IllegalArgumentException("지원하지 않는 알림 타입: " + type);
        }

        Notification notification = factory.createNotification();
        notification.send(to, message);
    }

    public List<String> getSupportedTypes() {
        return new ArrayList<>(factoryMap.keySet());
    }
}
```

### 4. 컨트롤러

```java
@RestController
@RequestMapping("/api/notifications")
@RequiredArgsConstructor
public class NotificationController {

    private final NotificationService notificationService;

    @PostMapping("/{type}")
    public ResponseEntity<String> send(
            @PathVariable String type,
            @RequestBody NotificationRequest request) {

        notificationService.send(type, request.getTo(), request.getMessage());
        return ResponseEntity.ok("알림 발송 완료");
    }

    @GetMapping("/types")
    public ResponseEntity<List<String>> getTypes() {
        return ResponseEntity.ok(notificationService.getSupportedTypes());
    }
}

@Getter
@AllArgsConstructor
@NoArgsConstructor
public class NotificationRequest {
    private String to;
    private String message;
}
```

### 사용 예시 (API 호출)

```bash
# 이메일 알림 발송
curl -X POST http://localhost:8080/api/notifications/email \
  -H "Content-Type: application/json" \
  -d '{"to": "user@example.com", "message": "주문 완료"}'

# SMS 알림 발송
curl -X POST http://localhost:8080/api/notifications/sms \
  -H "Content-Type: application/json" \
  -d '{"to": "010-1234-5678", "message": "배송 시작"}'

# 지원 타입 조회
curl http://localhost:8080/api/notifications/types
# ["email", "sms", "push"]
```

### 새로운 알림 타입 추가하기

카카오톡 알림을 추가하려면:

```java
// 1. 제품 구현
@Component
public class KakaoNotification implements Notification {
    @Override
    public void send(String to, String message) {
        log.info("💬 카카오톡 발송 - 수신: {}, 내용: {}", to, message);
    }

    @Override
    public String getType() {
        return "kakao";
    }
}

// 2. 팩토리 구현
@Component
public class KakaoFactory implements NotificationFactory {
    @Override
    public Notification createNotification() {
        return new KakaoNotification();
    }

    @Override
    public String getSupportedType() {
        return "kakao";
    }
}
```

**기존 코드 수정 없이** 바로 `/api/notifications/kakao`로 호출 가능!

---

## 실생활 예제 - 게임 캐릭터 생성 시스템

RPG 게임에서 플레이어가 선택한 직업에 따라 다른 특성을 가진 캐릭터를 생성하는 시스템을 구현해보겠습니다.

```java
// 게임 캐릭터 인터페이스
interface GameCharacter {
    void attack();
    void defend();
    void useSpecialSkill();
    String getCharacterInfo();
}

// 전사 캐릭터
class Warrior implements GameCharacter {
    private String name;
    private int health = 150;
    private int damage = 25;

    public Warrior(String name) {
        this.name = name;
    }

    @Override
    public void attack() {
        System.out.println(name + " 전사가 검으로 " + damage + "의 피해를 입힙니다!");
    }

    @Override
    public void defend() {
        System.out.println(name + " 전사가 방패로 방어합니다! (피해 50% 감소)");
    }

    @Override
    public void useSpecialSkill() {
        System.out.println(name + " 전사가 '분노의 일격'을 사용합니다! (피해 2배)");
    }

    @Override
    public String getCharacterInfo() {
        return String.format("전사 %s - 체력: %d, 공격력: %d", name, health, damage);
    }
}

// 마법사 캐릭터
class Mage implements GameCharacter {
    private String name;
    private int health = 80;
    private int mana = 120;
    private int damage = 35;

    public Mage(String name) {
        this.name = name;
    }

    @Override
    public void attack() {
        System.out.println(name + " 마법사가 파이어볼로 " + damage + "의 마법 피해를 입힙니다!");
    }

    @Override
    public void defend() {
        System.out.println(name + " 마법사가 마법 보호막을 사용합니다! (마나 -10)");
    }

    @Override
    public void useSpecialSkill() {
        System.out.println(name + " 마법사가 '메테오'를 시전합니다! (광역 피해)");
    }

    @Override
    public String getCharacterInfo() {
        return String.format("마법사 %s - 체력: %d, 마나: %d, 마법력: %d", name, health, mana, damage);
    }
}

// 궁수 캐릭터
class Archer implements GameCharacter {
    private String name;
    private int health = 100;
    private int arrows = 50;
    private int damage = 20;

    public Archer(String name) {
        this.name = name;
    }

    @Override
    public void attack() {
        if (arrows > 0) {
            System.out.println(name + " 궁수가 화살로 " + damage + "의 피해를 입힙니다! (남은 화살: " + (--arrows) + ")");
        } else {
            System.out.println(name + " 궁수의 화살이 떨어졌습니다!");
        }
    }

    @Override
    public void defend() {
        System.out.println(name + " 궁수가 재빠르게 회피합니다! (회피율 70%)");
    }

    @Override
    public void useSpecialSkill() {
        System.out.println(name + " 궁수가 '관통사격'을 사용합니다! (적 관통)");
    }

    @Override
    public String getCharacterInfo() {
        return String.format("궁수 %s - 체력: %d, 화살: %d, 공격력: %d", name, health, arrows, damage);
    }
}

// 캐릭터 생성 팩토리 추상 클래스
abstract class CharacterCreator {
    // 팩토리 메서드 - 서브클래스에서 구현
    public abstract GameCharacter createCharacter(String playerName);

    // 캐릭터 생성 및 초기화를 담당하는 템플릿 메서드
    public GameCharacter createAndInitializeCharacter(String playerName) {
        GameCharacter character = createCharacter(playerName);

        System.out.println("=== 캐릭터 생성 완료 ===");
        System.out.println(character.getCharacterInfo());
        System.out.println("초기 장비를 지급합니다...");
        System.out.println("튜토리얼을 시작합니다...");

        return character;
    }
}

// 전사 생성 팩토리
class WarriorCreator extends CharacterCreator {
    @Override
    public GameCharacter createCharacter(String playerName) {
        System.out.println("용맹한 전사를 생성합니다...");
        return new Warrior(playerName);
    }
}

// 마법사 생성 팩토리
class MageCreator extends CharacterCreator {
    @Override
    public GameCharacter createCharacter(String playerName) {
        System.out.println("지혜로운 마법사를 생성합니다...");
        return new Mage(playerName);
    }
}

// 궁수 생성 팩토리
class ArcherCreator extends CharacterCreator {
    @Override
    public GameCharacter createCharacter(String playerName) {
        System.out.println("민첩한 궁수를 생성합니다...");
        return new Archer(playerName);
    }
}

// 게임 시스템
public class GameCharacterSystem {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("=== RPG 게임 캐릭터 생성 ===");
        System.out.print("플레이어 이름을 입력하세요: ");
        String playerName = scanner.nextLine();

        System.out.println("\n직업을 선택하세요:");
        System.out.println("1. 전사 (높은 체력과 방어력)");
        System.out.println("2. 마법사 (강력한 마법 공격)");
        System.out.println("3. 궁수 (빠른 원거리 공격)");
        System.out.print("선택 (1-3): ");

        int choice = scanner.nextInt();
        CharacterCreator creator = getCharacterCreator(choice);

        if (creator != null) {
            GameCharacter character = creator.createAndInitializeCharacter(playerName);

            // 캐릭터 테스트
            System.out.println("\n=== 캐릭터 능력 테스트 ===");
            character.attack();
            character.defend();
            character.useSpecialSkill();
        } else {
            System.out.println("잘못된 선택입니다.");
        }

        scanner.close();
    }

    // 팩토리 선택 로직
    private static CharacterCreator getCharacterCreator(int choice) {
        return switch (choice) {
            case 1 -> new WarriorCreator();
            case 2 -> new MageCreator();
            case 3 -> new ArcherCreator();
            default -> null;
        };
    }
}
```

**실행 결과 예시:**
```
=== RPG 게임 캐릭터 생성 ===
플레이어 이름을 입력하세요: 드래곤슬레이어

직업을 선택하세요:
1. 전사 (높은 체력과 방어력)
2. 마법사 (강력한 마법 공격)
3. 궁수 (빠른 원거리 공격)
선택 (1-3): 1

용맹한 전사를 생성합니다...
=== 캐릭터 생성 완료 ===
전사 드래곤슬레이어 - 체력: 150, 공격력: 25
초기 장비를 지급합니다...
튜토리얼을 시작합니다...

=== 캐릭터 능력 테스트 ===
드래곤슬레이어 전사가 검으로 25의 피해를 입힙니다!
드래곤슬레이어 전사가 방패로 방어합니다! (피해 50% 감소)
드래곤슬레이어 전사가 '분노의 일격'을 사용합니다! (피해 2배)
```

## 기본 예제 코드 (Java)

```java
// Product 인터페이스
interface Product {
    void use();
}

// ConcreteProduct A
class ConcreteProductA implements Product {
    @Override
    public void use() {
        System.out.println("Using Product A");
    }
}

// ConcreteProduct B
class ConcreteProductB implements Product {
    @Override
    public void use() {
        System.out.println("Using Product B");
    }
}

// Creator (Factory) 추상 클래스
abstract class Creator {
    // 팩토리 메서드 (서브클래스가 구현)
    public abstract Product factoryMethod();

    public void someOperation() {
        Product product = factoryMethod();
        product.use();
    }
}

// ConcreteCreator A
class ConcreteCreatorA extends Creator {
    @Override
    public Product factoryMethod() {
        return new ConcreteProductA();
    }
}

// ConcreteCreator B
class ConcreteCreatorB extends Creator {
    @Override
    public Product factoryMethod() {
        return new ConcreteProductB();
    }
}

// 사용 예시
public class Client {
    public static void main(String[] args) {
        Creator creatorA = new ConcreteCreatorA();
        creatorA.someOperation(); // "Using Product A" 출력

        Creator creatorB = new ConcreteCreatorB();
        creatorB.someOperation(); // "Using Product B" 출력
    }
}
```

## 장점

- **느슨한 결합**: 클라이언트와 구체적인 제품 클래스 간의 결합을 분리합니다.
- **단일 책임 원칙(SRP)**: 제품 생성 코드를 한 곳으로 모아 코드의 유지보수성을 높입니다.
- **개방-폐쇄 원칙(OCP)**: 새로운 제품 유형을 추가할 때 기존 클라이언트 코드를 수정하지 않고 새로운 생성자 클래스를 추가할 수 있습니다.
- **코드 재사용성**: 공통적인 객체 생성 로직을 추상 클래스에서 정의하여 재사용할 수 있습니다.

## 단점

- **코드 복잡성 증가**: 패턴을 구현하기 위해 많은 새로운 클래스(인터페이스, 생성자, 제품 클래스 등)가 필요하므로 코드의 전체적인 복잡성이 증가할 수 있습니다.
- **클래스 수 증가**: 새로운 제품 타입마다 해당하는 팩토리 클래스를 만들어야 하므로 클래스 수가 늘어납니다.

## 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Abstract Factory** | Factory Method의 확장 - 관련 객체 군(family)을 생성 |
| **Template Method** | Factory Method는 Template Method의 특수한 형태 |
| **Prototype** | Factory 없이 복제로 객체 생성 (대안적 접근) |
| **Singleton** | Factory를 싱글톤으로 구현하는 경우가 많음 |

### Factory Method vs Abstract Factory

```java
// Factory Method: 하나의 제품 생성
interface NotificationFactory {
    Notification createNotification();  // 단일 제품
}

// Abstract Factory: 관련 제품군 생성
interface UIFactory {
    Button createButton();      // 제품 1
    Checkbox createCheckbox();  // 제품 2
    TextField createTextField(); // 제품 3
}
```

| 비교 | Factory Method | Abstract Factory |
|------|---------------|------------------|
| 생성 대상 | 단일 제품 | 관련 제품군 |
| 확장 방법 | 서브클래스 추가 | 새 팩토리 구현 |
| 복잡도 | 낮음 | 높음 |
| 사용 사례 | 알림, 로거 | UI 테마, DB 드라이버 |