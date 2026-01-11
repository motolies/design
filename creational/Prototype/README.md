# 프로토타입 패턴 (Prototype Pattern)

## 정의

프로토타입 패턴은 기존 객체를 복제하여 새로운 객체를 생성하는 생성 디자인 패턴입니다. 클래스로부터 객체를 생성하는 것이 비용이 많이 들거나 복잡할 때, 기존 인스턴스를 복제하여 새로운 인스턴스를 만드는 것이 더 효율적일 수 있습니다.

## 🎯 한눈에 보기

| 항목 | 설명 |
|------|------|
| **핵심** | new 대신 기존 객체를 **복제**하여 새 객체 생성 |
| **비유** | 세포 분열 - 기존 세포를 복제하여 새로운 세포 생성 |
| **언제** | 객체 생성 비용이 크거나, 비슷한 객체를 많이 만들 때 |
| **Spring** | `@Scope("prototype")`, `BeanUtils.copyProperties()` |

### 핵심 구성요소
```
Prototype       → clone() 메서드를 정의하는 인터페이스
ConcretePrototype → 자신을 복제하는 clone() 구현
Client          → clone()을 호출하여 객체 복제
```

### new vs clone
```java
// new: 처음부터 다시 생성 (비용 높음)
Monster m = new Monster();
m.loadFromDatabase();    // DB 조회
m.initializeGraphics();  // 리소스 로딩

// clone: 기존 객체 복제 (비용 낮음)
Monster m2 = existingMonster.clone();  // 즉시 완료!
```

## 구조 (Structure)

```mermaid
classDiagram
    class Prototype {
        <<interface>>
        +clone(): Prototype
    }

    class ConcretePrototype1 {
        -field1: String
        -field2: int
        +clone(): Prototype
        +operation(): void
    }

    class ConcretePrototype2 {
        -fieldA: String
        -fieldB: boolean
        +clone(): Prototype
        +operation(): void
    }

    class Client {
        -prototype: Prototype
        +operation(): void
    }

    Prototype <|.. ConcretePrototype1
    Prototype <|.. ConcretePrototype2
    Client --> Prototype : uses

    note for Prototype "복제 메서드를 정의하는 인터페이스"
    note for ConcretePrototype1 "자신을 복제하는 구체적인 구현"
```

## 동작 흐름

```mermaid
sequenceDiagram
    participant Client
    participant PrototypeRegistry
    participant Prototype
    participant Clone

    Note over PrototypeRegistry: 프로토타입들이 미리 등록됨

    Client->>PrototypeRegistry: getPrototype("typeA")
    PrototypeRegistry->>Prototype: clone()

    Note over Prototype: 자신을 복제하여<br/>새 객체 생성

    Prototype->>Clone: new Clone(this.fields)
    Prototype-->>PrototypeRegistry: clone 객체 반환
    PrototypeRegistry-->>Client: clone 객체 반환

    Client->>Clone: customize()
    Note over Clone: 필요에 따라<br/>복제본 수정
```

### 복제 과정 상세

```mermaid
sequenceDiagram
    participant Original as 원본 객체
    participant Clone as 복제 객체

    Note over Original: 이미 초기화된 상태<br/>(DB 로딩, 설정 완료)

    Original->>Clone: 얕은 복사 (super.clone())
    Note over Clone: 기본 필드 복사됨

    Original->>Clone: 깊은 복사 (참조 객체)
    Note over Clone: 참조 객체도 별도 복사

    Clone-->>Clone: 독립적인 객체 완성
```

## 사용 이유

- **성능 최적화**: 복잡한 객체 생성 과정을 피하고 기존 객체를 복제하여 성능을 향상시킵니다.
- **동적 객체 생성**: 런타임에 생성할 객체의 타입이 결정될 때 유용합니다.
- **복잡한 초기화 비용 절약**: 데이터베이스 연결, 파일 로딩 등 비용이 큰 초기화 작업을 한 번만 수행합니다.

