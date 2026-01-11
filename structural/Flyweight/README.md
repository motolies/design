# Flyweight 패턴 (플라이웨이트 패턴)

## 개요

Flyweight 패턴은 많은 수의 객체를 효율적으로 지원하기 위해 메모리 사용을 최소화하는 구조 패턴입니다. 객체의 내재적 상태(intrinsic state)를 공유하고 외재적 상태(extrinsic state)를 별도로 관리하여 메모리 사용량을 줄입니다.

## 🎯 한눈에 보기

| 항목 | 설명 |
|------|------|
| **핵심** | 공통 데이터를 **공유**하여 메모리 절약 |
| **비유** | 체스 게임 - 말의 이미지는 공유, 위치만 개별 저장 |
| **언제** | 유사한 객체가 수천~수만 개 필요할 때 |
| **Spring** | `@Cacheable`, String Pool, Integer Cache |

### 핵심 개념: 상태 분리
```
객체 상태를 두 가지로 분리:

내재적 상태 (Intrinsic)     외재적 상태 (Extrinsic)
├── 공유 가능               ├── 공유 불가
├── 변하지 않음             ├── 상황마다 다름
├── Flyweight에 저장        ├── Context에 저장
└── 예: 폰트, 색상, 이미지   └── 예: 위치, 크기

100만 개 문자 렌더링 시:
- Flyweight 없이: 100만 개 폰트 정보 저장
- Flyweight 사용: 26개 알파벳 + 100만 개 위치 정보
→ 메모리 수십 배 절약!
```

## 구조

```mermaid
classDiagram
    class FlyweightFactory {
        -flyweights: Map~String, Flyweight~
        +getFlyweight(key: String): Flyweight
        +getCreatedFlyweightsCount(): int
    }

    class Flyweight {
        <<interface>>
        +operation(extrinsicState: Context)
    }

    class ConcreteFlyweight {
        -intrinsicState: String
        +operation(extrinsicState: Context)
    }

    class UnsharedConcreteFlyweight {
        -allState: String
        +operation(extrinsicState: Context)
    }

    class Context {
        -extrinsicState: String
        -flyweight: Flyweight
        +operation()
    }

    class Client {
        -contexts: List~Context~
        +main()
    }

    Client --> FlyweightFactory
    Client --> Context
    Context --> Flyweight
    FlyweightFactory --> Flyweight
    ConcreteFlyweight ..|> Flyweight
    UnsharedConcreteFlyweight ..|> Flyweight
    FlyweightFactory ..> ConcreteFlyweight
```

## 주요 구성 요소

- **Flyweight**: 플라이웨이트 인터페이스로, 외재적 상태를 받아 작업을 수행하는 메서드를 정의합니다.
- **ConcreteFlyweight**: 내재적 상태를 저장하고 공유 가능한 플라이웨이트를 구현합니다.
- **UnsharedConcreteFlyweight**: 공유되지 않는 플라이웨이트 (모든 상태를 저장).
- **FlyweightFactory**: 플라이웨이트 인스턴스를 생성하고 관리하는 팩토리입니다.
- **Context**: 외재적 상태와 플라이웨이트 참조를 저장합니다.

## 동작 흐름

```mermaid
sequenceDiagram
    participant Client
    participant Factory as FlyweightFactory
    participant Cache as Cache (Map)
    participant Flyweight

    Client->>Factory: getFlyweight("A")
    Factory->>Cache: 캐시 확인
    Cache-->>Factory: 없음

    Factory->>Flyweight: new Flyweight("A")
    Factory->>Cache: 캐시에 저장
    Factory-->>Client: Flyweight 반환

    Note over Client: 외재적 상태와 함께 사용
    Client->>Flyweight: operation(위치, 크기)

    Client->>Factory: getFlyweight("A")
    Factory->>Cache: 캐시 확인
    Cache-->>Factory: 있음!
    Factory-->>Client: 캐시된 Flyweight 반환

    Note over Client,Flyweight: 같은 Flyweight 재사용!
```

### 메모리 절약 원리

