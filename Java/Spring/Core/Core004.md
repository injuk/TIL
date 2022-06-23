# Core
## 2022-06-23 Thu

### 스프링 컨테이너 생성하기
* 스프링 컨테이너는 다음과 같이 생성해볼 수 있다.
```
ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
```
* 상술한 코드에서 생성된 `ApplicationContext`가 스프링 컨테이너이다.
  * **`ApplicationContext`는 인터페이스이며, `AnnotationConfigApplicationContext`는 구현체**이다.
* 스프링 컨테이너를 직접 생성하는 경우, 다음과 같은 방식 중 하나로 생성해볼 수 있으며, 상술한 방식은 2.의 방식을 따라 생성한 경우이다.
  1. XML을 기반으로 생성하기
  2. 어노테이션 기반의 Java 설정 클래스로 생성하기
* 스프링 자체가 어노테이션을 적극적으로 활용하고 지원하는 프레임워크다보니, 최근에는 XML을 기반으로 하는 방식은 잘 사용되지 않는다.
* 더 정확하게는 스프링 컨테이너는 `ApplicationContext`와 `BeanFactory`로 구분할 수 있다. 
  * **일반적으로는 둘 중 더 자주 사용되는 `ApplicationContext`를 스프링 컨테이너**라고 한다.

### 스프링 컨테이너 생성 과정
* 스프링 컨테이너를 생성하기 위해서는 구성 정보를 전달해줄 필요가 있다.
  * **스프링 컨테이너는 전달된 구성 정보를 토대로 빈으로 등록할 객체를 결정**한다.
  * 상술한 예시에서, `new AnnotationConfigApplicationContext(AppConfig.class)`는 AppConfig를 구성 정보로 전달한다.
  * **이를 통해 스프링 컨테이너가 생성되며, 생성된 스프링 컨테이너는 내부적으로 스프링 빈 저장소를 포함**한다.
  * **스프링 빈 저장소는 키 - 값 목록으로 구성되며, 이 경우 키와 값은 각각 빈 이름과 빈 객체에 대응**된다. 
* 스프링 컨테이너가 생성되면 스프링 빈 저장소에 필요한 빈 객체를 등록한다.
  * 이 때, **스프링 컨테이너는 매개변수로 전달된 설정 클래스의 정보 중 @Bean 어노테이션이 명시된 메소드를 모두 호출하여 스프링 빈을 등록**한다.
  * 등록되는 빈 이름은 기본적으로 @Bean 어노테이션이 명시된 메소드의 이름을 사용하며, 필요시 `@Bean(name="ingnoh")`와 같이 직접 작성할 수도 있다.
    * **빈 이름은 반드시 유일한 이름을 지정해야하며, 그렇지 않은 경우 임의의 빈이 무시되는 등 예상치 못한 오류가 발생**할 수 있다.
    * 특히 실무에서의 빈 이름은 최대한 단순하고 명확해야하며, 절대 중복된 이름은 명명하지 말아야 한다.
  * 즉, **스프링 빈 저장소에 저장되는 빈 이름은 호출되는 메소드명이며, 해당 빈 이름에 대응되는 빈 객체에는 메소드로부터 반환된 값이 저장**된다. 
* **스프링 컨테이너가 생성된 후 필요한 빈 객체 정보를 관리하도록 구성이 완료된 스프링 빈 저장소를 스프링 빈이라고 지칭**할 수 있다.
* 여기까지 과정에서 스프링 빈이 준비되었으므로, 각 빈의 의존 관계를 설정한다.
  * **스프링 컨테이너는 설정 클래스의 정보를 기반으로 의존 관계를 주입**한다.
* **스프링은 빈 객체를 생성하고 의존 관계를 주입하는 단계가 명시적으로 구분되어 있다**.
  * 반면, AppConfig와 같이 Java 코드만으로 의존성을 주입하는 경우에는 두 단계가 구분되지 않는다.

### 컨테이너에 등록된 빈 조회하기
* **`ac.getBeanDefinitionNames()`를 통해 스프링 컨테이너에 등록된 모든 빈의 이름을 조회**할 수 있다.
  * 조회된 각 빈의 이름은 일반적으로 설정 파일로 전달된 클래스의 메소드이름이다.
* **`ac.getBean()`을 통해 빈 이름을 기준으로 대응되는 빈 객체를 조회**할 수 있다.
* 상술한 내용을 토대로 컨테이너에 등록한 빈을 모두 조회할 경우, 다음과 같은 테스트 코드를 활용할 수 있다.
  * getBeanDefinitionNames 메소드를 통해 모든 bean의 definitionName을 가져올 수 있다.
  * 가져온 배열을 순회하며, getBean 메소드를 통해 beanDefinitionName에 대응되는 빈 객체를 가져올 수 있다.
  * 이 경우, 스프링이 내부적인 동작을 위해 등록한 빈까지 모두 출력된다.
