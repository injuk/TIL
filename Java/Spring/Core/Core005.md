# Core
## 2022-06-24 Fri

### 싱글톤 컨테이너
* 스프링은 기본적으로 기업의 서비스 애플리케이션을 지원하기 위해 탄생하였다.
  * 대부분의 스프링 애플리케이션은 웹 애플리케이션이지만, 굳이 웹이 아닌 애플리케이션도 개발이 가능하다.
  * 반면, **기업용 웹 애플리케이션은 태생적으로 많은 고객이 동시에 요청을 전달하기 쉽다**.

### 기존 방식의 한계
* 스프링 없이 구현한 순수 DI 컨테이너 예시의 경우, 매 의존성 주입 요청마다 새로운 객체가 생성되는 한계가 존재한다.
  * 예를 들어 아래의 테스트 코드는 `MemberService`를 요청할 때마다 새로운 객체가 생성되어 반환되므로 반드시 실패한다.
```
@Test
@DisplayName("스프링 없는 순수한 DI 컨테이너의 한계.")
void pureContainer() {
    AppConfig appConfig = new AppConfig();

    MemberService memberService1 = appConfig.memberService();
    MemberService memberService2 = appConfig.memberService();

    System.out.println("memberService1 = " + memberService1);
    System.out.println("memberService2 = " + memberService2);
    
    assertThat(memberService1).isEqualTo(memberService2);
}
```
* 이 경우, 매 사용자의 요청마다 새로운 객체가 생성되어 JVM 메모리에 올라가게 된다.
  * 때문에 **기존 방식에서는 초당 1000건의 요청이 발생한다면 새로운 객체는 매 초 1000개가 생성되어야하며, 이는 매우 비효율적인 방식**이다.
  * 엎친 데 덮친 격으로, 만약 각 의존성이 또 다른 의존성 객체를 요청한다면 초당 생성되는 객체 수는 기하급수적으로 늘어갈 수 있다.
* 이렇듯 기존 방식은 메모리 낭비가 매우 심하므로, 싱글톤 패턴을 도입하여 문제를 해결할 필요성이 있다.
  * **싱글톤 패턴을 도입하는 경우, 객체가 항상 단 1개만 생성되는 것을 보장하여 모든 사용자의 요청에서 같은 객체를 공유하도록 할 수 있다**.

## 2022-06-25 Sat
### 싱글톤 패턴
* 싱글톤은 패턴이 적용된 클래스의 객체가 단 하나만 생성되는 것을 보장하는 패턴이다.
  * 이는 바꿔 말해 2개 이상의 객체가 존재하지 못하도록 막을 수 있어야 한다.
  * 예를 들어, 아래의 코드와 같이 생성자를 private으로 두어 외부에서 인스턴스화할 수 없도록 막을 수 있다.
```
public class SingletonService {
    private static final SingletonService instance = new SingletonService();

    private SingletonService() {}

    public static SingletonService getInstance() {
        return instance;
    }

    public void doSomething() {
        System.out.println("do something!!!");
    }
}
```
* 상술한 클래스는 클래스 로더에 의해 클래스가 로드되는 시점에 `instance`를 미리 하나 생성해둔다.
  * 해당 객체의 참조는 오로지 `getInstance()` 메소드를 통해서만 얻을 수 있다.
  * **생성자는 private으로 설정되어 외부에서 호출할 수 없으므로, `new SingletonService()`와 같은 호출은 불가능**하다.
* 싱글톤의 가장 큰 이점은 클래스의 인스턴스화에 드는 비용이 클수록 효율성이 더 좋아진다는 점이다.
  * **극단적으로, 객체를 생성하는데 드는 비용이 1000이라면 단순히 객체를 참조하는 비용은 1 정도로 생각해도 무방**하다.
* 상술한 객체는 아래와 같은 테스트 코드를 통해 동일성을 검증할 수 있다.
  * isSameAs는 Java의 `==` 비교를 통한 동일성을 검사한다.
  * 반면, isEqualTo는 `equals()` 메소드를 통한 검증을 의미한다.
```
@Test
@DisplayName("싱글톤 객체는 유일성이 보장된다.")
void singletonInstanceTest() {
    SingletonService singletonService1 = SingletonService.getInstance();
    SingletonService singletonService2 = SingletonService.getInstance();

    System.out.println("singletonService1 = " + singletonService1);
    System.out.println("singletonService2 = " + singletonService2);
    System.out.println(singletonService1 == singletonService2);

    assertThat(singletonService1).isSameAs(singletonService2);
}
```
* 여기까지 싱글톤에 대해 익혔을 때, AppConfig의 모든 메소드에도 동일한 방식을 적용하여 `new` 연산자가 아닌 `getInstance()`를 활용하고 싶을 수 있다.
  * 그러나 **스프링 컨테이너는 기본적으로 모든 빈 객체를 싱글톤 패턴을 적용하여 관리**한다.
* 이 밖에도 싱글톤 패턴을 구현하는 방법은 너무나도 많지만, 상술한 방식은 그 중에서도 안전하고 단순한 경우에 속한다.

### 싱글톤의 한계
* 싱글톤 패턴은 수많은 사용자 요청에 대해서도 단 하나의 인스턴스만을 사용하도록하는 이점이 있으나, 다음과 같은 많은 문제점을 수반한다.
  1. 싱글톤을 적용하기 위해 구현 코드 자체가 길어진다.
  2. 클라이언트는 의존 관계상 항상 구체적인 구현체에 의존하게 되므로 DIP를 위배한다.
  3. 상술한 이유에서, OCP 역시 위반하기 쉽다.
  4. 싱글톤 객체의 내부 속성을 변경하거나 초기화하기 어렵다.
  5. 테스트하기 어렵다.
  6. private 생성자로 인해 자식 클래스를 만들기 어렵다.
* 이와 같은 이유에서, **싱글톤 패턴은 유연성을 크게 떨어트리므로 안티 패턴으로 취급되기도 한다**!

### 싱글톤 컨테이너
* **스프링 컨테이너는 상술한 싱글톤의 한계를 극복하는 동시에 객체 인스턴스를 1개만 생성하여 관리**한다.
  * 앞서 다룬 스프링 빈이 바로 싱글톤으로 관리되는 빈 객체이다.
* 스프링은 별도의 싱글톤 패턴 적용 과정 없이도 빈 객체를 싱글톤으로 관리한다.
  * AppConfig와 같은 Java 설정 파일을 예로 들어, @Bean이 명시된 메소드를 호출하며 반환된 싱글톤 객체를 스프링 컨테이너는 빈 객체로서 관리한다.
  * 때문에 싱글톤 패턴 적용을 위한 구현 코드를 작성할 필요가 없다.
  * 이렇듯 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라고도 지칭한다.
* **싱글톤 컨테이너로 인해 DIP와 OCP 위반, 또는 복잡한 테스트가 필요한 싱글톤 패턴의 단점에서 해방되어 자유롭게 싱글톤의 이점을 누릴 수 있다**.
* 덕분에 여러 사용자의 동시다발적인 요청에서도 매 번 객체를 생성할 필요 없이 단일 인스턴스로 응답이 가능하다.
  * 이 때, 스프링은 빈 객체를 반드시 싱글톤으로만 관리하는 것은 아니다.
  * 스프링은 추가 설정을 통해 빈 객체 요청시마다 새로운 인스턴스를 반환하도록하는 기능 역시 제공한다.
  * 그러나 **실무에서는 사실상 싱글톤 빈만 사용한다고 이해해도 무방**하다.