```mermaid
sequenceDiagram
    participant App as 애플리케이션
    participant Factory as FlyweightFactory
    participant Memory as 메모리

    Note over App,Memory: 10,000개 문자 렌더링

    App->>Factory: 'A' 문자 1,000개 요청
    Factory->>Memory: Flyweight 1개 생성
    Factory-->>App: 같은 Flyweight 1,000번 반환

    App->>Factory: 'B' 문자 500개 요청
    Factory->>Memory: Flyweight 1개 생성
    Factory-->>App: 같은 Flyweight 500번 반환

    Note over Memory: 총 Flyweight: 26개 (알파벳)<br/>위치 정보: 10,000개<br/>→ 메모리 99% 절약!
```

## 내재적 상태 vs 외재적 상태

### 내재적 상태 (Intrinsic State)
- 객체에 저장되며 여러 컨텍스트에서 공유 가능
- 변하지 않는 정보 (예: 문자의 폰트, 색상)
- 플라이웨이트 객체 내부에 저장

### 외재적 상태 (Extrinsic State)
- 컨텍스트에 의존적이며 공유할 수 없음
- 변할 수 있는 정보 (예: 문자의 위치, 크기)
- 클라이언트가 플라이웨이트에 전달

## 실제 사용 사례

### 1. 텍스트 에디터
- 문자 객체의 글꼴, 스타일은 내재적 상태
- 문자의 위치, 크기는 외재적 상태

### 2. 게임 개발
- 파티클 시스템에서 텍스처, 색상은 내재적 상태
- 위치, 속도는 외재적 상태

### 3. 웹 브라우저
- HTML 요소의 스타일 정보는 내재적 상태
- DOM 트리에서의 위치는 외재적 상태

### 4. 아이콘 시스템
- 아이콘 이미지는 내재적 상태
- 화면에서의 위치, 크기는 외재적 상태

## 초급 예제: 나무 심기 게임

수천 그루의 나무를 그리는 게임을 상상해보세요. 각 나무마다 이미지를 저장하면 메모리가 부족합니다.

```java
import java.util.*;

// 1. Flyweight - 공유되는 나무 타입 (내재적 상태)
class TreeType {
    private String name;      // 나무 이름
    private String texture;   // 텍스처 이미지 (용량 큼!)

    public TreeType(String name, String texture) {
        this.name = name;
        this.texture = texture;
        System.out.println("🌲 새 나무 타입 생성: " + name);
    }

    public void draw(int x, int y) {
        System.out.println("  " + name + " 그리기 at (" + x + "," + y + ")");
    }
}

// 2. Flyweight Factory - 나무 타입 캐시
class TreeFactory {
    private static Map<String, TreeType> cache = new HashMap<>();

    public static TreeType getTreeType(String name) {
        if (!cache.containsKey(name)) {
            cache.put(name, new TreeType(name, name + "_texture.png"));
        }
        return cache.get(name);
    }

    public static int getCacheSize() { return cache.size(); }
}

// 3. Context - 개별 나무 (외재적 상태: 위치)
class Tree {
    private int x, y;           // 외재적 상태
    private TreeType type;      // Flyweight 참조

    public Tree(int x, int y, TreeType type) {
        this.x = x;
        this.y = y;
        this.type = type;
    }

    public void draw() {
        type.draw(x, y);
    }
}

// 4. 사용 예제
public class FlyweightDemo {
    public static void main(String[] args) {
        List<Tree> forest = new ArrayList<>();

        // 1000그루 나무 심기 (3가지 타입만 생성됨!)
        String[] types = {"소나무", "참나무", "단풍나무"};

        for (int i = 0; i < 10; i++) {
            String typeName = types[i % 3];
            TreeType type = TreeFactory.getTreeType(typeName);
            forest.add(new Tree(i * 10, i * 5, type));
        }

        System.out.println("\n--- 숲 그리기 ---");
        forest.forEach(Tree::draw);

        System.out.println("\n📊 통계:");
        System.out.println("총 나무 수: " + forest.size());
        System.out.println("생성된 TreeType 수: " + TreeFactory.getCacheSize());
    }
}
```

**실행 결과:**
```
🌲 새 나무 타입 생성: 소나무
🌲 새 나무 타입 생성: 참나무
🌲 새 나무 타입 생성: 단풍나무

--- 숲 그리기 ---
  소나무 그리기 at (0,0)
  참나무 그리기 at (10,5)
  단풍나무 그리기 at (20,10)
  ...

📊 통계:
총 나무 수: 10
생성된 TreeType 수: 3
```

