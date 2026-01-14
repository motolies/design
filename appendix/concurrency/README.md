# 동시성 (Concurrency) 패턴

동시성 패턴은 멀티스레드 환경에서 안전하고 효율적으로 객체를 생성하고 관리하기 위한 패턴입니다. 스레드 안전성을 보장하면서도 성능을 최적화하는 것이 핵심입니다.

## 패턴 목록

| 패턴 | 설명 | 난이도 | Spring 활용 |
|------|------|--------|-------------|
| [Double Checked Locking](./DoubleCheckedLocking) | 멀티스레드 환경에서 지연 초기화 최적화 | 중 | Bean Lifecycle |
| [Object Pool](./ObjectPool) | 객체 재사용으로 생성 비용 절감 | 중 | HikariCP, ThreadPool |

## 핵심 개념

### 스레드 안전성 (Thread Safety)
여러 스레드가 동시에 접근해도 정확하게 동작하는 것을 보장합니다.

### 지연 초기화 (Lazy Initialization)
실제로 필요할 때까지 객체 생성을 미루어 리소스를 절약합니다.

### 객체 재사용 (Object Reuse)
비용이 큰 객체를 풀에 보관하고 재사용하여 성능을 향상시킵니다.

## 패턴 선택 가이드

### Double Checked Locking을 선택할 때
- 싱글턴 패턴을 멀티스레드 환경에서 구현할 때
- 지연 초기화가 필요하면서 동기화 오버헤드를 최소화하고 싶을 때
- 객체 생성 비용이 높아 최초 접근 시에만 생성하고 싶을 때

### Object Pool을 선택할 때
- 객체 생성/소멸 비용이 높을 때 (DB 커넥션, 스레드 등)
- 동시에 사용되는 객체 수에 제한이 필요할 때
- 리소스 재사용으로 메모리/시간 효율을 높이고 싶을 때

## 관련 GoF 패턴

| GoF 패턴 | 관계 |
|----------|------|
| [Singleton](../../creational/Singleton) | Double Checked Locking은 Singleton의 스레드 안전 구현 방법 |
| [Flyweight](../../structural/Flyweight) | Object Pool과 유사하게 객체 재사용 개념 적용 |