## 적용 상황

프로토타입 패턴은 다음과 같은 상황에서 특히 유용합니다:

### 1. 객체 생성 비용이 높은 경우
- **복잡한 그래픽 객체**: 3D 모델, 이미지 처리 결과
- **데이터베이스 쿼리 결과**: 복잡한 조인 쿼리의 결과 객체
- **네트워크 리소스**: API 호출 결과, 원격 데이터

### 2. 다양한 설정의 객체가 필요한 경우
- **게임 아이템**: 기본 무기에서 다양한 옵션이 추가된 변형
- **문서 템플릿**: 기본 양식에서 부분적으로 수정된 문서
- **UI 컴포넌트**: 기본 스타일에서 색상, 크기 등이 변경된 컴포넌트

### 3. 객체의 클래스가 런타임에 결정되는 경우
```java
// 나쁜 예: 하드코딩된 객체 생성
if (type.equals("warrior")) {
    return new Warrior(); // 새로운 타입 추가시 코드 수정 필요
}

// 좋은 예: 프로토타입 패턴 사용
return prototypeRegistry.getPrototype(type).clone(); // 설정으로 관리 가능
```

## 얕은 복사 vs 깊은 복사

프로토타입 패턴을 구현할 때 가장 중요한 고려사항은 복사의 깊이입니다.

### 얕은 복사 (Shallow Copy)

```java
class ShallowCopyExample implements Cloneable {
    private String name;
    private List<String> items;

    public ShallowCopyExample(String name) {
        this.name = name;
        this.items = new ArrayList<>();
    }

    @Override
    public ShallowCopyExample clone() {
        try {
            return (ShallowCopyExample) super.clone(); // 얕은 복사
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }

    // getter, setter 메서드들...
}

// 문제점: 참조 객체가 공유됨
ShallowCopyExample original = new ShallowCopyExample("Original");
original.getItems().add("Item1");

ShallowCopyExample copy = original.clone();
copy.getItems().add("Item2"); // original의 items에도 영향!

System.out.println(original.getItems()); // [Item1, Item2] - 예상치 못한 변경!
```

### 깊은 복사 (Deep Copy)

```java
class DeepCopyExample implements Cloneable {
    private String name;
    private List<String> items;

    public DeepCopyExample(String name) {
        this.name = name;
        this.items = new ArrayList<>();
    }

    @Override
    public DeepCopyExample clone() {
        try {
            DeepCopyExample cloned = (DeepCopyExample) super.clone();
            cloned.items = new ArrayList<>(this.items); // 깊은 복사
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }

    // getter, setter 메서드들...
}

// 안전한 복사: 각각 독립적인 객체
DeepCopyExample original = new DeepCopyExample("Original");
original.getItems().add("Item1");

DeepCopyExample copy = original.clone();
copy.getItems().add("Item2"); // original에 영향 없음

System.out.println(original.getItems()); // [Item1] - 예상대로 동작
System.out.println(copy.getItems()); // [Item1, Item2]
```

## 초급 예제: 문서 복제 시스템

가장 간단한 프로토타입 패턴 예제입니다. 문서를 복제하여 새 문서를 만드는 상황입니다.