**핵심 포인트:**
- `TreeType`: 무거운 데이터(텍스처) 포함 → 공유
- `Tree`: 가벼운 데이터(위치) 포함 → 개별 저장
- 1000그루를 그려도 `TreeType`은 3개만 생성!

## 복잡한 실생활 예제: 대규모 온라인 게임의 파티클 시스템

```java
// 외재적 상태를 나타내는 컨텍스트
class ParticleContext {
    private double x, y, z;          // 위치
    private double velocityX, velocityY, velocityZ;  // 속도
    private double size;             // 크기
    private double rotation;         // 회전
    private double life;             // 생명력 (0.0 ~ 1.0)
    private long creationTime;       // 생성 시간

    public ParticleContext(double x, double y, double z, double velocityX,
                          double velocityY, double velocityZ, double size) {
        this.x = x;
        this.y = y;
        this.z = z;
        this.velocityX = velocityX;
        this.velocityY = velocityY;
        this.velocityZ = velocityZ;
        this.size = size;
        this.rotation = 0.0;
        this.life = 1.0;
        this.creationTime = System.currentTimeMillis();
    }

    // 파티클 업데이트 (물리 계산)
    public void update(double deltaTime) {
        // 위치 업데이트
        x += velocityX * deltaTime;
        y += velocityY * deltaTime;
        z += velocityZ * deltaTime;

        // 중력 효과
        velocityY -= 9.8 * deltaTime;

        // 회전 업데이트
        rotation += 180 * deltaTime; // 초당 180도 회전

        // 생명력 감소
        long currentTime = System.currentTimeMillis();
        double totalLifespan = 5000; // 5초
        life = Math.max(0, 1.0 - (currentTime - creationTime) / totalLifespan);
    }

    public boolean isAlive() {
        return life > 0;
    }

    // Getters
    public double getX() { return x; }
    public double getY() { return y; }
    public double getZ() { return z; }
    public double getSize() { return size; }
    public double getRotation() { return rotation; }
    public double getLife() { return life; }

    public void setSize(double size) { this.size = size; }
}

// Flyweight 인터페이스
interface ParticleFlyweight {
    void render(ParticleContext context, GraphicsContext graphics);
    String getType();
    void applyEffect(ParticleContext context, String effectType);
}

// 구체적인 Flyweight 구현들
class FireParticle implements ParticleFlyweight {
    private final String texture;
    private final String[] colors;
    private final double baseOpacity;

    public FireParticle() {
        this.texture = "fire_particle.png";
        this.colors = new String[]{"#FF4500", "#FF6347", "#FFD700", "#FF8C00"};
        this.baseOpacity = 0.8;
    }

    @Override
    public void render(ParticleContext context, GraphicsContext graphics) {
        // 생명력에 따른 색상과 투명도 계산
        double life = context.getLife();
        String color = colors[(int)(life * (colors.length - 1))];
        double opacity = baseOpacity * life;

        graphics.setColor(color);
        graphics.setOpacity(opacity);
        graphics.setTexture(texture);
        graphics.drawParticle(
            context.getX(), context.getY(), context.getZ(),
            context.getSize() * life, // 생명력에 따라 크기 감소
            context.getRotation()
        );
    }

    @Override
    public String getType() {
        return "FIRE";
    }

    @Override
    public void applyEffect(ParticleContext context, String effectType) {
        switch (effectType) {
            case "EXPLOSION":
                context.setSize(context.getSize() * 1.5);
                break;
            case "WIND":
                // 바람 효과는 외재적 상태에서 처리
                break;
        }
    }
}

class SmokeParticle implements ParticleFlyweight {
    private final String texture;
    private final String color;
    private final double baseOpacity;

    public SmokeParticle() {
        this.texture = "smoke_particle.png";
        this.color = "#696969";
        this.baseOpacity = 0.6;
    }

    @Override
    public void render(ParticleContext context, GraphicsContext graphics) {
        double life = context.getLife();
        double opacity = baseOpacity * life * 0.5; // 연기는 더 투명

        graphics.setColor(color);
        graphics.setOpacity(opacity);
        graphics.setTexture(texture);
        graphics.drawParticle(
            context.getX(), context.getY(), context.getZ(),
            context.getSize() * (2.0 - life), // 시간이 지날수록 크기 증가
            context.getRotation()
        );
    }

    @Override
    public String getType() {
        return "SMOKE";
    }

    @Override
    public void applyEffect(ParticleContext context, String effectType) {
        switch (effectType) {
            case "DISPERSE":
                context.setSize(context.getSize() * 2.0);
                break;
        }
    }
}

class WaterParticle implements ParticleFlyweight {
    private final String texture;
    private final String color;
    private final double baseOpacity;

    public WaterParticle() {
        this.texture = "water_particle.png";
        this.color = "#4169E1";
        this.baseOpacity = 0.9;
    }

    @Override
    public void render(ParticleContext context, GraphicsContext graphics) {
        double life = context.getLife();

        graphics.setColor(color);
        graphics.setOpacity(baseOpacity * life);
        graphics.setTexture(texture);
        graphics.drawParticle(
            context.getX(), context.getY(), context.getZ(),
            context.getSize(),
            context.getRotation()
        );
    }

    @Override
    public String getType() {
        return "WATER";
    }

    @Override
    public void applyEffect(ParticleContext context, String effectType) {
        switch (effectType) {
            case "SPLASH":
                context.setSize(context.getSize() * 0.8);
                break;
        }
    }
}

// Flyweight Factory
class ParticleFactory {
    private static final Map<String, ParticleFlyweight> flyweights = new HashMap<>();
    private static int createdFlyweights = 0;

    public static ParticleFlyweight getParticle(String type) {
        ParticleFlyweight flyweight = flyweights.get(type);

        if (flyweight == null) {
            switch (type.toUpperCase()) {
                case "FIRE":
                    flyweight = new FireParticle();
                    break;
                case "SMOKE":
                    flyweight = new SmokeParticle();
                    break;
                case "WATER":
                    flyweight = new WaterParticle();
                    break;
                default:
                    throw new IllegalArgumentException("Unknown particle type: " + type);
            }
            flyweights.put(type, flyweight);
            createdFlyweights++;
            System.out.println("새로운 " + type + " 파티클 타입 생성. 총 생성된 타입: " + createdFlyweights);
        }

        return flyweight;
    }

    public static int getCreatedFlyweightsCount() {
        return createdFlyweights;
    }

    public static void printFlyweightInfo() {
        System.out.println("=== Flyweight 정보 ===");
        System.out.println("생성된 Flyweight 타입 수: " + createdFlyweights);
        System.out.println("저장된 타입들: " + flyweights.keySet());
    }
}

// 그래픽 컨텍스트 (시뮬레이션용)
class GraphicsContext {
    private String currentColor;
    private double currentOpacity;
    private String currentTexture;

    public void setColor(String color) {
        this.currentColor = color;
    }

    public void setOpacity(double opacity) {
        this.currentOpacity = opacity;
    }

    public void setTexture(String texture) {
        this.currentTexture = texture;
    }

    public void drawParticle(double x, double y, double z, double size, double rotation) {
        System.out.printf("파티클 렌더링: 위치(%.1f, %.1f, %.1f) 크기=%.1f 회전=%.1f° " +
                         "색상=%s 투명도=%.2f 텍스처=%s%n",
                         x, y, z, size, rotation, currentColor, currentOpacity, currentTexture);
    }
}

// 파티클 시스템 관리자
class ParticleSystem {
    private final List<ParticleContext> particles;
    private final GraphicsContext graphics;
    private final Map<ParticleContext, ParticleFlyweight> particleTypes;

    public ParticleSystem() {
        this.particles = new ArrayList<>();
        this.graphics = new GraphicsContext();
        this.particleTypes = new HashMap<>();
    }

    public void addParticle(String type, double x, double y, double z,
                           double velocityX, double velocityY, double velocityZ, double size) {
        ParticleContext context = new ParticleContext(x, y, z, velocityX, velocityY, velocityZ, size);
        ParticleFlyweight flyweight = ParticleFactory.getParticle(type);

        particles.add(context);
        particleTypes.put(context, flyweight);
    }

    public void update(double deltaTime) {
        Iterator<ParticleContext> iterator = particles.iterator();
        while (iterator.hasNext()) {
            ParticleContext particle = iterator.next();
            particle.update(deltaTime);

            if (!particle.isAlive()) {
                particleTypes.remove(particle);
                iterator.remove();
            }
        }
    }

    public void render() {
        System.out.println("=== 파티클 렌더링 (총 " + particles.size() + "개) ===");
        for (ParticleContext particle : particles) {
            ParticleFlyweight flyweight = particleTypes.get(particle);
            flyweight.render(particle, graphics);
        }
    }

    public void addExplosionEffect(double x, double y, double z) {
        System.out.println("폭발 효과 생성!");

        // 화염 파티클들
        for (int i = 0; i < 50; i++) {
            double angle = Math.random() * 2 * Math.PI;
            double speed = 20 + Math.random() * 30;
            double velocityX = Math.cos(angle) * speed;
            double velocityY = 10 + Math.random() * 20;
            double velocityZ = Math.sin(angle) * speed;

            addParticle("FIRE", x, y, z, velocityX, velocityY, velocityZ, 1.0 + Math.random());
        }

        // 연기 파티클들
        for (int i = 0; i < 30; i++) {
            double angle = Math.random() * 2 * Math.PI;
            double speed = 5 + Math.random() * 10;
            double velocityX = Math.cos(angle) * speed;
            double velocityY = 2 + Math.random() * 5;
            double velocityZ = Math.sin(angle) * speed;

            addParticle("SMOKE", x, y, z, velocityX, velocityY, velocityZ, 2.0 + Math.random());
        }
    }

    public void applyGlobalEffect(String effectType) {
        System.out.println("글로벌 효과 적용: " + effectType);
        for (ParticleContext particle : particles) {
            ParticleFlyweight flyweight = particleTypes.get(particle);
            flyweight.applyEffect(particle, effectType);
        }
    }

    public void printSystemInfo() {
        System.out.println("=== 파티클 시스템 정보 ===");
        System.out.println("활성 파티클 수: " + particles.size());

        Map<String, Integer> typeCount = new HashMap<>();
        for (ParticleFlyweight flyweight : particleTypes.values()) {
            typeCount.merge(flyweight.getType(), 1, Integer::sum);
        }

        System.out.println("타입별 파티클 수:");
        typeCount.forEach((type, count) -> System.out.println("  " + type + ": " + count + "개"));
    }
}

// 클라이언트 코드
public class ParticleSystemDemo {
    public static void main(String[] args) {
        ParticleSystem system = new ParticleSystem();

        System.out.println("=== 파티클 시스템 시뮬레이션 시작 ===");

        // 초기 파티클들 생성
        system.addParticle("FIRE", 0, 0, 0, 1, 5, 0, 1.0);
        system.addParticle("SMOKE", 0, 2, 0, 0, 3, 0, 1.5);
        system.addParticle("WATER", 5, 0, 0, -2, 8, 0, 0.8);

        // 폭발 효과 (많은 파티클 생성)
        system.addExplosionEffect(10, 0, 10);

        ParticleFactory.printFlyweightInfo();
        system.printSystemInfo();

        // 시뮬레이션 루프
        for (int frame = 0; frame < 3; frame++) {
            System.out.println("\n=== 프레임 " + (frame + 1) + " ===");

            // 업데이트 (0.1초씩)
            system.update(0.1);

            // 몇 개 파티클만 렌더링 (출력 제한)
            if (frame == 0) {
                system.render();
            }

            // 글로벌 효과 적용
            if (frame == 1) {
                system.applyGlobalEffect("EXPLOSION");
            }

            system.printSystemInfo();
        }

        // 최종 통계
        System.out.println("\n=== 최종 통계 ===");
        ParticleFactory.printFlyweightInfo();

        // 메모리 사용량 비교
        int totalParticles = 100; // 실제 시나리오에서는 수천~수만 개
        int flyweightTypes = ParticleFactory.getCreatedFlyweightsCount();

        System.out.println("메모리 효율성:");
        System.out.println("Flyweight 없이: " + totalParticles + "개의 파티클 객체 필요");
        System.out.println("Flyweight 사용: " + flyweightTypes + "개의 Flyweight + " +
                          totalParticles + "개의 Context 객체");
        System.out.println("메모리 절약률: " +
                          String.format("%.1f%%", (1.0 - (double)flyweightTypes / totalParticles) * 100));
    }
}
```

