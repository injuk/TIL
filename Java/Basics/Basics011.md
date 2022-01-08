# Java
## 2022-01-08 Sat

### 에러, 예외
* 에러는 크게 두 종류로 나뉜다.
  1. 컴파일 에러: 컴파일 단계에서 발생하는 에러
  2. 런타임 에러: 컴파일은 성공했지만, App.의 실행 도중 발생하는 에러
* 이 중 런타임 에러는 다시 에러와 예외로 나뉜다.
  1. 에러: App.에 작성한 코드에 의해 조치될 수 없는 오류
  2. 예외: 개발자의 적절한 코드 구현을 통해 비정상적인 App.의 종료를 막을 수 있는 오류
* Java에서는 런타임 에러 또한 각각 Error와 Exception 클래스로 정의해두었다.
  * 이 때, 둘 모두 Object 클래스를 상속받는다.

### Exception, RuntimeException
* 모든 '예외'의 최고 조상 클래스이며, 크게 다음과 같은 두 종류로 나뉜다.
  1. Exception 클래스 및 자식 클래스들
  2. RuntimeException 클래스 및 자식 클래스들
    * RuntimeException 역시 Exception의 자식이지만, 편의상 이렇게 분류하였다.
* RuntimeException 클래스들은 주로 개발자의 실수에 의해 발생되는 예외이다.
  * 예를 들어 0 나누기 예외가 있다.
* Exception 클래스들은 주로 외부의 영향으로 발생하는 예외이며, 사용자들의 동작에 의해 발생하는 경우가 많다.
  * 예를 들어 FileNotFoundException 등 파일을 찾지 못하거나, 입력 데이터 형식이 잘못 된 경우가 있다.

### 예외 처리
* 예외 처리는 App. 실행 중 발생할 수 있는 예외에 대비하는 코드를 작성하는 것을 말한다.
* 적절한 예외 처리는 예외가 발생하더라도 App.을 종료시키지 않고, 계속해서 실행되도록 한다.
* 예외 처리되지 않은 예외는 uncaught exception으로서 JVM의 예외 처리기(UncaughtExceptionHandler)가 처리한 후 화면에 원인을 출력한다.
* 예외 처리를 위한 코드는 다음과 같은 형식을 사용한다.
```
try {
} catch (Exception e) {
}
```
* 하나의 try에는 여러 종류의 예외를 처리할 수 있도록 하나 이상의 catch 블록이 자리할 수 있다.
* 이 때, 발생한 예외의 종류와 ㅊ일치하는 단 하나의 catch 문만이 실행된다.
  * break가 없는 switch문처럼 동작하지는 않는다는 의미이다. catch는 최대 하나만 실행된다!
* 발생한 예외와 일치하는 catch 블록이 없다면 예외는 처리되지 않는다.
* try 블록 또는 catch 블록 내부에서도 예외가 발생할 수 있으므로, 별도의 try - catch 문을 포함하도록 작성할 수 있다.
* try 블록 내부에서 예외가 발생하면, 발생한 예외와 일치하는 예외를 처리하는 catch 블록을 찾는다.
  * 있으면, 해당 catch 블록의 코드를 수행한다.
  * 없으면, 해당 예외는 처리되지 못한다.
  * 예외가 발생한 시점에서 try 문을 벗어나므로, try 블록에서 예외가 발생한 직후의 코드부터는 실행되지 않는다.
      * 때문에 try 블록에 포함시킬 코드의 범위를 적절히 결정지을 수 있어야 한다.
* **예외가 발생하면 발생한 예외에 해당하는 인스턴스가 생성**된다.
  * 예외가 발생한 문장이 try 블록에 있었다면, 예외를 처리할 수 있는 catch 블록을 찾는 식으로 동작하게 된다.
* catch 블록의 괄호에는 처리하고자 하는 예외 타입의 참조 변수를 선언해두어야 한다.
* try 블록에서 예외가 발생하면, try와 가장 가까운 catch 블록부터 하나씩 instanceof 연산자로 확인하며 검사하고, 검사 결과가 true인 catch 블록에서 해당 예외를 처리한다.
  * 때문에 Exception 클래스를 처리하도록 작성된 catch 블록은 try 블록에서 발생한 모든 예외를 catch한다.

