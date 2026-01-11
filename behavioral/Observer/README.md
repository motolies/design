# 옵저버 패턴 (Observer Pattern)

## 정의

옵저버 패턴은 한 객체(주제, Subject)의 상태가 변경되면 그 객체에 의존하는 다른 객체(옵저버, Observer)들에게 자동으로 알림을 보내고 업데이트하는 행동 디자인 패턴입니다. 이는 일대다(one-to-many)의 의존 관계를 정의합니다.

## 🎯 한눈에 보기

| 항목 | 설명 |
|------|------|
| **핵심** | 상태 변경 시 관련된 모든 객체에 자동으로 알림 |
| **비유** | 유튜브 구독: 새 영상 업로드 → 모든 구독자에게 알림 |
| **언제** | 이벤트 발생 시 여러 객체가 반응해야 할 때 |
| **Spring** | `@EventListener`, `ApplicationEvent`, `ApplicationEventPublisher` |

> **💡 주문 완료 시 여러 작업이 필요할 때...**
>
> **❌ Before (직접 호출)**
> ```java
> orderService.complete() {
>     emailService.send();        // 이메일
>     smsService.send();          // SMS
>     pointService.add();         // 포인트
>     inventoryService.update();  // 재고
>     // 새 기능 추가마다 코드 수정!
> }
> ```
>
> **✅ After (옵저버 패턴)**
> ```java
> eventPublisher.publish(OrderCompletedEvent)
> // ↓ 자동으로 모든 리스너에게 전파
> // → EmailListener, SmsListener, PointListener, ...
> ```

## 구조 (Structure)

```mermaid
classDiagram
    class Subject {
        <<interface>>
        +registerObserver(Observer): void
        +removeObserver(Observer): void
        +notifyObservers(): void
    }

    class ConcreteSubject {
        -observers: List~Observer~
        -state: Object
        +registerObserver(Observer): void
        +removeObserver(Observer): void
        +notifyObservers(): void
        +getState(): Object
        +setState(Object): void
    }

    class Observer {
        <<interface>>
        +update(Object): void
    }

    class ConcreteObserverA {
        -subject: Subject
        +update(Object): void
    }

    class ConcreteObserverB {
        -subject: Subject
        +update(Object): void
    }

    Subject <|.. ConcreteSubject
    Observer <|.. ConcreteObserverA
    Observer <|.. ConcreteObserverB
    Subject o--> Observer : notifies
    ConcreteSubject --> Observer : "1..*"

    note for Subject "상태 변경을 알리는 주제"
    note for Observer "상태 변경에 반응하는 관찰자"
```

## 동작 흐름 (시퀀스 다이어그램)

```mermaid
sequenceDiagram
    participant Subject as Subject<br/>(OrderService)
    participant Observer1 as Observer 1<br/>(EmailService)
    participant Observer2 as Observer 2<br/>(PointService)
    participant Observer3 as Observer 3<br/>(InventoryService)

    Note over Subject,Observer3: 1. 옵저버 등록
    Observer1->>Subject: registerObserver()
    Observer2->>Subject: registerObserver()
    Observer3->>Subject: registerObserver()

    Note over Subject,Observer3: 2. 상태 변경 발생
    Subject->>Subject: 주문 완료 처리

    Note over Subject,Observer3: 3. 모든 옵저버에게 알림
    Subject->>Observer1: update(OrderEvent)
    Subject->>Observer2: update(OrderEvent)
    Subject->>Observer3: update(OrderEvent)

    Observer1-->>Subject: 이메일 발송 완료
    Observer2-->>Subject: 포인트 적립 완료
    Observer3-->>Subject: 재고 감소 완료
```

## 사용 이유

- **느슨한 결합**: 주제(Subject)와 옵저버(Observer)는 서로의 구체적인 구현을 알 필요 없이, 추상적인 인터페이스를 통해 상호작용합니다. 이로 인해 두 컴포넌트 간의 결합도가 낮아집니다.
- **동적 관계**: 런타임에 동적으로 옵저버를 추가하거나 제거할 수 있어 유연한 관계 설정이 가능합니다.
- **이벤트 기반 시스템**: GUI 이벤트 처리, 뉴스 구독 시스템, 주식 가격 변동 알림 등 상태 변화에 따라 여러 객체에 변경 사항을 전파해야 하는 경우에 널리 사용됩니다.

