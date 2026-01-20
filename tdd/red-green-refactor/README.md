# Red-Green-Refactor 사이클

## 정의

Red-Green-Refactor는 TDD의 핵심 사이클로, **실패하는 테스트 작성(Red)** → **테스트 통과(Green)** → **코드 개선(Refactor)** 단계를 반복합니다.

## 🎯 한눈에 보기

| 단계 | 설명 | 목표 |
|------|------|------|
| **🔴 Red** | 실패하는 테스트 먼저 작성 | 요구사항 정의 |
| **🟢 Green** | 테스트를 통과하는 최소 코드 작성 | 기능 구현 |
| **🔵 Refactor** | 코드 개선 (테스트는 계속 통과) | 품질 향상 |

## 사이클 다이어그램

```mermaid
flowchart LR
    RED["🔴 Red<br/>테스트 작성<br/>(실패)"]
    GREEN["🟢 Green<br/>최소 코드<br/>(통과)"]
    REFACTOR["🔵 Refactor<br/>코드 개선<br/>(통과 유지)"]

    RED --> GREEN --> REFACTOR --> RED

    style RED fill:#f44336,color:#fff
    style GREEN fill:#4caf50,color:#fff
    style REFACTOR fill:#2196f3,color:#fff
```

## 각 단계 상세

### 🔴 Red - 실패하는 테스트 작성

> "테스트가 실패하는 것을 확인해야 한다"

```java
@Test
@DisplayName("두 수를 더하면 합계가 반환된다")
void add_TwoNumbers_ReturnsSum() {
    // Given
    Calculator calculator = new Calculator();

    // When
    int result = calculator.add(2, 3);

    // Then
    assertThat(result).isEqualTo(5);
}
```

이 시점에서 `Calculator` 클래스가 없으므로 **컴파일 에러**가 발생합니다.
이것이 바로 **Red** 상태입니다!

**왜 실패를 먼저 확인할까?**

| 이유 | 설명 |
|------|------|
| 테스트 검증 | 테스트가 실제로 동작하는지 확인 |
| 요구사항 명확화 | 무엇을 구현할지 명확히 정의 |
| 과잉 구현 방지 | 필요한 것만 구현하도록 유도 |

### 🟢 Green - 테스트 통과하는 최소 코드

> "테스트를 통과하는 가장 간단한 코드를 작성한다"

```java
public class Calculator {

    public int add(int a, int b) {
        return a + b;  // 최소한의 구현
    }
}
```

**Green 단계의 핵심 원칙:**

```mermaid
flowchart TD
    A["테스트 통과가 목표"] --> B["아름다운 코드 X"]
    A --> C["완벽한 설계 X"]
    A --> D["최소한의 코드 O"]

    style D fill:#4caf50,color:#fff
    style B fill:#f44336,color:#fff
    style C fill:#f44336,color:#fff
```

| 원칙 | 설명 |
|------|------|
| **KISS** | 가장 단순하게 구현 |
| **최소 코드** | 테스트 통과에 필요한 것만 |
| **빠른 피드백** | 빨리 Green 상태로 |

### 🔵 Refactor - 코드 개선

> "동작은 유지하면서 코드 품질을 개선한다"

리팩토링 전:
```java
public class Calculator {

    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }

    public int multiply(int a, int b) {
        return a * b;
    }
}
```

리팩토링 후 (예: 검증 로직 추가):
```java
public class Calculator {

    public int add(int a, int b) {
        validateInput(a, b);
        return a + b;
    }

    public int subtract(int a, int b) {
        validateInput(a, b);
        return a - b;
    }

    public int multiply(int a, int b) {
        validateInput(a, b);
        return a * b;
    }

    private void validateInput(int a, int b) {
        // 공통 검증 로직
    }
}
```

**Refactor 단계의 핵심:**

| 원칙 | 설명 |
|------|------|
| **테스트 통과 유지** | 리팩토링 후에도 모든 테스트 통과 |
| **작은 단위** | 한 번에 조금씩 변경 |
| **자주 실행** | 변경마다 테스트 실행 |

## 실전 예제: 할인 계산기

### 요구사항
- 10% 할인을 적용하는 기능
- 원가에서 할인 금액을 뺀 최종 가격 반환

### 🔴 Red - 테스트 먼저

```java
class DiscountCalculatorTest {

    @Test
    @DisplayName("10% 할인을 적용하면 할인된 가격이 반환된다")
    void applyDiscount_10Percent_ReturnsDiscountedPrice() {
        // Given
        DiscountCalculator calculator = new DiscountCalculator();
        int originalPrice = 10000;

        // When
        int discountedPrice = calculator.applyDiscount(originalPrice, 10);

        // Then
        assertThat(discountedPrice).isEqualTo(9000);
    }
}
```