### printStackTrace, getMessage 메소드
* 예외 발생시 생성되는 예외 인스턴스에는 발생한 예외에 대한 정보가 담겨 있다.
* printStackTrace와 getMessage 메소드는 예외 인스턴스에 담긴 예외 관련 정보를 확인하기 위해 호출할 수 있다.
  * printStackTrace: 예외 발생시 call stack에 있었던 메소드의 정보와 예외 메시지를 출력한다.
  * getMessage: 예외 발생시 인스턴스가 갖고 있는 메시지를 반환한다.
* try - catch와 같은 예외 처리의 궁극적인 목적은 예외에 의한 App.의 비정상 종료를 막는 데에 있는 반면, 위 두 메소드는 catch 블록에서 예외의 원인을 확인할 수 있도록 돕는다.
* 아래의 코드를 실행해보자.
```
class Main {
  public static void main(String[] args) throws Exception {
     first();
  }
  static void first() throws Exception {
      second();
  }
  static void second() throws Exception {
      third();
  }
  static void third() throws Exception {
      throw new Exception("Bye World!");
  }
}
```
* 이 경우, 아래와 같은 stackTrace를 확인할 수 있다.
```
Exception in thread "main" java.lang.Exception: Bye World!
	at Main.third(Main.java:12)
	at Main.second(Main.java:9)
	at Main.first(Main.java:6)
	at Main.main(Main.java:3)
```
* 이러한 에러 메시지는 다음과 같은 의미를 갖는다.
  1. 예외 발생 시점에 call stack에는 third, second, first, main 메소드가 있었다.
  2. 예외가 실제로 발생한 곳은 stackTrace의 가장 윗 줄인 third 메소드이다.
  3. 메소드의 호출 흐름은 가장 아래의 main으로부터 > first > second > thrid 순이다.

### multi catch
* catch 블록에서 예외를 처리할 때, 처리 절차가 동일하다면 여러 catch 블록을 '|' 기호를 통해 하나로 합칠 수 있다.
```
try {}
catch(ExceptionA ea) { ea.printStackTrace(); }
catch(ExceptionB eb) { eb.printStackTrace(); }
// 위 내용은 아래와 같이 | 기호를 통해 줄일 수 있다.
try {}
catch(ExceptionA | ExceptionB e) { e.printStackTrace(); }
```
* 이 때, 두 예외가 부모 자식 관계라면 컴파일 에러가 발생한다.
  * 위 경우라면 부모 예외 클래스만 활용하는 식으로 수정되어야 한다.

### throw
* 개발자가 throw 문을 통해 고의로 예외를 발생시킬 수 있다.
* 이 경우, new 연산자를 통해 예외 인스턴스를 생성한 후 throw를 통해 해당 인스턴스(또는 참조 변수)를 명시하면 예외가 발생된다.
  * Exception 클래스는 생성자에 문자열을 인자로 전달하는 것으로 Exception 인스턴스의 메시지를 설정할 수 있다.
  * Exception 인스턴스의 메시지는 getMessage() 메소드를 통해 확인할 수 있다.
```
Exception e = new Exception("Test Exception");
throw e;
// 또는 throw new Exception("Test Exception");
```

### checked, unchecked
* 아래의 코드를 확인해보자.
```
class Main {
  public static void main(String[] args) {
    // throw new Exception("Test"); // 1.
    // throw new RuntimeException("Test"); // 2.
  }
}
```
* 1.과 2.를 각각 주석해제한 후 컴파일하면 결과는 다음과 같다.
  * 1.을 주석해제할 경우 컴파일이 되지 않는다.
  * 2.를 주석해제할 경우, 정상적으로 컴파일되나 실행 중 에러가 발생한다.
* 위 두 줄 모두 명백하게 예외를 발생시킬 것이 분명하며, 1.은 예외 처리를 하지 않으면 컴파일 자체도 실패하는 것으로 보아 예외 처리가 강제되는 것을 알 수 있다.
* 그러나 2.의 경우에서 미루어 보았을 때, **개발자의 실수로 발생하는 런타임 에러는 예외 처리를 강제하지 않는 것**을 알 수 있다.
  * 만약 Exception처럼 RuntimeException도 예외 처리를 강제한다면, 참조 변수를 사용하는 모든 곳에서 try - catch 블록을 사용해야 했을 것이다.