## 적용 상황

옵저버 패턴은 다음과 같은 상황에서 특히 유용합니다:

### 1. 실시간 데이터 모니터링
- **주식 거래 시스템**: 주식 가격 변동을 여러 화면에 실시간 업데이트
- **IoT 센서 데이터**: 온도, 습도 센서 데이터를 여러 대시보드에 전파
- **게임 점수 시스템**: 점수 변화를 순위표, UI, 업적 시스템에 알림

### 2. 이벤트 기반 시스템
- **GUI 이벤트**: 버튼 클릭, 텍스트 변경 등의 이벤트 처리
- **알림 시스템**: 이메일, SMS, 푸시 알림 등 다중 채널 알림
- **로그 시스템**: 하나의 이벤트를 여러 로그 파일에 기록

### 3. 상태 변화 전파가 필요한 경우
```java
// 나쁜 예: 직접 결합으로 확장성 부족
class UserService {
    public void updateUser(User user) {
        // 사용자 정보 업데이트
        emailService.sendEmail(user); // 직접 호출
        smsService.sendSMS(user);     // 직접 호출
        // 새로운 알림 방식 추가 시 코드 수정 필요!
    }
}

// 좋은 예: 옵저버 패턴으로 확장성 확보
class UserService extends Subject {
    public void updateUser(User user) {
        // 사용자 정보 업데이트
        notifyObservers(user); // 모든 등록된 옵저버에게 알림
    }
}
```

## 초급 예제 - 5분 만에 이해하기

유튜브 구독 시스템으로 옵저버 패턴을 이해해봅시다.

```java
// 1단계: 옵저버 인터페이스 (구독자)
interface Subscriber {
    void onNewVideo(String videoTitle);
}

// 2단계: 구체적인 옵저버들 (구독자 종류)
class EmailSubscriber implements Subscriber {
    private String email;

    public EmailSubscriber(String email) {
        this.email = email;
    }

    @Override
    public void onNewVideo(String videoTitle) {
        System.out.println("📧 " + email + "로 알림: 새 영상 '" + videoTitle + "'");
    }
}

class AppSubscriber implements Subscriber {
    private String userId;

    public AppSubscriber(String userId) {
        this.userId = userId;
    }

    @Override
    public void onNewVideo(String videoTitle) {
        System.out.println("📱 " + userId + "님 앱 알림: 새 영상 '" + videoTitle + "'");
    }
}

// 3단계: Subject (유튜브 채널)
class YouTubeChannel {
    private String channelName;
    private List<Subscriber> subscribers = new ArrayList<>();

    public YouTubeChannel(String channelName) {
        this.channelName = channelName;
    }

    public void subscribe(Subscriber subscriber) {
        subscribers.add(subscriber);
        System.out.println("✅ 새 구독자 등록!");
    }

    public void unsubscribe(Subscriber subscriber) {
        subscribers.remove(subscriber);
        System.out.println("❌ 구독 해제");
    }

    public void uploadVideo(String title) {
        System.out.println("\n🎬 " + channelName + " 채널에 새 영상 업로드: " + title);
        notifySubscribers(title);
    }

    private void notifySubscribers(String title) {
        for (Subscriber subscriber : subscribers) {
            subscriber.onNewVideo(title);
        }
    }
}

// 4단계: 사용
public class Main {
    public static void main(String[] args) {
        YouTubeChannel channel = new YouTubeChannel("코딩 채널");

        // 구독자 등록
        Subscriber email = new EmailSubscriber("user@example.com");
        Subscriber app = new AppSubscriber("user123");

        channel.subscribe(email);
        channel.subscribe(app);

        // 새 영상 업로드 → 모든 구독자에게 자동 알림!
        channel.uploadVideo("옵저버 패턴 완벽 정리");
    }
}
```

**출력:**
```
✅ 새 구독자 등록!
✅ 새 구독자 등록!

🎬 코딩 채널 채널에 새 영상 업로드: 옵저버 패턴 완벽 정리
📧 user@example.com로 알림: 새 영상 '옵저버 패턴 완벽 정리'
📱 user123님 앱 알림: 새 영상 '옵저버 패턴 완벽 정리'
```

