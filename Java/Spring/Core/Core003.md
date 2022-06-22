# Core
## 2022-06-21 Tue

### 새로운 할인 정책을 테스트하기
* 현재의 설계에서, 할인을 담당하는 역할은 별도의 인터페이스인 `DiscountPolicy`로 분리되어 있기 때문에 새로운 할인 정책을 추가히기 쉽다.
* 이러한 코드가 추가된 경우, 반드시 대응되는 테스트 케이스를 만드는 것이 중요하다.
  * 이 때, `@DisplayName("테스트에서 보여지는 문자열.")` 등의 어노테이션을 활용하여 테스트 케이스의 의도를 명시할 수 있다.
* 또한 테스트 케이스는 항상 성공하는 행복 경로 뿐만이 아닌, 실패하거나 놓치기 쉬운 경계 테스트 역시 충실히 작성하는 것이 바람직하다.

### 새로운 할인 정책의 적용
* 요구 사항의 변경으로 인해 새로 정의한 할인 정책으로 애플리케이션의 동작을 수정하는 경우, 도메인 서비스를 다음과 같이 수정해야 한다.
  * 아래의 경우, 다형성을 활용하여 `DiscountPolicy`를 멤버 변수로 갖기에 수정은 비교적 쉽다.
```
public class OrderServiceImpl implements OrderService {

    private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
//    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
    
    private final MemberRepository memberRepository = new InMemoryMemberRepository();

    @Override
    public Order create(Long memberId, String productName, Long price) {
        Member member = memberRepository.findById(memberId);
        Long discountPrice = discountPolicy.discount(member, price);

        return new Order(memberId, productName, price, discountPrice);
    }
}
```
* 객체 지향 설계를 준수하였기에 변경 사항을 반영하기 쉬우나, 동시에 이는 치명적인 문제점을 갖고 있다.

### 기존 애플리케이션의 한계
* 할인 정책을 별도의 역할인 `DiscountPolicy` 인터페이스로 분리하였으나, 할인 정책을 사용하는 것은 서비스 객체이다.
  * 즉, 서비스는 할인 정책에 의존하는 클라이언트 객체이다.
* 때문에 **애플리케이션에서 사용할 할인 정책을 수정하려면 반드시 도메인 서비스 객체에도 수정**이 가해져야만 한다.
* 즉, **다형성을 위해 역할과 구현을 분리하여 인터페이스와 구현 객체를 분리했으나, 개방 폐쇄 원칙과 의존성 역전 원칙은 준수할 수 없다**.
  * 현재의 **도메인 서비스는 기능을 변경하기 위해 클라이언트 코드를 반드시 수정해야하므로, 개방 폐쇄 원칙을 준수하지 못한다**.
  * **멤버 변수 discountPolicy는 `new RateDiscountPolicy();` 와 같이 상세한 구현체에 의존하므로, 의존성 역전 원칙을 준수하지 못한다**.
* 이는 결국 **상세한 구현체에 의존하기에 발생하는 문제이며, 어떤 객체가 또 다른 임의의 객체에게 의존하는 순간 변경이 전파될 가능성은 커지게 된다**.
  * 상술한 코드의 경우, 설계 덕분에 변경을 최소화하였으나 그럼에도 여전히 개방 폐쇄 원칙을 위배했다는 사실은 변함이 없다.

### 한계 극복하기
* 현재까지 작성한 클라이언트 코드는 현재 인터페이스 뿐만 아니라 실제 구현 클래스에도 의존하므로 상술한 한게에서 자유로울 수 없다.
* 이는 **구체 클래스가 변경될 때 클라이언트 코드에 변경이 전파되는 것이 원인이므로, 의존성 역전 원칙을 위배하지 않도록 추상화에만 의존하도록 수정**한다.
  * 즉, 의존 관계를 변경하여 클라이언트 코드가 인터페이스에만 의존하도록 한다.
* 이에 따라 상술한 코드는 추상화에만 의존하도록 구현부를 다음과 같이 제거해줄 수 있다.
```
private DiscountPolicy discountPolicy;
```
* 그러나 수정된 코드의 경우, 추상화에만 의존하지만 실제 구현체가 없기에 테스트 진행시 NPE가 발생하게 된다.
  * 즉, 의존성 역전 원칙은 준수하였으나 정작 애플리케이션이 동작하지 않게 되었다.