```java
// 1. 프로토타입 인터페이스 - 복제 기능 정의
interface Document extends Cloneable {
    Document copy();  // clone 대신 copy라는 이름 사용 (더 직관적)
    void print();
}

// 2. 구체적인 프로토타입 - 이력서 문서
class Resume implements Document {
    private String name;
    private String content;

    public Resume(String name, String content) {
        this.name = name;
        this.content = content;
        // 실제로는 여기서 복잡한 초기화가 일어날 수 있음
        System.out.println("📄 새 이력서 생성 (비용 높음)");
    }

    // 복제용 생성자 (깊은 복사)
    private Resume(Resume other) {
        this.name = other.name;
        this.content = other.content;
        System.out.println("📋 이력서 복제 (비용 낮음)");
    }

    @Override
    public Document copy() {
        return new Resume(this);  // 복제본 반환
    }

    public void setName(String name) { this.name = name; }

    @Override
    public void print() {
        System.out.println("이력서: " + name);
        System.out.println("내용: " + content);
    }
}

// 3. 사용 예제
public class PrototypeDemo {
    public static void main(String[] args) {
        // 원본 이력서 생성 (비용 높음)
        Resume original = new Resume("홍길동", "Java 개발자 경력 5년");

        // 복제하여 새 이력서 생성 (비용 낮음)
        Resume copy1 = (Resume) original.copy();
        copy1.setName("김철수");  // 이름만 변경

        Resume copy2 = (Resume) original.copy();
        copy2.setName("이영희");

        System.out.println("\n--- 결과 ---");
        original.print();
        copy1.print();
        copy2.print();
    }
}
```

**실행 결과:**
```
📄 새 이력서 생성 (비용 높음)
📋 이력서 복제 (비용 낮음)
📋 이력서 복제 (비용 낮음)

--- 결과 ---
이력서: 홍길동
내용: Java 개발자 경력 5년
이력서: 김철수
내용: Java 개발자 경력 5년
이력서: 이영희
내용: Java 개발자 경력 5년
```

**핵심 포인트:**
- 원본은 한 번만 생성하고, 나머지는 복제
- 복제 후 필요한 부분만 수정
- `new`로 생성하는 것보다 `copy()`가 훨씬 빠름

## 실생활 예제 - 게임 몬스터 복제 시스템

RPG 게임에서 다양한 몬스터를 효율적으로 생성하는 시스템을 프로토타입 패턴으로 구현해보겠습니다.

