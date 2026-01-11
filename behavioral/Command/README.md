# 커맨드 패턴 (Command Pattern)

## 정의

커맨드 패턴은 요청을 객체로 캡슐화하여 다른 객체들을 서로 다른 요청, 큐, 로그 요청으로 매개변수화할 수 있게 해주는 행동 디자인 패턴입니다. 또한 실행 취소(undo) 기능을 지원할 수 있습니다.

## 🎯 한눈에 보기

| 항목 | 설명 |
|------|------|
| **핵심** | 요청(행동)을 객체로 만들어 저장·취소·재실행 가능하게 |
| **비유** | 식당에서 주문서를 작성하는 것 - 주문서(Command)가 주방(Receiver)에 전달되어 요리 실행 |
| **언제** | 실행 취소(Undo), 작업 큐, 매크로 기능이 필요할 때 |
| **Spring** | `@Async`, `@Scheduled`, `TransactionTemplate`, 이벤트 기반 처리 |

### 핵심 구성요소
```
Client       → 명령 객체를 생성하고 Invoker에 전달
Command      → 실행할 작업을 캡슐화한 인터페이스 (execute, undo)
Invoker      → 명령을 저장하고 실행을 요청 (리모컨, 버튼)
Receiver     → 실제 작업을 수행하는 객체 (조명, 에어컨)
```

## 구조 (Structure)

```mermaid
classDiagram
    class Client {
        +main(): void
    }

    class Invoker {
        -command: Command
        +setCommand(Command): void
        +executeCommand(): void
    }

    class Command {
        <<interface>>
        +execute(): void
        +undo(): void
    }

    class ConcreteCommand {
        -receiver: Receiver
        -state: Object
        +execute(): void
        +undo(): void
    }

    class Receiver {
        +action(): void
        +getState(): Object
        +setState(Object): void
    }

    Client --> Invoker : creates
    Client --> ConcreteCommand : creates
    Client --> Receiver : creates
    Invoker --> Command : uses
    Command <|.. ConcreteCommand
    ConcreteCommand --> Receiver : calls action

    note for Command "명령을 캡슐화하는 인터페이스"
    note for Invoker "명령을 실행하는 객체"
    note for Receiver "실제 작업을 수행하는 객체"
```

## 동작 흐름 (Sequence Diagram)

```mermaid
sequenceDiagram
    participant Client as 👤 Client
    participant Invoker as 📱 Invoker (리모컨)
    participant Command as 📋 Command
    participant Receiver as 💡 Receiver (조명)

    Note over Client,Receiver: 1. 명령 설정 단계
    Client->>Command: new LightOnCommand(light)
    Client->>Invoker: setCommand(lightOnCommand)

    Note over Client,Receiver: 2. 명령 실행 단계
    Client->>Invoker: pressButton()
    Invoker->>Command: execute()
    Command->>Receiver: turnOn()
    Receiver-->>Command: 조명 켜짐
    Command-->>Invoker: 실행 완료
    Invoker->>Invoker: lastCommand = command

    Note over Client,Receiver: 3. 실행 취소 단계
    Client->>Invoker: pressUndoButton()
    Invoker->>Command: undo()
    Command->>Receiver: turnOff()
    Receiver-->>Command: 조명 꺼짐
```

## 사용 이유

- **요청 캡슐화**: 요청자와 수신자를 분리하여 요청을 객체로 캡슐화합니다.
- **실행 취소 기능**: 명령의 이전 상태를 저장하여 undo/redo 기능을 구현할 수 있습니다.
- **지연 실행**: 명령을 나중에 실행하거나 원격에서 실행할 수 있습니다.
- **로깅**: 명령들을 기록하여 시스템 복구나 디버깅에 활용할 수 있습니다.
- **매크로 기능**: 여러 명령을 조합하여 복합 명령을 만들 수 있습니다.

## 적용 상황

커맨드 패턴은 다음과 같은 상황에서 특히 유용합니다:

### 1. GUI 애플리케이션
- **메뉴/버튼 액션**: 동일한 기능을 다른 UI 요소에서 실행
- **실행 취소/재실행**: 텍스트 편집기, 그래픽 툴
- **매크로 기능**: 일련의 작업을 자동화

### 2. 요청 큐잉 및 스케줄링
- **작업 큐**: 백그라운드에서 처리할 작업들
- **지연 실행**: 특정 시간에 실행되는 작업
- **원격 실행**: 네트워크를 통한 원격 명령 실행

### 3. 트랜잭션 시스템
```java
// 나쁜 예: 직접 호출로 롤백 어려움
class BankService {
    public void transfer(Account from, Account to, double amount) {
        from.withdraw(amount);  // 실패 시 롤백 복잡
        to.deposit(amount);     // 실패 시 앞의 작업 롤백 필요
    }
}

// 좋은 예: 커맨드 패턴으로 트랜잭션 관리
interface BankCommand {
    void execute();
    void undo();
}

class TransferCommand implements BankCommand {
    // 트랜잭션 단위로 실행/롤백 가능
}
```

