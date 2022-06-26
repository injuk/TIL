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

### 컴포넌트 스캔 시작 위치
* 프로젝트 내의 모든 Java 클래스를 컴포넌트 스캔하면 너무 긴 시간이 필요하므로, 꼭 필요한 위치를 기준으로 탐색을 시작하도록 위치를 명시할 수 있다.
  * 최악의 경우, 외부 라이브러리로 가져온 코드까지 탐색해버릴 수도 있다.
```
@ComponentScan(basePackages="hello.core")
```
* `basePackages`는 탐색 위치를 지정하며, 명시된 패키지를 포함하여 하위 패키지를 모두 탐색한다.
  * 또는 `basePackages={"hello.core", "hello.service"}`와 같이 여러 패키지를 명시할 수도 있다.
  * 해당 **값을 지정하지 않는 경우, @ComponentScan 어노테이션이 명시된 클래스의 패키지를 시작 위치**로 한다.
* 또는 `basePackageClasses=AutoAppConfig.class` 어노테이션을 통해 명시한 클래스가 포함된 패키지를 기준으로 탐색을 시작할 수도 있다.
* **일반적으로 권장되는 방식은 상술한 패키지 위치 속성을 명시하지 않고 프로젝트의 최상단에 설정 정보 파일을 두는 것**이다.
  * 예를 들어, `hello.core` 패키지가 최상단이라면 `AutoAppConfig` 클래스를 해당 패키지에 작성한다.
  * 이를 통해 **하위 패키지는 모두 컴포넌트 스캔의 대상이 되며, 또한 설정 정보가 애플리케이션을 대표할 수 있다는 관점에서도 바람직한 방식**이다.
* 패키지를 명시하지 않는 방식은 @ComponentScan을 포함하는 @SpringBootApplication 어노테이션을 활용하는 스프링 부트에서도 기본으로 채택한다.
  * 이로 인해 스프링 부트 애플리케이션은 굳이 @ComponentScan 어노테이션을 직접 명시할 필요가 없다.

### 컴포넌트 스캔 탐색 대상
* 컴포넌트 스캔은 @Component 어노테이션 뿐만 아니라 다음의 어노테이션이 명시된 클래스 역시 탐색 대상으로 한다.
  1. @Controller: 스프링 MVC 컨트롤러에서 사용한다.
  2. @Service: 스프링 비즈니스 로직에서 사용한다.
  3. @Repository: 스프링 데이터 접근 계층에서 사용한다.
  4. @Configuration: 스프링 설정 정보에서 사용한다.
* 사실, 상술한 모든 어노테이션은 내부적으로 @Component 어노테이션을 포함하기 때문에 탐색 대상이 된다.
  * 이러한 **어노테이션의 포함 관계를 통한 인식은 스프링 자체적인 기능으로, 순수 Java만으로는 어노테이션의 상속 또는 포함 관계를 인식할 수 없다**.
* **상술한 어노테이션들이 명시된 경우, 스프링은 해당 정보를 컴포넌트 스캔만을 위해 사용하는 것이 아니라 추가적인 부가 기능을 제공하도록 동작**한다.
  1. @Controller: 스프링 MVC 컨트롤러로 인식한다.
  2. @Repository: 스프링 데이터 접근 계층으로 인식하여 데이터 계층의 예외를 스프링 예외로 변환한다.
     * 이렇듯 **스프링이 제공하는 추상화된 예외로 변환해주지 않는 경우, 데이터베이스가 바뀔 때마다 코드 전체적인 수정이 필요**해진다.
     * 이는 데이터베이스마다 다른 구체적인 예외를 사용하기 때문이다.
  3. @Configuration: 스프링 설정 정보로 인식하여 빈 객체를 싱글톤으로 관리하도록 지원한다.
  4. @Service: 유일하게 **추가 기능을 제공하지 않는 어노테이션이지만, 개발자들이 해당 클래스에 핵심 비즈니스 로직이 포함된다는 것을 인지하도록 돕는다**.
     * 이를 통해 개발자는 핵심 비즈니스 로직과 관련된 메소드에 @Transaction 등의 어노테이션을 명시할지 결정할 수 있다.
     * **이는 트랜잭션이 핵심 비즈니스 로직과 함께 시작해야 하기 때문이며, 실무에서도 서비스 클래스에 트랜잭션을 자주 적용하는 편**이다.

### 필터
* 컴포넌트 스캔 대상을 추가하거나 제거하기 위해 다음의 필터를 사용할 수 있다.
  1. includeFilters: 컴포넌트 스캔 대상을 추가로 명시한다.
  2. excludeFilters: 컴포넌트 스캔에서 제외할 대상을 명시한다.
* 이 때, 필터로 지정할 수 있는 필터 타입은 다음과 같은 다섯 가지 종류가 존재한다.
  1. ANNOTATION: 기본값으로 적용되며, 어노테이션을 인식하여 동작한다. 
  2. ASSIGNABLE_TYPE: 지정한 타입과 그 자식 타입을 인식하여 동작한다.
  3. ASPECTJ: AspectJ 패턴을 사용하여 동작한다.
  4. REGEX: 정규표현식을 사용하여 동작한다.
  5. CUSTOM: `TypeFilter`라는 인터페이스를 구현하여 필터링 조건을 직접 구현한다.
* **대부분의 경우 @Component 어노테이션만으로도 충분하므로 필터는 자주 사용되지 않는다**.
  * 간혹 excludeFilters는 사용해야만 하는 경우가 발생하지만, includeFilters는 사실상 사용할 필요가 없다.
* **최근 스프링 부트에서는 컴포넌트 스캔을 기본으로 제공하며, 이 때 옵션을 수정하기보다는 최대한 스프링의 기본 설정에 맞추어 사용하는 것이 권장**된다.