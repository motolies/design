# 어댑터 패턴 (Adapter Pattern)

## 정의

어댑터 패턴은 한 클래스의 인터페이스를 클라이언트가 기대하는 다른 인터페이스로 변환하는 구조 디자인 패턴입니다. 어댑터를 사용하면 호환되지 않는 인터페이스를 가진 클래스들이 함께 작동할 수 있습니다.

## 사용 이유

- **호환성 문제 해결**: 서로 다른 인터페이스를 가진 두 클래스를 직접 수정하지 않고도 함께 사용하고 싶을 때 유용합니다.
- **기존 코드 재사용**: 레거시 시스템이나 외부 라이브러리의 클래스를 새로운 시스템의 인터페이스에 맞춰 재사용할 수 있습니다.
- **유연성 증가**: 클라이언트 코드는 대상 인터페이스에만 의존하므로, 실제 구현 클래스가 변경되더라도 클라이언트 코드는 영향을 받지 않습니다.

## 예제 코드 (Java)

```java
// Target Interface: 클라이언트가 사용하려는 인터페이스
interface Target {
    void request();
}

// Adaptee: 재사용하고 싶은 기존 클래스 (호환되지 않는 인터페이스)
class Adaptee {
    public void specificRequest() {
        System.out.println("Called specificRequest() in Adaptee");
    }
}

// Adapter: Adaptee의 인터페이스를 Target 인터페이스에 맞게 변환
class Adapter implements Target {
    private Adaptee adaptee;

    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void request() {
        // 클라이언트의 요청(request)을 Adaptee가 이해할 수 있는 방식(specificRequest)으로 변환
        adaptee.specificRequest();
    }
}

// 사용 예시
public class Client {
    public static void main(String[] args) {
        Adaptee adaptee = new Adaptee();
        Target adapter = new Adapter(adaptee);

        // 클라이언트는 Target 인터페이스를 통해 Adaptee의 기능을 사용
        adapter.request();
    }
}
```

## 장점

- **재사용성 향상**: 기존 클래스의 코드를 변경하지 않고 새로운 시스템에 통합할 수 있습니다.
- **결합도 감소**: 클라이언트와 구현 클래스 사이의 결합도를 낮춥니다.
- **단일 책임 원칙(SRP)**: 인터페이스 변환 로직을 어댑터 클래스에 캡슐화하여 코드의 응집도를 높입니다.

## 단점

- **코드 복잡성 증가**: 새로운 어댑터 클래스를 추가해야 하므로 전체적인 코드의 양과 복잡성이 늘어날 수 있습니다.