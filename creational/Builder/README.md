# 빌더 패턴 (Builder Pattern)

## 정의

빌더 패턴은 복잡한 객체를 생성하는 과정을 단계별로 분리하여, 동일한 생성 절차에서 서로 다른 표현 결과를 만들 수 있도록 하는 생성 디자인 패턴입니다.

## 사용 이유

- **복잡한 객체 생성**: 생성자에 많은 매개변수가 필요하거나, 객체 생성 과정이 여러 단계로 구성되어 복잡할 때 유용합니다.
- **가독성 향상**: 메서드 체이닝(Method Chaining)을 통해 각 필드에 어떤 값이 설정되는지 명확하게 알 수 있어 코드의 가독성이 높아집니다.
- **일관성 유지**: 객체 생성 과정을 디렉터(Director) 클래스로 중앙에서 관리하면, 복잡한 객체를 일관성 있게 생성할 수 있습니다.
- **불변 객체 생성**: 생성 과정이 완료된 후에는 변경할 수 없는 불변(Immutable) 객체를 만드는 데 용이합니다.

## 예제 코드 (Java)

```java
// Product 클래스
class Product {
    private String partA;
    private String partB;
    private String partC;

    // 빌더를 통해서만 인스턴스화 가능하도록 private 생성자
    private Product(Builder builder) {
        this.partA = builder.partA;
        this.partB = builder.partB;
        this.partC = builder.partC;
    }

    @Override
    public String toString() {
        return "Product [partA=" + partA + ", partB=" + partB + ", partC=" + partC + "]";
    }

    // Static nested Builder 클래스
    public static class Builder {
        // 필수 매개변수
        private final String partA;

        // 선택적 매개변수
        private String partB = "DefaultB";
        private String partC = "DefaultC";

        public Builder(String partA) {
            this.partA = partA;
        }

        public Builder setPartB(String partB) {
            this.partB = partB;
            return this; // 메서드 체이닝을 위해 this 반환
        }

        public Builder setPartC(String partC) {
            this.partC = partC;
            return this;
        }

        public Product build() {
            return new Product(this);
        }
    }
}

// 사용 예시
public class Client {
    public static void main(String[] args) {
        // 필수 값만 설정
        Product product1 = new Product.Builder("PartA_Value").build();
        System.out.println(product1);

        // 모든 값 설정
        Product product2 = new Product.Builder("PartA_Value")
                                      .setPartB("PartB_Value")
                                      .setPartC("PartC_Value")
                                      .build();
        System.out.println(product2);
    }
}
```

## 장점

- **코드 가독성**: 필요한 값만 설정하고, 메서드 이름을 통해 어떤 값을 설정하는지 명확히 알 수 있습니다.
- **객체 생성 제어**: 객체를 생성하는 단계를 세밀하게 제어할 수 있습니다.
- **불변성**: `build()` 메서드가 호출되기 전까지는 객체가 생성되지 않으며, 한 번 생성된 후에는 상태를 변경하지 않는 불변 객체를 쉽게 만들 수 있습니다.

## 단점

- **코드 복잡성 증가**: 빌더 클래스를 추가로 작성해야 하므로 코드의 양이 늘어납니다. 간단한 객체를 생성할 때는 오버헤드가 될 수 있습니다.
- **내부 빌더의 불변성**: 빌더 자체는 불변이 아니므로, `build()` 메서드 호출 전에 빌더의 상태가 변경될 수 있습니다.