## 초급 예제 - 텍스트 편집기 Undo (5분 이해)

가장 친숙한 Undo 기능을 커맨드 패턴으로 구현합니다.

```java
import java.util.Stack;

// 1. Command 인터페이스 - 모든 명령의 공통 규약
interface TextCommand {
    void execute();  // 실행
    void undo();     // 취소
}

// 2. Receiver - 실제 작업을 수행하는 객체
class TextEditor {
    private StringBuilder text = new StringBuilder();

    public void insert(String str) {
        text.append(str);
        System.out.println("입력: \"" + str + "\" → 현재 텍스트: \"" + text + "\"");
    }

    public void delete(int length) {
        if (length <= text.length()) {
            String deleted = text.substring(text.length() - length);
            text.delete(text.length() - length, text.length());
            System.out.println("삭제: \"" + deleted + "\" → 현재 텍스트: \"" + text + "\"");
        }
    }

    public String getText() { return text.toString(); }
}

// 3. Concrete Command - 텍스트 입력 명령
class InsertCommand implements TextCommand {
    private TextEditor editor;
    private String textToInsert;

    public InsertCommand(TextEditor editor, String text) {
        this.editor = editor;
        this.textToInsert = text;
    }

    @Override
    public void execute() {
        editor.insert(textToInsert);
    }

    @Override
    public void undo() {
        editor.delete(textToInsert.length());
    }
}

// 4. Invoker - 명령 실행 및 히스토리 관리
class CommandManager {
    private Stack<TextCommand> history = new Stack<>();

    public void executeCommand(TextCommand command) {
        command.execute();
        history.push(command);  // 히스토리에 저장
    }

    public void undo() {
        if (!history.isEmpty()) {
            System.out.print("↩️ Undo: ");
            history.pop().undo();
        } else {
            System.out.println("취소할 작업이 없습니다.");
        }
    }
}

// 5. 사용 예시
public class TextEditorDemo {
    public static void main(String[] args) {
        TextEditor editor = new TextEditor();
        CommandManager manager = new CommandManager();

        // 텍스트 입력
        manager.executeCommand(new InsertCommand(editor, "Hello"));
        manager.executeCommand(new InsertCommand(editor, " World"));
        manager.executeCommand(new InsertCommand(editor, "!"));

        // Undo 실행
        manager.undo();  // "!" 삭제
        manager.undo();  // " World" 삭제

        System.out.println("최종 결과: \"" + editor.getText() + "\"");
    }
}
```

**실행 결과:**
```
입력: "Hello" → 현재 텍스트: "Hello"
입력: " World" → 현재 텍스트: "Hello World"
입력: "!" → 현재 텍스트: "Hello World!"
↩️ Undo: 삭제: "!" → 현재 텍스트: "Hello World"
↩️ Undo: 삭제: " World" → 현재 텍스트: "Hello"
최종 결과: "Hello"
```

### 핵심 포인트
```
1. Command = 실행할 작업 + 취소할 방법을 하나의 객체로 묶음
2. 히스토리 = Stack에 명령을 쌓아서 Undo 구현
3. Invoker = 명령 실행만 담당, 무슨 작업인지는 몰라도 됨
```

## 실생활 예제 - 스마트 홈 제어 시스템

다양한 IoT 기기들을 제어하는 스마트 홈 시스템을 커맨드 패턴으로 구현해보겠습니다.