* 결국 **또 다른 어떠한 객체가 클라이언트인 도메인 서비스에 적절한 `DiscountPolicy` 객체를 생성하여 주입해주어야 한다**.

## 2022-06-22 Wed
### 관심사 분리하기
* 배우는 본인의 역할에만 집중해야하듯, 다른 배역을 고용하는 역할을 수행하지 말아야 한다.
  * 즉, **관심사는 분리되어야 하며 각각의 배우는 상대 역이 누구든지 간에 즉시 공연할 수 있어야 한다**.
* 이렇듯 **전체적인 공연을 구성하고, 배우를 섭외하여 배우를 역할에 할당하는 것은 배우 개개인이 아닌 공연 기획자의 역할**이다.
* 애플리케이션 역시 공연 기획자 역할을 수행할 객체를 추가해야하며, 결과적으로 배우와 공연 기획자의 책임을 명확히 분리할 수 있어야 한다.

### AppConfig 추가하기
* 상술한 애플리케이션은 도메인 서비스가 직접 구현체를 선택하여 `new` 연산자를 활용한다.
  * 이는 비유하자면 배우가 상대 역할의 배우까지 선택하는 꼴이며, 명백이 배우의 책임을 벗어나는 행위이다.
* 공연 기획자의 예시와 같이 애플리케이션에서도 전체적인 동작 방식을 조율하는 역할이 정의되어야 하며, 이를 수행할 AppConfig 객체를 추가할 수 있다.
* AppConfig와 같은 설정 클래스는 크게 다음과 같은 역할을 수행할 수 있어야 한다.
  1. 실제 구현 객체를 생성한다.
  2. **생성된 구현 객체를 적절한 클라이언트에 연결**한다.
* 때문에 추상화와 구현 모두에 의존하던 기존의 도메인 서비스들은 다음과 같이 필요한 의존성을 외부로부터 주입받을 수 있도록 수정되어야 한다.
  * 이는 의존성의 생성자 주입 방식에 해당한다.
```
public class MemberServiceImpl implements MemberService {
    // 더 이상 상세한 구현체에 의존하지 않으며, 추상화에만 의존한다.
    private final MemberRepository memberRepository;
    
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member find(Long id) {
        return memberRepository.findById(id);
    }
}
```
* 이에 맞추어 작성된 AppConfig의 구현 예시는 다음과 같다.
  * **AppConfig는 애플리케이션의 실제 동작에 필요한 구현체를 생성하고, 참조를 생성자를 통해 주입하여 두 객체의 의존 관계를 만들어준다**.
```
public class AppConfig {
    public MemberService memberService() {
        return new MemberServiceImpl(
                new InMemoryMemberRepository()
        );
    }
    public OrderService orderService() {
        return new OrderServiceImpl(
                new FixDiscountPolicy(),
                new InMemoryMemberRepository()
        );
    }
}
```
* 이를 토대로 **도메인 서비스는 완전히 추상화에만 의존하므로 의존성 역전 원칙을 준수하며, 구체적인 클래스에 대한 어떠한 정보도 갖지 않게 된다**.
  * 즉, **AppConfig가 도입되어 객체를 생성하고 연결하는 역할과 기능을 실행하는 역할이 명확히 구분되어 관심사가 분리**되었다.
* 이렇듯 생성자 의존성 주입을 사용하는 경우, 다음의 이점을 얻을 수 있다.
  1. 각 도메인 서비스는 추상화에만 의존하며, 어떠한 구현체에도 의존하지 않는다.
  2. 각 도메인 서비스 입장에서, 생성자에 실제로 어떠한 구현체가 주입될지는 알 수 없다.
  3. 각 도메인 서비스가 알고 있는 것은 생성자를 통해 주입된 객체가 주어진 역할(=인터페이스)을 수행할 수 있다는 사실 뿐이다.
  4. 때문에 **각각의 도메인 서비스는 의존 관계에 대한 고민은 모두 외부의 AppConfig에 맡기고, 자신의 맡은 바 책임에만 집중할 수 있게 된다**.
* 이는 **도메인 객체의 입장에서 보면 마치 외부로부터 의존 관계가 주입되는 것처럼 보이므로, DI 또는 의존 관계 주입이라고 표현**한다.

