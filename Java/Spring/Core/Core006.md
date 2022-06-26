# Core
## 2022-06-26 Sun

### 컴포넌트 스캔이란?
* 이전까지 사용했던 @Bean 어노테이션 또는 XML 파일을 통해 모든 빈 객체를 직접 명시하는 방식은 확실하지만, 다음과 같은 문제를 수반하기 쉽다.
  1. 관리할 빈 객체가 많아질수록 설정 파일이 커진다.
  2. 이로 인해 빈 객체를 추가하거나 제거하는 과정에서 휴먼 에러가 발생할 가능성이 높아진다.
  3. 무엇보다, 귀찮다.
* 스프링은 설정 정보 없이도 자동으로 스프링 빈 객체를 관리하기 위한 다음과 같은 기능을 제공한다.
  1. 컴포넌트 스캔: 스프링 빈을 자동으로 등록하기 위한 기능이다.
  2. @Autowired 어노테이션: 스프링 빈 객체 간의 의존 관계를 자동으로 주입하기 위한 기능이다.
* **컴포넌트 스캔은 @ComponentScan 어노테이션이 명시된 경우에 시동되며, @Component 어노테이션이 명시된 모든 클래스를 찾아 스프링 빈으로 등록**한다.

### AutoAppConfig 클래스 작성하기
* 기존 AppConfig 클래스를 수정하기보다는 새로운 클래스를 작성하여 컴포넌트 스캔을 테스트하기 위해 다음과 같이 작성한다.
  * @ComponentScan 어노테이션을 통해 컴포넌트 스캔을 시동한다.
  * 이 때, **기존 AppConfig 클래스는 컴포넌트 스캔의 대상에서 제외하기 위해 excludeFilters 옵션을 사용**한다.
    * 반면, **실무에서는 @Configuration 어노테이션이 붙는 설정 파일을 굳이 컴포넌트 스캔 대상에서 제외하지는 않는다**.
  * 아래와 같은 excludeFilters 옵션은 컴포넌트 스캔 대상에서 @Configuration 어노테이션이 할당된 클래스는 빈 객체로 등록하지 않는다.
```
@Configuration
@ComponentScan(
        excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
)
public class AutoAppConfig {}
```
* 상술한 AutoAppConfig 클래스는 기존의 AppConfig와 다르게 @Bean 어노테이션을 명시하는 메소드를 전혀 갖지 않는다.

### @Component 어노테이션 명시하기
* 컴포넌트 스캔은 말 그대로 @Component 어노테이션이 명시된 모든 클래스를 스프링 빈으로 등록한다.
  * 때문에 스프링 빈으로 등록하고자 하는 모든 클래스에 해당 어노테이션을 명시할 필요가 있다.
* 그러나 이러한 방식의 경우, AppConfig 방식과 달리 의존 관계를 설정해줄 방법이 없어보일 수 있다.
* 때문에 **다음과 같이 의존 관계가 필요한 컴포넌트의 생성자에 의존 관계를 자동으로 주입하는 @Autowired 어노테이션을 명시**한다.
```
private final DiscountPolicy discountPolicy;
private final MemberRepository memberRepository;

@Autowired
public OrderServiceImpl(DiscountPolicy discountPolicy, MemberRepository memberRepository) {
    this.discountPolicy = discountPolicy;
    this.memberRepository = memberRepository;
}
```
* 이 경우, **@Autowired에 의해 스프링은 MemberRepository와 DiscountPolcy 타입에 맞는 빈 객체를 스프링 컨테이너로부터 찾아 주입**해주게 된다.
  * 세부적인 동작 원리는 다르지만, 이는 마치 `ac.getBean(MemberRepository.class)`를 활용하는 것과 유사하다. 

