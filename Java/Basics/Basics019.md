# Java
## 2022-01-15 Sat

### 람다식
* JDK 1.8에서 추가된 기능으로, **메소드를 하나의 식으로 표현**할 수 있게 한다.
* 람다식은 메소드를 간략하면서 명확한 식으로 표현할 수 있게 한다.
    * 메소드의 이름과 반환값이 없는 함수이므로, 익명 함수라고도 한다.
* 람다식을 사용했을 떄의 부수 효과는,
    1. 메소드를 추가하기 위한 클래스를 만들 필요 없이 메소드의 역할을 수행한다.
        * 메소드는 객체의 동작이나 행위를 의미하는 시멘틱이 강하며, 클래스에 종속적이다.
        * 람다식은 이러한 제약이 없으므로 함수라는 용어의 사용이 더 적합할 수 있다.
            * 정확하게는 **람다식은 익명 클래스의 인스턴스에 포함된 메소드처럼 동작**한다.
    2. 메소드의 매개 변수로 전달될 수 있다.
    3. 메소드의 결과로 반환될 수 있다.
        * 2.와 3.은 메소드(함수)가 변수처럼 취급되는 일급 함수의 특징을 말한다.

### 람다식의 작성
* 람다식은 익명 함수이므로 메소드의 이름과 반환 타입을 제거한다.
* 또한 매개 변수 선언부와 메소드 바디 사이에 화살표(->)를 추가한다.
```
(매개변수들) -> { 메소드 바디 }
```
* 반환 값이 있는 메소드 바디의 경우, return문 대신 식(expression)으로 대체할 수 있다.
    * 이 경우 식의 실행 결과가 자동으로 반환되며, 실행문이 아닌 식이므로 세미콜론(;)을 생략할 수 있다.
* 람다식에 선언된 **매개변수 타입은 추론이 가능한 경우에 생략이 가능**하다.
    * 사실상 대부분의 경우에 추론이 가능하므로, 많은 경우에 생략이 가능하다.
    * 람다식에 반환형이 없는 이유도 람다식의 결과가 항상 추론될 수 있기 때문이다.
* 선언된 매개 변수가 하나인 경우 괄호를 생략할 수 있다. 그러나 타입과 함께 명시한 경우에는 생략할 수 없다.
    * a -> a + 2는 가능하지만 int a -> a + 2는 불가능하다. 타입이 없어야 괄호를 생략할 수 있다.
* 메소드 바디의 문장이 하나인 경우, 중괄호를 생략할 수 있다.
    * **중괄호가 생략된 메소드 바디는 세미콜론(;)을 붙이지 않아야 한다**.
    * **메소드 바디의 문장이 return문인 경우에는 중괄호를 생략할 수 없다**.

### 함수형 인터페이스
* 람다식은 실제로는 익명 클래스의 객체와 동등하다. 즉, 실제로는 익명 클래스에 포함된 메소드이다.
* 상술한 이유에서, 람다식을 호출하려면 익명 클래스의 메소드를 호출하는 과정이 필요하다.
    * 그렇다면 익명 클래스를 사용하기 위한 참조 변수는 어떤 타입으로 선언해야 할까?
```
@FunctionalInterface
interface Temp {
  int add(int a, int b);
}

class Main {
  public static void main(String[] args) {
    Temp temp = new Temp() { // 익명 클래스를 활용한 인터페이스의 구현
      @Override
      public int add(int a, int b) {
        return a + b;
      }
    };
    System.out.println(temp.add(5, 10)); // 익명 클래스를 활용한 방식, 15

    Temp lambda = (a, b) -> a + b; // 익명 클래스를 활용한 인터페이스의 구현 절차를 람다식으로 대체
    System.out.println(lambda.add(3, 4)); // 람다식을 활용한 방식, 7
  }
}
```
* 위 예시에서 미루어보았을 때, 인터페이스를 구현한 익명 클래스는 람다식과 동등하다. 그렇기에 빌드가 되고, App.이 성공적으로 실행된다.
    * **이는 람다식이 실제로는 익명 클래스로부터 인스턴스화된 인스턴스이기 때문**이다.
    * 또한 이렇게 Temp 인터페이스를 구현한 익명 클래스의 add 메소드와 람다식의 반환형, 메소드 시그니쳐가 일치하기 때문이다.
* 또한 하나의 메소드만이 정의된 인터페이스를 구현한 익명 클래스의 인스턴스를 사용하여 람다식을 다루는 것은 Java의 특징에도 잘 맞는다.
* 결론적으로 **람다식은 인터페이스를 통해 다루어진다**.
    * 또한, 이렇게 **람다식을 다루기 위해 정의된 인터페이스를 Functional Interface(함수형 인터페이스)**라고 한다.