```java
import java.util.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

// 명령 인터페이스
interface SmartHomeCommand {
    void execute();
    void undo();
    String getDescription();
    LocalDateTime getExecutedTime();
    boolean isUndoable();
}

// 추상 명령 클래스 (공통 기능 구현)
abstract class AbstractSmartHomeCommand implements SmartHomeCommand {
    protected String description;
    protected LocalDateTime executedTime;
    protected boolean undoable;

    public AbstractSmartHomeCommand(String description) {
        this.description = description;
        this.undoable = true;
    }

    @Override
    public String getDescription() {
        return description;
    }

    @Override
    public LocalDateTime getExecutedTime() {
        return executedTime;
    }

    @Override
    public boolean isUndoable() {
        return undoable;
    }

    protected void markExecuted() {
        this.executedTime = LocalDateTime.now();
    }
}

// 조명 시스템 (Receiver)
class SmartLight {
    private String location;
    private boolean isOn;
    private int brightness; // 0-100
    private String color;

    public SmartLight(String location) {
        this.location = location;
        this.isOn = false;
        this.brightness = 100;
        this.color = "White";
    }

    public void turnOn() {
        isOn = true;
        System.out.println("💡 " + location + " 조명이 켜졌습니다.");
    }

    public void turnOff() {
        isOn = false;
        System.out.println("💡 " + location + " 조명이 꺼졌습니다.");
    }

    public void setBrightness(int brightness) {
        this.brightness = Math.max(0, Math.min(100, brightness));
        System.out.println("💡 " + location + " 조명 밝기: " + this.brightness + "%");
    }

    public void setColor(String color) {
        this.color = color;
        System.out.println("💡 " + location + " 조명 색상: " + color);
    }

    // getter 메서드들
    public String getLocation() { return location; }
    public boolean isOn() { return isOn; }
    public int getBrightness() { return brightness; }
    public String getColor() { return color; }
}

// 에어컨 시스템 (Receiver)
class SmartAirConditioner {
    private String location;
    private boolean isOn;
    private int temperature; // 16-30도
    private String mode; // "COOL", "HEAT", "FAN", "DRY"

    public SmartAirConditioner(String location) {
        this.location = location;
        this.isOn = false;
        this.temperature = 24;
        this.mode = "COOL";
    }

    public void turnOn() {
        isOn = true;
        System.out.println("❄️ " + location + " 에어컨이 켜졌습니다. (모드: " + mode + ", 온도: " + temperature + "°C)");
    }

    public void turnOff() {
        isOn = false;
        System.out.println("❄️ " + location + " 에어컨이 꺼졌습니다.");
    }

    public void setTemperature(int temperature) {
        this.temperature = Math.max(16, Math.min(30, temperature));
        System.out.println("❄️ " + location + " 에어컨 온도: " + this.temperature + "°C");
    }

    public void setMode(String mode) {
        this.mode = mode;
        System.out.println("❄️ " + location + " 에어컨 모드: " + mode);
    }

    // getter 메서드들
    public String getLocation() { return location; }
    public boolean isOn() { return isOn; }
    public int getTemperature() { return temperature; }
    public String getMode() { return mode; }
}

// 스피커 시스템 (Receiver)
class SmartSpeaker {
    private String location;
    private boolean isOn;
    private int volume; // 0-100
    private String currentSong;

    public SmartSpeaker(String location) {
        this.location = location;
        this.isOn = false;
        this.volume = 50;
        this.currentSong = "";
    }

    public void turnOn() {
        isOn = true;
        System.out.println("🔊 " + location + " 스피커가 켜졌습니다.");
    }

    public void turnOff() {
        isOn = false;
        System.out.println("🔊 " + location + " 스피커가 꺼졌습니다.");
    }

    public void setVolume(int volume) {
        this.volume = Math.max(0, Math.min(100, volume));
        System.out.println("🔊 " + location + " 스피커 볼륨: " + this.volume);
    }

    public void playMusic(String song) {
        this.currentSong = song;
        System.out.println("🎵 " + location + " 스피커에서 '" + song + "' 재생 중");
    }

    public void stopMusic() {
        this.currentSong = "";
        System.out.println("⏹️ " + location + " 스피커 음악 정지");
    }

    // getter 메서드들
    public String getLocation() { return location; }
    public boolean isOn() { return isOn; }
    public int getVolume() { return volume; }
    public String getCurrentSong() { return currentSong; }
}

// 조명 제어 명령들
class LightOnCommand extends AbstractSmartHomeCommand {
    private SmartLight light;
    private boolean previousState;

    public LightOnCommand(SmartLight light) {
        super("조명 켜기 - " + light.getLocation());
        this.light = light;
    }

    @Override
    public void execute() {
        previousState = light.isOn();
        light.turnOn();
        markExecuted();
    }

    @Override
    public void undo() {
        if (!previousState) {
            light.turnOff();
            System.out.println("↩️ 조명 켜기 명령 취소");
        }
    }
}

class LightOffCommand extends AbstractSmartHomeCommand {
    private SmartLight light;
    private boolean previousState;

    public LightOffCommand(SmartLight light) {
        super("조명 끄기 - " + light.getLocation());
        this.light = light;
    }

    @Override
    public void execute() {
        previousState = light.isOn();
        light.turnOff();
        markExecuted();
    }

    @Override
    public void undo() {
        if (previousState) {
            light.turnOn();
            System.out.println("↩️ 조명 끄기 명령 취소");
        }
    }
}

class LightBrightnessCommand extends AbstractSmartHomeCommand {
    private SmartLight light;
    private int newBrightness;
    private int previousBrightness;

    public LightBrightnessCommand(SmartLight light, int brightness) {
        super("조명 밝기 조절 - " + light.getLocation() + " (" + brightness + "%)");
        this.light = light;
        this.newBrightness = brightness;
    }

    @Override
    public void execute() {
        previousBrightness = light.getBrightness();
        light.setBrightness(newBrightness);
        markExecuted();
    }

    @Override
    public void undo() {
        light.setBrightness(previousBrightness);
        System.out.println("↩️ 조명 밝기 조절 명령 취소 (이전: " + previousBrightness + "%)");
    }
}

// 에어컨 제어 명령들
class AirConditionerOnCommand extends AbstractSmartHomeCommand {
    private SmartAirConditioner ac;
    private boolean previousState;

    public AirConditionerOnCommand(SmartAirConditioner ac) {
        super("에어컨 켜기 - " + ac.getLocation());
        this.ac = ac;
    }

    @Override
    public void execute() {
        previousState = ac.isOn();
        ac.turnOn();
        markExecuted();
    }

    @Override
    public void undo() {
        if (!previousState) {
            ac.turnOff();
            System.out.println("↩️ 에어컨 켜기 명령 취소");
        }
    }
}

class AirConditionerTemperatureCommand extends AbstractSmartHomeCommand {
    private SmartAirConditioner ac;
    private int newTemperature;
    private int previousTemperature;

    public AirConditionerTemperatureCommand(SmartAirConditioner ac, int temperature) {
        super("에어컨 온도 조절 - " + ac.getLocation() + " (" + temperature + "°C)");
        this.ac = ac;
        this.newTemperature = temperature;
    }

    @Override
    public void execute() {
        previousTemperature = ac.getTemperature();
        ac.setTemperature(newTemperature);
        markExecuted();
    }

    @Override
    public void undo() {
        ac.setTemperature(previousTemperature);
        System.out.println("↩️ 에어컨 온도 조절 명령 취소 (이전: " + previousTemperature + "°C)");
    }
}

// 스피커 제어 명령들
class SpeakerPlayCommand extends AbstractSmartHomeCommand {
    private SmartSpeaker speaker;
    private String song;
    private String previousSong;
    private boolean wasPlaying;

    public SpeakerPlayCommand(SmartSpeaker speaker, String song) {
        super("음악 재생 - " + speaker.getLocation() + " ('" + song + "')");
        this.speaker = speaker;
        this.song = song;
    }

    @Override
    public void execute() {
        previousSong = speaker.getCurrentSong();
        wasPlaying = !previousSong.isEmpty();

        if (!speaker.isOn()) {
            speaker.turnOn();
        }
        speaker.playMusic(song);
        markExecuted();
    }

    @Override
    public void undo() {
        if (wasPlaying) {
            speaker.playMusic(previousSong);
        } else {
            speaker.stopMusic();
        }
        System.out.println("↩️ 음악 재생 명령 취소");
    }
}

// 매크로 명령 (복합 명령)
class MacroCommand extends AbstractSmartHomeCommand {
    private List<SmartHomeCommand> commands;

    public MacroCommand(String description, List<SmartHomeCommand> commands) {
        super("매크로: " + description);
        this.commands = new ArrayList<>(commands);
    }

    @Override
    public void execute() {
        System.out.println("🎯 매크로 실행 시작: " + description);
        for (SmartHomeCommand command : commands) {
            command.execute();
        }
        markExecuted();
        System.out.println("✅ 매크로 실행 완료");
    }

    @Override
    public void undo() {
        System.out.println("↩️ 매크로 실행 취소 시작: " + description);
        // 역순으로 undo 실행
        for (int i = commands.size() - 1; i >= 0; i--) {
            if (commands.get(i).isUndoable()) {
                commands.get(i).undo();
            }
        }
        System.out.println("✅ 매크로 실행 취소 완료");
    }
}

// 널 명령 (아무것도 하지 않는 명령)
class NoCommand extends AbstractSmartHomeCommand {
    public NoCommand() {
        super("빈 명령");
        this.undoable = false;
    }

    @Override
    public void execute() {
        // 아무것도 하지 않음
    }

    @Override
    public void undo() {
        // 아무것도 하지 않음
    }
}

// 스마트 홈 리모컨 (Invoker)
class SmartHomeRemote {
    private SmartHomeCommand[] onCommands;
    private SmartHomeCommand[] offCommands;
    private Stack<SmartHomeCommand> commandHistory;
    private SmartHomeCommand lastCommand;

    public SmartHomeRemote() {
        onCommands = new SmartHomeCommand[7];  // 7개 슬롯
        offCommands = new SmartHomeCommand[7];
        commandHistory = new Stack<>();

        SmartHomeCommand noCommand = new NoCommand();
        for (int i = 0; i < 7; i++) {
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
        lastCommand = noCommand;
    }

    public void setCommand(int slot, SmartHomeCommand onCommand, SmartHomeCommand offCommand) {
        if (slot >= 0 && slot < 7) {
            onCommands[slot] = onCommand;
            offCommands[slot] = offCommand;
        }
    }

    public void onButtonPressed(int slot) {
        if (slot >= 0 && slot < 7) {
            onCommands[slot].execute();
            commandHistory.push(onCommands[slot]);
            lastCommand = onCommands[slot];
        }
    }

    public void offButtonPressed(int slot) {
        if (slot >= 0 && slot < 7) {
            offCommands[slot].execute();
            commandHistory.push(offCommands[slot]);
            lastCommand = offCommands[slot];
        }
    }

    public void undoButtonPressed() {
        if (lastCommand.isUndoable()) {
            lastCommand.undo();
        } else {
            System.out.println("❌ 실행 취소할 수 없는 명령입니다.");
        }
    }

    public void executeCommand(SmartHomeCommand command) {
        command.execute();
        commandHistory.push(command);
        lastCommand = command;
    }

    public void displayStatus() {
        System.out.println("\n📱 스마트 홈 리모컨 상태");
        System.out.println("=".repeat(40));
        for (int i = 0; i < 7; i++) {
            System.out.println("슬롯 " + (i + 1) + ": " + onCommands[i].getDescription()
                             + " / " + offCommands[i].getDescription());
        }
        System.out.println("마지막 명령: " + lastCommand.getDescription());
        System.out.println("명령 기록 수: " + commandHistory.size());
    }

    public void displayHistory() {
        System.out.println("\n📜 명령 실행 기록");
        System.out.println("=".repeat(40));
        if (commandHistory.isEmpty()) {
            System.out.println("실행된 명령이 없습니다.");
            return;
        }

        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("HH:mm:ss");
        for (int i = Math.max(0, commandHistory.size() - 10); i < commandHistory.size(); i++) {
            SmartHomeCommand cmd = commandHistory.get(i);
            String time = cmd.getExecutedTime() != null ?
                cmd.getExecutedTime().format(formatter) : "N/A";
            System.out.println((i + 1) + ". [" + time + "] " + cmd.getDescription());
        }
    }
}

// 스마트 홈 제어 시스템 데모
public class SmartHomeSystemDemo {
    public static void main(String[] args) throws InterruptedException {
        // 1. 스마트 기기들 생성
        SmartLight livingRoomLight = new SmartLight("거실");
        SmartLight bedroomLight = new SmartLight("침실");
        SmartAirConditioner livingRoomAC = new SmartAirConditioner("거실");
        SmartSpeaker kitchenSpeaker = new SmartSpeaker("주방");

        // 2. 리모컨 생성
        SmartHomeRemote remote = new SmartHomeRemote();

        System.out.println("🏠 스마트 홈 제어 시스템 시작");
        System.out.println("=".repeat(50));

        // 3. 명령들 생성 및 리모컨에 설정
        SmartHomeCommand livingRoomLightOn = new LightOnCommand(livingRoomLight);
        SmartHomeCommand livingRoomLightOff = new LightOffCommand(livingRoomLight);
        SmartHomeCommand bedroomLightOn = new LightOnCommand(bedroomLight);
        SmartHomeCommand bedroomLightOff = new LightOffCommand(bedroomLight);
        SmartHomeCommand acOn = new AirConditionerOnCommand(livingRoomAC);
        SmartHomeCommand acOff = new LightOffCommand(livingRoomLight); // 예시로 다른 명령 설정

        remote.setCommand(0, livingRoomLightOn, livingRoomLightOff);
        remote.setCommand(1, bedroomLightOn, bedroomLightOff);
        remote.setCommand(2, acOn, acOff);

        // 4. 기본 명령 실행 테스트
        System.out.println("📱 기본 명령 실행 테스트");
        remote.onButtonPressed(0);  // 거실 조명 켜기
        Thread.sleep(1000);

        remote.onButtonPressed(1);  // 침실 조명 켜기
        Thread.sleep(1000);

        remote.onButtonPressed(2);  // 에어컨 켜기
        Thread.sleep(1000);

        // 5. 실행 취소 테스트
        System.out.println("\n↩️ 실행 취소 테스트");
        remote.undoButtonPressed();  // 마지막 명령 취소
        Thread.sleep(1000);

        // 6. 복잡한 명령들 직접 실행
        System.out.println("\n🎛️ 고급 제어 테스트");
        remote.executeCommand(new LightBrightnessCommand(livingRoomLight, 30));
        Thread.sleep(1000);

        remote.executeCommand(new AirConditionerTemperatureCommand(livingRoomAC, 22));
        Thread.sleep(1000);

        remote.executeCommand(new SpeakerPlayCommand(kitchenSpeaker, "좋은 아침 플레이리스트"));
        Thread.sleep(1000);

        // 7. 매크로 명령 테스트 - "영화 모드"
        System.out.println("\n🎬 매크로 명령 테스트 - '영화 모드'");
        List<SmartHomeCommand> movieModeCommands = Arrays.asList(
            new LightOffCommand(livingRoomLight),
            new LightBrightnessCommand(bedroomLight, 10),
            new AirConditionerTemperatureCommand(livingRoomAC, 20),
            new SpeakerPlayCommand(kitchenSpeaker, "영화 OST 모음")
        );

        MacroCommand movieMode = new MacroCommand("영화 모드", movieModeCommands);
        remote.executeCommand(movieMode);
        Thread.sleep(2000);

        // 8. 매크로 실행 취소
        System.out.println("\n↩️ 매크로 실행 취소 테스트");
        remote.undoButtonPressed();
        Thread.sleep(1000);

        // 9. "취침 모드" 매크로
        System.out.println("\n🌙 매크로 명령 테스트 - '취침 모드'");
        List<SmartHomeCommand> sleepModeCommands = Arrays.asList(
            new LightOffCommand(livingRoomLight),
            new LightOffCommand(bedroomLight),
            new AirConditionerTemperatureCommand(livingRoomAC, 26),
            new SpeakerPlayCommand(kitchenSpeaker, "수면 음악")
        );

        MacroCommand sleepMode = new MacroCommand("취침 모드", sleepModeCommands);
        remote.executeCommand(sleepMode);

        // 10. 상태 및 기록 확인
        remote.displayStatus();
        remote.displayHistory();

        System.out.println("\n🎯 스마트 홈 제어 시스템 데모 완료!");
    }
}
```

