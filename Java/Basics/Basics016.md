# Java
## 2022-01-12 Wed

### enums
* enums(열거형)는 JDK 1.5부터 추가된 개념이며, **서로 관련된 상수를 편리하게 선언 및 사용하기 위해** 제공되는 기능이다.
* typesafe하므로, 값이 같아도 타입이 다르면 에러가 발생한다.
* Java에서는 원칙적으로 상수의 값이 바뀌면 해당 상수를 참조하는 모든 소스가 다시 컴파일되어야 한다.
* 그러나 **enums의 상수를 사용한다면 기존의 소스를 다시 컴파일할 필요가 없다**.
* enums에 정의된 상수에 접근하려면 마치 static 멤버처럼 [열거형이름].[변수명]으로 사용한다.
```
public class EnumTest {
    public static void main(String[] args) {
        Tester tester = new Tester();
        System.out.println(tester.testEnum); // ONE
    }
}

class Tester {
    TestEnum testEnum;

    Tester() {
        testEnum = TestEnum.ONE;
    }
}

enum TestEnum {
    ONE,
    TWO,
    THREE,
}
```
* enums는 switch의 조건식에도 사용할 수 있다. 이 때, [열거형].[변수]가 아닌 아래와 같은 형식으로 작성한다.
```
public class EnumTest {
    public static void main(String[] args) {
        Tester tester = new Tester();
        switch (tester.testEnum) {
            case ONE: break;
            case TWO: break;
        }
    }
}
// 생략
```
* 내부적으로 열거형 상수는 각각의 열거형 인스턴스(위 예시의 경우, TestEnum 타입 인스턴스)이다.
  * 각 상수는 static final 형식으로 인스턴스의 주소를 저장한다.
  * 이 값은 변하지 않으므로, 열거형 상수 간 비교는 비교 연산자 ==를 사용할 수 있다. 
    * 즉, equals를 사용하지 않더라도 비교가 가능하다.

### enums에 값 추가하기
* 필요하다면 enums의 상수에 값을 추가할 수 있다. 
  * 이 경우, 반드시 **enums의 상수에 추가한 값에 대응되는 enum의 멤버 변수와 생성자의 추가가 필요**하다.
```
public class EnumTest {
    public static void main(String[] args) {
        System.out.println(TestEnum.THREE.getStr());
    }
}

enum TestEnum {
    ONE("I"),
    TWO("II"),
    THREE("III"); // 반드시 세미 콜론을 사용한다.

    String str; // 열거형에 String이 할당되었으므로, 이 값을 내부적으로 유지할 수 있는 변수를 선언한다.
    TestEnum(String str) { // 지정된 값을 저장할 수 있도록 생성자도 정의한다.
        this.str = str;
    }

    public String getStr() {
        return this.str; // 열거형에 저장된 String 값을 반환할 수 있도록 getter를 정의한다.
    }
}
```
* **enums에 생성자를 추가하였으나, 다른 클래스들과 같이 생성자를 통한 인스턴스화는 불가능**하다.\
  * enums의 생성자는 암시적으로 private이기 때문이다.
* 열거형 상수에 할당하는 값은 하나 이상일 수 있다. 이 경우, 할당된 값과 같은 형태의 생성자 및 멤버 변수가 추가되어야 한다.
```
public class EnumTest {
    public static void main(String[] args) {
        System.out.println(TestEnum.SEVEN.getStr());
        System.out.println(TestEnum.SEVEN.getNum());
    }
}

enum TestEnum {
    ONE("I", 1),
    TWO("II", 2),
    THREE("III", 3),
    SEVEN("VII", 7); // 하나 이상의 값을 할당할 수 있다.

    String str;
    int num; // 추가된 값에 대응되는 멤버 변수를 추가한다.
    TestEnum(String str, int num) { // 열거형 상수에 할당한 값과 같은 형태의 생성자를 추가해야 한다.
        this.str = str;
        this.num = num;
    }

    public String getStr() {
        return this.str;
    }
    public int getNum() {
        return this.num;
    }
}
```

### enums와 메소드
* enums에도 메소드를 추가할 수 있다. 이 때, **enums의 메소드는 반드시 추상 메소드**여야 한다.
  * 각 열거형 상수는 **추상 메소드를 반드시 구현**해야 한다.
```
public class EnumTest {
    public static void main(String[] args) {
        TestEnum.ONE.hello();
        TestEnum.TWO.hello();
        TestEnum.THREE.hello();
        TestEnum.SEVEN.hello();
    }
}

enum TestEnum {
    ONE("I", 1) { // [상수]([인자]) { 메소드 구현 } 형태를 따른다.
        void hello() {
            System.out.println("[ONE]: " + this.str);
            System.out.println("[ONE]: " + this.num);
        }
    },
    TWO("II", 2) {
        void hello() {
            System.out.println("[TWO]: " + this.str);
            System.out.println("[TWO]: " + this.num);
        }
    },
    THREE("III", 3) {
        void hello() {
            System.out.println("[THREE]: " + this.str);
            System.out.println("[THREE]: " + this.num);
        }
    },
    SEVEN("VII", 7) {
        void hello() {
            System.out.println("[SEVEN]: " + this.str);
            System.out.println("[SEVEN]: " + this.num);
        }
    };

    String str;
    int num;
    TestEnum(String str, int num) {
        this.str = str;
        this.num = num;
    }

    abstract void hello(); // 추상 메소드의 정의. 각 열거형 상수는 이를 반드시 구현해야 한다.
}
```
* 추상 메소드이므로, 열거형 상수마다 메소드를 달리 구현하는 것으로 각각 별개의 동작을 수행할 수 있다. 
  * 아쉽게도 enums에 메소드를 작성할 일은 많지 않다.
