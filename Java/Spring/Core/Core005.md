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

### 싱글톤 방식의 주의사항
* **싱글톤 패턴, 또는 싱글톤 컨테이너 등 싱글톤을 적용하는 모든 객체는 반드시 무상태를 유지**해야 한다.
  * 이는 여러 클라이언트 코드가 단일 객체를 공유하기 때문이며, 상태를 갖는다면 각 클라이언트의 동작에 의해 서로 영향을 받을 수 있다.
* 객체를 무상태로 설게하기 위해서는 다음과 같은 기준을 따를 수 있어야 한다.
  1. 특정한 클라이언트에만 의존하는 필드를 갖지 않아야 한다.
  2. 특정한 클라이언트가 임의로 필드의 값을 변경할 수 없어야 한다.
  3. '가능한 한' 읽기 동작만을 지원해야 한다.
  4. **필드를 대신하여 Java에서 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용**해야 한다.
* 예를 들어 아래와 같은 코드는 전형적으로 장애가 발생하기 쉬운 형태의 서비스 객체이다.
```
public class StatefulService {
    private int price;
    public void order(String name, int price) {
        System.out.println("name: " + name + ", price: " + price);
        this.price = price;
    }

    public int getPrice() {
        return price;
    }
}
```
* **스프링 빈에 공유가 가능한 필드를 설정할 경우, 애플리케이션은 큰 장애로 이어지기 쉽다**.
  * **이러한 문제는 실무에서도 종종 마주칠 수 있으며, 그 때마다 매우 어렵게 해결해야하는 유형**에 속한다.
* **공유 필드는 항상 조심해야하며, 스프링 빈 객체는 가능한 한 무상태로 설계하는 것이 바람직**하다.
* 상술한 코드의 경우, 공유 필드를 제거하고 지역변수를 활용하는 것으로 문제를 해결할 수 있다.
```
public class StatefulService {
    public int order(String name, int price) {
        System.out.println("name: " + name + ", price: " + price);
        
        return price;
    }
}
```

## 2022-06-26 Sun
### @Configuration 어노테이션과 싱글톤
* 그러나 기존에 작성한 AppConfig만 봐서는 마치 싱글톤이 깨지는 것처럼 보일 수 있다.
  * 예를 들어, 아래의 코드에서는 memberService 메소드와 orderService 메소드 각각이 memberRepository 메소드를 호출한다.
  * 때문에 memberRepository는 호출 시마다 리포지토리 객체를 반환하여 마치 두 개의 리포지토리 인스턴스가 존재할 것처럼 보인다.
  * **개발 과정에서 발생하는 이러한 석연치 않은 부분들은 테스트 코드를 통해 확인하는 것이 바람직**하다.
```
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new InMemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(discountPolicy(), memberRepository());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}
```
* 그러나 이를 테스트해볼 경우, 모든 memberRepository는 동일한 객체임을 알 수 있다.
  * **생성자 또는 `{}` 문법에 `System.out.prinln`을 작성하더라도 최초 생성 1회만 로그가 출력되는 것을 확인**할 수 있다.
* 결국 **스프링은 어떠한 방식으로든 항상 빈 객체의 싱글톤을 보장하고자 내부적으로 무언가를 수행할 것이지만, 이는 Java 코드 상에서는 잘 보이지 않는다**.

### @Configuration과 바이트 코드 조작
* 스프링 컨테이너는 싱글톤 레지스트리이므로 빈 객체가 싱글톤이 되도록 유지해야한다.
  * 그러나 스프링은 개발자에 의해 작성된 Java 코드까지 수정할 능력은 없다.
  * **때문에 스프링은 내부적으로 바이트 코드를 조작하는 라이브러리를 사용**한다.
* 스프링이 내부적으로 바이트 코드를 조작하는 라이브러리를 사용한다는 증거는 다음과 같은 테스트 코드로 확인할 수 있다.
  * 스프링은 @Configuration 어노테이션이 명시된 클래스 역시 빈으로 등록하므로 아래와 같은 동작이 가능하다.
```
@Test
void configurationDeepDive() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    AppConfig appConfig = ac.getBean(AppConfig.class);
    System.out.println("appConfig = " + appConfig.getClass());
}
```
* 상술한 테스트 코드를 실행할 경우, 다음과 같은 출력을 확인할 수 있다.
  * 정상적이라면 `appConfig = class hello.core.AppConfig` 처럼 보여져야하지만, 아래의 출력은 더 복잡한 결과를 반환한다.