**실행 결과 예시:**
```
🏠 스마트 홈 제어 시스템 시작
==================================================
📱 기본 명령 실행 테스트
💡 거실 조명이 켜졌습니다.
💡 침실 조명이 켜졌습니다.
❄️ 거실 에어컨이 켜졌습니다. (모드: COOL, 온도: 24°C)

↩️ 실행 취소 테스트
❄️ 거실 에어컨이 꺼졌습니다.
↩️ 에어컨 켜기 명령 취소

🎛️ 고급 제어 테스트
💡 거실 조명 밝기: 30%
❄️ 거실 에어컨 온도: 22°C
🔊 주방 스피커가 켜졌습니다.
🎵 주방 스피커에서 '좋은 아침 플레이리스트' 재생 중

🎬 매크로 명령 테스트 - '영화 모드'
🎯 매크로 실행 시작: 매크로: 영화 모드
💡 거실 조명이 꺼졌습니다.
💡 침실 조명 밝기: 10%
❄️ 거실 에어컨 온도: 20°C
🎵 주방 스피커에서 '영화 OST 모음' 재생 중
✅ 매크로 실행 완료
```

## 기본 예제 코드 (Java)

```java
// Command Interface
interface Command {
    void execute();
    void undo();
}

// Receiver
class Light {
    private boolean isOn = false;

    public void turnOn() {
        isOn = true;
        System.out.println("조명이 켜졌습니다.");
    }

    public void turnOff() {
        isOn = false;
        System.out.println("조명이 꺼졌습니다.");
    }

    public boolean isOn() {
        return isOn;
    }
}

// Concrete Commands
class LightOnCommand implements Command {
    private Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.turnOn();
    }

    @Override
    public void undo() {
        light.turnOff();
    }
}

class LightOffCommand implements Command {
    private Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.turnOff();
    }

    @Override
    public void undo() {
        light.turnOn();
    }
}

// Invoker
class RemoteControl {
    private Command command;
    private Command lastCommand;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void pressButton() {
        command.execute();
        lastCommand = command;
    }

    public void pressUndoButton() {
        if (lastCommand != null) {
            lastCommand.undo();
        }
    }
}

// Client
public class CommandPatternDemo {
    public static void main(String[] args) {
        Light light = new Light();
        Command lightOn = new LightOnCommand(light);
        Command lightOff = new LightOffCommand(light);

        RemoteControl remote = new RemoteControl();

        remote.setCommand(lightOn);
        remote.pressButton();  // 조명 켜기

        remote.setCommand(lightOff);
        remote.pressButton();  // 조명 끄기

        remote.pressUndoButton();  // 실행 취소 (조명 켜기)
    }
}
```

