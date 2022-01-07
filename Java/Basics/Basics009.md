# Java
## 2022-01-07 Fri

### 인터페이스
* 인터페이스 또한 추상 메소드를 갖는, 일종의 추상 클래스이다.
  * 그러나 모든 메소드가 추상 메소드인 식으로 추상화 정도가 추상 클래스보다 높다.
  * 인터페이스는 완성된 메소드나 멤버 변수를 가질 수 없다.
* 인터페이스는 Object로부터 내려오는 상속 흐름이 없다.
* 인터페이스는 추상 메소드와 상수만을 가질 수 있다.
  * 모든 메소드는 public abstract이다.
    * JDK 1.8부터 static과 default도 허용한다.
  * 모든 상수는 public static final이다.
  * 위의 제어자 조합은 생략할 수 있다. (어차피 모든 멤버에 적용될 것이기 때문!) 
    * 생략된 제어자는 컴파일 시점에 추가된다.
* **인터페이스는 그 자체로 사용하기보다는 다른 클래스의 작성에 도움을 주기 위해 작성**된다.
```
interafacte Temp {
    public static final [상수명] = [값];
    public abstract [메소드명]([매개 변수 정보]);
}
```
* 인터페이스도 상속을 주고 받을 수 있지만, 인터페이스 간에만 가능하다.
* 인터페이스 간의 상속은 **다중 상속을 지원**한다.
  * 상속 받은 자식 인터페이스는 부모 인터페이스들에 정의된 모든 멤버를 상속 받는다.
```
interface TempA {}
interface TempB {}
interface Child extends TempA, TempB {}
```
* 인터페이스는 일반적으로 **'어떠한 기능을 구현할 목적으로' 추상 메소드를 작성**한다.

### 인터페이스의 구현
* 추상 클래스와 마찬가지로 인터페이스 자체로는 인스턴스를 생성할 수 없다.
* 인터페이스의 내용을 완성하는 클래스가 추가로 필요하며, 이 때 인터페이스의 내용을 구현하는 implements 키워드를 사용한다. 
  * 추상 클래스를 상속 받는 자식 클래스는 부모인 추상 클래스를 확장하므로 extends를 사용하는 것과 대비된다.
* 클래스가 인터페이스를 구현하되, 인터페이스의 모든 기능을 구현하고 싶지 않다면 추상 클래스를 사용한다.
```
interface Temp {
  void hello();
}
class Hello implements Temp {
  public void hello() { ; }
  // 추상 메소드의 바디를 정의한다.
  // public으로 오버라이딩해야 한다. 구현 클래스는 인터페이스의 메소드보다 적은 범위의 제어자를 가질 수 없다.
}
interface Tbd {
  void determined();
  void tbd();
}
abstract class TempTbd implements Tbd {
  public void determined(){ ; }
  // 인터페이스의 모든 기능을 구현하지 않으려면 abstract 클래스여야 한다.
}
```
* 인터페이스의 명명은 ~able과 같이 '**특정 기능을 할 수 있을 것**'을 암시하는 것을 권장한다.
  * 이를 통해 해당 인터페이스의 기능을 쉽게 유추할 수 있으며, 이 인터페이스를 구현한 클래스가 무엇을 할 수 있는지 쉽게 알 수 있게 한다.
* 인터페이스의 구현 관계 역시 instanceof로 체크할 수 있다.
  * 이 때, 특정 클래스가 구현한 인터페이스의 계층도에 있는 모든 인터페이스에 대해 true가 반환된다.
* 인터페이스를 구현하는 클래스 역시 메소드의 접근 제어 지시자를 더 좁은 범위로 수정할 수 없다.
  * 인터페이스의 메소드는 public abstract이므로, 구현 클래스의 메소드는 항상 public이 된다.
* 참고: 다음과 같은 인터페이스가 존재한다고 하자.
```
interface Movable {
  void move();
}
```
* 위 인터페이스를 세 개의 클래스가 구현한다고 할 경우, move 메소드에 대한 동작은 세 클래스에서 모두 오버라이딩 되어야 한다.
* 만약 move 메소드가 하는 역할이 세 클래스에서 모두 같다면, 하나의 클래스에서 수정된 내용은 세 클래스에서도 일관성 유지를 위해 모두 수정되어야 한다.
* 이러한 유지보수 상의 문제는 Movable을 구현하는 클래스를 하나 만들고, 기존의 세 클래스에서 해당 클래스를 인스턴스화하여 사용하는 것으로 대체할 수 있다.
  * 이는 인터페이스를 구현하던 기존 클래스의 코드가 모두 같은 경우를 가정한다. 이러한 방식은 하나의 클래스에서 오버라이딩하여 유지보수성을 향상시킨다.