### 컴포넌트 스캔과 @Autowired의 관계
* **컴포넌트 스캔 방식을 활용하면 @Autowired 어노테이션의 사용은 필수적**이다.
* **@Autowired 어노테이션이 명시된 경우, 스프링은 생성자의 매개변수를 확인하여 적절한 타입의 빈 객체를 자동으로 연결**한다.
* AppConfig 방식의 경우 빈 객체의 등록과 의존 관계 주입을 모두 코드 형태로 수동으로 작성해야 했다.
  * 반면 **컴포넌트 스캔을 사용할 경우, 빈 객체는 자동으로 생성되지만 설정 정보를 명시할 장소가 없기 때문에 의존 관계를 수동으로 설정할 방법이 없다**.
  * 때문에 빈 객체로 등록할 컴포넌트를 명시하는 것과 동시에 의존 관계의 주입 역시 함께 해결해야 한다.

### AutoAppConfig 테스트하기
* AutoAppConfig 역시 기존의 AppConfig와 같은 방식으로 테스트 케이스를 작성할 수 있다.
  * `AnnotationConfigApplicationContext` 의 생성자에 AutoAppConfig.class를 전달한다.
  * 이렇게 생성된 `ApplicationContext` 역시 getBean을 통해 원하는 스프링 빈을 조회할 수 있다.
```
@Test
@DisplayName("컴포넌트 스캔으로 스프링 빈을 등록할 수 있다.")
void componentScan() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);

    MemberService bean = ac.getBean(MemberService.class);

    assertThat(bean).isInstanceOf(MemberService.class);
}
```
* 또한, 컴포넌트 스캔을 통해 등록되는 빈은 아래와 같은 로그를 통해 확인할 수 있다.
```
19:33:57.910 [main] DEBUG org.springframework.context.annotation.ClassPathBeanDefinitionScanner - Identified candidate component class: file [/spring/core/out/production/classes/hello/core/discount/domain/RateDiscountPolicy.class]
19:33:57.911 [main] DEBUG org.springframework.context.annotation.ClassPathBeanDefinitionScanner - Identified candidate component class: file [/spring/core/out/production/classes/hello/core/member/repository/InMemoryMemberRepository.class]
19:33:57.911 [main] DEBUG org.springframework.context.annotation.ClassPathBeanDefinitionScanner - Identified candidate component class: file [/spring/core/out/production/classes/hello/core/member/service/MemberServiceImpl.class]
19:33:57.911 [main] DEBUG org.springframework.context.annotation.ClassPathBeanDefinitionScanner - Identified candidate component class: file [/spring/core/out/production/classes/hello/core/order/service/OrderServiceImpl.class]
```

### @ComponentScan과 빈 객체의 이름
* @ComponentScan은 기본적으로 @Component 어노테이션이 명시된 모든 클래스를 찾아 스프링 빈으로 등록한다.
  * 이 때, **등록되는 스프링 빈의 이름은 대상 Java 클래스의 이름을 `camelCase`로 변환한 문자열**이다.
  * 때문에 `MemberServiceImpl` 클래스는 `memberServiceImple` 이라는 이름으로 스프링 컨테이너에 등록된다.
* 등록되는 스프링 빈의 이름은 직접 지정할 수도 있으며, `@Component("memberServiceImpl")` 형태로 명시한다.
  * 이는 특수한 경우에만 사용되는 기능이며, 일반적으로는 기본으로 적용되는 빈 이름을 사용한다.

### @Autowired와 의존 관계 자동 주입
* **컴포넌트의 생성자에 @Autowired 어노테이션이 명시된 경우, 스프링 컨테이너는 자동으로 적절한 빈 객체를 찾아 의존 관계를 주입**한다. 
  * 이 때, **기본적인 빈 객체 조회 전략은 `ac.getBean(MemberService.class)`와 같이 생성자에 명시된 매개변수의 타입을 기반**으로 한다.
  * **생성자에 명시된 의존성이 여럿인 경우에도 스프링 컨테이너는 적절한 모든 빈 객체를 찾아 주입**한다.