### AppConfig의 결론
* **AppConfig가 도입됨으로써 객체의 사용과 생성 및 연결 관심사는 확실히 분리되었으며, 해당 객체는 마치 공연 기획자 같은 역할을 수행**한다.
* **AppConfig는 역할에 맞는 구현 클래스를 선택하여 생성하며, 이로 인해 애플리케이션의 전체 구성을 책임**진다.
* 이로 인해 도메인 서비스 등의 각 객체는 의존성 생성에 대한 고민 없이 오로지 자신의 기능에만 집중하면 된다.

### AppConfig 리팩토링
* 상술한 현재의 AppConfig는 역할이 명시적으로 드러나지 않으므로, 역할이 잘 드러나도록 리팩토링하는 것이 바람직하다.
* 이 경우, 아래와 같이 리팩토링을 적용하여 AppConfig 클래스의 역할을 명시적으로 드러낼 수 있게 된다.
```
public class AppConfig {
    public MemberService memberService() {
        return new MemberServiceImpl(
                memberRepository()
        );
    }

    private MemberRepository memberRepository() {
        return new InMemoryMemberRepository();
    }

    public OrderService orderService() {
        return new OrderServiceImpl(
                discountPolicy(),
                memberRepository()
        );
    }
    
    public DiscountPolicy discountPolicy() {
        return new FixDiscountPolicy();
    }
}
```
* 이렇게 **리팩토링된 AppConfig는 제공되는 메소드의 시그니쳐만으로도 무슨 역할을 수행하는지 명시적으로 드러나게 된다**.
  * AppConfig는 각 메소드를 통해 역할과, 동적으로 적용될 구현까지도 명시적으로 드러낸다. 
  * 나아가 **AppConfig 클래스만 확인하더라도 애플리케이션의 전체적인 역할과 관계, 실제 구현까지 한 눈에 보일 수 있게 된다**.
  * 때문에 AppConfig 하나만으로도 전체 애플리케이션을 빠르게 파악할 수 있게 된다.

### 애플리케이션의 기능 변경하기
```
> 애플리케이션의 기능을 변경하는 경우, 설정 영역만을 수정하되 사용 영역은 전혀 손대지 않아도 되는 것이 이상적인 애플리케이션의 상태이다.
```
* **AppConfig 클래스가 추가되었으므로, 애플리케이션은 크게 사용 영역과 설정 영역으로 분리**되었다.
  * AppConfig는 각 객체를 생성하고 연결짓는 설정 영역에 속하는 객체이다.
* 만약 `DiscountPolicy`의 실제 구현체를 변경하여 애플리케이션에 적용되는 할인 정책을 수정할 경우, 개발자는 오로지 AppConfig의 내용만을 수정하면 된다.
  * 즉, **설정 영역에 대해서만 수정하면 사용 영역에 존재하는 어떠한 코드도 수정할 필요가 없다**.
  * 반면, **구체적인 구현에 대해 알아야 할 책임이 있는 구성 영역은 수정 사항을 반영하기 위해 당연히 변경**되어야 한다.
* 이 시점에서 의존성 역전 원칙은 물론 개방 폐쇄 원칙까지 준수하는 애플리케이션이 완성된다.

### 애플리케이션에서의 SRP, DIP, OCP 적용
* 기존 코드의 경우, 각 도메인 서비스는 객체를 생성하고, 연결하고, 기능을 실행하는 너무 많은 책임을 갖기에 SRP를 준수하지 못했다.
  * 이는 **AppConfig 객체를 도입하여 객체의 생성과 구현 책임을 할당하는 관심사의 분리를 통해 SRP를 적용**할 수 있게 되었다.
* 기존 코드의 경우, 각 도메인 서비스는 추상화와 동시에 구현 클래스에도 의존하였기에 DIP를 준수하지 못했다.
  * 때문에 각 도메인 서비스가 추상화에만 의존하도록 하였으나, 이 경우 구현 클래스를 사용할 수 없는 문제가 발생할 수 있다.
  * 이는 **AppConfig로부터 필요한 의존성을 생성자를 통해 주입 받는 방법으로 DIP를 준수**할 수 있게 된다.
* **임의의 클래스가 다형성을 잘 활용하고 클라이언트가 DIP를 준수했을 때 OCP를 적용할 수 있는 가능성이 열리게 된다**.
  * 애플리케이션은 사용 영역과 구성 영역으로 분리할 수 있으며, 이 과정에서 AppConfig 클래스를 도입한다.
  * **AppConfig가 도입됨으로써 설정 영역을 수정하는 것을 통해 애플리케이션의 기능을 손쉽게 확장시킬 수 있으며, 기존 클라이언트는 수정할 필요가 없다**.
  * 즉, **소프트웨어 요소를 추가하는 등의 방법을 통해 기능을 확장하더라도 사용 영역에 대한 변경은 닫혀 있는 상태를 유지**할 수 있다.