**핵심 포인트:**
- `YouTubeChannel`(Subject)은 구독자 목록만 관리
- 새 영상 업로드 시 모든 구독자에게 자동 알림
- 새로운 알림 방식 추가 = 새 Subscriber 클래스만 추가

---

## Spring Boot 예제

Spring의 이벤트 시스템으로 옵저버 패턴을 구현합니다. **Spring에서는 직접 옵저버 패턴을 구현할 필요 없이 `@EventListener`를 사용합니다!**

### 프로젝트 구조
```
src/main/java/com/example/order/
├── event/
│   └── OrderCompletedEvent.java    # 이벤트 클래스
├── listener/
│   ├── EmailNotificationListener.java
│   ├── PointAccumulationListener.java
│   └── InventoryUpdateListener.java
├── service/
│   └── OrderService.java           # 이벤트 발행
└── controller/
    └── OrderController.java
```

### 1. 이벤트 클래스 정의

```java
@Getter
@AllArgsConstructor
public class OrderCompletedEvent {
    private Long orderId;
    private Long customerId;
    private int totalAmount;
    private List<String> productNames;
    private LocalDateTime orderedAt;

    public OrderCompletedEvent(Long orderId, Long customerId, int totalAmount, List<String> productNames) {
        this.orderId = orderId;
        this.customerId = customerId;
        this.totalAmount = totalAmount;
        this.productNames = productNames;
        this.orderedAt = LocalDateTime.now();
    }
}
```

### 2. 이벤트 리스너들 (옵저버)

```java
@Component
@Slf4j
public class EmailNotificationListener {

    @EventListener
    public void handleOrderCompleted(OrderCompletedEvent event) {
        log.info("📧 이메일 발송 - 주문 #{}, 고객 #{}, 금액: {}원",
                event.getOrderId(), event.getCustomerId(), event.getTotalAmount());
        // 실제로는 메일 서비스 호출
    }
}

@Component
@Slf4j
public class PointAccumulationListener {

    @EventListener
    public void handleOrderCompleted(OrderCompletedEvent event) {
        int points = event.getTotalAmount() / 100;  // 1% 적립
        log.info("💰 포인트 적립 - 고객 #{}, {}포인트 적립",
                event.getCustomerId(), points);
        // 실제로는 포인트 서비스 호출
    }
}

@Component
@Slf4j
public class InventoryUpdateListener {

    @EventListener
    public void handleOrderCompleted(OrderCompletedEvent event) {
        log.info("📦 재고 감소 - 상품: {}",
                String.join(", ", event.getProductNames()));
        // 실제로는 재고 서비스 호출
    }
}

// 비동기 처리가 필요한 경우
@Component
@Slf4j
public class SmsNotificationListener {

    @Async  // 비동기 처리
    @EventListener
    public void handleOrderCompleted(OrderCompletedEvent event) {
        log.info("📱 SMS 발송 (비동기) - 주문 #{}", event.getOrderId());
        // 실제로는 SMS API 호출
    }
}
```

### 3. 이벤트 발행 서비스

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderService {

    private final ApplicationEventPublisher eventPublisher;

    public Long completeOrder(Long customerId, int totalAmount, List<String> products) {
        // 1. 주문 처리 로직
        Long orderId = saveOrder(customerId, totalAmount, products);
        log.info("✅ 주문 완료 - 주문 #{}", orderId);

        // 2. 이벤트 발행 → 모든 리스너에게 자동 전파!
        OrderCompletedEvent event = new OrderCompletedEvent(
                orderId, customerId, totalAmount, products
        );
        eventPublisher.publishEvent(event);

        return orderId;
    }

    private Long saveOrder(Long customerId, int totalAmount, List<String> products) {
        // DB 저장 로직
        return System.currentTimeMillis();  // 임시 ID
    }
}
```

### 4. 컨트롤러

```java
@RestController
@RequestMapping("/api/orders")
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;

    @PostMapping
    public ResponseEntity<Map<String, Object>> createOrder(@RequestBody OrderRequest request) {
        Long orderId = orderService.completeOrder(
                request.getCustomerId(),
                request.getTotalAmount(),
                request.getProducts()
        );

        return ResponseEntity.ok(Map.of(
                "orderId", orderId,
                "message", "주문이 완료되었습니다."
        ));
    }
}