```java
import java.util.*;

// 게임 아이템 클래스
class Item implements Cloneable {
    private String name;
    private int value;
    private String type;

    public Item(String name, int value, String type) {
        this.name = name;
        this.value = value;
        this.type = type;
    }

    @Override
    public Item clone() {
        try {
            return (Item) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }

    // getter, setter 메서드들
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getValue() { return value; }
    public void setValue(int value) { this.value = value; }
    public String getType() { return type; }

    @Override
    public String toString() {
        return String.format("%s (%s, 가치: %d골드)", name, type, value);
    }
}

// 몬스터 능력치 클래스
class Stats implements Cloneable {
    private int health;
    private int attack;
    private int defense;
    private int speed;

    public Stats(int health, int attack, int defense, int speed) {
        this.health = health;
        this.attack = attack;
        this.defense = defense;
        this.speed = speed;
    }

    @Override
    public Stats clone() {
        try {
            return (Stats) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }

    // getter, setter 메서드들
    public int getHealth() { return health; }
    public void setHealth(int health) { this.health = health; }
    public int getAttack() { return attack; }
    public void setAttack(int attack) { this.attack = attack; }
    public int getDefense() { return defense; }
    public void setDefense(int defense) { this.defense = defense; }
    public int getSpeed() { return speed; }
    public void setSpeed(int speed) { this.speed = speed; }

    @Override
    public String toString() {
        return String.format("HP:%d, 공격:%d, 방어:%d, 속도:%d", health, attack, defense, speed);
    }
}

// 프로토타입 인터페이스
interface MonsterPrototype extends Cloneable {
    MonsterPrototype clone();
    void displayInfo();
    void levelUp(int levels);
    String getType();
}

// 기본 몬스터 클래스
abstract class BaseMonster implements MonsterPrototype {
    protected String name;
    protected String type;
    protected int level;
    protected Stats stats;
    protected List<Item> dropItems;
    protected String[] skills;
    protected Map<String, Integer> resistances;

    public BaseMonster(String name, String type, int level, Stats stats) {
        this.name = name;
        this.type = type;
        this.level = level;
        this.stats = stats;
        this.dropItems = new ArrayList<>();
        this.resistances = new HashMap<>();
    }

    @Override
    public BaseMonster clone() {
        try {
            BaseMonster cloned = (BaseMonster) super.clone();

            // 깊은 복사 수행
            cloned.stats = this.stats.clone();
            cloned.dropItems = new ArrayList<>();
            for (Item item : this.dropItems) {
                cloned.dropItems.add(item.clone());
            }
            cloned.skills = this.skills.clone();
            cloned.resistances = new HashMap<>(this.resistances);

            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public void levelUp(int levels) {
        this.level += levels;
        // 레벨업에 따른 능력치 증가
        stats.setHealth(stats.getHealth() + levels * 10);
        stats.setAttack(stats.getAttack() + levels * 3);
        stats.setDefense(stats.getDefense() + levels * 2);
        stats.setSpeed(stats.getSpeed() + levels * 1);

        System.out.println(name + "이(가) " + levels + "레벨 상승! (현재 레벨: " + this.level + ")");
    }

    @Override
    public String getType() {
        return type;
    }

    public void addDropItem(Item item) {
        dropItems.add(item);
    }

    public void setSkills(String[] skills) {
        this.skills = skills;
    }

    public void addResistance(String element, int value) {
        resistances.put(element, value);
    }

    @Override
    public void displayInfo() {
        System.out.println("=== " + name + " (레벨 " + level + ") ===");
        System.out.println("타입: " + type);
        System.out.println("능력치: " + stats);

        if (skills != null && skills.length > 0) {
            System.out.println("스킬: " + Arrays.toString(skills));
        }

        if (!resistances.isEmpty()) {
            System.out.println("저항력: " + resistances);
        }

        if (!dropItems.isEmpty()) {
            System.out.println("드롭 아이템:");
            for (Item item : dropItems) {
                System.out.println("  - " + item);
            }
        }
        System.out.println();
    }
}

// 구체적인 몬스터 클래스들
class Orc extends BaseMonster {
    public Orc() {
        super("오크", "휴머노이드", 5, new Stats(80, 25, 15, 10));

        // 오크 특성 설정
        addDropItem(new Item("오크의 도끼", 150, "무기"));
        addDropItem(new Item("가죽 갑옷", 100, "방어구"));
        setSkills(new String[]{"강타", "포효"});
        addResistance("물리", 10);
    }
}

class Dragon extends BaseMonster {
    public Dragon() {
        super("드래곤", "용족", 20, new Stats(300, 80, 60, 25));

        // 드래곤 특성 설정
        addDropItem(new Item("드래곤 비늘", 500, "재료"));
        addDropItem(new Item("드래곤 하트", 1000, "재료"));
        addDropItem(new Item("드래곤 소드", 2000, "무기"));
        setSkills(new String[]{"화염브레스", "비행", "꼬리치기", "용의 포효"});
        addResistance("화염", 90);
        addResistance("물리", 50);
        addResistance("마법", 30);
    }
}

class Goblin extends BaseMonster {
    public Goblin() {
        super("고블린", "휴머노이드", 2, new Stats(30, 15, 8, 20));

        // 고블린 특성 설정
        addDropItem(new Item("구리 동전", 5, "재화"));
        addDropItem(new Item("낡은 단검", 25, "무기"));
        setSkills(new String[]{"빠른 공격", "도주"});
        addResistance("독", 5);
    }
}

class Slime extends BaseMonster {
    public Slime() {
        super("슬라임", "젤리", 1, new Stats(40, 10, 5, 8));

        // 슬라임 특성 설정
        addDropItem(new Item("슬라임 젤리", 10, "재료"));
        setSkills(new String[]{"산성 공격"});
        addResistance("물리", 20);
        addResistance("독", 100);
    }
}

// 프로토타입 매니저 (레지스트리)
class MonsterPrototypeManager {
    private Map<String, MonsterPrototype> prototypes;

    public MonsterPrototypeManager() {
        prototypes = new HashMap<>();
        initializePrototypes();
    }

    private void initializePrototypes() {
        // 기본 프로토타입들 등록
        prototypes.put("orc", new Orc());
        prototypes.put("dragon", new Dragon());
        prototypes.put("goblin", new Goblin());
        prototypes.put("slime", new Slime());

        System.out.println("몬스터 프로토타입 매니저 초기화 완료!");
        System.out.println("등록된 몬스터 타입: " + prototypes.keySet());
        System.out.println();
    }

    public MonsterPrototype createMonster(String type) {
        MonsterPrototype prototype = prototypes.get(type.toLowerCase());
        if (prototype == null) {
            throw new IllegalArgumentException("알 수 없는 몬스터 타입: " + type);
        }
        return prototype.clone();
    }

    public MonsterPrototype createMonster(String type, int level) {
        MonsterPrototype monster = createMonster(type);
        if (level > 1) {
            monster.levelUp(level - 1); // 현재 레벨 - 1만큼 레벨업
        }
        return monster;
    }

    public void registerPrototype(String type, MonsterPrototype prototype) {
        prototypes.put(type.toLowerCase(), prototype);
        System.out.println("새로운 몬스터 프로토타입 등록: " + type);
    }

    public Set<String> getAvailableTypes() {
        return prototypes.keySet();
    }
}

// 게임 몬스터 스포너
class MonsterSpawner {
    private MonsterPrototypeManager prototypeManager;
    private Random random;

    public MonsterSpawner(MonsterPrototypeManager manager) {
        this.prototypeManager = manager;
        this.random = new Random();
    }

    public List<MonsterPrototype> spawnRandomMonsters(int count) {
        List<MonsterPrototype> monsters = new ArrayList<>();
        String[] types = {"goblin", "orc", "slime", "dragon"};

        for (int i = 0; i < count; i++) {
            String randomType = types[random.nextInt(types.length)];
            int randomLevel = random.nextInt(10) + 1; // 1~10 레벨

            MonsterPrototype monster = prototypeManager.createMonster(randomType, randomLevel);
            monsters.add(monster);
        }

        return monsters;
    }

    public MonsterPrototype spawnBoss(String type) {
        MonsterPrototype boss = prototypeManager.createMonster(type);
        boss.levelUp(15); // 보스는 기본 레벨에서 +15 레벨
        return boss;
    }
}

// 게임 데모 클래스
public class MonsterPrototypeDemo {
    public static void main(String[] args) {
        // 1. 프로토타입 매니저 초기화
        MonsterPrototypeManager manager = new MonsterPrototypeManager();
        MonsterSpawner spawner = new MonsterSpawner(manager);

        // 2. 기본 몬스터들 생성 테스트
        System.out.println("=== 기본 몬스터 생성 테스트 ===");
        MonsterPrototype basicOrc = manager.createMonster("orc");
        MonsterPrototype basicDragon = manager.createMonster("dragon");

        basicOrc.displayInfo();
        basicDragon.displayInfo();

        // 3. 레벨이 다른 같은 종류 몬스터 생성
        System.out.println("=== 다양한 레벨의 오크 생성 ===");
        MonsterPrototype lowLevelOrc = manager.createMonster("orc", 3);
        MonsterPrototype highLevelOrc = manager.createMonster("orc", 10);

        lowLevelOrc.displayInfo();
        highLevelOrc.displayInfo();

        // 4. 랜덤 몬스터 스포닝
        System.out.println("=== 랜덤 몬스터 스포닝 ===");
        List<MonsterPrototype> randomMonsters = spawner.spawnRandomMonsters(5);
        for (MonsterPrototype monster : randomMonsters) {
            monster.displayInfo();
        }

        // 5. 보스 몬스터 생성
        System.out.println("=== 보스 몬스터 등장 ===");
        MonsterPrototype dragonBoss = spawner.spawnBoss("dragon");
        dragonBoss.displayInfo();

        // 6. 성능 테스트 - 많은 몬스터 생성
        System.out.println("=== 성능 테스트: 1000마리 몬스터 생성 ===");
        long startTime = System.currentTimeMillis();

        List<MonsterPrototype> army = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            army.add(manager.createMonster("goblin"));
        }

        long endTime = System.currentTimeMillis();
        System.out.println("1000마리 고블린 생성 완료!");
        System.out.println("소요 시간: " + (endTime - startTime) + "ms");
        System.out.println("생성된 몬스터 수: " + army.size());
    }
}
```

