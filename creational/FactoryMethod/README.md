# 팩토리 메서드 패턴 (Factory Method Pattern)

## 정의

팩토리 메서드 패턴은 객체를 생성하기 위한 인터페이스를 정의하지만, 어떤 클래스의 인스턴스를 생성할지에 대한 결정은 서브클래스에서 내리도록 하는 생성 디자인 패턴입니다. 즉, 객체 생성 과정을 서브클래스에 위임하는 것입니다.

## 사용 이유

- **결합도 감소**: 객체를 생성하는 코드와 사용하는 코드를 분리하여 결합도를 낮출 수 있습니다. 클라이언트 코드는 구체적인 클래스 이름 대신 인터페이스에만 의존하게 됩니다.
- **유연성 및 확장성 증가**: 새로운 종류의 객체를 추가해야 할 때, 기존 팩토리 코드를 수정하는 대신 새로운 서브클래스를 추가하여 팩토리 메서드를 오버라이드하면 되므로 시스템의 확장성이 향상됩니다.

## 예제 코드 (Java)

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

## 단점

- **코드 복잡성 증가**: 패턴을 구현하기 위해 많은 새로운 클래스(인터페이스, 생성자, 제품 클래스 등)가 필요하므로 코드의 전체적인 복잡성이 증가할 수 있습니다.