```
class Impl implements Movable {
  public void move(){ ; };
}
class TempA {
  Impl impl = new Impl();
  void move() { imple.move(); }
}
class TempB {
  Impl impl = new Impl();
  void move() { imple.move(); }
}
```

### 인터페이스 다중상속
* Java는 다중 상속에서 기인하는 여러 문제(부모 클래스들이 같은 이름의 멤버를 갖는 경우의 복잡성, 등)로 인해 다중 상속을 지원하지 않는다.
  * 그러나 C++과 같은 언어에서는 다중 상속을 지원하므로, 이는 Java의 단점으로 비추어질 수 있다.
  * 때문에 Java 진영에서는 인터페이스로 다중 상속을 지원하도록 한다고 함.
  * 그러니 인터페이스 = 다중 상속에 사용 이라고 오해하지 말자!

### 인터페이스 다형성
* 인터페이스를 구현한 클래스 또한 자식 클래스의 일종이므로, 해당 인터페이스 타입의 참조 변수로 구현한 클래스를 참조할 수 있다.
  * 이 경우, 해당 참조 변수로는 인터페이스의 멤버만 호출할 수 있다.
```
class Main {
  public static void main(String[] args) {
    Temp temp = new Concrete(); // 인터페이스 타입의 참조 변수로 구현 클래스를 참조한다.
    // temp.hello(); // Temp의 멤버가 아닌 구현 클래스의 멤버이므로 호출할 수 없다.
  }
}
interface Temp {
  void temp();
}
class Concrete implements Temp {
   void temp() { ; };
   void hello() { ; };
}
```
* 따라서 **인터페이스 역시 메소드의 매개 변수로 지정될 수 있다**!
  * 이러한 메소드는 메소드 호출 시 해당 인터페이스를 구현한 클래스의 인스턴스를 인자로 넘겨주어야 한다.
* 당연히 메소드의 반환형으로도 지정될 수 있다.
  * 역시 해당 **메소드가 반환형으로 지정된 인터페이스를 구현한 클래스를 반환해야 함을 의미**한다.
* 인터페이스의 다형성은 클라이언트 - 서버 구조에서 적합하다. 클라이언트의 App.을 변경하지 않고 서버 측 변경을 반영하는 것만으로 사용자가 수정된 새로운 App.을 사용할 수 있다.

### 인터페이스의 장점
* 기존 상속 관계를 깨트리지 않고, 필요한 클래스에만 공통된 기능을 추가할 수 있다.
  * 즉, 서로 관계 없는 클래스들에게 관계를 맺어줄 수 있다.
  * 이 경우 인터페이스를 구현한 클래스들은 원래 서로와 관계가 없으므로, 한 인터페이스의 수정이 다른 인터페이스에 영향을 주지 않는다.
  
### 인터페이스를 왜 사용하는가?
* 다음과 같은 구조는 TempB 이외에 TempC, TempD가 신규 작성될 경우 TempA에 대응되는 메소드가 계속해서 추가되어야 한다.
  * 이러한 관계를 직접적인 관계라고도 한다.
```
class Main {
  public static void main(String[] args) {
    TempA temp = new TempA();
    temp.helloB(new TempB());
    temp.helloB(new TempC()); // 이걸 위해
  }
}
class TempA {
  public void helloB(TempB temp) { temp.tempMethod(); }
  public void helloB(TempC temp) { temp.tempMethod(); } // 여기가 추가되어야 한다.
}
class TempB {
  public void tempMethod(){ System.out.println("Hello World!"); }
}

class TempC {
  public void tempMethod(){ System.out.println("Bye World!"); }
}
```
* 반면 다음과 같은 구조는 TempC, D가 계속 추가되더라도 TempA가 수정될 필요가 없다.
  * 이 경우, TempA 클래스는 실제로 호출되는 클래스의 이름을 알 필요도 없다. 인터페이스만 알면 되기 때문!
    * 심지어, **인터페이스를 구현하는 클래스가 실제로 존재하지 않아도 TempA를 완성할 수 있으므로 코드를 선작업**해둘 수 있다.
  * 이는 기존의 직접적인 관계가 인터페이스를 통한 간접적인 관계로 변경된 것에 해당한다.