## Spring Boot에서의 Command 패턴

Spring에서 커맨드 패턴은 **비동기 작업 처리**, **트랜잭션 관리**, **이벤트 기반 처리**에 자주 활용됩니다.

### 1. 주문 처리 시스템 (실무 예제)

```java
// Command 인터페이스
public interface OrderCommand {
    void execute();
    void undo();
    String getDescription();
}

// Receiver - 실제 작업 수행
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;

    public Order createOrder(OrderRequest request) {
        Order order = Order.from(request);
        return orderRepository.save(order);
    }

    public void cancelOrder(Long orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
        order.cancel();
        orderRepository.save(order);
    }

    public void reserveStock(Long orderId) {
        inventoryService.reserve(orderId);
    }

    public void releaseStock(Long orderId) {
        inventoryService.release(orderId);
    }

    public void processPayment(Long orderId, int amount) {
        paymentService.charge(orderId, amount);
    }

    public void refundPayment(Long orderId) {
        paymentService.refund(orderId);
    }
}
```

```java
// Concrete Commands - 각 작업을 명령으로 캡슐화
@Component
@Scope("prototype")  // 매번 새 인스턴스 생성
@RequiredArgsConstructor
public class CreateOrderCommand implements OrderCommand {
    private final OrderService orderService;
    private OrderRequest request;
    private Order createdOrder;

    public CreateOrderCommand init(OrderRequest request) {
        this.request = request;
        return this;
    }

    @Override
    public void execute() {
        createdOrder = orderService.createOrder(request);
    }

    @Override
    public void undo() {
        if (createdOrder != null) {
            orderService.cancelOrder(createdOrder.getId());
        }
    }

    @Override
    public String getDescription() {
        return "주문 생성: " + request.getProductName();
    }

    public Order getCreatedOrder() {
        return createdOrder;
    }
}

@Component
@Scope("prototype")
@RequiredArgsConstructor
public class ReserveStockCommand implements OrderCommand {
    private final OrderService orderService;
    private Long orderId;

    public ReserveStockCommand init(Long orderId) {
        this.orderId = orderId;
        return this;
    }

    @Override
    public void execute() {
        orderService.reserveStock(orderId);
    }

    @Override
    public void undo() {
        orderService.releaseStock(orderId);
    }

    @Override
    public String getDescription() {
        return "재고 예약: 주문 " + orderId;
    }
}

@Component
@Scope("prototype")
@RequiredArgsConstructor
public class ProcessPaymentCommand implements OrderCommand {
    private final OrderService orderService;
    private Long orderId;
    private int amount;

    public ProcessPaymentCommand init(Long orderId, int amount) {
        this.orderId = orderId;
        this.amount = amount;
        return this;
    }

    @Override
    public void execute() {
        orderService.processPayment(orderId, amount);
    }

    @Override
    public void undo() {
        orderService.refundPayment(orderId);
    }

    @Override
    public String getDescription() {
        return "결제 처리: " + amount + "원";
    }
}
```

