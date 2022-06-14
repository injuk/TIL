# Basics
## 2022-06-13 Mon

### 스프링 빈과 의존 관계
* 특정한 도메인의 컨트롤러는 해당 도메인의 서비스 클래스의 객체를 통해 작업을 수행하므로, 두 객체 간에는 의존 관계가 존재한다.
  * 이 경우, **컨트롤러는 서비스에 의존하는 셈**이다.
* 이러한 **의존 관계는 개발자가 직접 코드로 구현할 수도 있지만, 스프링의 경우 어노테이션을 활용하여 더욱 간단하게 사용이 가능**하다.
```
@Controller
public class MemberController {}
```
* @Controller 와 같은 어노테이션이 붙은 클래스의 경우, 다음과 같은 순서에 의해 객체가 생성되어 관리된다.
  1. 스프링이 실행되는 경우, 스프링 컨테이너가 생성된다.
  2. 스프링 컨테이너에 @Controller 어노테이션이 명시된 MemberController 객체가 자동으로 생성되어 삽입된다.
  3. 이렇게 **스프링 컨테이너에 삽입된 객체들은 스프링 자체적인 관리가 가능**하다.
* 이러한 일련의 과정을 다른 표현으로 `스프링 컨테이너에 의해 스프링 빈이 관리된다.` 라고도 할 수 있다.
* 스프링을 적용하지 않는 경우, 컨트롤러가 서비스에 의존하도록 하려면 코드는 다음과 같이 작성되어야 한다.
```
public class MemberController {

    private final MemberService = new MemberService();
}
```
* 반면 **스프링을 사용하는 경우, 필요한 의존성은 스프링 컨테이너에 등록하고 필요한 경우에 이를 받아서 사용하도록 구현**되어야 한다.
  * 이 경우 @Autowired 어노테이션을 활용할 수 있으며, 다음과 같이 작성한다.
```
@Controller
public class MemberController {

    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```
* MemberController는 스프링이 동작되며 스프링 컨테이너에 의해 관리되고, 생성 시점에 생성자가 호출된다.
  * 이 때, **생성자에 @Autowired 어노테이션이 명시된 경우 스프링은 스프링 컨테이너에 등록된 의존성을 가져와 연결**시킨다.
  * 즉, **의존성은 우선 스프링 컨테이너에 등록되어야 하므로 당연히 서비스 객체 역시 스프링 컨테이너에 빈 형태로 등록되어 있어야 한다**. 
* 이렇듯 각 객체를 스프링 컨테이너가 관리할 수 있도록 하는 어노테이션에는 @Controller 외에도 @Service, @Repository 등이 존재한다.
  * 이는 컨트롤러, 서비스, 리포지토리 구조가 오랜 기간 정형화된 패턴이기 때문이며, 스프링은 용도에 맞는 어노테이션을 기본적으로 제공한다.
* **빈에 주입될 의존성은 기본적으로 자동으로 선택되나, 동일한 기능을 수행하는 여러 의존성이 존재하는 경우 다음과 같이 실행에 실패**할 수 있다.
```
Parameter 0 of constructor in hello.hellospring.service.MemberService required a single bean, but 2 were found:
	- memoryMemberRepository: defined in file [/absolute/path/MemoryMemberRepository.class]
	- testRepository: defined in file [/absolute/path/TestRepository.class]


Action:

Consider marking one of the beans as @Primary, updating the consumer to accept multiple beans, or using @Qualifier to identify the bean that should be consumed
```
* 상술한 방식과 같이, **객체 간의 여러 복잡한 의존성 관계를 스프링이 자동으로 관리하여 필요한 의존성을 생성 및 주입하는 것을 의존성 주입**이라고 한다.

### 스프링 빈을 등록하는 방법
* 스프링 컨테이너가 관리하는 객체는 빈이라고 하며, 이러한 빈을 등록할 수 있는 방법은 크게 다음과 같다.
  1. 컴포넌트 스캔을 활용한 의존 관계의 자동 설정
  2. Java 코드 상에서 직접 빈을 등록
* **스프링 개발자는 상술한 두 방식을 모두 숙지해야 하며, 왠만한 클래스는 모두 스프링 빈으로 등록하는 것이 바람직**하다.
  * 이는 각 객체를 스프링 빈으로 등록했을 경우 얻어지는 이점이 많기 때문이다.
* **스프링 빈으로 등록되는 객체는 기본적으로 싱글톤이므로, 동일한 스프링 빈은 모두 같은 인스턴스를 사용하는 것이 보장**된다.
  * 추가 설정으로 여러 인스턴스를 갖도록 할 수 있으나, **정말 특별한 경우를 제외하면 일반적으로 싱글톤을 적용**한다.

### 컴포넌트 스캔이란?
* 상술한 코드인 **@Controller와 @Service, @Repository 어노테이션을 활용하는 방식은 컴포넌트 스캔을 활용한 방식에 해당**한다.
  * 사실 @Component 어노테이션만을 명시해도 스프링 빈에 자동으로 등록되며, 상술한 세 어노테이션은 각각 내부적으로 @Component 어노테이션을 포함한다.
