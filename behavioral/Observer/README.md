# 옵저버 패턴 (Observer Pattern)

## 정의

옵저버 패턴은 한 객체(주제, Subject)의 상태가 변경되면 그 객체에 의존하는 다른 객체(옵저버, Observer)들에게 자동으로 알림을 보내고 업데이트하는 행동 디자인 패턴입니다. 이는 일대다(one-to-many)의 의존 관계를 정의합니다.

## 사용 이유

- **느슨한 결합**: 주제(Subject)와 옵저버(Observer)는 서로의 구체적인 구현을 알 필요 없이, 추상적인 인터페이스를 통해 상호작용합니다. 이로 인해 두 컴포넌트 간의 결합도가 낮아집니다.
- **동적 관계**: 런타임에 동적으로 옵저버를 추가하거나 제거할 수 있어 유연한 관계 설정이 가능합니다.
- **이벤트 기반 시스템**: GUI 이벤트 처리, 뉴스 구독 시스템, 주식 가격 변동 알림 등 상태 변화에 따라 여러 객체에 변경 사항을 전파해야 하는 경우에 널리 사용됩니다.

## 예제 코드 (Java)

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

## 단점

- **예상치 못한 업데이트**: 옵저버가 많아지면 상태 변경 시 어떤 순서로 알림이 가는지 제어하기 어렵고, 변경 사항 전파가 복잡해질 수 있습니다.
- **메모리 누수**: 옵저버가 명시적으로 등록 해제되지 않으면, 주제 객체가 메모리에서 해제되지 않아 메모리 누수가 발생할 수 있습니다.
- **성능 문제**: 너무 많은 옵저버가 등록되어 있거나 업데이트 로직이 복잡한 경우, 알림을 보내는 과정에서 성능 저하가 발생할 수 있습니다.