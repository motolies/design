# 퍼사드 패턴 (Facade Pattern)

## 정의

퍼사드 패턴은 복잡한 서브시스템에 대해 간단하고 통일된 인터페이스를 제공하는 구조 디자인 패턴입니다. '퍼사드(Facade)'는 건물의 정면을 의미하며, 복잡한 내부 구조를 가리고 단순한 외부 모습만 보여주는 것처럼, 이 패턴은 복잡한 내부 로직을 숨기고 클라이언트에게 필요한 기능만 노출하는 역할을 합니다.

## 사용 이유

- **복잡성 감소**: 여러 개의 복잡한 클래스들로 구성된 서브시스템을 직접 사용하는 대신, 단순한 퍼사드 클래스를 통해 사용함으로써 클라이언트 코드의 복잡성을 크게 줄일 수 있습니다.
- **결합도 감소**: 클라이언트는 퍼사드 인터페이스에만 의존하게 되므로, 서브시스템의 내부 클래스들이 변경되더라도 클라이언트 코드는 영향을 받지 않습니다. 이는 서브시스템과 클라이언트 간의 결합도를 낮추는 효과를 가져옵니다.
- **계층화된 구조**: 시스템을 여러 계층으로 구성할 때, 각 계층의 진입점으로 퍼사드를 사용하여 계층 간의 통신을 단순화할 수 있습니다.

## 예제 코드 (Java)

```java
// 복잡한 서브시스템의 일부인 클래스들
class SubsystemA {
    public void operationA() {
        System.out.println("SubsystemA: Ready!");
    }
}

class SubsystemB {
    public void operationB() {
        System.out.println("SubsystemB: Go!");
    }
}

class SubsystemC {
    public void operationC() {
        System.out.println("SubsystemC: Fire!");
    }
}

// Facade 클래스: 서브시스템에 대한 단순한 인터페이스 제공
class Facade {
    private SubsystemA subsystemA;
    private SubsystemB subsystemB;
    private SubsystemC subsystemC;

    public Facade() {
        this.subsystemA = new SubsystemA();
        this.subsystemB = new SubsystemB();
        this.subsystemC = new SubsystemC();
    }

    // 클라이언트를 위한 단순화된 메서드
    public void performOperation() {
        System.out.println("Facade: Initiating operations.");
        subsystemA.operationA();
        subsystemB.operationB();
        subsystemC.operationC();
        System.out.println("Facade: Operations completed.");
    }
}

// 사용 예시
public class Client {
    public static void main(String[] args) {
        Facade facade = new Facade();
        // 클라이언트는 복잡한 내부 구조를 몰라도 퍼사드를 통해 작업을 수행
        facade.performOperation();
    }
}
```

## 장점

- **단순성**: 클라이언트가 복잡한 서브시스템을 쉽게 사용할 수 있도록 단순한 인터페이스를 제공합니다.
- **느슨한 결합**: 클라이언트와 서브시스템 간의 결합을 분리하여 시스템의 유연성과 유지보수성을 높입니다.
- **코드 가독성 향상**: 서브시스템의 사용법이 명확해져 코드의 가독성이 좋아집니다.

## 단점

- **전지전능한 객체 가능성**: 퍼사드 클래스가 너무 많은 책임을 떠맡아 모든 서브시스템에 강하게 결합된 '전지전능한 객체(God Object)'가 될 위험이 있습니다.
- **불필요한 간접 계층**: 서브시스템이 간단한 경우, 퍼사드를 도입하는 것이 오히려 불필요한 복잡성을 추가할 수 있습니다.