* 만약 하기 코드의 new TempB(), new TempC() 코드가 마음에 들지 않는다면, 인스턴스화 및 반환을 담당하는 팩토리 클래스를 작성하여 해결해볼 수 있다.
```
class Main {
  public static void main(String[] args) {
    TempA temp = new TempA();
    // B, C 등은 Enum으로 바꾸면 더 좋다.
    temp.helloB(TempInterfaceFactory.getInstance("B"));
    temp.helloB(TempInterfaceFactory.getInstance("C"));
  }
}
class TempA {
  public void helloB(TempInterface temp) { temp.tempMethod(); }
}

interface TempInterface {
    void tempMethod();
}
class TempB implements TempInterface {
  public void tempMethod(){ System.out.println("Hello World!"); }
}

class TempC implements TempInterface {
  public void tempMethod(){ System.out.println("Bye World!"); }
}

class TempInterfaceFactory {
    public static TempInterface getInstance(String type) {
        TempInterface temp;
        // B, C 등은 Enum으로 바꾸면 더 좋다.
        switch(type) {
            case "B": temp = new TempB(); break;
            case "C": temp = new TempC(); break;
            default: temp = null; break;
        }
        return temp;
    }
}
```
* 추가적으로, 인터페이스 타입의 참조 변수로도 .toString() 등 Object 클래스의 메소드를 사용할 수 있다.
  * 어차피 모든 클래스는 Object를 상속받았을 것이기 때문에 허용되는 것이라고 한다!

### 인터페이스의 static 메소드
* 원래 클래스 메소드(static)는 인스턴스와 독립적인 메소드이므로, 인터페이스에 추가하지 않을 이유는 없었다.
  * 그러나 인터페이스의 러닝 커브를 낮추기 위해 반드시 추상 메소드만 존재해야 한다는 규칙에 예외를 두지 않은 것이라고 한다.
* JDK 1.8부터는 인터페이스에도 public static 메소드를 작성할 수 있도록 규칙이 변경되었다.
```
class Main {
  public static void main(String[] args) {
    TempInterface.hello(); // 인터페이스를 별도 클래스로 구현하지 않아도 호출이 가능하다!
  }
}
interface TempInterface {
    static void hello() { System.out.println("Helloooo"); }
}
```

### 인터페이스의 default 메소드
* 부모 클래스에 신규 메소드를 추가하는 것은 쉽다. 자식 클래스도 해당 메소드를 멤버로서 상속받기 때문이다.
* 반면 인터페이스의 경우, 인터페이스를 구현하는 클래스가 많을 수록 메소드 추가의 문제가 커진다.
  * 인터페이스를 구현한 클래스 모두 해당 메소드를 정의해야하기 때문이다.
* 이러한 경우, default 키워드를 통해 신규 메소드를 인터페이스에 추가할 수 있다.
  * default는 접근 제어자 default와 관계 없다. 인터페이스의 default 메소드는 public이다!
* 이는 마치 부모 클래스에 신규 메소드를 추가한 것과 유사하며, 구현 클래스들이 해당 클래스를 구현할 필요가 없어지므로 추가 수정이 필요가 없다.
* 그러나 아래와 같이 여러 인터페이스에서 동일한 메소드 이름을 사용하는 경우 에러가 발생할 수 있다.
  * 이는 인터페이스를 구현하는 구현 클래스에서 메소드를 오버라이드하는 것으로 해결할 수 있다.
```
class Main {
  public static void main(String[] args) {
    Impl i = new Impl();
    i.hello();
  }
}
interface TempA {
    // default는 접근 제어자가 아니다.
    public default void hello() { System.out.println("Helloooo"); }
}

interface TempB {
    public default void hello() { System.out.println("Byeeeeee"); }
}

class Impl implements TempA, TempB {
    // 이대로 실행하면 에러가 발생한다. 이러한 경우, 아랫 줄과 같이 메소드를 오버라이드 해서 사용해야 한다.
    // public void hello() { System.out.println("Overrideee"); }
}
```
* 만약 부모 클래스를 상속 받고, 특정 인터페이스를 구현도 하는데 부모 클래스의 메소드와 인터페이스의 default 메소드의 이름이 같다면 부모 클래스의 메소드가 우선된다.
  * 자식 클래스에도 동명의 메소드가 있다면, 자식 클래스의 메소드가 호출된다.
  * 즉, 메소드 명이 중복되는 경우 자식 클래스 > 부모 클래스 > 인터페이스의 default 메소드 순의 우선 순위를 갖는다.
```
class Main {
  public static void main(String[] args) {
    Child child = new Child();
    
    child.hello();
  }
}
interface TempA {
    public default void hello() { System.out.println("Helloooo"); }
}

class Parent {
    public void hello() { System.out.println("Parent method!"); }
}

// class Child implements TempA {
class Child extends Parent implements TempA {
    // public void hello() {
    //     System.out.println("Child method");
    // }
}
```