### IoC - 제어의 역전
* **제어의 역전은 프레임워크가 개발자의 코드를 호출하는 기법을 의미하며, 이는 스프링에만 국한된 개념은 아니다**.
  * 일반적으로는 개발자가 모든 것을 생성하고, 연결하고, 사용하는 흐름이 자연스럽다.
  * 그러나 제어의 역전을 적용할 경우 개발자가 작성한 코드를 생성하고 연결하는 것은 프레임워크 등이 수행하기 때문에 코드에 대한 제어권이 역전된다.
* 상술한 코드에서도 AppConfig의 도입으로 인해 다음과 같은 이유에서 제어의 역전이 적용된 것으로 이해할 수 있다.
  1. 모든 구현 객체는 실행만을 담당하며, 객체의 생성과 연결 등의 제어 흐름은 AppConfig가 관리한다.
  2. 의존성이 필요한 객체들은 인터페이스를 호출하지만, 실제로 어떠한 구현체를 호출할지는 알 수 없다. 알 필요 없다.
* 이렇듯 **애플리케이션의 제어 흐름을 직접 관리하는 것이 아닌, 외부에서 관리하는 것을 제어의 역전**이라고 한다.
* 제어의 역전 관점에서는 프레임워크와 라이브러리를 쉽게 구분할 수도 있다.
  1. **프레임워크는 개발자가 작성한 코드를 제어하고, 대신 실행**한다.
     * 이러한 이유에서 junit은 개발자가 테스트 케이스를 작성만 하면 대신 실행해주므로, 프레임워크가 맞다.
  2. **라이브러리는 개발자가 작성한 코드가 제어의 흐름을 관리**한다.

### DI - 의존 관계 주입
* **모든 의존 관계는 정적인 클래스 의존 관계와, 런타임에 결정되는 동적인 객체 의존 관계를 구분**해야 한다.
* **정적인 클래스 의존 관계의 경우, 클래스 상단에 명시되는 import 문을 통해 쉽게 확인**할 수 있다.
  * 때문에 정적인 의존 관계는 애플리케이션을 실행하지 않더라도 알 수 있다.
  * **그러나 정적인 클래스 의존 관계만으로는 런타임에 실제로 어떠한 객체가 주입될지는 알 수 없다**.
* 반면 **동적인 객체 인스턴스 의존 관계는 애플리케이션의 런타임에 생성된 객체 인스턴스의 참조가 연결되는 의존 관계**이다.
* 이렇듯 **애플리케이션의 런타임에 외부에서 실제 구현체를 생성하고, 클라이언트에 전달하여 의존 관계를 연결시키는 것이 의존 관계 주입**이다.
  * 의존 관계 주입은 내부적으로는 객체 인스턴스를 생성하고, 참조값을 전달하는 식으로 동작한다.
  * **의존 관계 주입을 통해 클라이언트 코드를 변경하지 않고서도 클라이언트의 의존성을 쉽게 변경**할 수 있다.
  * 또한, **의존 관계 주입을 활용하면 정적인 클래스 의존 관계를 변경하지 않고서도 동적인 객체 인스턴스의 의존 관계를 쉽게 변경**할 수 있다.

### IoC 컨테이너 - 또는 DI 컨테이너
* 상술한 코드에서 구현한 **AppConfig처럼, 객체를 생성하여 관리하고 의존 관계를 연결해주는 것을 IoC 컨테이너 또는 DI 컨테이너라고 지칭**한다.
  * 이 중, **의존 관계를 주입하는 기능에 초점을 맞추어 최근에는 아주 범용적인 용어인 IoC보다는 DI 컨테이너라는 표현을 선호**한다.
  * 같은 의미로 어셈블러, 또는 오브젝트 팩토리라는 용어를 사용할 수도 있다.
* 상술한 이유에서 AppConfig 역시 DI 컨테이너이며, 스프링 또한 DI 컨테이너 기능을 제공한다.

