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