@Getter
@AllArgsConstructor
@NoArgsConstructor
public class OrderRequest {
    private Long customerId;
    private int totalAmount;
    private List<String> products;
}
```

### 사용 예시

```bash
curl -X POST http://localhost:8080/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": 1,
    "totalAmount": 50000,
    "products": ["노트북 파우치", "무선 마우스"]
  }'
```

**출력 로그:**
```
✅ 주문 완료 - 주문 #1704123456789
📧 이메일 발송 - 주문 #1704123456789, 고객 #1, 금액: 50000원
💰 포인트 적립 - 고객 #1, 500포인트 적립
📦 재고 감소 - 상품: 노트북 파우치, 무선 마우스
📱 SMS 발송 (비동기) - 주문 #1704123456789
```

### 새로운 기능 추가하기

주문 완료 시 슬랙 알림을 추가하려면 **리스너만 추가**:

```java
@Component
@Slf4j
public class SlackNotificationListener {

    @EventListener
    public void handleOrderCompleted(OrderCompletedEvent event) {
        log.info("💬 슬랙 알림 - 새 주문 발생! #{}번 주문, {}원",
                event.getOrderId(), event.getTotalAmount());
        // 슬랙 API 호출
    }
}
```

**기존 코드 수정 없이** 슬랙 알림이 자동으로 동작!

### 조건부 처리

```java
@Component
public class VipNotificationListener {

    @EventListener(condition = "#event.totalAmount >= 100000")
    public void handleVipOrder(OrderCompletedEvent event) {
        // 10만원 이상 주문만 처리
        log.info("🌟 VIP 주문 감지! - 금액: {}원", event.getTotalAmount());
    }
}
```

---

## 실생활 예제 - 주식 가격 모니터링 시스템

실시간 주식 가격 변화를 여러 화면과 시스템에 전파하는 시스템을 옵저버 패턴으로 구현해보겠습니다.

```java
import java.util.*;
import java.text.DecimalFormat;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

// 주식 데이터 클래스
class StockData {
    private String symbol;
    private double price;
    private double previousPrice;
    private int volume;
    private LocalDateTime timestamp;

    public StockData(String symbol, double price, double previousPrice, int volume) {
        this.symbol = symbol;
        this.price = price;
        this.previousPrice = previousPrice;
        this.volume = volume;
        this.timestamp = LocalDateTime.now();
    }

    public double getChangePercent() {
        if (previousPrice == 0) return 0;
        return ((price - previousPrice) / previousPrice) * 100;
    }

    public double getChangeAmount() {
        return price - previousPrice;
    }

    public boolean isIncrease() {
        return price > previousPrice;
    }

    // getter 메서드들
    public String getSymbol() { return symbol; }
    public double getPrice() { return price; }
    public double getPreviousPrice() { return previousPrice; }
    public int getVolume() { return volume; }
    public LocalDateTime getTimestamp() { return timestamp; }

    @Override
    public String toString() {
        DecimalFormat df = new DecimalFormat("#,##0.00");
        String changeStr = String.format("%s%.2f (%.2f%%)",
            isIncrease() ? "+" : "", getChangeAmount(), getChangePercent());
        return String.format("%s: $%s %s", symbol, df.format(price), changeStr);
    }
}

// 주식 옵저버 인터페이스
interface StockObserver {
    void onStockUpdate(StockData stockData);
    String getObserverName();
}

// 주식 주제 인터페이스
interface StockSubject {
    void registerObserver(StockObserver observer);
    void removeObserver(StockObserver observer);
    void notifyObservers(StockData stockData);
}

// 주식 가격 서비스 (ConcreteSubject)
class StockPriceService implements StockSubject {
    private List<StockObserver> observers;
    private Map<String, StockData> stocks;

    public StockPriceService() {
        this.observers = new ArrayList<>();
        this.stocks = new HashMap<>();
    }

    @Override
    public void registerObserver(StockObserver observer) {
        observers.add(observer);
        System.out.println("✅ " + observer.getObserverName() + " 등록됨");
    }

    @Override
    public void removeObserver(StockObserver observer) {
        observers.remove(observer);
        System.out.println("❌ " + observer.getObserverName() + " 제거됨");
    }

