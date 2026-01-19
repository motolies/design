# Static 서비스 테스트 (순수 단위 테스트)

## 정의

Static 서비스 테스트는 **Spring 컨텍스트 없이** 순수하게 Java 코드만으로 테스트하는 방식입니다. 가장 빠르고 단순한 테스트 형태입니다.

## 🎯 한눈에 보기

| 항목 | 설명 |
|------|------|
| **핵심** | Spring 없이 순수 Java로 테스트 |
| **대상** | static 메서드, 유틸리티 클래스, POJO |
| **속도** | ⚡ 가장 빠름 (밀리초 단위) |
| **어노테이션** | `@Test`, `@ParameterizedTest` |

> **💡 언제 사용하나요?**
>
> ```java
> // 이런 클래스들에 적합!
> public class StringUtils {
>     public static boolean isEmpty(String str) { ... }
> }
>
> public class PriceCalculator {
>     public int calculate(int price, int quantity) { ... }
> }
> ```

## 기본 테스트 작성

### 테스트할 클래스

```java
public class StringUtils {

    public static boolean isEmpty(String str) {
        return str == null || str.trim().isEmpty();
    }

    public static String reverse(String str) {
        if (str == null) return null;
        return new StringBuilder(str).reverse().toString();
    }
}
```

### 테스트 코드

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import static org.assertj.core.api.Assertions.*;

class StringUtilsTest {

    @Test
    @DisplayName("null이면 isEmpty는 true를 반환한다")
    void isEmpty_NullInput_ReturnsTrue() {
        // When
        boolean result = StringUtils.isEmpty(null);

        // Then
        assertThat(result).isTrue();
    }

    @Test
    @DisplayName("빈 문자열이면 isEmpty는 true를 반환한다")
    void isEmpty_EmptyString_ReturnsTrue() {
        // When
        boolean result = StringUtils.isEmpty("");

        // Then
        assertThat(result).isTrue();
    }

    @Test
    @DisplayName("공백만 있으면 isEmpty는 true를 반환한다")
    void isEmpty_WhitespaceOnly_ReturnsTrue() {
        // When
        boolean result = StringUtils.isEmpty("   ");

        // Then
        assertThat(result).isTrue();
    }

    @Test
    @DisplayName("문자가 있으면 isEmpty는 false를 반환한다")
    void isEmpty_HasContent_ReturnsFalse() {
        // When
        boolean result = StringUtils.isEmpty("hello");

        // Then
        assertThat(result).isFalse();
    }
}
```

## AssertJ 활용법

### 기본 검증

```java
import static org.assertj.core.api.Assertions.*;

// 동등성 검증
assertThat(result).isEqualTo(expected);
assertThat(result).isNotEqualTo(other);

// Boolean 검증
assertThat(result).isTrue();
assertThat(result).isFalse();

// Null 검증
assertThat(result).isNull();
assertThat(result).isNotNull();
```

### 문자열 검증

```java
String text = "Hello World";

assertThat(text).startsWith("Hello");
assertThat(text).endsWith("World");
assertThat(text).contains("lo Wo");
assertThat(text).doesNotContain("Bye");
assertThat(text).hasSize(11);
assertThat(text).isEqualToIgnoringCase("hello world");
```

### 숫자 검증

```java
int number = 100;

assertThat(number).isPositive();
assertThat(number).isGreaterThan(50);
assertThat(number).isLessThanOrEqualTo(100);
assertThat(number).isBetween(0, 200);
```

### 컬렉션 검증

```java
List<String> names = List.of("Alice", "Bob", "Charlie");

assertThat(names).hasSize(3);
assertThat(names).contains("Alice", "Bob");
assertThat(names).containsExactly("Alice", "Bob", "Charlie");
assertThat(names).doesNotContain("David");
assertThat(names).isNotEmpty();
```

### 예외 검증

```java
@Test
@DisplayName("0으로 나누면 ArithmeticException이 발생한다")
void divide_ByZero_ThrowsException() {
    // Given
    Calculator calculator = new Calculator();

    // When & Then
    assertThatThrownBy(() -> calculator.divide(10, 0))
            .isInstanceOf(ArithmeticException.class)
            .hasMessageContaining("zero");
}