## 기본 Flyweight 패턴 예제

```java
// Flyweight 인터페이스
interface CharacterFlyweight {
    void display(int row, int column, String font, int size);
}

// 구체적인 Flyweight
class Character implements CharacterFlyweight {
    private final char symbol; // 내재적 상태

    public Character(char symbol) {
        this.symbol = symbol;
        System.out.println("새로운 문자 객체 생성: " + symbol);
    }

    @Override
    public void display(int row, int column, String font, int size) {
        // 외재적 상태 (row, column, font, size)를 사용하여 렌더링
        System.out.printf("문자 '%c' 표시: 위치(%d, %d) 폰트=%s 크기=%d%n",
                         symbol, row, column, font, size);
    }
}

// Flyweight Factory
class CharacterFactory {
    private static final Map<Character, CharacterFlyweight> flyweights = new HashMap<>();

    public static CharacterFlyweight getCharacter(char symbol) {
        CharacterFlyweight flyweight = flyweights.get(symbol);
        if (flyweight == null) {
            flyweight = new Character(symbol);
            flyweights.put(symbol, flyweight);
        }
        return flyweight;
    }

    public static int getFlyweightCount() {
        return flyweights.size();
    }
}

// 컨텍스트 클래스
class TextContext {
    private int row, column;
    private String font;
    private int size;
    private CharacterFlyweight character;

    public TextContext(int row, int column, String font, int size, char symbol) {
        this.row = row;
        this.column = column;
        this.font = font;
        this.size = size;
        this.character = CharacterFactory.getCharacter(symbol);
    }

    public void display() {
        character.display(row, column, font, size);
    }
}
```

