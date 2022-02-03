# DesignPattern
## 2022-02-03 Thu

## Singleton Pattern
### 기본 개념
* 프로세스에서 하나의 인스턴스만 생성되도록 하는 방식이다.
* 프로세스에서 하나의 클래스를 여러 인스턴스로 만들었다면, 각 인스턴스는 다른 메모리 주소를 갖게 된다.
* 싱글톤 인스턴스는 일반적으로 static 인스턴스로, 애플리케이션의 시작부터 끝까지 유지된다.
  * 아무런 호출을 하지 않은 상태에는 null이지만, 첫 호출 이후에 인스턴스화가 이루어지게 된다.
  * 따라서 반환된 인스턴스들의 주소는 항상 같게 된다.
* 싱글톤의 핵심은 하나의 애플리케이션의 실행 도중 단 하나의 인스턴스만을 만들게 하는 것이다.
* 싱글톤은 일반적으로 인스턴스 하나가 리소스를 많이 차지하는 경우나, Network I/O 등에 활용된다.

### 코드 예시
* new로 생성한 경우와 getInstance 메소드를 사용한 경우로 나누어 작성하였다.
```
public class SingletonMain {
    public static void main(String[] args) {
        // 일반적인 호출
        Singleton singleton1 = new Singleton();
        Singleton singleton2 = new Singleton();
        System.out.println(Objects.equals(singleton1, singleton2));

        // 싱글톤 적용
        Singleton singleton3 = Singleton.getInstance();
        Singleton singleton4 = Singleton.getInstance();
        System.out.println(Objects.equals(singleton3, singleton4));
    }
}

class Singleton {
    public static Singleton singleton = null;

    public static Singleton getInstance() {
        if(Objects.isNull(singleton)) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```