**실행 결과**: ❌ 컴파일 에러 (DiscountCalculator 없음)

### 🟢 Green - 최소 구현

```java
public class DiscountCalculator {

    public int applyDiscount(int price, int discountPercent) {
        int discountAmount = price * discountPercent / 100;
        return price - discountAmount;
    }
}
```

**실행 결과**: ✅ 테스트 통과

### 🔴 Red - 추가 테스트

```java
@Test
@DisplayName("할인율이 0이면 원래 가격이 반환된다")
void applyDiscount_ZeroPercent_ReturnsOriginalPrice() {
    // Given
    DiscountCalculator calculator = new DiscountCalculator();

    // When
    int result = calculator.applyDiscount(10000, 0);

    // Then
    assertThat(result).isEqualTo(10000);
}

@Test
@DisplayName("할인율이 100이면 0원이 반환된다")
void applyDiscount_100Percent_ReturnsZero() {
    // Given
    DiscountCalculator calculator = new DiscountCalculator();

    // When
    int result = calculator.applyDiscount(10000, 100);

    // Then
    assertThat(result).isEqualTo(0);
}
```

**실행 결과**: ✅ 모든 테스트 통과

### 🔵 Refactor - 코드 개선

```java
public class DiscountCalculator {

    public int applyDiscount(int price, int discountPercent) {
        validatePrice(price);
        validateDiscountPercent(discountPercent);

        int discountAmount = calculateDiscountAmount(price, discountPercent);
        return price - discountAmount;
    }

    private void validatePrice(int price) {
        if (price < 0) {
            throw new IllegalArgumentException("가격은 0 이상이어야 합니다");
        }
    }

    private void validateDiscountPercent(int percent) {
        if (percent < 0 || percent > 100) {
            throw new IllegalArgumentException("할인율은 0~100 사이여야 합니다");
        }
    }

    private int calculateDiscountAmount(int price, int discountPercent) {
        return price * discountPercent / 100;
    }
}
```

### 🔴 Red - 예외 케이스 테스트

```java
@Test
@DisplayName("할인율이 음수면 예외가 발생한다")
void applyDiscount_NegativePercent_ThrowsException() {
    // Given
    DiscountCalculator calculator = new DiscountCalculator();

    // When & Then
    assertThatThrownBy(() -> calculator.applyDiscount(10000, -10))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessageContaining("0~100");
}
```

## 사이클 반복 시각화

```mermaid
sequenceDiagram
    participant Dev as 개발자
    participant Test as 테스트
    participant Code as 프로덕션 코드

    rect rgb(255, 200, 200)
        Note over Dev,Code: 🔴 Red Phase
        Dev->>Test: 테스트 작성
        Test-->>Dev: ❌ 실패
    end

    rect rgb(200, 255, 200)
        Note over Dev,Code: 🟢 Green Phase
        Dev->>Code: 최소 구현
        Test-->>Dev: ✅ 통과
    end

    rect rgb(200, 200, 255)
        Note over Dev,Code: 🔵 Refactor Phase
        Dev->>Code: 코드 개선
        Test-->>Dev: ✅ 여전히 통과
    end

    Note over Dev,Code: 반복...
```

## TDD 마인드셋

### Do's ✅

| 항목 | 설명 |
|------|------|
| 작은 단계 | 한 번에 하나의 테스트만 |
| 빠른 피드백 | 자주 테스트 실행 |
| 단순한 시작 | 가장 쉬운 케이스부터 |
| 실패 확인 | Red 상태 꼭 확인 |

### Don'ts ❌

| 항목 | 설명 |
|------|------|
| 한 번에 많이 | 여러 기능 동시 구현 |
| 완벽 추구 | Green에서 완벽한 코드 |
| 테스트 건너뛰기 | Red 없이 바로 구현 |
| 리팩토링 미루기 | Refactor 건너뛰기 |

## 💡 팁

### 작은 단계로 진행하기

TDD의 핵심은 **아주 작은 단계**로 진행하는 것입니다.

