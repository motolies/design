# 코드 품질 (Code Quality) 패턴

코드 품질 패턴은 더 안전하고 유지보수하기 쉬운 코드를 작성하기 위한 패턴입니다. 일반적인 프로그래밍 실수를 예방하고, 코드의 명확성과 안정성을 향상시킵니다.

## 패턴 목록

| 패턴 | 설명 | 난이도 | Spring 활용 |
|------|------|--------|-------------|
| [Null Object](./NullObject) | null 대신 기본 동작을 수행하는 객체 사용 | 하 | Optional<T>, 기본 구현체 |

## 핵심 개념

### Null 안전성 (Null Safety)
NullPointerException을 예방하기 위한 방어적 프로그래밍 기법입니다.

### 다형성 활용 (Polymorphism)
조건문 대신 다형성을 사용하여 코드 분기를 처리합니다.

### 기본 동작 (Default Behavior)
특수한 경우에도 일관된 동작을 제공합니다.

## 패턴 선택 가이드

### Null Object를 선택할 때
- null 체크 로직이 코드 전체에 퍼져 있을 때
- "값이 없음"이 유효한 비즈니스 상태일 때
- 다형성을 활용하여 조건문을 줄이고 싶을 때
- Optional보다 더 풍부한 기본 동작이 필요할 때

## Null Object vs Optional

| 관점 | Null Object | Optional<T> |
|------|-------------|-------------|
| 동작 | 기본 동작 수행 | 값 존재 여부만 표현 |
| 사용 | 인터페이스 구현체 | 반환 타입 래핑 |
| 장점 | 다형성, 풍부한 기본 동작 | 간단, JDK 표준 |
| 적합 | 복잡한 기본 로직 | 단순한 값 존재 확인 |

## 관련 GoF 패턴

| GoF 패턴 | 관계 |
|----------|------|
| [Strategy](../../behavioral/Strategy) | Null Object는 "아무것도 하지 않는 전략"으로 볼 수 있음 |
| [State](../../behavioral/State) | Null Object는 "null 상태"를 명시적으로 표현 |
| [Singleton](../../creational/Singleton) | Null Object는 보통 싱글턴으로 구현 |