    @Override
    public void notifyObservers(StockData stockData) {
        System.out.println("\n📢 주식 가격 변동 알림: " + stockData);
        for (StockObserver observer : observers) {
            observer.onStockUpdate(stockData);
        }
    }

    public void updateStockPrice(String symbol, double newPrice, int volume) {
        StockData previousData = stocks.get(symbol);
        double previousPrice = previousData != null ? previousData.getPrice() : newPrice;

        StockData newData = new StockData(symbol, newPrice, previousPrice, volume);
        stocks.put(symbol, newData);

        notifyObservers(newData);
    }

    public StockData getStockData(String symbol) {
        return stocks.get(symbol);
    }

    public Set<String> getAllSymbols() {
        return stocks.keySet();
    }
}

// 트레이딩 대시보드 (ConcreteObserver)
class TradingDashboard implements StockObserver {
    private String name;
    private Map<String, StockData> watchList;

    public TradingDashboard(String name) {
        this.name = name;
        this.watchList = new HashMap<>();
    }

    @Override
    public void onStockUpdate(StockData stockData) {
        watchList.put(stockData.getSymbol(), stockData);

        String trendIcon = stockData.isIncrease() ? "📈" : "📉";
        String timeStr = stockData.getTimestamp().format(DateTimeFormatter.ofPattern("HH:mm:ss"));

        System.out.println(String.format("[%s] %s %s %s (거래량: %,d)",
            name, trendIcon, stockData, timeStr, stockData.getVolume()));
    }

    @Override
    public String getObserverName() {
        return "Trading Dashboard (" + name + ")";
    }

    public void displayWatchList() {
        System.out.println("\n=== " + name + " 관심종목 ===");
        if (watchList.isEmpty()) {
            System.out.println("관심종목이 없습니다.");
            return;
        }

        for (StockData stock : watchList.values()) {
            String trendIcon = stock.isIncrease() ? "📈" : "📉";
            System.out.println(trendIcon + " " + stock);
        }
    }
}

// 알림 서비스 (ConcreteObserver)
class AlertService implements StockObserver {
    private Map<String, Double> alertThresholds;

    public AlertService() {
        this.alertThresholds = new HashMap<>();
    }

    @Override
    public void onStockUpdate(StockData stockData) {
        Double threshold = alertThresholds.get(stockData.getSymbol());
        if (threshold != null) {
            if (Math.abs(stockData.getChangePercent()) >= threshold) {
                sendAlert(stockData);
            }
        }
    }

    @Override
    public String getObserverName() {
        return "Alert Service";
    }

    public void setAlertThreshold(String symbol, double threshold) {
        alertThresholds.put(symbol, threshold);
        System.out.println("🚨 " + symbol + " 알림 임계값 설정: ±" + threshold + "%");
    }

    private void sendAlert(StockData stockData) {
        String alertType = stockData.isIncrease() ? "급등" : "급락";
        System.out.println(String.format("🚨 [긴급알림] %s %s! %.2f%% 변동",
            stockData.getSymbol(), alertType, Math.abs(stockData.getChangePercent())));
    }
}

// 포트폴리오 매니저 (ConcreteObserver)
class PortfolioManager implements StockObserver {
    private String managerName;
    private Map<String, Integer> holdings; // 종목별 보유량
    private double totalValue;

    public PortfolioManager(String managerName) {
        this.managerName = managerName;
        this.holdings = new HashMap<>();
        this.totalValue = 0.0;
    }

    @Override
    public void onStockUpdate(StockData stockData) {
        Integer shares = holdings.get(stockData.getSymbol());
        if (shares != null && shares > 0) {
            double stockValue = stockData.getPrice() * shares;
            double changeValue = stockData.getChangeAmount() * shares;

            System.out.println(String.format("[%s] %s 포지션 변동: $%,.2f (%s$%,.2f)",
                managerName, stockData.getSymbol(), stockValue,
                changeValue >= 0 ? "+" : "", changeValue));

            updateTotalValue();
        }
    }

    @Override
    public String getObserverName() {
        return "Portfolio Manager (" + managerName + ")";
    }