```
@Test
void findAllBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for (String beanDefinitionName : beanDefinitionNames) {
        Object bean = ac.getBean(beanDefinitionName);
        System.out.println("name = " + beanDefinitionName + " object = " + bean);
    }
}
```
* 스프링이 내부적인 동작을 위해 등록한 빈을 제외하고 확인하고 싶은 경우, 다음과 같은 테스트 코드를 활용할 수 있다.
  * 빈의 종류를 분류하기 위해 beanDefinition의 getRole 메소드를 활용할 수 있다.
  * **`ROLE_APPLICATION`은 스프링이 내부적인 동작을 위해 등록한 빈이 아닌, 애플리케이션 개발을 위해 개발자가 등록한 빈을 의미**한다.
  * 반면, `ROLE_INFRASTRUCTURE`는 스프링이 내부적인 동작을 위해 자동으로 등록하는 빈을 의미한다.
```
@Test
void findApplicationBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for (String beanDefinitionName : beanDefinitionNames) {
        BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

        if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("name = " + beanDefinitionName + " object = " + bean);
        }
    }
}
```

### getBean
* **getBean 메소드를 사용하는 것은 스프링 컨테이너로부터 스프링 빈 객체를 찾는 가장 기본적인 방법**이다.
  * **해당 메소드는 `(빈 이름, 타입)` 또는 `(타입)` 형태로 사용하며, 대상 빈 객체가 존재하지 않는 경우 예외가 발생**한다.
* getBean 메소드를 활용하여 조회하는 경우, 인터페이스 또는 구체 클래스 중 어느 것을 매개 변수로 전달하더라도 정상 동작한다.
  * 그러나 **역할에 집중하고 유연성을 높이기 위해 구체 클래스보다는 인터페이스를 명시하는 것이 바람직**하다. 
```
@Test
@DisplayName("빈 이름으로 조회하기.")
void findBeanByName() {
    MemberService memberService = ac.getBean("memberService", MemberService.class);
    // 아래와 같은 방식도 가능하지만, 권장되지는 않는다.
    // MemberService memberService = ac.getBean("memberService", MemberServiceImpl.class);

    assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
}
```

### 예외 테스트 케이스 작성하기
* 상술한 코드는 getBean 메소드의 동작성을 테스트 코드로 이해하고 있으나, 예외 발생에 대한 테스트가 부족하다.
* 이러한 **예외 발생 케이스의 테스트는 다음과 같이 `org.junit.jupiter.api.Assertions` API를 통해 테스트가 가능**하다.
  * assertThrows 메소드는 두 개의 매개변수를 받아 동작하며, 각각 예외 클래스와 람다식을 전달받는다.
  * 이 때, 아래의 **테스트 케이스는 두 번째 람다식이 실패했을 때 첫 번째 매개변수에 명시된 예외가 발생해야 함을 의미**한다.
```
@Test
@DisplayName("빈 이름으로 조회 실패.")
void cannotFindBeanByName() {
    assertThrows(
            NoSuchBeanDefinitionException.class,
            () -> ac.getBean("ingnohService", MemberService.class)
    );
}
```

### Spring 컨테이너에 동일한 타입이 둘 이상 등록된 경우
* getBean 메소드를 통해 타입으로 조회하는 경우, 같은 타입의 빈 객체가 둘 이상이면 `NoUniqueBeanDefinitionException` 예외가 발생한다.
  * 이 경우, 빈 이름을 명시하는 것으로 오류를 해결할 수 있다.
  * 또는 getBeansOfType 메소드를 통해 해당 타입의 모든 빈 목록을 조회할 수도 있다.
* getBeansOfType 메소드는 다음과 같이 코드를 작성하여 테스트해볼 수 있다.
  * 이 때, 해당 메소드는 Map을 반환한다.
```
@Test
@DisplayName("임의의 타입을 모두 조회하기.")
void findAllBeanByType() {
    Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
    for (String key : beansOfType.keySet()) {
        System.out.println("key = " + key + " value = " + beansOfType.get(key));
    }
    System.out.println("beansOfType = " + beansOfType);
    assertThat(beansOfType.size()).isEqualTo(2);
}
```
* 추후 다뤄볼 @Autowired 등의 기술 역시 모두 내부적으로는 이를 기반으로 동작한다.

## 2022-06-24 Fri
### 상속 관계의 Spring 빈 조회하기
* **임의의 빈을 조회하는 경우, 자식 타입의 빈이 존재한다면 이마저도 모두 조회**한다.
  * 따라서 Java 언어의 최고 조상인 Object.class로 조회할 경우, 모든 스프링 빈을 조회할 수 있다.

### 실무에서의 ApplicationContext
* 사실 **실무에서는 `ac.getBean()`과 같은 작업을 하는 일이 드물다**.
  * 대부분의 경우 스프링 컨테이너에 의한 자동 의존 관계 주입만을 활용한다.
* 종종 순수 Java 코드로 쓰여진 레거시 애플리케이션에서 스프링 컨테이너를 직접 생성해야하는 경우가 있으며, 그 때 상술한 내용을 활용할 수 있다.

### BeanFactory란?
* **스프링 컨테이너의 최상위 인터페이스이며, 스프링 빈을 관리하고 조회하는 역할을 수행**한다.
* getBean 메소드를 제공하며, 이 외에도 여태까지 활용한 대부분의 기능을 빈 팩토리가 제공한다.

