# HeadFirst
## 2022-11-02 Wed

### 커맨드 패턴의 필요성
```
> 때로는 작업을 요청하는 측과 작업을 처리하는 측을 분리하여 캡슐화해야할 수 있다.
> 이 때, 중간 계층인 커맨드 객체를 두는 것으로 두 객체를 손쉽게 분리할 수 있다.
```
* 예를 들어 **작업을 요청했을 때 처리될 수 있는 작업의 종류는 매우 많으나, 작업 간의 공통된 인터페이스를 추출하기는 어려운 경우가 있을 수 있다**.
  * 이 경우, 엎친 데 덮친 격으로 작업의 종류가 추후에 추가되는 것을 고려한다면 문제는 더욱 복잡해진다.
* 변화를 캡슐화하기 위해서는 두 요청자와 처리자 사이에 위치하여 각 객체를 분리하는 역할을 수행할 무언가가 필요하며, 커맨드 객체는 이를 적절히 수행할 수 있다.

### 커맨드 패턴의 구성 요소
```
> 커맨드 패턴은 크게 클라이언트 객체로부터 시작하여 리시버와 커맨드, 인보커 객체로 구성된다.
```
* **커맨드 패턴을 시작하는 클라이언트 객체는 커맨드 객체를 생성**한다.
* **커맨드 객체는 리시버의 정보를 포함하며, 행동을 캡슐화하는 유일한 메소드인 execute**를 갖는다.
  * 이 때, 해당 메소드는 리시버 객체가 갖는 임의의 행동을 처리하는 식으로 동작한다.
* **인보커 객체에는 클라이언트 객체가 setCommand 메소드를 호출하여 커맨드 객체를 설정하며, 설정된 커맨드 객체를 적시까지 보관하는 역할을 맡는다**.
  * **인보커 객체는 필요한 시점에 자신이 참조하는 커맨드 객체의 execute 메소드를 호출하며, 이로 인해 리시버 객체에 저장된 메소드가 호출**된다.

### 인보커의 로딩
* 인보커는 기본적으로 클라이언트에서 생성한 커맨드 객체를 저장하기 위한 setCommand 메소드를 제공한다.
  * 해당 메소드를 통해 **저장된 커맨드 객체는 추후 적절한 시점에 클라이언트가 명령 실행을 요청했을 때 커맨드 객체의 명령을 실행**한다.
* 이렇듯 일회성 작업 뿐만 아니라 인보커 객체는 커맨드 객체를 한 번 로딩한 후에는 다음과 같이 동작하도록 구현될 수도 있다.
  1. 작업을 처리한 후에 커맨드 객체의 참조를 제거한다.
  2. 또는 저장해둔 커맨드 객체를 통해 명령을 여러 차례 실행한다.

## 2022-11-03 Thu
### 커맨드 객체 정의하기
* **커맨드 객체는 반드시 인터페이스를 기반으로 정의되는 반면, 인터페이스의 구조 자체는 매우 간단**하다.
  * 이 때, **인터페이스의 메소드는 자유롭게 이름을 정의해도 좋지만 일반적으로는 execute라는 명명을 사용**한다.
```java
public interface Command {
    void execute();
} 
```
* 이러한 **커맨드 객체를 구현하는 것은 다음과 같으며, 반드시 생성자로부터 커맨드 객체를 처리할 리시버 객체를 전달하여 멤버 변수로 참조**할 수 있어야 한다.
  * 아래 **예시의 경우, 리시버를 호출하는 메소드는 execute 뿐이므로 Bird가 여러 동작을 처리할 수 있더라도 동작 당 하나의 커맨드 객체를 정의**한다.
```java
public class FlyCommand implements Command {
    // 생성자로부터 커맨드 객체로 제어할 리시버 객체를 전달한 후, 멤버 변수로 참조하도록 구현한다.
    private Bird bird;
    
    public FlyCommand(Bird bird) {
        this.bird = bird;
    }
    
    // 커맨드 인터페이스를 구현하며, execute 메소드가 호출되었을 때 리시버 객체에게 작업을 요청한다.
    @Override
    public void execute() {
        bird.fly();
    }
}
```

### 커맨드 객체 사용하기
* 상술했듯 **커맨드 인터페이스와 그 구현체를 정의했다면, 이는 다음과 같이 간단한 인보커 객체를 구현하는 것으로 사용이 가능**하다.
  * 아래의 예시에서 확인할 수 있듯, 필요한 경우에 커맨드 객체는 설정자를 활용하여 언제든지 교체가 가능하다.
