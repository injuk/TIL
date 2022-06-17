# Basics
## 2022-06-18 Sat

### AOP가 필요한 상황
* 예를 들어 모든 리소스에 대해 호출 시간을 남겨야하는 상황이 발생했다면, 모든 메소드에 중복 코드를 추가하는 방안을 생각할 수도 있다.
  * 이를 회원 서비스의 가입 메소드에서 구현한다면, 아마도 다음과 같은 형태를 띄게 될 것이다.
  * 이 때, 예외가 발생했더라도 반드시 시간을 측정할 수 있도록 finally 구문을 사용하였다.
```
public Long join(Member member) {
    long start = System.currentTimeMillis();
    try {
        validateDuplicateMember(member);

        memberRepository.save(member);
        return member.getId();
    } finally {
        long finish = System.currentTimeMillis();
        long time = finish - start;
        System.out.println("join = " + time + "ms");
    }
}
```
* 특히 시간 측정 요구 사항 같은 경우는 다음과 같은 문제점이 수반된다.
  1. 회원 서비스에서, 시간과 관련된 기능은 애초에 핵심적인 관심사도 아니다.
     * **이 경우, 시간을 측정하는 로직은 모든 비즈니스 로직의 공통 관심사**가 될 것이다.
  2. 클래스마다 핵심 관심사와 공통 관심사가 뒤섞여 가독성과 유지보수성이 떨어진다.
  3. 공통 메소드를 도입하기도 어려운 편이고, 템플릿 메소드 패턴을 적용하더라도 복잡하다.
  4. 반복적인 과정에서 휴먼 에러가 발생하기 쉽다.
  5. 모든 것을 성공적으로 도입하였더라도, 수많은 위치에 흩어진 중복 코드로 인해 유지보수성은 지속적으로 떨어져간다.
* 이렇듯 **많은 기능은 공통 관심 사항과 핵심 관심 사항으로 나뉠 수 있으며, AOP는 공통 관심 사항을 원하는 위치에 손쉽게 적용할 수 있도록 지원**한다.

### AOP란?
* 상술한 시나리오는 많은 개발자들을 괴롭게 하는 상황이며, 이를 해결하려는 노력이 계속해서 있어왔다.
  * 이를 해결하기 위한 고민 과정에서 관점 지향적인 해결책을 제안하려 시도한 것이 AOP의 시작이다.
* **AOP는 공통 관심 사항과 핵심 관심 사항을 분리하는 데에 집중하며, 공통 관심 사항을 한 곳으로 모아 원하는 곳에 적용하는 방식을 채택**한다.
  * **이를 가능하게 한 것이 AOP라는 개념이며, 스프링은 어노테이션을 활용하여 AOP를 적극적으로 지원하는 프레임워크**이다.
* AOP는 악명 높은 개념이지만, 언제 왜 사용하는지를 중심으로 이해한다면 쉬운 응용이 가능한 개념이다.

### AOP 적용하기
* 시간 측정 로직의 경우, AOP를 적용하기 위해 다음과 같은 별도의 클래스를 작성할 수 있다.
  * 이 때, **반드시 @Aspect 어노테이션을 명시**하도록 한다.
* 또한 @Around 어노테이션을 통해 AOP가 적용된 기능의 실행 지점을 명시할 수 있다.
  * 예를 들어 아래의 코드의 경우, 모든 메소드의 호출 전 후에 실행해야할 로직에서 동작할 코드를 명시하였다.
  * 이는 시간 측정 로직 시나리오에 정확히 들어맞는 어노테이션이며, 유사한 동작을 수행하는 @Before 또는 @After를 명시할 수도 있다.
  * 이 때, 어노테이션의 인자로는 대상이 될 패키지와 클래스 경로를 명시한다.
```
@Aspect
@Component
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        System.out.println("[START] " + joinPoint.toLongString());

        try {
            return joinPoint.proceed();
        } finally {
            long elapsedTime = System.currentTimeMillis() - start;
            System.out.println("[FINISH] " + joinPoint.toLongString() + " " + elapsedTime + "ms");
        }
    }
}
```
* 상술한 코드와 같이 AOP로 적용하기 위한 기능은 반드시 빈으로 등록해야한다.
  * 이 때, @Component 어노테이션을 명시하여 컴포넌트 스캔의 대상으로 만들 수도 있다.
  * 그러나 일반적으로는 평범하여 정형화된 서비스, 리포지토리, 컨트롤러를 컴포넌트 스캔의 대상으로 설정한다.
  * 반면 **AOP로 적용하는 기능은 특별하므로 누구나 쉽게 이를 알아챌 수 있어야 하며, 이로 인해 Config 클래스에 명시적으로 등록하는 방식이 선호**된다.
  * 즉, 정형화되지 않은 AOP 기능은 바로 인지가 가능해야 한다. 
