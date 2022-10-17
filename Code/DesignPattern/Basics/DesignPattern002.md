# DesignPattern
## 2022-02-02 Wed

## Factory Pattern
### 기본 개념
* 팩토리 패턴은 객체의 생성을 별도의 클래스에 위임하는 기법이다.
* 팩토리 패턴의 장점은 복잡한 인스턴스의 생성 방식을 클라이언트 코드에서 알 필요가 없게 만들 수 있다는 점이다.
  * 아래의 예시에서, 클라이언트 코드는 Factory 클래스의 main 메소드이다.
  * 아래의 예시는 간단하지만, 시스템에 따라서 인스턴스의 생성 과정은 얼마든지 복잡해질 수 있다.
* 팩토리 패턴은 메소드로 구현될 수도, 클래스로 구현될 수도 있다.

### 예시 코드
```
public class Factory {
    public static void main(String[] args) {
        AnimalFactory animalFactory = new AnimalFactory();

        Animal owl = create(Bird.OWL);
        owl.move();

        Animal duck = animalFactory.create(Bird.DUCK);
        duck.move();
    }

    // 메소드로 구현한 팩토리
    public static Animal create(Bird kind) {
        switch (kind) {
            case DUCK:
                return new Duck();
            case OWL:
                return new Owl();
            default:
                throw new IllegalStateException("Unexpected value: " + kind);
        }
    }
}

enum Bird {
    DUCK,
    OWL,
}

interface Animal {
    void move();
}

class Duck implements Animal {
    @Override
    public void move() {
        System.out.println("Duck Duck");
    }
}

class Owl implements Animal {
    @Override
    public void move() {
        System.out.println("Owl Owl");
    }
}

// 클래스로 구현한 팩토리
class AnimalFactory {

    public Animal create(Bird kind) {
        switch (kind) {
            case DUCK:
                return new Duck();
            case OWL:
                return new Owl();
            default:
                throw new IllegalStateException("Unexpected value: " + kind);
        }
    }
}

```

## Factory Method Pattern
### 기본 개념
* 팩토리 패턴은 팩토리에 의해 생성되는 객체가 일관된 과정을 통해 생성된다.
* 그러나 팩토리 자체에 새로운 기능을 추가하고 싶은 경우도 얼마든지 생길 수 있다.
  * 예를 들어, **생성되는 인스턴스마다 다른 기능을 추가로 적용하고 싶다면 기존의 팩토리 패턴으로는 구현이 어려울 수 있다**.
* **팩토리 메소드 패턴은 팩토리 자체를 인터페이스로 추상화하며, 생성 메소드를 자식 팩토리 클래스에서 구현하도록 유도**한다.
  * 때문에 구현된 팩토리에서는 팩토리 메소드를 포함하게 될 것이며, 상세한 생성 절차 및 추가 메소드 등을 자유롭게 구현할 수 있다.
  * 이러한 이유에서 해당 패턴에는 팩토리 메소드 패턴이라는 이름이 붙게 된다.

### 예시 코드
```
public class FactoryMethod {

    public static void main(String[] args) {
        DuckFactory duckFactory = new DuckFactory();
        OwlFactory owlFactory = new OwlFactory();

        Animal duck = duckFactory.create();
        duck.move();
        Animal duck2 = duckFactory.doSomethingToNewDuck();
        duck2.move();
        System.out.println(duckFactory.getCount());


        Animal owl = owlFactory.create();
        owl.move();
        owlFactory.printCount();
    }
}

interface Animal {
    void move();
}

class Duck implements Animal {
    @Override
    public void move() {
        System.out.println("Duck Duck");
    }
}

class Owl implements Animal {
    @Override
    public void move() {
        System.out.println("Owl Owl");
    }
}

interface OriginFactory {
    public Animal create();
}

class DuckFactory implements OriginFactory {
    int duckCount;
    @Override
    public Animal create() {
        this.duckCount++;
        return new Duck();
    }

    public int getCount() {
        return this.duckCount;
    }

    public Animal doSomethingToNewDuck() {
        System.out.println("Do something to duck!!!");
        return create();
    }
}

class OwlFactory implements OriginFactory {
    int owlCount;
    @Override
    public Animal create() {
        this.owlCount++;
        return new Owl();
    }

    public void printCount() {
        System.out.println(this.owlCount);
    }
}
```
* **팩토리 메소드 패턴의 핵심은 팩토리 메소드를 갖는 인터페이스**에 있다.
* 때문에 팩토리 메소드를 갖는 클래스들은 팩토리 메소드를 갖지만, 팩토리 이외의 기능을 포함하는 경우가 많다.
    * 즉, **팩토리 메소드를 갖는 클래스의 책임이 팩토리 기능이 아닌 경우가 많다**.