**실행 결과 예시:**
```
몬스터 프로토타입 매니저 초기화 완료!
등록된 몬스터 타입: [goblin, slime, orc, dragon]

=== 기본 몬스터 생성 테스트 ===
=== 오크 (레벨 5) ===
타입: 휴머노이드
능력치: HP:80, 공격:25, 방어:15, 속도:10
스킬: [강타, 포효]
저항력: {물리=10}
드롭 아이템:
  - 오크의 도끼 (무기, 가치: 150골드)
  - 가죽 갑옷 (방어구, 가치: 100골드)

=== 드래곤 (레벨 20) ===
타입: 용족
능력치: HP:300, 공격:80, 방어:60, 속도:25
스킬: [화염브레스, 비행, 꼬리치기, 용의 포효]
저항력: {화염=90, 물리=50, 마법=30}
드롭 아이템:
  - 드래곤 비늘 (재료, 가치: 500골드)
  - 드래곤 하트 (재료, 가치: 1000골드)
  - 드래곤 소드 (무기, 가치: 2000골드)

=== 다양한 레벨의 오크 생성 ===
오크이(가) 2레벨 상승! (현재 레벨: 3)
=== 오크 (레벨 3) ===
타입: 휴머노이드
능력치: HP:100, 공격:31, 방어:19, 속도:12

오크이(가) 5레벨 상승! (현재 레벨: 10)
=== 오크 (레벨 10) ===
타입: 휴머노이드
능력치: HP:130, 공격:40, 방어:25, 속도:15
```