    public void addHolding(String symbol, int shares) {
        holdings.put(symbol, holdings.getOrDefault(symbol, 0) + shares);
        System.out.println(managerName + "가 " + symbol + " " + shares + "주 보유");
    }

    private void updateTotalValue() {
        // 실제로는 모든 보유 종목의 현재 가치를 계산
        totalValue += 1000; // 예시
    }

    public void displayPortfolio() {
        System.out.println("\n=== " + managerName + " 포트폴리오 ===");
        for (Map.Entry<String, Integer> entry : holdings.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue() + "주 보유");
        }
    }
}

// 시장 데이터 분석기 (ConcreteObserver)
class MarketAnalyzer implements StockObserver {
    private List<StockData> priceHistory;
    private int maxHistorySize;

    public MarketAnalyzer(int maxHistorySize) {
        this.priceHistory = new ArrayList<>();
        this.maxHistorySize = maxHistorySize;
    }

    @Override
    public void onStockUpdate(StockData stockData) {
        priceHistory.add(stockData);

        // 최대 크기 유지
        if (priceHistory.size() > maxHistorySize) {
            priceHistory.remove(0);
        }

        analyzeMarketTrend(stockData);
    }

    @Override
    public String getObserverName() {
        return "Market Analyzer";
    }

    private void analyzeMarketTrend(StockData stockData) {
        if (Math.abs(stockData.getChangePercent()) > 5.0) {
            String trend = stockData.isIncrease() ? "강세" : "약세";
            System.out.println(String.format("[분석] %s 시장 %s 신호 감지 (변동률: %.2f%%)",
                stockData.getSymbol(), trend, stockData.getChangePercent()));
        }
    }

    public void generateReport() {
        System.out.println("\n=== 시장 분석 리포트 ===");
        if (priceHistory.isEmpty()) {
            System.out.println("분석할 데이터가 없습니다.");
            return;
        }

        Map<String, List<StockData>> symbolData = new HashMap<>();
        for (StockData data : priceHistory) {
            symbolData.computeIfAbsent(data.getSymbol(), k -> new ArrayList<>()).add(data);
        }

        for (Map.Entry<String, List<StockData>> entry : symbolData.entrySet()) {
            String symbol = entry.getKey();
            List<StockData> data = entry.getValue();

            double avgChange = data.stream()
                .mapToDouble(StockData::getChangePercent)
                .average()
                .orElse(0.0);

            System.out.println(String.format("%s: 평균 변동률 %.2f%% (%d회 거래)",
                symbol, avgChange, data.size()));
        }
    }
}