### ApplicationContext란?
* 상속 계층상 BeanFactory 인터페이스를 상속받는 또 다른 인터페이스이다.
* **애플리케이션 컨텍스트는 빈을 관리하고 조회하는 기능은 모두 BeanFactory로부터 상속받아 제공**한다.
* 또한 **애플리케이션의 개발 과정에서는 빈 관리 기능 이외에도 많은 부가기능이 필요하며, 애플리케이션 컨텍스트가 이 부분을 담당**한다.
  1. MessageSource를 활용한 i18n 기능
  2. 환경 변수 처리 기능
  3. 애플리케이션 이벤트 기능
  4. 파일 등의 조회시 사용할 수 있는 편리한 리소스 조회 기능
* 이러한 **부가 기능들은 일반적인 애플리케이션에서 자주 사용되는 공통 기능이므로, 애플리케이션 컨텍스트는 이들 인터페이스를 함께 상속하여 기능을 제공**한다.

### BeanFactory와 ApplicationContext의 관계
* 애플리케이션 컨텍스트 인터페이스는 빈 팩토리 인터페이스를 상속 받아 빈 관리 기능을 제공한다.
* 이에 덧붙여 **애플리케이션 컨텍스트는 여러 부가 기능 인터페이스를 다중 상속하므로 애플리케이션 개발에 필요한 여러 공통 기능을 함께 제공**한다.
  * 때문에 빈 팩토리를 직접 사용하는 일은 거의 없으며, 대신 여러 편리한 기능을 제공하는 애플리케이션 컨텍스트를 사용한다.
* **빈 팩토리 또는 애플리케이션 컨텍스트 자체를 스프링 컨테이너라고 지칭**한다.

### Spring 컨테이너가 지원하는 다양한 설정 형식
* 스프링 컨테이너는 Java 코드 이외에도 XML 등 다양한 설정 형식을 유연하게 지원할 수 있도록 설계되어 있다.
  * 어노테이션 기반의 Java 설정 파일을 지원하는 `AnnotationConfigApplicationContext` 클래스처럼, 설정 형식 별로 대응되는 클래스를 제공한다.
  * 예를 들어 `GenericXmlApplicationContext` 클래스는 Java 파일에 명시된 어노테이션 정보가 아닌 xml 확장자 파일을 설정 정보로 사용한다.
  * 또한, 이들 구현체는 모두 `ApplicationContext` 인터페이스를 구현한다.
* 필요한 경우, 개발자는 `AnnotationConfigApplicationContext` 클래스를 구현하여 자신이 원하는 방식을 정의할 수도 있다.

### XML 설정 형식 사용해보기
* **스프링 부트가 주류로 자리잡은 현재에는 XML 기반의 설정 방식이 자주 사용되지 않지만, 레거시 애플리케이션의 유지보수를 위해 이를 사용**해야할 수 있다.
  * 덧붙여 XML 기반의 설정은 컴파일 없이 빈 설정 정보를 변경할 수 있다는 장점 또한 존재한다.
* 해당 방식은 `GenericXmlApplicationContext` 클래스를 사용하며, xml 확장자로 작성된 설정 파일을 생성자에 전달하여 스프링 컨테이너를 생성한다.
* 상술한 AppConfig.class를 그대로 appConfig.xml로 옮기려면, `resources/appConfig.xml`을 생성하고 다음과 같이 작성한다.
  * IDE의 지원을 받아 Spring Configuration 파일을 생성하면 더욱 쉽게 진행할 수 있다.
  * bean 항목은 스프링 컨테이너가 관리할 개별 빈 객체를 의미한다.
  * constructor-arg 항목의 경우 생성자를 활용한 의존성 주입이 필요한 경우를 의미하며, ref 속성을 통해 의존성 대상 id를 명시한다.
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="memberService" class="hello.core.member.service.MemberServiceImpl" >
        <constructor-arg name="memberRepository" ref="memberRepository" />
    </bean>

    <bean id="memberRepository" class="hello.core.member.repository.InMemoryMemberRepository" />

    <bean id="orderService" class="hello.core.order.service.OrderServiceImpl" >
        <constructor-arg name="discountPolicy" ref="discountPolicy" />
        <constructor-arg name="memberRepository" ref="memberRepository" />
    </bean>

    <bean id="discountPolicy" class="hello.core.discount.domain.RateDiscountPolicy" />
</beans>
```
* 상술한 appConfig.xml의 경우 다음과 같은 코드를 작성하여 애플리케이션 컨텍스트를 생성할 수 있다.
  * 이 때, `GenericXmlApplicationContext` 클래스는 자동으로 `resources` 폴더 하위에 위치한 appConfig.xml을 읽어들여 동작한다.
  * 해당 방식은 설정 정보를 불러들이는 방식이 어노테이션 기반에서 xml 파일 기반으로 바뀐 것을 제외하고는 사용성 측면에서 달라지는 것이 전혀 없다.
```
ApplicationContext ac = new GenericXmlApplicationContext("appConfig.xml");
```