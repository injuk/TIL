# DesignPattern
## 2022-02-03 Thu

## Facade Pattern
### 기본 개념
* 파사드는 건물의 앞면을 의미한다.
* 이와 유사하게, 파사드 패턴은 복잡한 세부사항은 뒷면에 숨기고 간단한 전면 인터페이스만 제공하는 패턴이다. 
* 개발 과정에서 여러 라이브러리나 클래스를 조합하여 복잡하게 호출해야 하는 경우, 이 과정을 대리해주는 인터페이스용 클래스를 고려할 수 있다.
  * 결과 클라이언트는 복잡한 실제 호출 과정을 알 필요 없으며, 간단한 인터페이스를 통해 기능을 간단하게 활용할 수 있다.
* 파사드 패턴은 여러 클래스의 메소드를 묶는 것 외에도, 라이브러리들을 묶어 간단한 인터페이스를 제공하는 데에도 활용할 수 있다.

### 예시 코드
* 여러 클래스의 복잡한 호출을 Facade 클래스의 processing 메소드에서 모두 호출한다.
* 때문에 클라이언트(이 경우, FacadeMain) 코드에서는 복잡한 호출 절차 없이 processing 메소드만을 활용할 수 있다.
```
public class FacadeMain {
    public static void main(String[] args) {
        Facade facade = new Facade();
        facade.processing();
    }
}

class FirstProcess {
    public void step1() {
        System.out.println("FirstProcess: step1");
    }
    public void step2() {
        System.out.println("FirstProcess: step2");
    }
}
class SecondProcess {
    public void step1() {
        System.out.println("SecondProcess: step1");
    }
    public void step2() {
        System.out.println("SecondProcess: step2");
    }
    public void step3() {
        System.out.println("SecondProcess: step3");
    }
}
class ThirdProcess {
    public void step1() {
        System.out.println("ThirdProcess: step1");
    }
}
class Facade {

    private FirstProcess first;
    private SecondProcess second;
    private ThirdProcess third;

    public Facade() {
        this.first = new FirstProcess();
        this.second = new SecondProcess();
        this.third = new ThirdProcess();
    }

    public void processing() {
        first.step1();
        first.step2();

        second.step1();
        second.step2();
        second.step3();

        third.step1();
    }
}
```