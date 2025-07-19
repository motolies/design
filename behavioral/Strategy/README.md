# 전략 패턴 (Strategy Pattern)

## 정의

전략 패턴은 알고리즘군을 정의하고 각각을 캡슐화하여 서로 바꿔 쓸 수 있도록 만드는 행동 디자인 패턴입니다. 전략을 사용하면 클라이언트로부터 알고리즘을 분리하여 독립적으로 변경할 수 있습니다.

## 사용 이유

- **알고리즘 교체**: 런타임에 여러 알고리즘(전략) 중 하나를 선택하여 사용해야 할 때 유용합니다.
- **코드의 유연성 및 확장성**: 새로운 전략을 추가해야 할 때, 기존 컨텍스트 코드를 수정하지 않고 새로운 전략 클래스만 추가하면 되므로 개방-폐쇄 원칙(OCP)을 준수할 수 있습니다.
- **복잡한 조건문 제거**: `if-else` 또는 `switch` 문으로 다양한 전략을 분기 처리하는 코드를 피하고, 각 전략을 별도의 클래스로 캡슐화하여 코드의 가독성과 유지보수성을 높일 수 있습니다.

## 예제 코드 (Java)

```java
// Strategy Interface: 모든 전략들이 구현해야 할 공통 인터페이스
interface Strategy {
    int doOperation(int num1, int num2);
}

// ConcreteStrategy A: 덧셈 전략
class OperationAdd implements Strategy {
    @Override
    public int doOperation(int num1, int num2) {
        return num1 + num2;
    }
}

// ConcreteStrategy B: 뺄셈 전략
class OperationSubtract implements Strategy {
    @Override
    public int doOperation(int num1, int num2) {
        return num1 - num2;
    }
}

// ConcreteStrategy C: 곱셈 전략
class OperationMultiply implements Strategy {
    @Override
    public int doOperation(int num1, int num2) {
        return num1 * num2;
    }
}

// Context: 전략을 사용하는 클래스
class Context {
    private Strategy strategy;

    public Context(Strategy strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }

    public int executeStrategy(int num1, int num2) {
        return strategy.doOperation(num1, num2);
    }
}

// 사용 예시
public class Client {
    public static void main(String[] args) {
        Context context = new Context(new OperationAdd());
        System.out.println("10 + 5 = " + context.executeStrategy(10, 5));

        context.setStrategy(new OperationSubtract());
        System.out.println("10 - 5 = " + context.executeStrategy(10, 5));

        context.setStrategy(new OperationMultiply());
        System.out.println("10 * 5 = " + context.executeStrategy(10, 5));
    }
}
```

## 장점

- **유연한 알고리즘 전환**: 런타임에 동적으로 알고리즘을 변경할 수 있습니다.
- **확장성**: 새로운 전략을 쉽게 추가할 수 있습니다 (개방-폐쇄 원칙).
- **코드 단순화**: 복잡한 조건 분기문을 제거하고 코드를 더 깔끔하게 만들 수 있습니다.

## 단점

- **클래스 수 증가**: 각 전략을 별도의 클래스로 만들어야 하므로, 간단한 로직에 적용할 경우 클래스의 수가 늘어나는 단점이 있습니다.
- **클라이언트의 책임**: 클라이언트가 어떤 전략을 사용해야 하는지 알아야 하므로, 클라이언트와 특정 전략 간에 약간의 결합이 생길 수 있습니다.