* **람다식을 다루기 위한 제약 사항으로, 함수형 인터페이스에는 하나의 추상 메소드만 정의**되어 있어야 한다.
    * 그래야만 람다식과 인터페이스의 메소드가 1:1로 매칭될 수 있기 때문이다.
    * 그러나 인터페이스의 static 메소드와 default 메소드에는 개수 제한이 없다.
    * 이러한 **함수형 인터페이스의 제약은 @FunctionalInterface 어노테이션을 통해 쉽게 검증할 수 있으므로, 적극적으로 사용**하도록 하자.
* 람다식은 일급 함수로서의 특징을 갖는다.
```
@FunctionalInterface
interface FunctionalTempInterface {
  int method(int a, int b);
}
class FI {
  void runLambda(FunctionalTempInterface temp) {
    int result = temp.method(3, 55);
    System.out.println(result);
  }
  FunctionalTempInterface getLambda() {
    return (a, b) -> a * b;
  }
}

class Main {
  public static void main(String[] args) {
    FI fi = new FI();

    fi.runLambda(fi.getLambda()); // 165
  }
}
```
* FunctionalTempInterface는 람다식을 사용하기 위해 하나의 메소드를 정의한 함수형 인터페이스이다.
* FI는 runLambda와 getLambda 두 메소드를 갖는다.
    1. runLambda: 함수형 인터페이스를 매개 변수로 받고, 형식에 맞게 int형을 반환받도록 메소드를 호출한 후 결과를 출력한다.
    2. getLambda: 람다식을 반환받는다.
* Main에서는 getLambda를 통해 람다식을 반환받고, 이 람다식을 그대로 runLambda에 전달한다.
* 이렇듯 **함수의 역할을 하는 람다식을 마치 변수처럼 메소드간에 주고 받도록** 할 수 있다.
    * 실제로는 익명 클래스의 인스턴스를 메소드 간에 주고 받는 것이지만, 람다식 개념이 도입됨으로써 식이 더 간결해지고 가독성이 좋아진다.
* 람다식의 타입이 함수형 인터페이스로 명시된 인터페이스와 일치하는 것은 아니다.
      * 그러나 인터페이스를 구현한 클래스의 인스턴스화 동일하므로 인터페이스로의 형변환을 허용하며, 이러한 형변환은 생략이 가능하다.
      * 인터페이스로 형변환된 람다식은 Object 클래스로 형변환이 가능하다. 람다식을 바로 Object 클래스로 형변환하는 것은 불가능하다.

### java.util.function
* 대부분의 메소드는 타입이나 매개 변수 정보, 반환형이 비슷하다.
* 이러한 사실을 전제로 java.util.function 패키지에는 **자주 쓰이는 형식의 메소드 형태를 기반으로 함수형 인터페이스를 정의**해두었다.
   * 매번 함수형 인터페이스를 정의하기보다는 해당 패키지의 인터페이스를 활용하는 것이 바람직하다.
* 매개 변수가 하나인 함수형 인터페이스:
   1. Supplier: 매개 변수 없이 하나의 값을 반환한다.
   2. Consumer: 매개 변수를 하나 받지만 값을 반환하지 않는다.
   3. Function: 매개 변수를 하나 받아 하나의 값을 반환한다.
      * 자식으로는 UnaryOperator 인터페이스가 있다.
      * UnaryOperator는 T apply(T t)형태이다. 
   4. Predicate: 매개 변수를 하나 받아 boolean을 반환한다.
      * 조건식 표현에 유용하다.
```
import java.util.function.Predicate;

class Main {
  public static void main(String[] args) {
    Predicate<Integer> isZero = num -> num == 0;

    System.out.println(isZero.test(0)); // test는 Predicate 인터페이스에 정의된 메소드이다.
    System.out.println(isZero.test(10)); // boolean test(T t);
    
    Function<Integer, String> toStr = num -> num.toString();
    System.out.println(toStr.apply(10)); // apply는 Function 인터페이스에 정의된 메소드이다.
    // R apply(T t);
  }
}
```
* Function과 Predicate의 차이는 조건식 표현에 시멘틱상 Predicate가 더 적합하다는 점이다.
* 매개 변수가 두 개인 함수형 인터페이스:
   * 이름 앞에 접두사 Bi가 붙는다.
   1. BiConsumer: 매개 변수를 두 개 받지만 값을 반환하지 않는다.
   2. BiPredicate: 매개 변수를 두 개 받아 boolean을 반환한다. 
   3. BiFunction: 매개 변수를 두 개 받아 하나의 값을 반환한다.
      * BinaryOperator: 자식 인터페이스이며, T apply(T t, T t)형태를 갖는다. 
