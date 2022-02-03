# DesignPattern
## 2022-02-04 Fri

## Adapter Pattern
### 기본 개념
* 하나의 인터페이스를 다른 인터페이스로 전환하기 위해 사용되는 디자인 패턴이다.
* 예를 들어, 클라이언트 코드의 인터페이스와 호출되는 목적 클래스의 인터페이스가 다른 경우 일반적으로 둘은 상호작용할 수 없다.
  * 그러나 Adapter 클래스를 추가하여 이를 해결할 수 있다.
  * 클라이언트 코드 입장에서는 Adapter의 인터페이스만 보이므로, 목적 클래스가 캡슐화되어 숨겨진다.
  * 이러한 탓에 Adapter 클래스는 Wrapper 클래스처럼 보여질 수 있다.
* Adapter 패턴은 두 클래스 간에 맞지 않는 기존의 인터페이스로 맞춰주기 위해 활용할 수 있다. 

### 코드 예시
* Person을 구현하는 클래스들은 talk, walk 인터페이스를 제공한다.
* Bird를 구현하는 클래스들은 tweet, fly 인터페이스를 제공한다.
  * 즉, Bird 인스턴스들은 Person의 인터페이스와 다른 인터페이스를 제공한다.
* 때문에 클라이언트 코드(이 경우, main 메소드)는 Person을 사용하듯이 Bird를 사용할 수 없다.
* 이러한 문제점은 Adapter 패턴으로 해결할 수 있으며, 이를 위해 Person을 구현하는 BirdAdapter 클래스를 정의한다.
  * BirdAdapter는 생성자를 통해 Bird 인스턴스를 전달 받고, 이를 내부 인스턴스 변수로 갖는다.
  * BirdAdapter는 Person과 같은 talk, walk 인터페이스 제공하며, 이를 재정의하는 과정에서 전달 받은 Bird 인스턴스를 적절히 활용한다.
```
public class AdapterMain {
    public static void main(String[] args) {
        Person woman = new Woman();
        woman.talk();
        woman.walk();

        Person man = new Man();
        man.talk();
        man.walk();

        Bird owl = new Owl();
        // 아래는 owl이 talk, walk 인터페이스를 제공하지 않으므로 실행 불가하다.
        // owl.talk();
        // owl.walk();
        owl.tweet();
        owl.fly();
        BirdAdapter birdAdapter = new BirdAdapter(owl);
        birdAdapter.talk();
        birdAdapter.walk();
    }
}

interface Person {
    void talk();
    void walk();
}
class Woman implements Person {
    @Override
    public void talk() {
        System.out.println("Woman talking");
    }
    @Override
    public void walk() {
        System.out.println("Woman walking");
    }
}
class Man implements Person {
    @Override
    public void talk() {
        System.out.println("Man talking");
    }
    @Override
    public void walk() {
        System.out.println("Man walking");
    }
}
interface Bird {
    void tweet();
    void fly();
}
class Owl implements Bird {
    @Override
    public void tweet() {
        System.out.println("Owl tweeting");
    }
    @Override
    public void fly() {
        System.out.println("Owl flying");
    }
}
class BirdAdapter implements Person {
    private final Bird bird;
    public BirdAdapter(Bird bird) {
        this.bird = bird;
    }
    @Override
    public void talk() {
        bird.tweet();
    }
    @Override
    public void walk() {
        bird.fly();
    }
}
```