// 또는
@Test
void divide_ByZero_ThrowsException_Alternative() {
    Calculator calculator = new Calculator();

    assertThatExceptionOfType(ArithmeticException.class)
            .isThrownBy(() -> calculator.divide(10, 0))
            .withMessageContaining("zero");
}
```

## 파라미터 테스트

### @ValueSource - 단일 값 테스트

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

@ParameterizedTest
@DisplayName("양수는 isPositive가 true를 반환한다")
@ValueSource(ints = {1, 5, 10, 100, 1000})
void isPositive_PositiveNumbers_ReturnsTrue(int number) {
    // When
    boolean result = NumberUtils.isPositive(number);

    // Then
    assertThat(result).isTrue();
}

@ParameterizedTest
@DisplayName("빈 문자열이면 isEmpty가 true를 반환한다")
@ValueSource(strings = {"", "   ", "\t", "\n"})
void isEmpty_BlankStrings_ReturnsTrue(String input) {
    assertThat(StringUtils.isEmpty(input)).isTrue();
}
```

### @NullAndEmptySource - null과 빈 값 테스트

```java
@ParameterizedTest
@NullAndEmptySource
@DisplayName("null이거나 빈 문자열이면 isEmpty가 true를 반환한다")
void isEmpty_NullOrEmpty_ReturnsTrue(String input) {
    assertThat(StringUtils.isEmpty(input)).isTrue();
}
```

### @CsvSource - 여러 파라미터 테스트

```java
@ParameterizedTest
@DisplayName("두 수를 더하면 합계가 반환된다")
@CsvSource({
    "1, 2, 3",
    "0, 0, 0",
    "-1, 1, 0",
    "100, 200, 300"
})
void add_TwoNumbers_ReturnsSum(int a, int b, int expected) {
    // Given
    Calculator calculator = new Calculator();

    // When
    int result = calculator.add(a, b);

    // Then
    assertThat(result).isEqualTo(expected);
}
```

### @MethodSource - 복잡한 테스트 데이터

```java
@ParameterizedTest
@DisplayName("할인율에 따라 정확한 할인 금액이 계산된다")
@MethodSource("discountTestCases")
void applyDiscount_VariousRates_ReturnsCorrectPrice(
        int originalPrice, int discountPercent, int expectedPrice) {

    // Given
    DiscountCalculator calculator = new DiscountCalculator();

    // When
    int result = calculator.applyDiscount(originalPrice, discountPercent);

    // Then
    assertThat(result).isEqualTo(expectedPrice);
}

static Stream<Arguments> discountTestCases() {
    return Stream.of(
        Arguments.of(10000, 10, 9000),   // 10% 할인
        Arguments.of(10000, 50, 5000),   // 50% 할인
        Arguments.of(10000, 0, 10000),   // 할인 없음
        Arguments.of(10000, 100, 0)      // 100% 할인
    );
}
```

## Static 메서드 모킹

### mockStatic() 사용법

외부 라이브러리의 static 메서드를 모킹해야 할 때 사용합니다.

```java
import org.mockito.MockedStatic;
import static org.mockito.Mockito.*;

class OrderServiceTest {

    @Test
    @DisplayName("현재 시간 기준으로 주문 시간이 설정된다")
    void createOrder_SetsCurrentTime() {
        // Given
        LocalDateTime fixedTime = LocalDateTime.of(2024, 1, 15, 10, 30);

        try (MockedStatic<LocalDateTime> mockedStatic = mockStatic(LocalDateTime.class)) {
            mockedStatic.when(LocalDateTime::now).thenReturn(fixedTime);

            // When
            Order order = new Order();
            order.setOrderTime(LocalDateTime.now());

            // Then
            assertThat(order.getOrderTime()).isEqualTo(fixedTime);
        }
        // try 블록이 끝나면 자동으로 mock 해제
    }
}
```

### UUID 모킹 예제