## Java 표준 라이브러리의 Flyweight 예제

### String 풀 (String Pool)
```java
public class StringPoolExample {
    public static void main(String[] args) {
        // 문자열 리터럴은 String Pool에서 관리 (Flyweight)
        String str1 = "Hello";
        String str2 = "Hello";
        String str3 = new String("Hello");

        System.out.println(str1 == str2);  // true (같은 객체 참조)
        System.out.println(str1 == str3);  // false (다른 객체)
        System.out.println(str1 == str3.intern()); // true (intern으로 풀에서 가져옴)
    }
}
```

### Integer 캐싱
```java
public class IntegerCacheExample {
    public static void main(String[] args) {
        // -128 ~ 127 범위의 Integer는 캐시됨 (Flyweight)
        Integer a = 100;
        Integer b = 100;
        Integer c = 200;
        Integer d = 200;

        System.out.println(a == b);  // true (캐시된 객체)
        System.out.println(c == d);  // false (캐시 범위 벗어남)
    }
}
```

## Spring Boot에서의 Flyweight 패턴

### 1. @Cacheable을 이용한 Flyweight 구현

```java
// Flyweight - 상품 상세 정보 (무거운 데이터)
@Getter @Setter
public class ProductDetail {
    private String description;      // 상품 설명
    private List<String> images;     // 이미지 URL 목록
    private Map<String, String> specs;  // 상세 스펙
    // ... 용량이 큰 데이터들
}

// Flyweight Factory 역할 - Spring Cache 활용
@Service
@RequiredArgsConstructor
public class ProductDetailService {
    private final ProductRepository productRepository;

    @Cacheable(value = "productDetails", key = "#productId")
    public ProductDetail getProductDetail(Long productId) {
        System.out.println("📦 DB에서 상품 상세 로딩: " + productId);
        // 비용이 큰 DB 조회
        return productRepository.findDetailById(productId);
    }
}

// Context - 사용자별 장바구니 항목 (외재적 상태)
@Entity
@Getter @Setter
public class CartItem {
    @Id @GeneratedValue
    private Long id;

    private Long productId;    // Flyweight 참조
    private int quantity;      // 외재적 상태
    private Long userId;       // 외재적 상태
}

// 사용하는 서비스
@Service
@RequiredArgsConstructor
public class CartService {
    private final ProductDetailService productDetailService;
    private final CartItemRepository cartItemRepository;

    public CartItemDto getCartItemWithDetail(Long cartItemId) {
        CartItem item = cartItemRepository.findById(cartItemId)
            .orElseThrow();

        // 캐시된 상품 정보 재사용 (Flyweight)
        ProductDetail detail = productDetailService.getProductDetail(
            item.getProductId());

        return new CartItemDto(item, detail);
    }
}
```