* 결국 **컴포넌트 스캔 방식이란, 스프링이 시작될 때 @Component 어노테이션이 명시된 클래스는 모두 객체를 생성하여 스프링 컨테이너에 등록하는 방식**이다.
  * 나아가 **생성자에 명시된 @Autowired 어노테이션은 이렇게 생성된 빈 간의 의존 관계를 설정해주기 위해 명시하는 어노테이션**이다.
* **@SprintBootApplication 어노테이션이 명시된 클래스의 `package` 정보를 기준으로 하위 패키지에 위치한 모든 컴포넌트는 스프링 빈으로 등록**된다.
  * 즉, **동일한 `package` 정보를 갖거나 하위 패키지에 위치한 컴포넌트는 모두 컴포넌트 스캔의 대상**이 된다.
  * 절대 불가능한 것은 아니지만, 별도의 추가 설정 없이는 기본적인 컴포넌트 스캔의 동작 방식이 상술한 바와 같다.

## 2022-06-14 Tue
### Java 코드 상에서 직접 빈을 등록하기
* 컴포넌트 스캔을 사용하는 방법 이외에도 **직접 설정 파일에 스프링 빈을 등록할 수도 있다**.
  * 예를 들어, @Service나 @Repository와 같은 어노테이션을 제거하면 애플리케이션은 정상 동작하지 않을 수 있다.
  * 이 경우, SpringConfig 파일을 만들어 직접 빈을 등록할 수 있다.
```
@Configuration
public class SpringConfig {
    
    @Bean
    public MemberService memberService() {
        return new MemberService(
                memberRepository()
        );
    }
    
    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```
* 이러한 @Configuration 파일이 존재하는 경우, 스프링은 동작 과정에서 이를 인식하고 @Bean 어노테이션이 명시된 메소드들을 호출한다.
  * 이 결과로 반환되는 객체는 스프링 빈으로 등록된다.
* 상술한 코드의 경우, 다음과 같은 순서로 동작한다.
  1. 스프링이 켜지면서 memberService와 memberRepository 모두를 빈에 등록한다.
  2. 이 과정에서, 스프링 빈에 등록되어 있는 memberRepository를 memberService에 주입한다.

### 의존성 주입 분류
* 의존성을 주입하는 방법은 크게 필드 주입, 세터 주입, 생성자 주입의 세가지로 나누어 볼 수 있다.
* **의존 관계는 실행 중에 동적으로 변하는 경우가 거의 없으므로, 기본적으로 생성자 주입이 권장되는 방식**이다.
  1. 필드 주입 방식은 동적인 변경 가능성을 완전히 박탈하므로 권장되지 않는다.
  2. 세터 주입의 경우 세터 메소드가 public 등 관대한 정책으로 정의되므로, 의존성이 의도치 않게 변경될 가능성이 있다.
  3. **생성자 주입은 가장 권장되는 방식이며, 애플리케이션이 실행되며 조립되는 과정에서 의존성이 한 번만 주입되는 것이 보장**된다.
* 이 때, 동적인 변경이란 런타임에서의 조작으로 인해 상태가 변경되는 것을 가리킨다.

### Autowired 어노테이션
* @Autowired 어노테이션이 명시된 경우, 스프링은 스프링 컨테이너에 등록된 빈을 토대로 필요한 의존성을 자동으로 주입해준다.
* 당연히 **스프링 컨테이너에 빈으로 등록되지 않은 클래스에는 해당 어노테이션을 명시하여도 스프링의 관리 대상이 아니므로 달라지는 것이 없다**.
  * 또는 스프링에 의해서가 아닌 사용자의 new 연산자를 통해 생성된 객체 역시 @Autowired의 이점을 누릴 수 없는 예시이다.

### 컴포넌트 스캔과 Config 파일 비교하기
* 기본적인 사용성은 @Service 등의 어노테이션을 활용하는 컴포넌트 스캔 방식이 편리하다.
* 실무의 경우, **정형화된 패턴인 컨트롤러 / 서비스 / 리포지토리 클래스에는 컴포넌트 스캔 방식을 적용**한다.
* 반면, **정형화되지 않거나 상황에 따라 구현 클래스를 수정해야하는 경우에는 설정 파일을 통해 스프링 빈으로 등록**한다.
  * 예를 들어, PoC 과정에서 인메모리 저장소를 활용하도록 도입된 클래스를 생각할 수 있다.
  * **추후 해당 코드를 개선하여 새로운 클래스가 추가되더라도, 스프링 설정 파일을 활용하면 운영 중인 코드를 전혀 손대지 않고도 수정을 반영**할 수 있다.
* **스프링 설정 파일을 사용하는 경우, 의존성 구현체가 변경되더라도 설정 파일 하나만을 수정**하면 된다.
  * 반면, 컴포넌트 스캔 방식은 @Service, @Controller, @Repository 파일 등을 하나하나 뒤져봐야 한다.