### Spring 기반의 AppConfig
* AppConfig에 @Configuration과 @Bean 어노테이션을 적절히 명시할 경우, AppConfig는 Spring에 의해 관리되는 설정 객체가 된다.
* 스프링이 관리하는 AppConfig를 사용하고자 하는 경우에는 다음과 같이 코드를 작성해볼 수 있다.
  * ApplicationContext는 사실상 스프링 컨테이너이며, 모든 빈 객체를 관리해준다.
  * AnnotationConfigApplicationContext는 ApplicationContext를 생성하기 위해 사용할 수 있다.
    * 해당 객체는 어노테이션을 기반으로 설정된 AppConfig를 기반으로 ApplicationContext를 구성하기 위해 사용한다.
    * **이를 통해 AppConfig에 명시된 설정 정보를 토대로 스프링은 스프링 컨테이너를 통해 모든 빈 객체를 관리**한다.
```
public class MemberApplication {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);

        Member member = new Member(1L, "member", Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.find(1L);
        System.out.println("member: " + member.getName());
        System.out.println("found: " + findMember.getName());
    }
}
```
* ApplicationContext로부터 필요한 의존성을 가져오기 위해 getBean 메소드를 사용한다.
  * 이 때, **스프링 컨테이너에는 기본적으로 @Bean 어노테이션이 명시된 메소드의 이름으로 객체가 등록**된다.
  * 때문에 getBean의 첫 번째 매개변수로는 가져올 빈 객체를 생성하는 메소드 이름을 전달하며, 반환 객체의 타입을 두 번째 매개변수에 명시한다.
* 이제 MemberApplication을 실행할 경우, 다음과 같은 로그를 확인할 수 있다.
  * 위에서부터 다섯 개의 Bean은 스프링 자체적인 필요에 의해 항상 생성하는 객체이다.
  * 아래에서부터 다섯 개의 Bean은 AppConfig를 포함하여 스프링 컨테이너에서 관리하고자 등록한 빈 객체들이다.
  * **해당 로그를 통해 각 객체가 AppConfig의 메소드 명과 같은 이름으로 빈 객체를 등록하는 것을 확인**할 수 있다.
  * 때문에 스프링 컨테이너에게 해당 이름을 전달하여 필요한 빈 객체를 가져와 활용할 수 있게 된다.
```
17:06:47.751 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@569cfc36
17:06:47.763 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
17:06:47.851 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerProcessor'
17:06:47.852 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
17:06:47.854 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
17:06:47.854 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
17:06:47.859 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'appConfig'
17:06:47.863 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'memberService'
17:06:47.877 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'memberRepository'
17:06:47.879 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'orderService'
17:06:47.879 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'discountPolicy'
```

### Spring 컨테이너
* **`ApplicationContext`가 스프링 컨테이너**이다.
* 기존 코드의 경우 AppConfig를 통해 직접 객체를 생성하여 DI를 수행하였으나, 스프링을 활용할 경우 스프링 컨테이너를 통해 사용할 수 있다.
* 상술한 코드에서, 스프링 컨테이너는 @Configuration 어노테이션이 명시된 AppConfig를 구성 정보로 사용한다.
  * 이 때, **@Bean 어노테이션이 명시된 모든 메소드를 호출하여 반환된 객체를 스프링 컨테이너에 등록**한다.
  * 이러한 과정을 통해 **스프링 컨테이너에 등록된 모든 객체를 스프링 빈**이라고 한다.
  * 스프링 빈은 @Bean 어노테이션이 명시된 메소드 명을 빈의 이름으로 하여 등록된다.
* 해당 방식의 경우 AppConfig 객체에서 직접 필요한 의존성을 주입받을 필요 없이 스프링 컨테이너를 통해 필요한 빈 객체를 조회해야 한다.
  * 반면 기존 방식의 경우, Java 코드를 통해 객체를 생성하고 관리해야 했다면 해당 방식은 스프링 컨테이너에 빈을 등록하여 조회해야한다는 차이점이 존재한다.
* 이는 즉 **스프링에게 환경 정보가 명시된 @Configuration 클래스를 넘겨주고, 스프링은 해당 클래스를 토대로 필요한 빈 객체를 생성하여 관리하는 방식**이다.
  * 이후, 필요한 객체는 스프링 컨테이너에게 요청하여 조회한 후에야 사용할 수 있게 된다.
  * 추후 알아볼 **@Autowired 역시 유사한 원리로 스프링 컨테이너에게 필요한 빈 객체를 요청하여 주입받는 방식**이다.