```java
// 1단계: 가장 단순한 케이스
@Test void add_ZeroAndZero_ReturnsZero() {
    assertThat(calculator.add(0, 0)).isEqualTo(0);
}

// 2단계: 양수 하나
@Test void add_ZeroAndPositive_ReturnsPositive() {
    assertThat(calculator.add(0, 5)).isEqualTo(5);
}

// 3단계: 양수 둘
@Test void add_TwoPositives_ReturnsSum() {
    assertThat(calculator.add(3, 5)).isEqualTo(8);
}

// 4단계: 음수 케이스
@Test void add_NegativeNumbers_ReturnsSum() {
    assertThat(calculator.add(-3, -5)).isEqualTo(-8);
}
```

> **"큰 걸음보다 작은 발걸음이 더 빠르다"**

### 테스트 먼저 커밋하기

```bash
# Red 단계에서 커밋 (테스트만)
git add src/test/
git commit -m "test: add 기능 테스트 추가 (Red)"

# Green 단계에서 커밋 (구현)
git add src/main/
git commit -m "feat: add 기능 구현 (Green)"

# Refactor 단계에서 커밋
git commit -am "refactor: add 메서드 리팩토링"
```

이렇게 하면 **테스트가 먼저 존재한다는 것을 기록**으로 남길 수 있습니다.

### 삼각측량 (Triangulation)

하나의 테스트만으로 구현이 완성되지 않을 때, **여러 테스트를 추가하여 일반화**합니다.

```java
// 첫 번째 테스트 → 하드코딩으로 통과 가능
@Test void add_1And1_Returns2() {
    assertThat(calculator.add(1, 1)).isEqualTo(2);
}

// 이 구현으로 통과! (하지만 하드코딩)
public int add(int a, int b) {
    return 2;  // 하드코딩
}

// 두 번째 테스트 추가 → 삼각측량
@Test void add_2And3_Returns5() {
    assertThat(calculator.add(2, 3)).isEqualTo(5);
}

// 이제 일반화 필요
public int add(int a, int b) {
    return a + b;  // 일반화된 구현
}
```

### ZOMBIES 원칙

테스트 케이스 순서를 정할 때 참고하는 원칙입니다.

| 문자 | 의미 | 예시 |
|------|------|------|
| **Z** | Zero | 빈 컬렉션, 0, null |
| **O** | One | 단일 요소, 1개 |
| **M** | Many | 여러 요소, 다수 |
| **B** | Boundary | 경계값, 최대/최소 |
| **I** | Interface | 인터페이스 정의 |
| **E** | Exception | 예외 상황 |
| **S** | Simple | 단순한 시나리오부터 |

```java
// Z: Zero
@Test void isEmpty_EmptyList_ReturnsTrue() { ... }

// O: One
@Test void size_OneElement_ReturnsOne() { ... }

// M: Many
@Test void size_MultipleElements_ReturnsCount() { ... }

// B: Boundary
@Test void add_MaxInteger_ThrowsOverflow() { ... }

// E: Exception
@Test void get_InvalidIndex_ThrowsException() { ... }
```

## 자주 하는 실수

### 1. 테스트 없이 구현 시작

```java
// ❌ 코드 먼저 작성
public int add(int a, int b) {
    return a + b;
}
// 나중에 테스트 작성... (TDD 아님!)

// ✅ 테스트 먼저!
@Test void add() {
    assertThat(new Calculator().add(1, 2)).isEqualTo(3);
}
// → 컴파일 에러 → Calculator 클래스 생성 → add 메서드 구현
```

### 2. 너무 큰 단계로 진행

```java
// ❌ 한 번에 복잡한 기능 전체 테스트
@Test void processOrder_CompleteFlow() {
    // 재고 확인, 결제, 배송, 알림까지 전부...
}

// ✅ 작은 단위로 분리
@Test void validateStock_SufficientStock_ReturnsTrue() { ... }
@Test void processPayment_ValidCard_ReturnsSuccess() { ... }
@Test void createShipment_ValidOrder_ReturnsTrackingNumber() { ... }
```

### 3. Red 상태 확인 건너뛰기

```java
// ❌ 테스트 작성 후 바로 구현
// (테스트가 실패하는지 확인 안 함)

// ✅ 반드시 실패 확인
// 1. 테스트 작성
// 2. 테스트 실행 → 실패 확인 (Red)
// 3. 구현
// 4. 테스트 실행 → 성공 확인 (Green)
```

**왜 Red 확인이 중요한가?**
- 테스트가 실제로 실행되는지 확인
- 항상 성공하는 잘못된 테스트 방지
- 테스트가 올바른 것을 검증하는지 확인

## 관련 문서

| 문서 | 설명 |
|------|------|
| [메인으로 돌아가기](../README.md) | TDD 가이드 메인 |
| [Static 서비스 테스트](../static-service/README.md) | 순수 단위 테스트 시작하기 |