```java
public class LetItFly {
    // 인보커 객체 역시 생성자를 활용하여 커맨드를 멤버 변수로 참조한다. 
    private Command command;
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    // 적절한 시점에 해당 메소드가 호출된 경우, 자신의 멤버 변수로 참조된 커맨드 객체의 execute 메소드를 호출한다.
    public void runCommand() {
        command.execute();
    }
}
```
* 이제 커맨드 객체와 인보커 객체가 모두 준비되었으므로, 이를 클라이언트가 호출하는 상황 역시도 다음과 같이 작성해볼 수 있다.
```java
class Main {
    // main 메소드는 클라이언트 객체의 역할을 임시로 수행한다.
    public static void main(String[] args) {
        // 클라이언트 객체는 적절한 인보커에게 원하는 커맨드 객체를 전달한다.
        LetItFly invoker = new LetItFly();

        // 반면, 커맨드 객체에는 반드시 요청을 실제로 처리할 리시버 객체가 전달되어야 한다.
        Bird bird = new Bird();
        invoker.setCommand(new FlyCommand(bird));
        
        // 적절한 시점에 클라이언트가 인보커 객체를 호출하여 원하는 작업의 처리를 요청한다.
        invoker.runCommand(); // I Believe I can fly!!!
    }
}

class Bird {
    public void fly() {
        System.out.println("I Believe I can fly!!!");
    }
}

interface Command {
    void execute();
}

class FlyCommand implements Command {
    private Bird bird;
    
    public FlyCommand(Bird bird) {
        this.bird = bird;
    }
    
    @Override
    public void execute() {
        bird.fly();
    }
}

class LetItFly {
    private Command command;
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void runCommand() {
        command.execute();
    }
}
```

### 커맨드 패턴 정의하기
```
> 커맨드 패턴을 사용할 경우, 요청 내역 자체를 객체로 캡슐화하여 객체를 서로 다른 요청 내역에 따라 매개변수화할 수 있다.
> 때문에 커맨드 패턴을 적절히 활용하면 요청을 큐에 저장하거나, 로깅하거나 작업 자체를 취소할 수도 있다.
```
* **커맨드 객체는 일련의 행동을 임의의 리시버 객체와 연결하는 것으로 요청 자체를 캡슐화**한다.
  * 이를 위해서는 행동과 리시버를 하나의 객체에 담은 후, execute 메소드 하나만을 외부로 공개해야 한다.
  * 때문에 **객체 외부에서 볼 때에는 어떤 리시버 객체가 무슨 작업을 하는지는 전혀 알지 못한 채, 단지 요청 처리를 위해 execute 메소드만을 호출**한다.
* 기본적인 커맨드 패턴을 적절히 이해한 경우, 큐와 로그 또는 작업 취소 및 메타 커맨드 패턴까지 쉽게 확장할 수 있다.

### 커맨드 패임의 구성 요소 별 책
* **클라이언트는 구체 커맨드 객체를 생성하고, 이에 적절한 리시버를 설정하는 역할을 수행**한다.
* 리시버 객체는 실제로 작업을 처리하므로, 작업의 처리 방법을 구체적으로 알고 있어도 상관이 없다.
* **인보커 객체는 커맨드 객체를 포함하며, 커맨드 객체의 execute 메소드를 호출하여 임의의 작업을 요청**한다. 
* 커맨드 객체는 추상 커맨드 객체와 구체 커맨드 객체로 분류되며, 각각 다음과 같은 작업을 처리한다.
  1. **추상 커맨드 객체는 모든 커맨드 객체에서 구현하는 인터페이스이며, execute 메소드를 정의하여 임의의 리시버에게 특정한 작업에 대한 처리를 위임**한다.
  2. **구체 커맨드 객체는 임의의 행동과 리시버를 연결하며, 인보커로부터의 execute 호출을 수신하면 리시버에게 작업을 위임하여 처리**한다.
* 이렇듯 **커맨드 객체에 구현되는 execute 메소드는 내부적으로는 멤버 변수로 참조하는 리시버 객체의 메소드를 호출하여 작업을 처리하는 식으로 동작**한다.