// 주식 거래 시스템 데모
public class StockTradingSystemDemo {
    public static void main(String[] args) throws InterruptedException {
        // 1. 주식 가격 서비스 생성
        StockPriceService priceService = new StockPriceService();

        // 2. 옵저버들 생성
        TradingDashboard mainDashboard = new TradingDashboard("메인 대시보드");
        TradingDashboard mobileDashboard = new TradingDashboard("모바일 앱");
        AlertService alertService = new AlertService();
        PortfolioManager johnPortfolio = new PortfolioManager("John");
        PortfolioManager sarahPortfolio = new PortfolioManager("Sarah");
        MarketAnalyzer analyzer = new MarketAnalyzer(10);

        // 3. 옵저버 등록
        priceService.registerObserver(mainDashboard);
        priceService.registerObserver(mobileDashboard);
        priceService.registerObserver(alertService);
        priceService.registerObserver(johnPortfolio);
        priceService.registerObserver(sarahPortfolio);
        priceService.registerObserver(analyzer);

        // 4. 알림 임계값 및 포트폴리오 설정
        alertService.setAlertThreshold("AAPL", 3.0);
        alertService.setAlertThreshold("GOOGL", 2.5);

        johnPortfolio.addHolding("AAPL", 100);
        johnPortfolio.addHolding("GOOGL", 50);
        sarahPortfolio.addHolding("AAPL", 200);
        sarahPortfolio.addHolding("MSFT", 150);

        System.out.println("\n" + "=".repeat(60));
        System.out.println("📊 실시간 주식 거래 시스템 시작");
        System.out.println("=".repeat(60));

        // 5. 주식 가격 변동 시뮬레이션
        priceService.updateStockPrice("AAPL", 150.00, 1000000);
        Thread.sleep(1000);

        priceService.updateStockPrice("GOOGL", 2800.00, 500000);
        Thread.sleep(1000);

        priceService.updateStockPrice("MSFT", 300.00, 800000);
        Thread.sleep(1000);

        // 큰 변동으로 알림 트리거
        priceService.updateStockPrice("AAPL", 155.50, 1500000); // +3.67% 상승
        Thread.sleep(1000);

        priceService.updateStockPrice("GOOGL", 2730.00, 750000); // -2.5% 하락
        Thread.sleep(1000);

        priceService.updateStockPrice("AAPL", 148.25, 1200000); // -4.66% 하락
        Thread.sleep(1000);

        // 6. 대시보드 및 포트폴리오 상태 확인
        mainDashboard.displayWatchList();
        johnPortfolio.displayPortfolio();
        sarahPortfolio.displayPortfolio();
        analyzer.generateReport();

        // 7. 일부 옵저버 제거 테스트
        System.out.println("\n" + "=".repeat(40));
        System.out.println("모바일 앱 연결 해제");
        System.out.println("=".repeat(40));
        priceService.removeObserver(mobileDashboard);

        priceService.updateStockPrice("AAPL", 152.00, 900000);

        System.out.println("\n📈 주식 거래 시스템 종료");
    }
}
```

**실행 결과 예시:**
```
✅ Trading Dashboard (메인 대시보드) 등록됨
✅ Trading Dashboard (모바일 앱) 등록됨
✅ Alert Service 등록됨
✅ Portfolio Manager (John) 등록됨
✅ Portfolio Manager (Sarah) 등록됨
✅ Market Analyzer 등록됨
🚨 AAPL 알림 임계값 설정: ±3.0%
🚨 GOOGL 알림 임계값 설정: ±2.5%
John가 AAPL 100주 보유
John가 GOOGL 50주 보유
Sarah가 AAPL 200주 보유
Sarah가 MSFT 150주 보유

============================================================
📊 실시간 주식 거래 시스템 시작
============================================================

📢 주식 가격 변동 알림: AAPL: $150.00 +0.00 (0.00%)
[메인 대시보드] 📈 AAPL: $150.00 +0.00 (0.00%) 14:30:15 (거래량: 1,000,000)
[모바일 앱] 📈 AAPL: $150.00 +0.00 (0.00%) 14:30:15 (거래량: 1,000,000)
[John] AAPL 포지션 변동: $15,000.00 (+$0.00)
[Sarah] AAPL 포지션 변동: $30,000.00 (+$0.00)

📢 주식 가격 변동 알림: AAPL: $155.50 +5.50 (3.67%)
[메인 대시보드] 📈 AAPL: $155.50 +5.50 (3.67%) 14:30:18 (거래량: 1,500,000)
🚨 [긴급알림] AAPL 급등! 3.67% 변동
[John] AAPL 포지션 변동: $15,550.00 (+$550.00)
[Sarah] AAPL 포지션 변동: $31,100.00 (+$1,100.00)
[분석] AAPL 시장 강세 신호 감지 (변동률: 3.67%)
```

## Java의 내장 Observer 패턴

Java에서는 `java.util.Observable`과 `java.util.Observer`를 제공했지만, Java 9부터 deprecated되었습니다. 대신 더 현대적인 방식들을 사용합니다:

```java
// 1. PropertyChangeListener 사용
import java.beans.PropertyChangeListener;
import java.beans.PropertyChangeSupport;

class ModernSubject {
    private PropertyChangeSupport support;
    private String property;

    public ModernSubject() {
        support = new PropertyChangeSupport(this);
    }

    public void addPropertyChangeListener(PropertyChangeListener listener) {
        support.addPropertyChangeListener(listener);
    }

    public void removePropertyChangeListener(PropertyChangeListener listener) {
        support.removePropertyChangeListener(listener);
    }

    public void setProperty(String newProperty) {
        String oldProperty = this.property;
        this.property = newProperty;
        support.firePropertyChange("property", oldProperty, newProperty);
    }
}

// 2. CompletableFuture와 함께 사용
import java.util.concurrent.CompletableFuture;