```java
// Invoker - 명령 실행 및 트랜잭션 관리
@Service
@Slf4j
public class OrderCommandInvoker {
    private final Deque<OrderCommand> executedCommands = new ArrayDeque<>();

    /**
     * 여러 명령을 순차 실행하고, 실패 시 자동 롤백
     */
    @Transactional
    public void executeWithRollback(List<OrderCommand> commands) {
        try {
            for (OrderCommand command : commands) {
                log.info("실행: {}", command.getDescription());
                command.execute();
                executedCommands.push(command);
            }
            log.info("✅ 모든 명령 실행 완료");
        } catch (Exception e) {
            log.error("❌ 명령 실행 실패: {}", e.getMessage());
            rollbackAll();
            throw new OrderProcessingException("주문 처리 실패", e);
        }
    }

    /**
     * 실행된 명령을 역순으로 취소
     */
    private void rollbackAll() {
        log.info("↩️ 롤백 시작 (실행된 명령 수: {})", executedCommands.size());
        while (!executedCommands.isEmpty()) {
            OrderCommand command = executedCommands.pop();
            try {
                log.info("취소: {}", command.getDescription());
                command.undo();
            } catch (Exception e) {
                log.error("롤백 실패: {}", command.getDescription(), e);
            }
        }
        log.info("✅ 롤백 완료");
    }

    public void clearHistory() {
        executedCommands.clear();
    }
}
```

