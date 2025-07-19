# 데코레이터 패턴 (Decorator Pattern)

## 정의

데코레이터 패턴은 객체의 기존 코드를 수정하지 않고 동적으로 새로운 책임이나 기능을 추가할 수 있게 해주는 구조 디자인 패턴입니다. 객체를 장식(Decorate)하는 것처럼 기능을 추가한다고 해서 데코레이터 패턴이라고 불립니다.

## 사용 이유

- **동적 기능 추가**: 런타임에 객체에 유연하게 기능을 추가하거나 제거하고 싶을 때 사용됩니다. 상속을 사용하면 컴파일 타임에 기능이 고정되지만, 데코레이터 패턴은 동적으로 기능을 조합할 수 있습니다.
- **클래스 폭발 방지**: 기능의 모든 조합에 대해 서브클래스를 만드는 대신, 기본 클래스와 여러 데코레이터 클래스를 조합하여 필요한 기능을 구성할 수 있습니다. 이는 클래스의 수를 기하급수적으로 늘리는 것을 방지합니다.
- **단일 책임 원칙(SRP) 준수**: 각 기능(장식)을 별도의 클래스로 분리하여 관리하므로, 각 클래스는 하나의 책임만 갖게 됩니다.

## 예제 코드 (Java)

```java
// Component Interface: 모든 컴포넌트와 데코레이터가 구현할 공통 인터페이스
interface Component {
    String operation();
}

// ConcreteComponent: 기본 기능을 가진 핵심 객체
class ConcreteComponent implements Component {
    @Override
    public String operation() {
        return "ConcreteComponent";
    }
}

// Decorator: 추상 데코레이터 클래스. Component 인터페이스를 구현하고 Component 객체를 가짐
abstract class Decorator implements Component {
    protected Component component;

    public Decorator(Component component) {
        this.component = component;
    }

    @Override
    public String operation() {
        return component.operation(); // 감싸고 있는 객체에 작업 위임
    }
}

// ConcreteDecoratorA: 새로운 기능을 추가하는 구체적인 데코레이터
class ConcreteDecoratorA extends Decorator {
    public ConcreteDecoratorA(Component component) {
        super(component);
    }

    @Override
    public String operation() {
        return "ConcreteDecoratorA(" + super.operation() + ")"; // 원래 기능에 새로운 기능 추가
    }
}

// ConcreteDecoratorB: 또 다른 새로운 기능을 추가하는 구체적인 데코레이터
class ConcreteDecoratorB extends Decorator {
    public ConcreteDecoratorB(Component component) {
        super(component);
    }

    @Override
    public String operation() {
        return "ConcreteDecoratorB(" + super.operation() + ")";
    }
}

// 사용 예시
public class Client {
    public static void main(String[] args) {
        // 기본 컴포넌트
        Component simple = new ConcreteComponent();
        System.out.println("기본 객체: " + simple.operation());

        // 데코레이터 A로 장식
        Component decoratorA = new ConcreteDecoratorA(simple);
        System.out.println("데코레이터 A 추가: " + decoratorA.operation());

        // 데코레이터 B로 장식
        Component decoratorB = new ConcreteDecoratorB(decoratorA);
        System.out.println("데코레이터 B 추가: " + decoratorB.operation());
    }
}
```

## 장점

- **유연성**: 컴파일 타임이 아닌 런타임에 객체에 새로운 기능을 추가하거나 제거할 수 있습니다.
- **확장성**: 기존 코드를 변경하지 않고 새로운 데코레이터 클래스를 추가하여 시스템을 쉽게 확장할 수 있습니다 (개방-폐쇄 원칙).
- **코드 중복 감소**: 상속을 통한 기능 조합으로 발생할 수 있는 많은 수의 서브클래스를 피할 수 있습니다.

## 단점

- **복잡성**: 데코레이터 체인이 길어지면 코드를 이해하고 디버깅하기 어려워질 수 있습니다.
- **작은 객체들의 증가**: 많은 작은 데코레이터 객체들이 생성되어 시스템을 복잡하게 만들 수 있습니다.