class AsyncObserver {
    public CompletableFuture<Void> handleUpdate(String data) {
        return CompletableFuture.runAsync(() -> {
            // 비동기 처리
            System.out.println("비동기 처리: " + data);
        });
    }
}
```

## 기본 예제 코드 (Java)

```java
import java.util.ArrayList;
import java.util.List;

// Observer Interface
interface Observer {
    void update(String message);
}

// Subject Interface
interface Subject {
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers();
}

// ConcreteSubject: 상태를 가지고 있으며, 상태 변경 시 옵저버에게 알림
class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String latestNews;

    public void setLatestNews(String news) {
        this.latestNews = news;
        notifyObservers(); // 상태 변경 시 모든 옵저버에게 알림
    }

    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(latestNews);
        }
    }
}

// ConcreteObserver: Subject의 상태 변화를 감지하고 업데이트함
class NewsSubscriber implements Observer {
    private String name;

    public NewsSubscriber(String name) {
        this.name = name;
    }

    @Override
    public void update(String news) {
        System.out.println(name + " received news: " + news);
    }
}

// 사용 예시
public class Client {
    public static void main(String[] args) {
        NewsAgency newsAgency = new NewsAgency();

        Observer subscriber1 = new NewsSubscriber("Subscriber 1");
        Observer subscriber2 = new NewsSubscriber("Subscriber 2");

        // 옵저버 등록
        newsAgency.registerObserver(subscriber1);
        newsAgency.registerObserver(subscriber2);

        // 상태 변경 및 알림
        newsAgency.setLatestNews("Breaking news: A new design pattern has been discovered!");

        System.out.println();

        // 옵저버 제거
        newsAgency.removeObserver(subscriber1);

        newsAgency.setLatestNews("Another news: The old pattern is now deprecated.");
    }
}
```

## 장점

- **높은 유연성과 재사용성**: 주제와 옵저버가 느슨하게 결합되어 있어 독립적으로 재사용하고 수정할 수 있습니다.
- **개방-폐쇄 원칙(OCP)**: 새로운 옵저버 유형을 추가할 때 기존 주제 코드를 수정할 필요가 없습니다.
- **런타임 관계 설정**: 애플리케이션 실행 중에 새로운 옵저버를 동적으로 추가하거나 제거할 수 있습니다.
- **브로드캐스팅**: 하나의 이벤트로 여러 객체에게 동시에 알림을 보낼 수 있습니다.

## 단점

- **예상치 못한 업데이트**: 옵저버가 많아지면 상태 변경 시 어떤 순서로 알림이 가는지 제어하기 어렵고, 변경 사항 전파가 복잡해질 수 있습니다.
- **메모리 누수**: 옵저버가 명시적으로 등록 해제되지 않으면, 주제 객체가 메모리에서 해제되지 않아 메모리 누수가 발생할 수 있습니다.
- **성능 문제**: 너무 많은 옵저버가 등록되어 있거나 업데이트 로직이 복잡한 경우, 알림을 보내는 과정에서 성능 저하가 발생할 수 있습니다.
- **순환 의존성**: 옵저버와 주제 간에 복잡한 상호작용이 있을 때 무한 루프가 발생할 수 있습니다.

## 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Mediator** | Observer는 직접 통신, Mediator는 중재자를 통한 통신 |
| **Command** | Observer의 알림을 Command 객체로 전달 가능 |
| **Strategy** | Observer의 update 로직을 Strategy로 교체 가능 |

### Observer vs Pub-Sub (발행-구독)

```java
// Observer: Subject가 Observer를 직접 알고 있음
class Subject {
    List<Observer> observers;  // 직접 참조
    void notify() {
        for (Observer o : observers) o.update();
    }
}

// Pub-Sub: Message Broker를 통한 간접 통신
class Publisher {
    MessageBroker broker;
    void publish(String topic, Event e) {
        broker.publish(topic, e);  // 구독자를 모름
    }
}
```

| 비교 | Observer | Pub-Sub |
|------|----------|---------|
| 결합도 | Subject가 Observer 알음 | Publisher는 Subscriber 모름 |
| 중재자 | 없음 (직접 통신) | Message Broker 필요 |
| 동기/비동기 | 주로 동기 | 주로 비동기 |
| 사용 예 | Spring @EventListener | Kafka, RabbitMQ |