```java
// 실제 사용 - OrderFacade에서 명령 조합
@Service
@RequiredArgsConstructor
public class OrderFacade {
    private final OrderCommandInvoker invoker;
    private final ObjectProvider<CreateOrderCommand> createOrderProvider;
    private final ObjectProvider<ReserveStockCommand> reserveStockProvider;
    private final ObjectProvider<ProcessPaymentCommand> paymentProvider;

    public Order placeOrder(OrderRequest request) {
        // 명령 객체들 생성
        CreateOrderCommand createOrder = createOrderProvider.getObject().init(request);

        // 먼저 주문 생성 실행
        createOrder.execute();
        Long orderId = createOrder.getCreatedOrder().getId();

        // 나머지 명령들 준비
        List<OrderCommand> commands = List.of(
            reserveStockProvider.getObject().init(orderId),
            paymentProvider.getObject().init(orderId, request.getTotalAmount())
        );

        // 트랜잭션 단위로 실행 (실패 시 자동 롤백)
        invoker.executeWithRollback(commands);

        return createOrder.getCreatedOrder();
    }
}
```

### 2. @Async와 함께 사용하기 (비동기 명령 큐)

```java
// 비동기 명령 인터페이스
public interface AsyncCommand {
    CompletableFuture<Void> executeAsync();
    String getCommandId();
}

// 이메일 발송 명령
@Component
@Scope("prototype")
@RequiredArgsConstructor
public class SendEmailCommand implements AsyncCommand {
    private final EmailService emailService;
    private String to;
    private String subject;
    private String body;
    private final String commandId = UUID.randomUUID().toString();

    public SendEmailCommand init(String to, String subject, String body) {
        this.to = to;
        this.subject = subject;
        this.body = body;
        return this;
    }

    @Override
    @Async("emailTaskExecutor")  // 비동기 실행
    public CompletableFuture<Void> executeAsync() {
        emailService.send(to, subject, body);
        return CompletableFuture.completedFuture(null);
    }

    @Override
    public String getCommandId() {
        return commandId;
    }
}

// 비동기 명령 큐 관리자
@Service
@Slf4j
public class AsyncCommandQueue {
    private final BlockingQueue<AsyncCommand> queue = new LinkedBlockingQueue<>();
    private final Map<String, CommandStatus> statusMap = new ConcurrentHashMap<>();

    public String enqueue(AsyncCommand command) {
        queue.offer(command);
        statusMap.put(command.getCommandId(), CommandStatus.PENDING);
        log.info("명령 큐에 추가: {}", command.getCommandId());
        return command.getCommandId();
    }

    @Scheduled(fixedDelay = 1000)  // 1초마다 큐 처리
    public void processQueue() {
        AsyncCommand command = queue.poll();
        if (command != null) {
            statusMap.put(command.getCommandId(), CommandStatus.RUNNING);
            command.executeAsync()
                .thenRun(() -> statusMap.put(command.getCommandId(), CommandStatus.COMPLETED))
                .exceptionally(e -> {
                    statusMap.put(command.getCommandId(), CommandStatus.FAILED);
                    return null;
                });
        }
    }

    public CommandStatus getStatus(String commandId) {
        return statusMap.getOrDefault(commandId, CommandStatus.UNKNOWN);
    }
}
```