```
appConfig = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$9c1df86
```
* 이는 스프링이 내부적으로 `CGLIB`이라는 바이트 코드 조작 라이브러리를 사용한 것을 의미한다.
  * 이를 통해 **스프링은 AppConfig를 상속하는 임의의 클래스를 만들고 스프링 빈 객체로 등록하여 관리**한다.
  * 때문에 스프링 컨테이너가 관리하는 빈 객체의 이름은 `appConfig`가 되지만, 실제 객체는 `AppConfig@CGLIB` 형태의 인스턴스가 할당된다.
  * 이렇게 **생성된 `AppConfig@CGLIB` 객체는 `AppConfig`의 자식 타입이므로, `AppConfig` 타입으로 조회가 가능**하다. 
* **결과적으로 개발자가 작성한 `AppConfig` 클래스의 객체가 아닌, 내부적으로 생성된 별도의 객체가 빈 객체들의 싱글톤을 보장**하게 된다.

### AppConfig@CGLIB의 동작 예상
* 실제 CGLIB 라이브러리는 매우 복잡하게 동작하지만, 간략화할 경우 다음과 같은 동작을 수행한다.
  1. @Bean 어노테이션이 할당된 메소드들을 확인한다.
  2. 해당 메소드가 반환하는 객체를 빈으로 등록하기 전에, 우선 스프링 컨테이너에 해당 객체가 이미 빈으로 등록되어 있는지 확인한다.
  3. **이미 등록되어 있다면 해당 빈 객체를 반환**한다.
  4. **아직 등록되어 있지 않다면, 우선 `AppConfig`의 기존 로직을 호출하여 새로운 객체를 생성하고 스프링 컨테이너에 빈 객체로 등록한 후에 반환**한다.
* 결국 **내부적으로는 @Bean이 명시된 메소드마다 빈 객체가 이미 등록되어 있으면 반환하고, 없으면 등록한 후에 반환하는 코드가 동적으로 생성**된다.
  * 이를 통해 @Configuration 어노테이션이 명시된 객체는 싱글톤을 보장할 수 있다.

### @Configuration을 제거하면?
* @Configuration 어노테이션을 제거하더라도 @Bean이 명시된 메소드들이 정상적으로 호출되어 빈 객체가 등록된다.
* 그러나 이 경우에는 빈 객체들의 싱글톤을 보장하는 `AppConfig@CGLIB`이 아닌 순수 Java 코드인 `AppConfig`가 그대로 사용된다.
  * 즉, **여러 클라이언트로부터 의존되는 객체를 중복해서 생성되는 기존 방식의 문제점이 발생**한다.
  * 기존에 작성해두었던 테스트 코드를 통해 모두 다른 객체가 생성된 것을 확인할 수 있다.
```
memberRepository = hello.core.member.repository.InMemoryMemberRepository@561868a0
repositoryByMemberService = hello.core.member.repository.InMemoryMemberRepository@2ea6e30c
repositoryByOrderService = hello.core.member.repository.InMemoryMemberRepository@6138e79a
```
* 또한, **이러한 방식으로 서비스 클래스를 빈 객체로 등록하는 과정에서 생성되어 주입되는 리포지토리 클래스는 스프링 빈이 아니다**.
  * 즉, 스프링 컨테이너가 관리하지 않는 리포지토리 객체가 생성되게 된다.

### 설정 정보 클래스에는 항상 @Configuration 어노테이션을 명시하기
* @Configuration 어노테이션 없이 @Bean 어노테이션만 명시하더라도 각 메소드에서 반환하는 객체를 스프링 빈으로 등록할 수 있다.
* 그러나 이러한 방식으로 생성된 객체들은 싱글톤이 보장되지 않는다.
  * 예를 들어, 서비스 객체에 의존성을 주입하기 위해 호출되는 경우에는 이미 등록된 빈 객체를 반환하는 것이 아닌 새로운 객체를 생성한다. 
* 때문에 **고민할 것 없이 설정 정보 용도로 작성한 클래스에는 반드시 @Configuration 어노테이션을 명시하는 것이 바람직**하다.
  * 이를 통해 스프링은 자연스럽게 빈 객체들의 싱글톤을 보장해준다.