### 2. 아이콘/에셋 캐시 서비스

```java
// Flyweight - 아이콘 데이터
@Getter
public class IconData {
    private final String name;
    private final byte[] imageBytes;  // 실제 이미지 데이터 (무거움)
    private final String mimeType;

    public IconData(String name, byte[] imageBytes, String mimeType) {
        this.name = name;
        this.imageBytes = imageBytes;
        this.mimeType = mimeType;
        System.out.println("🖼️ 아이콘 로드: " + name);
    }
}

// Flyweight Factory
@Component
public class IconCache {
    private final Map<String, IconData> cache = new ConcurrentHashMap<>();

    @Value("${icons.path}")
    private String iconBasePath;

    public IconData getIcon(String iconName) {
        return cache.computeIfAbsent(iconName, this::loadIcon);
    }

    private IconData loadIcon(String iconName) {
        // 파일 시스템에서 로드 (비용 큼)
        byte[] bytes = Files.readAllBytes(
            Path.of(iconBasePath, iconName + ".png"));
        return new IconData(iconName, bytes, "image/png");
    }

    public int getCacheSize() {
        return cache.size();
    }

    @Scheduled(fixedRate = 3600000) // 1시간마다
    public void reportCacheStats() {
        System.out.println("📊 아이콘 캐시 크기: " + cache.size());
    }
}

// 컨트롤러
@RestController
@RequestMapping("/api/icons")
@RequiredArgsConstructor
public class IconController {
    private final IconCache iconCache;

    @GetMapping("/{name}")
    public ResponseEntity<byte[]> getIcon(@PathVariable String name) {
        IconData icon = iconCache.getIcon(name);
        return ResponseEntity.ok()
            .contentType(MediaType.parseMediaType(icon.getMimeType()))
            .body(icon.getImageBytes());
    }
}
```