## 직렬화를 이용한 깊은 복사

복잡한 객체의 깊은 복사를 구현하는 다른 방법으로 직렬화를 사용할 수 있습니다.

```java
import java.io.*;

class SerializableMonster implements Serializable, Cloneable {
    private String name;
    private List<Item> items;

    // 직렬화를 이용한 깊은 복사
    public SerializableMonster deepClone() {
        try {
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(baos);
            oos.writeObject(this);

            ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bais);

            return (SerializableMonster) ois.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## 기본 예제 코드 (Java)

```java
// 프로토타입 인터페이스
interface Prototype extends Cloneable {
    Prototype clone();
}

// 구체적인 프로토타입 구현
class ConcretePrototype implements Prototype {
    private String field;

    public ConcretePrototype(String field) {
        this.field = field;
    }

    @Override
    public ConcretePrototype clone() {
        try {
            return (ConcretePrototype) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }

    public String getField() {
        return field;
    }

    public void setField(String field) {
        this.field = field;
    }
}

// 클라이언트
public class Client {
    public static void main(String[] args) {
        ConcretePrototype prototype = new ConcretePrototype("Original");
        ConcretePrototype copy = prototype.clone();

        System.out.println("Original: " + prototype.getField());
        System.out.println("Copy: " + copy.getField());

        copy.setField("Modified");
        System.out.println("After modification:");
        System.out.println("Original: " + prototype.getField());
        System.out.println("Copy: " + copy.getField());
    }
}
```

## Spring Boot에서의 Prototype 패턴

### 1. @Scope("prototype") - Spring의 프로토타입 스코프

```java
// Prototype 스코프 빈 정의
@Component
@Scope("prototype")
public class ShoppingCart {
    private List<CartItem> items = new ArrayList<>();
    private LocalDateTime createdAt;