## 2022-11-04 Fri
### 널 객체
* 커맨드 객체를 적절히 활용할 경우, 다음과 같이 아무 동작도 하지 않는 널 객체를 정의할 수 있다.
```java
public class NoCommand implements Command {
    public void execute(){}
}
```
* 이러한 널 객체는 굳이 리턴할 필요는 없으나 클라이언트가 null을 처리하지 않도록 하고 싶은 경우에 유용하게 활용이 가능하다.
  * 예를 들어, **인보커가 참조하는 멤버 변수에 대한 null 체크 로직을 추가하는 대신 생성자에서 우선 널 객체로 초기화**할 수 있다.
* 이렇듯 널 객체는 여러 디자인 패턴에서 매우 유용하게 사용되므로, 널 객체 자체를 일종의 디자인 패턴으로 분류하는 의견 역시 존재한다.

### 커맨드 객체의 역할
* **커맨드 객체는 인보커 객체와 리시버 객체를 분리하는 것으로, 작업의 요청과 작업 처리를 성공적으로 분리**한다.
  * 즉, **커맨드 객체는 리시버의 동작을 캡슐화하므로 인보커 객체가 단지 execute 메소드만을 호출할 수 있도록 한다**.
  * **커맨드 객체에는 실제 작업을 처리하는 리시버 객체의 참조가 포함되며, execute 메소드를 통해 리시버 객체의 메소드를 하나 이상 호출**하게 된다.
* 이 때, 커맨드 객체가 execute와 같은 단일 메소드만을 제공할 경우 람다 표현식을 사용하여 불필요한 커맨드 클래스의 수를 크게 줄일 수 있다.
```
class Main {
    public static void main(String[] args) {
        LetItFly invoker = new LetItFly();

        Bird bird = new Bird();
        
        // 이렇듯 기존 방식대로 커맨드 객체를 별도의 클래스로 선언하는 대신,
        // invoker.setCommand(new FlyCommand(bird));
        
        // 리시버를 직접 호출하는 람다 표현식을 작성할 수도 있으므로 FlyCommand 클래스를 제거할 수 있다.
        invoker.setCommand(() -> bird.fly());
        invoker.runCommand(); // I Believe I can fly!!!
    }
}
```

### 커맨드 객체의 작업 취소
* 예를 들어 **커맨드 객체를 통해 어떤 작업을 처리했을 때, UNDO 기능을 통해 이를 취소하려는 요구 사항이 발생**할 수 있다.
* **이는 커맨드 객체의 공통 기능으로 추가되어야하므로, 당연히 execute 메소드와 같은 undo 메소드의 추가가 필요**하다.
```java
public interface Command {
    void execute();
    void undo();
}
```
* 상술한 FlyCommand의 예제에서 undo 메소드를 반영하는 경우, 다음과 같은 코드를 작성할 수 있다.
  * 만일 StopCommand가 존재한다고 가정하면 해당 커맨드 객체의 undo 메소드는 `bird.fly();` 를 호출하는 식으로 구현된다.
```java
class FlyCommand implements Command {
    private Bird bird;
    
    public FlyCommand(Bird bird) {
        this.bird = bird;
    }
    
    @Override
    public void execute() {
        bird.fly();
    }
    
    @Override
    public void undo() {
        bird.stop();
    }
}
```
* 반면, **리시버가 여러 상태 값을 가져 이를 작업 취소 기능에 반영할 경우 커맨드 객체에 마지막으로 호출된 상태를 저장하는 별도의 멤버 변수가 필요**하다.
```java
class Bird {
    // 리시버 객체는 호출된 메소드에 따라 현재 상태를 멤버 변수로 저장한다.
    private String status;
    
    public Bird() {
        status = "STOP";
    }
    public void stop() {
        System.out.println("stop flying.");
        status = "STOP";
    }
    public void fast() {
        System.out.println("fly fast!!!");
        status = "FAST";
    }
    public void slow() {
        System.out.println("fly slow...");
        status = "SLOW";
    }
    
    // 리시버 객체는 자신이 관리하는 현재 상태 정보를 반환하기 위한 접근자를 제공한다.
    public String getStatus() {
        return status;
    }
}

class FlyCommand implements Command {
    private Bird bird;
    // 리시버 객체의 이전 상태를 저장하기 위한 멤버 변수를 커맨드 객체에 추가한다.
    private String prevStatus;
    
    public FlyCommand(Bird bird) {
        this.bird = bird;
    }
    
    @Override
    public void execute() {
        // execute 메소드가 호출된 경우, 리시버 객체에 작업을 위임하기 전에 리시버 객체의 현재 상태를 저장한다.
        prevStatus = bird.getStatus();
        bird.fly();
    }
    
    @Override
    public void undo() {
        // 리시버 객체의 이전 상태를 저장하는 멤버 변수의 값에 따라 이전 동작으로 되돌린다.
        if(prevStatus == 'FAST')
            bird.fast();
        else if(prevStatus == 'SLOW')
            bird.slow();
        else
            bird.stop();
    }
}
```