### 3. 공통 코드 캐시 (코드 테이블)

```java
// Flyweight - 공통 코드 (자주 사용되는 참조 데이터)
@Entity
@Getter
@Table(name = "common_code")
public class CommonCode {
    @Id
    private String code;
    private String name;
    private String groupCode;
    private String description;
    private int sortOrder;
}

// Flyweight Factory
@Service
@RequiredArgsConstructor
public class CommonCodeService {
    private final CommonCodeRepository repository;

    // 그룹별 코드 캐시
    private final Map<String, List<CommonCode>> groupCache =
        new ConcurrentHashMap<>();

    // 개별 코드 캐시
    private final Map<String, CommonCode> codeCache =
        new ConcurrentHashMap<>();

    @PostConstruct
    public void initCache() {
        System.out.println("📋 공통 코드 캐시 초기화 시작");
        List<CommonCode> allCodes = repository.findAll();

        // 개별 코드 캐시
        allCodes.forEach(code ->
            codeCache.put(code.getCode(), code));

        // 그룹별 캐시
        allCodes.stream()
            .collect(Collectors.groupingBy(CommonCode::getGroupCode))
            .forEach(groupCache::put);

        System.out.println("✅ 캐시 완료: " + codeCache.size() + "개 코드");
    }

    public CommonCode getCode(String code) {
        return codeCache.get(code);
    }

    public List<CommonCode> getCodesByGroup(String groupCode) {
        return groupCache.getOrDefault(groupCode, Collections.emptyList());
    }

    public String getCodeName(String code) {
        CommonCode cc = codeCache.get(code);
        return cc != null ? cc.getName() : code;
    }
}

// 주문에서 사용 - 상태 코드를 Flyweight로 참조
@Entity
@Getter @Setter
public class Order {
    @Id @GeneratedValue
    private Long id;

    private String statusCode;  // "ORDER_STATUS_01" 같은 코드 저장
    // ... 다른 필드들

    @Transient
    public String getStatusName(CommonCodeService codeService) {
        return codeService.getCodeName(statusCode);
    }
}
```