    public ShoppingCart() {
        this.createdAt = LocalDateTime.now();
        System.out.println("🛒 새 장바구니 생성: " + createdAt);
    }

    public void addItem(CartItem item) {
        items.add(item);
    }

    public List<CartItem> getItems() {
        return items;
    }
}

// 서비스에서 사용
@Service
@RequiredArgsConstructor
public class OrderService {
    private final ApplicationContext context;

    public ShoppingCart createNewCart() {
        // 호출할 때마다 새 인스턴스 생성
        return context.getBean(ShoppingCart.class);
    }
}
```

### 2. BeanUtils를 이용한 객체 복제

```java
@Entity
@Getter @Setter
public class Product {
    @Id @GeneratedValue
    private Long id;
    private String name;
    private BigDecimal price;
    private String description;
    private String category;
}

@Service
@RequiredArgsConstructor
public class ProductService {
    private final ProductRepository productRepository;

    // 기존 상품을 복제하여 새 상품 생성
    public Product copyProduct(Long originalId, String newName) {
        Product original = productRepository.findById(originalId)
            .orElseThrow(() -> new EntityNotFoundException("상품 없음"));

        Product copy = new Product();
        // 모든 속성 복사 (id 제외)
        BeanUtils.copyProperties(original, copy, "id");
        copy.setName(newName);

        return productRepository.save(copy);
    }
}
```

### 3. 프로토타입 레지스트리 패턴

```java
// 프로토타입 인터페이스
public interface ReportPrototype extends Cloneable {
    ReportPrototype copy();
    void generate();
    String getType();
}

// 구체적인 프로토타입들
@Component("sales")
public class SalesReport implements ReportPrototype {
    private String title = "매출 보고서";
    private LocalDate reportDate;
    private Map<String, Object> data = new HashMap<>();

    @Override
    public ReportPrototype copy() {
        SalesReport copy = new SalesReport();
        copy.title = this.title;
        copy.reportDate = LocalDate.now();
        copy.data = new HashMap<>(this.data);  // 깊은 복사
        return copy;
    }

    @Override
    public String getType() { return "sales"; }

    @Override
    public void generate() {
        System.out.println("📊 " + title + " 생성: " + reportDate);
    }
}

@Component("inventory")
public class InventoryReport implements ReportPrototype {
    private String title = "재고 보고서";
    private LocalDate reportDate;

    @Override
    public ReportPrototype copy() {
        InventoryReport copy = new InventoryReport();
        copy.title = this.title;
        copy.reportDate = LocalDate.now();
        return copy;
    }

    @Override
    public String getType() { return "inventory"; }

    @Override
    public void generate() {
        System.out.println("📦 " + title + " 생성: " + reportDate);
    }
}

// 프로토타입 레지스트리
@Component
@RequiredArgsConstructor
public class ReportRegistry {
    // Spring이 자동으로 모든 ReportPrototype 빈을 Map으로 주입
    private final Map<String, ReportPrototype> prototypes;

    public ReportPrototype createReport(String type) {
        ReportPrototype prototype = prototypes.get(type);
        if (prototype == null) {
            throw new IllegalArgumentException("Unknown report type: " + type);
        }
        return prototype.copy();
    }

    public Set<String> getAvailableTypes() {
        return prototypes.keySet();
    }
}

// 컨트롤러
@RestController
@RequestMapping("/api/reports")
@RequiredArgsConstructor
public class ReportController {
    private final ReportRegistry registry;

    @PostMapping("/{type}")
    public ResponseEntity<String> generateReport(@PathVariable String type) {
        ReportPrototype report = registry.createReport(type);
        report.generate();
        return ResponseEntity.ok("Report generated: " + type);
    }

