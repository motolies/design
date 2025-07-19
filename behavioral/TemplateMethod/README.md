# 템플릿 메서드 패턴 (Template Method Pattern)

## 정의

템플릿 메서드 패턴은 알고리즘의 골격(구조)을 상위 클래스에서 정의하고, 일부 단계를 서브클래스에서 구현하도록 하는 행동 디자인 패턴입니다. 이를 통해 알고리즘의 구조를 변경하지 않으면서 특정 단계를 서브클래스에서 재정의할 수 있습니다.

## 사용 이유

- **알고리즘 구조 고정**: 전체적으로는 동일하면서 부분적으로는 다른 구문으로 구성된 메서드의 코드 중복을 최소화하고 싶을 때 사용됩니다. 상위 클래스에서 알고리즘의 전체적인 흐름을 제어하고, 하위 클래스에서는 세부 내용을 커스터마이징할 수 있습니다.
- **코드 재사용성 향상**: 여러 클래스에서 공통으로 사용되는 알고리즘 로직을 추상 클래스에 정의하여 코드의 재사용성을 높입니다.
- **확장성**: 알고리즘의 특정 단계를 확장해야 할 때, 새로운 서브클래스를 추가하여 해당 부분만 오버라이드하면 되므로 유연한 확장이 가능합니다 (개방-폐쇄 원칙).

## 예제 코드 (Java)

```java
// AbstractClass: 템플릿 메서드를 정의하는 추상 클래스
abstract class Game {
    // 템플릿 메서드: 알고리즘의 골격을 정의하며 final로 선언하여 오버라이드를 막음
    public final void play() {
        // 공통적인 단계
        initialize();
        startPlay();
        // 서브클래스에서 구현할 단계
        playing();
        // 공통적인 단계
        endPlay();
    }

    // 공통적으로 구현된 메서드
    void initialize() {
        System.out.println("게임을 초기화합니다.");
    }

    void startPlay() {
        System.out.println("게임을 시작합니다.");
    }

    void endPlay() {
        System.out.println("게임을 종료합니다.");
    }

    // 서브클래스에서 구현해야 할 추상 메서드
    abstract void playing();
}

// ConcreteClass A: 축구 게임
class Football extends Game {
    @Override
    void playing() {
        System.out.println("축구 게임을 진행합니다!");
    }
}

// ConcreteClass B: 농구 게임
class Basketball extends Game {
    @Override
    void playing() {
        System.out.println("농구 게임을 진행합니다!");
    }
}

// 사용 예시
public class Client {
    public static void main(String[] args) {
        Game football = new Football();
        football.play();

        System.out.println();

        Game basketball = new Basketball();
        basketball.play();
    }
}
```

## 장점

- **코드 중복 감소**: 공통된 로직을 상위 클래스에 모아 코드 중복을 줄일 수 있습니다.
- **구조의 일관성 유지**: 알고리즘의 전체적인 구조를 변경하지 않고 특정 부분만 수정할 수 있어 일관성을 유지하기 쉽습니다.
- **유지보수 용이**: 공통 로직 수정 시 상위 클래스만 변경하면 되므로 유지보수가 편리합니다.

## 단점

- **유연성 제한**: 템플릿 메서드에서 정의한 알고리즘의 구조 자체는 변경하기 어렵습니다.
- **클래스 수 증가**: 각기 다른 구현에 대해 서브클래스를 만들어야 하므로 클래스의 수가 늘어날 수 있습니다.