### 3. Spring에서 커맨드 패턴이 사용되는 곳

| Spring 기능 | 커맨드 패턴 적용 |
|------------|-----------------|
| `@Async` | 메서드 호출을 비동기 태스크(Command)로 캡슐화 |
| `@Scheduled` | 스케줄된 작업을 Command로 관리 |
| `TransactionTemplate` | 트랜잭션 내 작업을 Command로 캡슐화 |
| `JdbcTemplate.execute(callback)` | SQL 실행을 Command 콜백으로 전달 |
| `ApplicationEvent` | 이벤트를 Command처럼 발행/처리 |

### Spring 패턴 활용 팁

```java
// 1. @Scope("prototype")으로 매번 새 Command 인스턴스 생성
@Component
@Scope("prototype")
public class MyCommand implements Command { ... }

// 2. ObjectProvider로 프로토타입 빈 주입
@RequiredArgsConstructor
public class CommandClient {
    private final ObjectProvider<MyCommand> commandProvider;

    public void doSomething() {
        MyCommand cmd = commandProvider.getObject();  // 새 인스턴스
    }
}

// 3. @Async + CompletableFuture로 비동기 명령
@Async
public CompletableFuture<Result> executeAsync() {
    // 비동기 실행
}

// 4. @Transactional + 롤백으로 안전한 명령 실행
@Transactional
public void executeWithRollback(List<Command> commands) {
    // 실패 시 자동 롤백
}
```

## 장점

- **요청자와 수신자 분리**: 요청을 보내는 객체와 처리하는 객체를 분리합니다.
- **실행 취소 기능**: 명령의 상태를 저장하여 undo/redo를 쉽게 구현할 수 있습니다.
- **매크로 기능**: 여러 명령을 조합하여 복합 명령을 만들 수 있습니다.
- **로깅과 트랜잭션**: 명령들을 기록하고 나중에 재실행하거나 롤백할 수 있습니다.
- **지연 실행**: 명령을 나중에 실행하거나 큐에 저장할 수 있습니다.

## 단점

- **클래스 수 증가**: 각 명령마다 별도의 클래스를 만들어야 하므로 클래스 수가 늘어납니다.
- **복잡성 증가**: 간단한 동작에 대해서도 명령 객체를 만들어야 하므로 코드가 복잡해질 수 있습니다.
- **메모리 사용량**: 명령 기록을 저장하면 메모리 사용량이 증가할 수 있습니다.

## 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Memento** | Command와 함께 사용하여 Undo를 구현. Command는 "무엇을 했는지", Memento는 "이전 상태"를 저장 |
| **Strategy** | 둘 다 객체로 행동을 캡슐화하지만, Strategy는 같은 목적의 다른 알고리즘, Command는 다른 목적의 다른 요청 |
| **Chain of Responsibility** | Command를 핸들러 체인으로 전달하여 처리할 수 있음 |
| **Composite** | MacroCommand처럼 여러 Command를 트리 구조로 조합 |
| **Prototype** | Command 객체를 복제하여 히스토리 저장 시 사용 가능 |

### 패턴 비교

```
Command vs Strategy:
├─ Command: "무엇을 할지"를 객체로 만듦 (요청의 캡슐화)
│   예: LightOnCommand, LightOffCommand (다른 요청)
└─ Strategy: "어떻게 할지"를 객체로 만듦 (알고리즘의 캡슐화)
    예: QuickSort, MergeSort (같은 목적, 다른 방법)

Command + Memento (Undo 구현):
├─ Command: execute() 시 Receiver의 현재 상태를 Memento로 저장
├─ Memento: Receiver의 상태 스냅샷 보관
└─ undo() 시: 저장된 Memento로 상태 복원
```