* 이렇듯 컴파일러 단에서 예외 처리 여부를 확인하는 Exception 클래스들은 '**checked 예외**'라고 한다.
* 반면 RuntimeException 클래스와 마찬가지로 컴파일러가 예외 처리를 확인하지 않는 클래스들은 '**unchecked 예외**'라고 한다.

### throws
* Java의 예외 처리 방법은 크게 다음과 같은 두 가지가 있다.
  1. try - catch 블록 사용
  2. 메소드 선언부에 throws [예외 이름] 작성
* 이러한 throws 키워드는 **해당 메소드에서 발생할 수 있는 예외의 목록을 명시**한다.
* 메소드의 선언 부에 throws를 통해 발생 가능한 예외가 작성되므로, **메소드의 사용자는 어떤 예외들이 처리되어야하는지 명시적으로 이해**할 수 있다.
  * 클래스 및 메소드는 작성 하는 사람은 물론 호출하는 사람까지 고려해야한다는 사실을 기억하자.
* 기존의 많은 언어들은 메소드 선언부에 예외를 명시하지 않으므로, 이 메소드를 사용할 때 어떤 예외를 처리해야하는지 대비가 어려운 경우가 많았다.
* 그러나 Java의 경우, 메소드 작성자가 **예외의 종류를 미리 메소드 선언부에 작성하는 것으로 메소드 호출자가 어떤 예외를 처리해야하는지 강요**할 수 있다.
* 자연스레 개발 속도가 향상되며, 예외에 보다 안전한 App.을 작성하는 것이 가능해진다.
  * 때문에 throws Exception과 같이 예외의 최고 조상을 발생시키는 메소드는 오버라이딩만 어렵게 하는 효과를 낳을 수 있다.
  * 메소드 오버라이딩시 부모 클래스에 작성된 메소드보다 더 적은 예외를 던질 수 없는 것을 기억하자.
* 즉, 메소드 작성자는 throws 키워드를 통해 해당 메소드 사용시 처리되어야 하는 예외의 종류를 명시하고,
* 메소드 호출자는 해당 메소드에서 발생할 수 있는 예외를 확인하고 적절한 조치를 취해야한다는 것이 throws의 의의이다.
* 때문에 **throws는 자신이 예외를 처리하는 것이 아니라 자신을 호출한 메소드에게 예외를 전달하는 것으로 예외 처리 책임을 떠넘기는 것**으로도 이해할 수 있다.
  * 이렇게 전달된 예외는 다시 다른 메소드에게 전달될 수 있으며, 최종적으로 main 메소드에서도 예외가 처리되지 않는 경우 App.이 비정상적으로 종료되게 된다.
  * 결국 **어느 한 메소드는 try - catch 블록을 활용하여 실제로 예외를 처리해주어야** 한다.
* 만약 자신이 작성한 메소드에 대한 API 문서를 작성한다면, 다음과 같은 사항을 고려해볼 수 있다.
  * 발생 가능한 예외가 Exception의 자식인 경우: 반드시 예외 처리가 되어야 하므로, throws를 통해 이를 분명히 명시한다.
  * 발생 가능한 예외가 RuntimeException의 자식인 경우: 반드시 예외 처리가 되어야 하지는 않으므로, throws를 통해 이를 명시하지 않아도 된다.
    * 그러나 해당 예외가 발생할 수 있음을 분명히 명시해두는 것이 좋다.
* 이렇듯 throws 키워드를 통해 메소드에 명시되는 예외는 일반적으로 **반드시 처리되어야 하는 예외**들이다.

### try - catch와 throws
* 메소드 자체적으로 try - catch를 통해 예외 처리를하면, 해당 메소드를 호출한 메소드는 예외 발생 사실을 인지할 수 없다.
* 반면 throws를 통해 자신을 호출한 메소드에게 예외를 떠넘기면, 메소드 호출자는 예외가 발생한 메소드를 호출한 라인에서 에러가 발생한 것으로 간주하고 예외 처리를 수행하게 된다.
  * 물론 이는 메소드 호출자에 try - catch 블록을 활용한 적절한 예외 처리가 되어 있는 경우의 이야기다.