### 인보커 객체에 작업 취소 기능 추가하기
* 상술한 커맨드 객체의 수정 사항에 맞추어 인보커 객체에 다음과 같이 몇 줄을 추가하면 간단하게 작업 취소 기능을 구현할 수 있다.
```java
class LetItFly {
  private Command command;
  // 작업 취소 기능에 대비하여 마지막으로 execute 메소드가 호출된 커맨드 객체를 참조하기 위한 멤버 변수를 추가한다.
  private Command lastInvoked;
  
  public LetItFly() {
      // NPE의 발생에 대비하여 NoCommand와 같은 널 객체를 활용할 수 있다.
      Command noCommand = new NoCommand();
      this.command = noCommand;
      this.lastInvoked = noCommand;
  }

  public void setCommand(Command command) {
    this.command = command;
  }

  public void runCommand() {
    command.execute();
    // 커맨드 객체의 execute 메소드가 호출되면 이를 멤버 변수로 참조한다.
    lastInvoked = command;
  }
  
  public void undo() {
      // 클라이언트 코드로부터 작업 취소 요청이 수신될 경우, 이를 커맨드 객체에 정의된 undo 메소드로 포워딩한다.
      lastInvoked.undo();
  }
}
```

## 2022-11-05 Sat
### 여러 동작을 한 번에 처리하는 커맨드 객체
* 상술한 예시의 **커맨드 객체는 하나의 작업만을 처리할 수 있는 것처럼 보이지만, 다음과 같이 구현한 커맨드는 한 번에 여러 커맨드 객체를 실행**할 수 있다.
  * 이 경우, **해당 커맨드 객체는 생성자와 멤버 변수를 통해 또 다른 커맨드를 배열로 전달 받아 활용**한다.
```java
class CommandOfCommands implements Command {
    // 여러 커맨드를 한 번의 execute 메소드 호출만으로 처리할 수 있도록, 커맨드 객체 배열을 멤버 변수로 갖는다.
    private Command[] commands;
    
    // 멤버 변수로 참조하는 커맨드 객체 배열은 생성자에서 초기화한다.
    public CommandOfCommands(Command[] commands) {
        this.commands = commands;
    }
    
    // 해당 커맨드 객체의 execute 메소드가 호출된 경우, 커맨드 객체 배열을 순회하며 모든 커맨드 객체의 execute 메소드를 호출한다.
    @Override
    public void execute() {
        for(Command command: commands) {
            command.execute();
        }
    }
    
    // undo 메소드의 구현 역시 동일한 방식을 사용할 수 있다.
    @Override
    public void undo() {
        for(Command command: commands) {
            command.undo();
        }
    }
}
```
* 이러한 매크로 커맨드 객체를 인보커 객체에 전달하는 경우, 클라이언트는 커맨드 객체의 배열을 만들어 전달하게 된다.

### 커맨드 패턴 주의사항
* 필요에 따라서는 리시버 객체 없이 커맨드 객체가 많은 것을 처리하는 스마트 커맨드 패턴을 정의할 수 있다.
  * 그러나 **이러한 방식은 인보커와 리시버를 분리하기 어렵고, 리시버 객체를 통해 커맨드를 매개변수화할 수 없다**.
* **undo 명령을 직전의 명령만 취소하는 것이 아닌, 여러 번 실행할 수 있도록 하기 위해서는 스택을 도입**할 수 있다.
  * 예를 들어, 커맨드 객체의 멤버 변수로 스택 자료 구조를 추가한 후에 execute 메소드가 호출될 때마다 이를 스택에 저장한다.
  * 그 이후로는 undo 메소드를 호출할 때마다 스택의 맨 위 항목부터 가져온 후에 undo 메소드를 호출할 수 있다.
* 매크로 커맨드 대신 임의의 커맨드 객체를 생성하여 execute 메소드로부터 다른 커맨드 객체의 execute 메소드를 직접 호출할 수도 있다.
  * 그러나 **이러한 방식은 유지보수성이 떨어지며, 무엇보다 매크로 커맨드는 커맨드 목록을 런타임에서 동적으로 결정할 수 있으므로 더 좋은 유연성을 갖는다**.