* 아래와 같이 @Configuration으로 등록된 클래스에 직접 @Bean으로 등록할 경우, 다른 개발자는 이 파일을 읽는 것 만으로도 AOP의 존재를 인지할 수 있다.
  * 물론, 그렇다고 해서 절대로 컴포넌트 스캔 방식을 사용하지 말아야하는 것은 아니다.
  * 아래의 코드도 그대로 실행하는 경우 에러가 발생하기에, 우선은 @Component 어노테이션을 적용하여 테스트를 진행해야 한다.
```
@Configuration
public class SpringConfig {
    // MemberRepository의 구현체는 스프링 데이터 JPA와 스프링 부트에 의해 자동으로 생성되어 주입된다.
    private final MemberRepository memberRepository;

    public SpringConfig(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository);
    }
    
    @Bean
    public TimeTraceAop timeTraceAop() {
        return new TimeTraceAop();
    }
}
```
* 참고사항으로, @Configuration 방식으로 실행할 경우 AOP 클래스에 다음과 같은 @Around 어노테이션을 명시해야 한다.
  * 아래와 같이 작성하지 않을 경우, SpringConfig 클래스에서 발생하는 순환 참조에 의해 스프링 부트 애플리케이션이 실행되지 않을 수 있다.
  * **실무에서는 대부분의 경우 패키지 단위로 AOP 적용 대상을 명시**하곤 한다.
```
@Around("execution(* hello.hellospring..*(..))" +
        "&& !target(hello.hellospring.SpringConfig)")
```

### AOP를 적용한 경우의 장점
* **상술한 시나리오의 모든 단점이 해소되며, 공통 관심 사항을 우아하게 분리하여 실행시킬 수 있다**.
  1. 회원 도메인의 핵심 관심 사항과 시간 측정이라는 공통 관심 사항은 분리된다.
  2. 시간 측정 로직은 공통 관심 사항으로 추출되므로, 가독성이 높아지며 유지보수성도 향상된다.
  3. 분리된 시간 측정 로직의 적용 대상은 유연하게 명시할 수 있다.
  4. **회원 도메인의 클래스는 핵심 관심 사항만을 깔끔하게 다룰 수 있게 된다**.

### Spring AOP 동작 원리
* **일반적인 구현 방식의 경우, 모든 도메인 기능의 호출 흐름은 의존성의 방향을 따른다**.
  * 예를 들어, 컨트롤러는 서비스에 의존하므로 컨트롤러의 메소드가 서비스의 메소드를 호출하게 된다.
* 이에 AOP 기능을 어느 위치에 적용할지 명시해두었다면, 스프링은 이러한 호출 흐름 사이에 AOP 메소드를 끼워넣는 방식으로 바꾼다.
* 예를 들어, 상술한 시나리오의 경우 다음과 같은 호출 흐름을 갖게 된다.
  1. **스프링이 동작을 시작하는 과정에서, AOP를 위해 가짜 서비스 클래스인 프록시 객체를 생성**한다.
  2. 프록시 객체를 생성한 후, 스프링 컨테이너는 빈으로 관리될 실제 서비스의 앞에 가짜 서비스인 프록시 객체를 세워둔다.
  3. **이후의 동작 과정에서, 가짜 서비스 객체인 프록시 객체의 실행이 끝난 후 `joinPoint.proceed()`를 호출하며 실제 서비스 객체를 호출**한다.
* 즉, **AOP가 적용된 경우 컨트롤러가 호출하는 것은 서비스의 실제 객체가 아닌 Java 프록시로 생성된 가짜 서비스 객체**이다.
  * 이렇듯 **스프링 컨테이너는 AOP가 적용되어야 하는 대상 빈에 대해 가짜 객체인 프록시를 생성**한다.
  * AOP는 프록시 객체를 통해 실행되며, `joinPoint.proceed()`가 호출된 시점에서부터 AOP 대상인 실제 객체가 호출된다.
* 실제로 컨트롤러에서 서비스 의존성을 주입받는 과정에서 `getClass()`를 통해 클래스 정보를 출력하면 다음과 같은 메시지를 확인할 수 있다.
  * 아래의 로그에서, EnhancerBySpringCGLIB은 서비스 객체를 복제하여 코드를 조작하는 역할을 수행하는 클래스이다. 
```
public MemberController(MemberService memberService) {
    this.memberService = memberService;
    System.out.println("memberService = " + memberService.getClass());
    // memberService = class hello.hellospring.service.MemberService$$EnhancerBySpringCGLIB$$c0396e61
}
```

### Spring DI와 AOP
* 스프링의 AOP가 가능한 이유 중 하나는 스프링 컨테이너가 빈을 관리하기 때문이다.
  * 이러한 구조를 사용하므로, 각 빈은 주입 받은 의존성을 활용하므로 스프링 컨테이너가 임의의 객체를 끼워넣기가 쉽다.
  * 반면 DI를 활용하지 않는 경우, 이러한 기술의 도입은 매우 어려운 일이 된다.
* 이렇듯 스프링은 프록시 기반의 AOP를 구현하였으며, 프록시 기반 방식 이외의 또 다른 기법으로는 컴파일 시점에 코드를 생성하여 추가하는 방법도 있다. 