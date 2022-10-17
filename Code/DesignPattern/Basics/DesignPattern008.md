# DesignPattern
## 2022-02-04 Fri

## Strategy Pattern
### 기본 개념
* 런타임에서 알고리즘을 선택하도록 하는 패턴이다.
* 전략 패턴이 적용된 함수는 변경되지 않으며, 어떤 동작을 수행할지 런타임에서 결정된다.

### 코드 예시
* 클래스의 동작 방식인 전략을 인터페이스로 정의하고(이 경우, Tarantino), 이를 구현하는 전략 클래스들을 작성한다.
* 전략에 따라 동작이 바뀌는 클래스를 정의하고(이 경우, Theater), setter로 전략을 수정할 수 있도록 한다.
  * 생성자를 통해 전략을 받아도 무방하지만, setter를 활용하는 것이 더 유연할 것 같다.
* 새로운 전략을 구현해야 하는 경우, 기존 클래스는 무엇도 수정하지 않고 전략 인터페이스를 구현하는 구현체를 추가한다.
  * 따라서 OCP에 위배되지 않는다.
```
public class StrategyMain {

    public static void main(String[] args) {
        Theater cgv = new Theater();
        cgv.setTarantino(new Django());
        cgv.playMovie();

        cgv.setTarantino(new Bastards());
        cgv.playMovie();
    }
}

interface Tarantino {
    void play();
}

class Django implements Tarantino {
    @Override
    public void play() {
        System.out.println("D is silent.");
    }
}
class Bastards implements Tarantino {
    @Override
    public void play() {
        System.out.println("Bingo!");
    }
}

class Theater {
    private Tarantino tarantino;

    public void setTarantino(Tarantino tarantino) {
        this.tarantino = tarantino;
    }

    public void playMovie() {
        tarantino.play();
    }
}
```