### 4. 폰트/스타일 캐시

```java
// Flyweight - 스타일 정의
@Getter
public class TextStyle {
    private final String fontFamily;
    private final int fontSize;
    private final String color;
    private final boolean bold;
    private final boolean italic;

    public TextStyle(String fontFamily, int fontSize, String color,
                     boolean bold, boolean italic) {
        this.fontFamily = fontFamily;
        this.fontSize = fontSize;
        this.color = color;
        this.bold = bold;
        this.italic = italic;
        System.out.println("🎨 스타일 생성: " + this);
    }

    @Override
    public String toString() {
        return String.format("%s %dpx %s%s%s",
            fontFamily, fontSize, color,
            bold ? " bold" : "", italic ? " italic" : "");
    }
}

// Flyweight Factory
@Component
public class TextStyleFactory {
    private final Map<String, TextStyle> styles = new ConcurrentHashMap<>();

    public TextStyle getStyle(String fontFamily, int fontSize, String color,
                              boolean bold, boolean italic) {
        String key = String.format("%s-%d-%s-%b-%b",
            fontFamily, fontSize, color, bold, italic);

        return styles.computeIfAbsent(key,
            k -> new TextStyle(fontFamily, fontSize, color, bold, italic));
    }

    // 미리 정의된 스타일
    public TextStyle getHeadingStyle() {
        return getStyle("Arial", 24, "#333333", true, false);
    }

    public TextStyle getBodyStyle() {
        return getStyle("Roboto", 14, "#666666", false, false);
    }

    public TextStyle getCaptionStyle() {
        return getStyle("Roboto", 12, "#999999", false, true);
    }

    public int getStyleCount() {
        return styles.size();
    }
}
```

## 장점

- **메모리 절약**: 많은 객체의 공통 부분을 공유하여 메모리 사용량을 크게 줄입니다.
- **성능 향상**: 객체 생성 비용을 줄여 애플리케이션 성능을 향상시킵니다.
- **확장성**: 대량의 객체를 효율적으로 처리할 수 있습니다.

## 단점

- **복잡성 증가**: 내재적/외재적 상태 분리로 코드 복잡성이 증가합니다.
- **외재적 상태 관리**: 클라이언트가 외재적 상태를 관리해야 하는 부담이 있습니다.
- **동기화 고려**: 멀티스레드 환경에서는 Flyweight Factory의 스레드 안전성을 고려해야 합니다.

## 관련 패턴

| 패턴 | 관계 | 비교 |
|------|------|------|
| **Singleton** | 조합 | FlyweightFactory는 보통 Singleton으로 구현 |
| **Factory Method** | 조합 | Flyweight 생성 시 Factory Method 사용 가능 |
| **Composite** | 조합 | Composite의 리프 노드를 Flyweight로 구현 |
| **Prototype** | 대안 | Prototype은 복제, Flyweight는 공유 |

### Flyweight vs Singleton vs Prototype

```java
// Singleton: 오직 하나만 존재
class AppConfig {
    private static final AppConfig INSTANCE = new AppConfig();
    public static AppConfig getInstance() { return INSTANCE; }
}

// Prototype: 복제하여 새 객체 생성
class Monster {
    public Monster clone() { return new Monster(this); }
}

// Flyweight: 같은 타입 공유, 상태는 분리
class TreeType {  // 공유됨 (예: 소나무, 참나무)
    private String texture;
}
class Tree {  // 개별 (예: 위치 정보)
    private TreeType type;  // 참조
    private int x, y;
}
```

**선택 기준:**
- 시스템에 하나만 필요 → **Singleton**
- 복제해서 수정 필요 → **Prototype**
- 공통 데이터 공유 + 개별 데이터 분리 → **Flyweight**

## 언제 사용할까?

1. 애플리케이션이 대량의 객체를 사용할 때
2. 객체 저장 비용이 높을 때
3. 대부분의 객체 상태를 외재적으로 만들 수 있을 때
4. 객체의 그룹들을 비교적 적은 수의 공유 객체로 대체할 수 있을 때
5. 애플리케이션이 객체의 정체성에 의존하지 않을 때