### 커맨드 패턴의 활용
* **커맨드 패턴을 적절히 활용하여 컴퓨테이션의 일부인 리시버로 대표되는 일련의 행동 패키지화하여 일급 객체 형태로 전달할 수도 있다**.
  * 이 경우, 클라이언트 애플리케이션에서는 커맨드 객체를 생성하고 오랜 시간이 지난 후에도 해당 컴퓨테이션을 호출할 수 있게 된다.
  * 무엇보다도, **다른 스레드에서 해당 컴퓨테이션을 호출할 수도 있는 큰 장점**을 얻게 된다.
* **상술한 이점으로 인해 커맨드 패턴은 스케쥴러나 스레드 풀, 또는 작업 큐와 같은 다양한 형태의 작업에 적용**할 수가 있다.
  * 예를 들어 **작업 큐의 경우, 큐 한 편에서는 커맨드를 계속해서 추가하고 다른 편에서는 스레드들이 커맨드를 처리하기 위해 대기**할 수 있다.
  * **각 스레드는 작업 큐로부터 작업을 가져온 후에 execute 메소드를 호출하고, 작업이 완료되면 자신이 처리한 커맨드 객체를 제거하는 식으로 동작**한다.
* 이러한 **작업 큐의 활용은 컴퓨테이션을 고정된 개수의 스레드로 제한할 수 있으며, 작업 큐 클래스 자체가 실제 작업을 수행하는 객체들과 완전히 분리**된다.
  * 또한, 각 스레드 역시 실제 작업의 처리에 종속되지 않고 매 번 다른 종류의 작업을 처리할 수 있는 유연성이 보장된다.
  * 이렇듯 **작업 큐 방식에서는 커맨드 객체를 큐에 삽입하는 것으로 이를 처리할 스레드가 자동으로 생성되고, execute 메소드를 호출하도록 구현**할 수 있다. 

### 커맨드 객체의 로그 기록과 복구 기능
* 커맨드 객체에 행동 기록과 복구 기능을 추가하고 싶은 경우, store와 load 메소드를 커맨드 인터페이스에 추가할 수 있다.
  * 기록 기능은 어떤 명령을 실행할 때마다 디스크에 실행 히스토리를 기록하여 복구 기능을 대비한다.
  * 반면, 복구 기능은 애플리케이션이 다운되었을 때 기록된 로그를 통해 커맨드 객체를 다시 로딩한 후에 일련의 기능을 호출하는 식으로 동작한다.
* 예를 들어 방대한 자료를 다루는 애플리케이션의 경우 로그를 통해 마지막 체크포인트 이후의 모든 작업을 저장하고, 시스템 다운시 이를 통해 복구할 수 있다.
  * 이 경우에는 모든 데이터의 변경을 기록하는 것이 아닌 특정 체크포인트 이후의 모든 행동만을 로그로 기록한다.
  * 이러한 기능을 고도화할 경우 애플리케이션의 일련의 작업에 트랜잭션과 롤백 기능을 적용할 수도 있다.

### 커맨드 패턴 - 결론
```
> 커맨드 패턴을 활용할 경우 메소드를 커맨드 객체로 캡슐화할 수 있으며, 필요시 객체를 저장하거나 전달하거나 메소드를 호출할 수도 있다.
```
* 커맨드 패턴을 활용할 경우, 요청하는 객체와 요청을 처리하는 객체를 분리할 수 있다.
* 이렇게 **두 개념을 분리하는 과정의 중심에 커맨드 객체가 포함되며, 커맨드 객체는 실제 행동을 처리하는 리시버를 캡슐화**한다.
* 커맨드 패턴에서는 작업을 요청하기 위해 인보커 객체가 커맨드 객체의 execute 메소드를 호출하고, 커맨드 객체는 리시버 객체에 작업을 위임한다.
* 커맨드 패턴에서는 인보커 객체를 매개변수화할 수 있으며, 런타임에서 동적으로 매개변수화를 설정할 수도 있다.
* 커맨드 패턴을 응용할 경우, 다음과 같은 기능을 구현해볼 수 있다.
  1. 커맨드 작업 취소 기능
  2. 커맨드를 확장하여 여러 커맨드를 한 번에 호출하는 매크로 커맨드 패턴
  3. 로그 및 트랜잭션 시스템