```java
@Test
@DisplayName("주문 ID는 UUID로 생성된다")
void createOrder_GeneratesUUID() {
    // Given
    UUID fixedUUID = UUID.fromString("123e4567-e89b-12d3-a456-426614174000");

    try (MockedStatic<UUID> mockedStatic = mockStatic(UUID.class)) {
        mockedStatic.when(UUID::randomUUID).thenReturn(fixedUUID);

        // When
        OrderService service = new OrderService();
        String orderId = service.generateOrderId();

        // Then
        assertThat(orderId).isEqualTo("123e4567-e89b-12d3-a456-426614174000");
    }
}
```

## 실전 예제: 가격 계산기

### 테스트할 클래스

```java
public class PriceCalculator {

    private static final int TAX_RATE = 10;  // 10%

    public int calculateTotalPrice(int unitPrice, int quantity) {
        validateInput(unitPrice, quantity);
        int subtotal = unitPrice * quantity;
        return addTax(subtotal);
    }

    public static int calculateTax(int price) {
        return price * TAX_RATE / 100;
    }

    private int addTax(int price) {
        return price + calculateTax(price);
    }

    private void validateInput(int unitPrice, int quantity) {
        if (unitPrice < 0) {
            throw new IllegalArgumentException("단가는 0 이상이어야 합니다");
        }
        if (quantity < 1) {
            throw new IllegalArgumentException("수량은 1 이상이어야 합니다");
        }
    }
}
```

### 테스트 코드

```java
class PriceCalculatorTest {

    private PriceCalculator calculator;

    @BeforeEach
    void setUp() {
        calculator = new PriceCalculator();
    }

    @Test
    @DisplayName("단가와 수량으로 총 가격(세금 포함)을 계산한다")
    void calculateTotalPrice_WithTax_ReturnsCorrectTotal() {
        // Given
        int unitPrice = 1000;
        int quantity = 5;

        // When
        int total = calculator.calculateTotalPrice(unitPrice, quantity);

        // Then
        // 1000 * 5 = 5000 + 세금 500 = 5500
        assertThat(total).isEqualTo(5500);
    }

    @ParameterizedTest
    @DisplayName("다양한 가격에 대해 세금이 정확히 계산된다")
    @CsvSource({
        "1000, 100",
        "5000, 500",
        "10000, 1000",
        "0, 0"
    })
    void calculateTax_VariousPrices_ReturnsCorrectTax(int price, int expectedTax) {
        // When
        int tax = PriceCalculator.calculateTax(price);

        // Then
        assertThat(tax).isEqualTo(expectedTax);
    }

    @Test
    @DisplayName("단가가 음수면 예외가 발생한다")
    void calculateTotalPrice_NegativeUnitPrice_ThrowsException() {
        assertThatThrownBy(() -> calculator.calculateTotalPrice(-1000, 1))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessageContaining("단가");
    }

    @Test
    @DisplayName("수량이 0이면 예외가 발생한다")
    void calculateTotalPrice_ZeroQuantity_ThrowsException() {
        assertThatThrownBy(() -> calculator.calculateTotalPrice(1000, 0))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessageContaining("수량");
    }
}
```

## 테스트 코드 패턴

### Given-When-Then 패턴

```java
@Test
@DisplayName("회원 등급이 VIP면 10% 추가 할인이 적용된다")
void calculateDiscount_VipMember_GetsExtraDiscount() {
    // Given (준비)
    DiscountCalculator calculator = new DiscountCalculator();
    Member vipMember = new Member("VIP");
    int price = 10000;

    // When (실행)
    int discountedPrice = calculator.calculate(price, vipMember);

    // Then (검증)
    assertThat(discountedPrice).isEqualTo(9000);
}
```

### 테스트 이름 규칙

| 패턴 | 예시 |
|------|------|
| `메서드_조건_결과` | `add_TwoPositiveNumbers_ReturnsSum` |
| `should_결과_when_조건` | `should_ReturnSum_when_AddingTwoNumbers` |
| `한글 DisplayName` | `@DisplayName("두 수를 더하면 합계가 반환된다")` |

## 관련 문서

| 문서 | 설명 |
|------|------|
| [메인으로 돌아가기](../README.md) | TDD 가이드 메인 |
| [서비스 빈 테스트](../service/README.md) | Mockito로 의존성 모킹 |
