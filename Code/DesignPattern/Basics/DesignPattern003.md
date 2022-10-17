# DesignPattern
## 2022-02-02 Wed

## Abstract Factory Pattern
### 기본 개념
* 팩토리 패턴과 팩토리 메소드 패턴은 모두 인스턴스화를 위한 방법을 다루었다.
* 추상 팩토리 패턴은 서로 관련이 있는 클래스들을 여러 방식으로 인스턴스화할 수 있을 때 유용하다.
```
        Product A   Product B
Class A     A           B
Class B     A           B
Class C     A           B
```
* Class A는 Product A에 맞출 수도 있고, Product B에 맞출 수도 있다.
  * Class B, C도 마찬가지로 Product 종류에 따라 달리 커스터마이징될 수 있다.
* 만약 Class A, B, C 모두가 상술한 조건에 맞는다면, 
  1. Product A에 들어가기 위한 A, B, C의 인스턴스 집합
  2. Product B에 들어가기 위한 A, B, C의 인스턴스 집합
  * 은 분명 공통 분모를 갖지만 세세한 구현에서 달라질 수 있다.

### 컴퓨터 예시
* 더 구체화한다면 컴퓨터 예시를 들어볼 수 있다.
```
         Samsung        Apple
CPU         A           B
VGA         A           B
RAM         A           B
```
* CPU, VGA, RAM은 Samsung과 Apple 집합을 위해 필수적이지만, A와 B에 따라 사양에 차이가 있을 수 있다.
* 이렇듯 **공통 분모를 갖는 인스턴스 집합을 만드는 경우라면 하나의 팩토리에서 여러 인스턴스를 만드는 것이 편리**하다.
* 상술한 예시에 맞는 추상 팩토리 패턴은 다음과 같은 순서로 구현해볼 수 있다.
  1. 추상 팩토리 인터페이스를 만든다.
  2. 추상 팩토리 인터페이스에는 CPU, VGA, RAM을 생성하기 위한 추상 메소드를 정의한다.
  3. Samsung, Apple 집합을 만들기 위한 각각의 팩토리 클래스를 추상 팩토리로부터 구현한다.
  4. 각각의 팩토리 클래스에서 추상 메소드를 구현한다.
* 구현된 팩토리 클래스는 각 회사에 맞는 CPU, VGA, RAM을 인스턴스화하게 된다.

### 부품 별 인터페이스화
* 반면 CPU, VGA, RAM에 주목해본다면, Samsung과 Apple용 제품은 세세한 세부사항을 차치하고 개념을 추상화할 수 있다.
  * 즉, **각 부품은 공통의 기능을 갖기 때문에 인터페이스화될 수 있다**.

### 예시 코드
```
public class AbstractFactoryMain {

    public static void main(String[] args) {
        AbstractFactory samsung = new SamsungFactory();
        AbstractFactory apple = new AppleFactory();

        createComputer(samsung);
        createComputer(apple);
    }

    public static void createComputer(AbstractFactory abstractFactory) {
        abstractFactory.createCpu().getClock();
        abstractFactory.createVga().getGpu();
        abstractFactory.createRam().getStorage();
    }
}

interface CPU {
    void getClock();
}
class SamsungCpu implements CPU {
    @Override
    public void getClock() {
        System.out.println("Samsung CPU");
    }
}
class AppleCpu implements CPU {
    @Override
    public void getClock() {
        System.out.println("Apple CPU");
    }
}

interface VGA {
    void getGpu();
}
class SamsungVga implements VGA {
    @Override
    public void getGpu() {
        System.out.println("Samsung VGA");
    }
}
class AppleVga implements VGA {
    @Override
    public void getGpu() {
        System.out.println("Apple VGA");
    }
}

interface RAM {
    void getStorage();
}
class SamsungRam implements RAM {
    @Override
    public void getStorage() {
        System.out.println("Samsung RAM");
    }
}
class AppleRam implements RAM {
    @Override
    public void getStorage() {
        System.out.println("Apple RAM");
    }
}

interface AbstractFactory {
    CPU createCpu();
    VGA createVga();
    RAM createRam();
}
class SamsungFactory implements AbstractFactory {
    @Override
    public CPU createCpu() {
        return new SamsungCpu();
    }

    @Override
    public VGA createVga() {
        return new SamsungVga();
    }

    @Override
    public RAM createRam() {
        return new SamsungRam();
    }
}
class AppleFactory implements AbstractFactory {
    @Override
    public CPU createCpu() {
        return new AppleCpu();
    }

    @Override
    public VGA createVga() {
        return new AppleVga();
    }

    @Override
    public RAM createRam() {
        return new AppleRam();
    }
}
```

### 결론
* 추상 팩토리 패턴은 각각 공통된 형태, 또는 테마를 갖는 인스턴스의 집합을 하나의 팩토리에서 인스턴스화하기 위해 사용한다.
  * 이 때, 팩토리 자체를 추상화하므로 인스턴스를 쉽게 생성할 수 있다.
* 만약 어떤 **인스턴스 집합이 여러 종류로 나뉘고, 상술한 예시와 같은 표(또는 테이블)의 형태를 가질 수 있다면 추상 팩토리 패턴의 적용을 고려**할 수 있다.

### 개인 리마인드용 - 팩토리, 팩토리 메소드, 추상 팩토리 패턴의 차이
* 팩토리: 단순히 인스턴스의 생성 책임만을 다른 메소드 또는 클래스에 넘긴다.
* 팩토리 메소드: 인스턴스의 생성 책임을 인스턴스 별로 구분된 팩토리 클래스에 넘기며, 각각의 팩토리 클래스는 팩토리 별 추가 기능을 자유롭게 구현할 수 있다. 
  * 팩토리 메소드 역시 팩토리 자체가 추상화되나, 단일 인스턴스를 생성하는 점에서 추상 팩토리와 다르다.
  * 때문에 추상화의 초점은 팩토리 자체가 아닌, 팩토리가 생성하는 인스턴스에 초점이 맞춰진다.
* 추상 팩토리: 공통된 특징이 있는 인스턴스 집합을 여러 형태로 생성할 필요가 있을 경우, 팩토리 자체를 추상화햐여 쉽게 인스턴스 집합을 생성할 수 있도록 한다.
  * 추상 팩토리 역시 팩토리 자체가 추상화되나, 공통된 특징을 인터페이스로 묶을 수 있는 클래스들의 집합을 생성하는 팩토리에 추상화 초점이 맞춰진다.
  * 추상 팩토리에서 생성되는 인스턴스 집합이 인터페이스와 같이 추상화되어 있다면, 어떠한 종류의 인스턴스 집합이든지 추상 팩토리로부터 구현할 수 있다.