    @GetMapping("/types")
    public Set<String> getReportTypes() {
        return registry.getAvailableTypes();
    }
}
```

### 4. ModelMapper를 이용한 DTO 복제

```java
@Configuration
public class ModelMapperConfig {
    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}

@Service
@RequiredArgsConstructor
public class UserService {
    private final ModelMapper modelMapper;
    private final UserRepository userRepository;

    // Entity → DTO 변환 (일종의 복제)
    public UserDTO copyToDto(User user) {
        return modelMapper.map(user, UserDTO.class);
    }

    // DTO → 새 Entity 생성 (복제 후 수정)
    public User createFromTemplate(UserDTO template, String newEmail) {
        User newUser = modelMapper.map(template, User.class);
        newUser.setId(null);  // 새 ID 발급을 위해
        newUser.setEmail(newEmail);
        return userRepository.save(newUser);
    }
}
```

### 5. 프로토타입 빈과 싱글톤 빈 함께 사용

```java
// 프로토타입 스코프 (요청마다 새 인스턴스)
@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestContext {
    private final String requestId = UUID.randomUUID().toString();
    private LocalDateTime timestamp = LocalDateTime.now();

    public String getRequestId() { return requestId; }
}

// 싱글톤 서비스에서 프로토타입 빈 사용
@Service
@RequiredArgsConstructor
public class AuditService {
    // proxyMode 덕분에 매번 새 인스턴스 획득
    private final RequestContext requestContext;

    public void log(String action) {
        System.out.println("[" + requestContext.getRequestId() + "] " + action);
    }
}
```

## 장점

- **성능 향상**: 복잡한 객체의 초기화 비용을 줄일 수 있습니다.
- **동적 객체 생성**: 런타임에 객체 타입을 결정하여 생성할 수 있습니다.
- **서브클래스 수 감소**: 팩토리 클래스 계층을 만들지 않아도 됩니다.
- **설정된 객체 복제**: 이미 설정이 완료된 복잡한 객체를 쉽게 복제할 수 있습니다.
- **객체 생성 과정 숨김**: 클라이언트는 복제 과정의 세부사항을 알 필요가 없습니다.

## 단점

- **복잡한 복제 로직**: 순환 참조가 있는 복잡한 객체의 깊은 복사는 구현이 어렵습니다.
- **clone() 메서드 오버라이드**: 모든 클래스에서 clone() 메서드를 구현해야 합니다.
- **얕은 복사 함정**: 기본 clone()은 얕은 복사를 수행하므로 참조 객체 문제가 발생할 수 있습니다.
- **상속 관계에서의 복잡성**: 상속 관계가 복잡할 때 clone() 구현이 어려워집니다.

## 관련 패턴

| 패턴 | 관계 | 비교 |
|------|------|------|
| **Factory Method** | 대안 | Factory는 클래스 기반 생성, Prototype은 인스턴스 기반 생성 |
| **Abstract Factory** | 조합 | Abstract Factory가 Prototype으로 제품을 생성할 수 있음 |
| **Singleton** | 반대 | Singleton은 하나만, Prototype은 여러 복제본 생성 |
| **Composite** | 조합 | 복잡한 트리 구조를 Prototype으로 복제 가능 |
| **Decorator** | 조합 | 장식된 객체를 Prototype으로 복제하여 재사용 |

### Factory Method vs Prototype

```java
// Factory Method: 클래스를 통해 생성
Monster monster = monsterFactory.create("orc");  // new Orc() 호출

// Prototype: 인스턴스를 복제하여 생성
Monster monster = orcPrototype.clone();  // 기존 객체 복제
```

**선택 기준:**
- 객체 생성 비용이 높음 → **Prototype**
- 객체 종류가 런타임에 결정됨 → **Prototype**
- 객체 생성 로직이 복잡함 → **Factory Method**
- 객체 타입별 다른 생성 로직 필요 → **Factory Method**