* **두 개 이상의 매개 변수를 갖는 함수형 인터페이스는 직접 정의해서 사용**해야 한다.

### 컬렉션 프레임워크와 함수형 인터페이스
* 컬렉션 프레임워크의 인터페이스에 포함된 default 메소드 중 일부는 함수형 인터페이스를 사용한다.
* Collection
   * removeIf: Preficate를 받아 조건에 맞는 요소를 삭제한다.
* List
   * replaceAll: UnaryOperator를 받아 모든 요소를 변환하여 대체한다.
* Iterable
   * forEach: Consumer를 받아 모든 요소에 정의된 작업을 수행한다.
* Map
   * compute: key와 BiFunction을 받아 해당 key에 정의된 작업을 수행한다.
   * forEach: BuConsumer를 받아 모든 요소에 대해 정의된 작업을 수행한다.
   * replaceAll: BiFunction을 받아 모든 요소를 변환하여 대체한다.
   * merge: key, value, BiFunction을 받아 모든 요소에 대해 정의된 병합 작업을 수행한다.

### Function의 합성과 Predicate의 결합
* Function 인터페이스에는 default 메소드인 andThen과 compose가 제공된다.
   1. andThen: f.andThen(g)의 형태로 사용하여 g(f(x))의 새로운 Function을 반환한다.
   2. compose: f.compose(g)의 형태로 사용하여 f(g(x))의 새로운 Function을 반환한다.
* Predicate 인터페이스에는 default 메소드인 and, or, negate가 제공된다.
   * 여러 조건식을 논리 연산자로 묶어 하나의 식을 만들듯, 여러 Predicate를 묶어 하나의 새로운 Predicate로 만들어주는데에 사용한다.
   * p.and(q), p.or(q), p.negate() 형식으로 사용한다.
   * p.negate()는 조건식 전체에 대한 부정을 나타낸다.
* Predicate.isEqual(Object o) 두 대상을 비교한 Predicate의 생성에 사용한다.
   * 첫 비교 대상은 isEqual의 매개 변수이고, 두 번째 비교 대상은 test의 매개 변수로 넣어준다.
```
class Main {
  public static void main(String[] args) {
    System.out.println(Predicate.isEqual(10).test(20)); // 10과 20을 비교하며, 결과는 false이다.
  }
}
```

### 메소드 참조
* 메소드 참조는 메소드를 간단하게 표현하는 람다식을 한 단계 더 간소화한다.
* 메소드 참조는 람다식이 하나의 메소드만 호출하는 경우에 사용할 수 있다.
```
class Main {
  public static void main(String[] args) {
    List<Integer> list = new ArrayList<>();
    list.add(10);
    list.add(20);
    list.add(30);

    list.forEach(num -> System.out.println(num));
    list.forEach(System.out::println); // 메소드 참조가 적용되었다. 윗 줄과 기능이 같다!
  }
}
```
* 람다식의 매개 변수가 둘이여도, 하나의 메소드만 호출하는 경우라면 메소드 참조가 가능하다.
```
class Main {
  public static void main(String[] args) {
    BiFunction<String, String, Boolean> bf1 = (str1, str2) -> str1.equals(str2);
    BiFunction<String, String, Boolean> bf2 = String::equals;
  }
}
```
* 특정 클래스의 메소드를 호출하는 경우에도 참조 변수를 활용하여 메소드 참조가 가능하다.
```
class Main {
  public static void main(String[] args) {
    Calculator calculator = new Calculator();
    Function<Integer, Integer> f = calculator::add10; // 참조 변수 calculator를 반드시 명시한다.
  }
}

class Calculator {
  public Integer add10(Integer num) {
    return num + 10;
  }
}
```
* **하나의 메소드만 호출하는 람다식은 [클래스명]::[메소드명] 또는 [참조변수]::[메소드명]으로 바꾸어 간소화**할 수 있다.
* 생성자 역시 같은 원리에서 메소드 참조가 가능하며, **[클래스명]::new 형태로 작성**한다.
   * 이 때, 매개 변수가 있는 생성자인 경우 적절한 함수형 인터페이스를 사용해야 한다.
```
class Main {
  public static void main(String[] args) {
    Supplier<Calculator> s = Calculator::new; // 매개 변수가 없는 생성자를 사용한 경우.
    Function<String, Calculator> f = Calculator::new; // String 매개 변수를 받는 생성자를 사용한 경우.
  }
}

class Calculator {
  String name;
  Calculator() {
    this.name = "temp";
  }
  Calculator(String name) {
    this.name = name;
  }
}
```
* 메소드 참조는 람다식을 마치 static 변수처럼 사용할 수 있도록 해주며, 코드 간소화에 유용한 테크닉이다.