* 결국, 두 방식 중 어떤 것을 사용하는지는 다음의 항목으로 결정할 수 있다.
  1. 메소드 내에서 자체적으로 예외를 처리해도 된다면 try - catch를 사용한다.
  2. 메소드를 호출한 다른 메소드가 발생한 예외에 대한 처리 책임이 있다면, throws 키워드를 통해 예외를 전달하고 메소드 호출자가 처리하도록 작업되어야 한다.

### finally
* 예외 발생 여부와 관계 없이 실행되어야할 코드를 작성한다.
* try - catch 블록의 마지막에 선택적으로 붙여 사용한다.
  * finally 블록을 사용하고자 하는 경우, 구조는 try - catch - finally가 된다.
  * try 또는 catch 블록에 명시적인 return;을 활용한 명시적인 메소드 종료가 있더라도 finally 블록의 코드가 실행된 후에 메소드가 종료된다.
* 예외가 발생하는 경우, 코드는 try > catch > finally 순으로 실행된다.
* 예외가 발생하지 않은 경우, 코드는 try > finally 순으로 실행된다.

### 사용자 정의 예외
* 필요에 따라 개발자가 자신의 예외를 정의하여 사용할 수 있다.
* 일반적으로 Exception 또는 RuntimeException을 상속받아 클래스를 작성한다.
  * 기존에는 Exception 클래스를 상속하는 checked 예외를 권장하였으나, 최근에는 try - catch 블록의 복잡함을 유발하는 checked 예외보다 RuntimeException을 상속받는 것이 추세라고 한다.
    * 이는 즉 필요에 따라 예외 처리 여부를 선택할 수 있는 unchecked 예외가 더 환영받고 있다는 의미이다.
  * 그래도 **되도록이면 자신만의 독창적인 예외 클래스를 작성하여 사용하는 것보다는 기존의 예외 클래스를 사용**하자.

### 메소드 호출자에서 예외 처리하기
* try - catch 블록을 통해 예외를 처리한 후, throw를 통해 예외를 다시 자신을 호출한 메소드에게 던질 수 있다.
* 이 때, catch 블록에서 발생한 예외를 적절히 처리한 후 throw 를 통해 예외를 메소드 호출자에게 전달하므로, 메소드 선언부에 throws 키워드가 작성되어야 한다.
* 이는 특정한 예외가 메소드 자체적으로도 처리되어야 하고, 메소드를 호출한 메소드 호출자에서도 처리될 필요가 있을 때 사용한다.

### chained exception
* 한 예외가 다른 예외를 발생시키는 경우를 말한다.
* 이는 Exception 클래스의 부모인 Throwable 클래스에 정의된 initCause()를 활용한다.
```
// 인자로 전달된 예외를 원인 예외로 등록한다.
Throwable initCause(Throwable cause)
// 원인 예외를 반환한다.
Throwable getCause()
```
* 예외의 연결 처리는 여러 예외를 하나의 예외로 묶어 처리하기 위함이다.
* 만약 내가 처리할 예외를 묶는 공통의 부모 클래스를 만들게 되면, catch 블록에서 실제로 발생한 예외를 알기 위해 instanceof를 반복해야 한다.
  * 또한 기존에 사용하던 예외의 상속 관계를 변경하는 것도 어렵다.
* 때문에 예외가 원인 예외를 포함할 수 있게 하였으며, 이 경우 예외들이 상속을 주고 받을 필요가 없어진다.
* 또한 이는 **checked 예외를 unchecked 예외로 바꿀 수 있다**는 장점이 있다.
* 처리할 필요 없는 checked 예외의 처리를 위해 무의미한 try - catch 블록을 작성하고 있었다면, 이를 uncheked 예외로 바꾸어 예외 처리를 강제하지 않도록 수정할 수 있다.
  * 이 때, initCause 메소드 대신 RuntimeException 클래스의 생성자를 사용할 수 있다.  
```
RuntimeException(Throwable cause)
```
* 다음은 위를 활용하여 checked 예외를 unchecked 예외로 래핑하는 예시이다.
```
class Main {
  public static void main(String[] args) {
     // initCause() 대신 RuntimeException의 생성자를 활용하여 원인 예외를 등록하였다.
     // 이러한 방식은 checked 예외를 unchecked 예외로 래핑할 수 있도록 하여 예외 처리를 강제하지 않는다.
     throw new RuntimeException(new Exception("Test!"));
  }
}
```
