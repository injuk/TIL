# Core
## 2022-06-27 Mon

### 의존 관계 주입 분류
* 의존 관계를 주입하는 방법은 크게 다음과 같이 분류된다.
  1. 생성자 주입
  2. setter 주입
  3. 필드 주입
  4. 일반 메소드 주입

### 생성자 주입
* 생성자 주입은 클래스의 생성자를 통해 의존 관계를 주입하며, 생성자 호출 시점에 단 1회만 의존 관계가 주입되는 것이 보장된다.
  * 따라서 **객체 간의 의존 관계가 필수적이고, 불변한 경우에 적용하는 것이 바람직**하다.
* **스프링 빈의 생성자가 단 하나만 존재하는 경우, @Autowired 어노테이션을 생략하더라도 의존 관계가 자동으로 주입**된다.
* 스프링이 컨테이너를 생성하고, 빈 객체를 등록하고, 의존 관계를 주입하는 생명 주기 속에서 생성자 주입은 빈 객체를 생성하는 시점에 의존 관계를 함께 주입한다.
  * 이는 스프링이 빈 객체를 등록하기 위해서는 객체를 생성해야 하고, 객체 생성을 위해 반드시 생성자를 호출해야하므로 어쩔 수 없는 동작이다.

### setter 주입
* **setter 메소드를 통해 의존 관계를 주입하는 방법이며, 의존 관계를 선택하거나 변경될 가능성이 있는 경우에 적용**할 수 있다.
```
private DiscountPolicy discountPolicy;
private MemberRepository memberRepository;

@Autowired
public void setDiscountPolicy(DiscountPolicy discountPolicy) {
    this.discountPolicy = discountPolicy;
    System.out.println("discountPolicy = " + discountPolicy);
}

@Autowired
public void setMemberRepository(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
    System.out.println("memberRepository = " + memberRepository);
}
```
* 스프링은 크게 다음과 같은 두 단계를 거쳐 빈 객체를 생성하고 의존 관계를 주입한다.
  1. 스프링 빈을 모두 컨테이너에 등록하는 단계.
  2. 등록된 스프링 빈들을 토대로 의존 관계를 자동으로 주입하는 단계.
* **@Autowired는 이 중 의존 관계를 주입하기 위해 사용하는 개념이므로, @Autowired 어노테이션을 명시하지 않는다면 의존성을 주입받을 수 없다**.
* 생성자 주입의 경우, 매개변수로 전달된 모든 의존성이 반드시 실재해야 한다.
  * 무엇보다도 **@Autowired 어노테이션 자체의 기본 동작 자체가 주입할 대상이 없으면 오류를 발생**시킨다.
  * 반면, **setter 주입의 경우 선택적으로 의존 관계를 주입하도록 아래와 같이 코드를 작성**할 수 있다.
```
@Autowired(required = false)
public void setDiscountPolicy(DiscountPolicy discountPolicy) {
    this.discountPolicy = discountPolicy;
}
```
* **해당 방식은 setter 메소드를 활용하므로, 의존성을 애플리케이션이 실행되는 도중에 동적으로 수정하는 것이 가능**하다.
  * 물론 이렇게 동적인 의존성을 사용하도록 코드를 작성하는 상황은 매우 드물다.

### 생성자 주입과 setter 주입
* 대부분의 경우에는 생성자 주입 방식을 사용한다.
* 반면, 의존성이 애플리케이션 실행 도중에 수정되어야할 필요가 있는 드문 경우에는 setter 주입을 활용한다.

### 필드 주입
* 필드 주입은 클래스의 멤버 변수에 @Autowired 어노테이션을 직접 명시하여 바로 주입하는 방식이다.
```
@Autowired private DiscountPolicy discountPolicy;
@Autowired private MemberRepository memberRepository;
```
* **코드가 매우 간결해지지만, 다음과 같은 이유에서 권장되지 않으므로 사용을 지양해야하는 방식**이다.
  1. 외부에서 변경이 어려우므로 테스트가 어렵다.
  2. DI 프레임워크가 없다면 동작하지 않는다.
* **상술한 단점은 모두 애플리케이션이 스프링 프레임워크 자체에 너무 크게 의존하게 된다는 공통점에서 기인**한다.
  * 스프링 없이는 아무 것도 할 수 없게 되므로, 테스트도 불가능해지는 것으로 이해할 수 있다.
  * 이를 해결하기 위해서는 각 의존성에 대해 setter 메소드를 추가해야하지만, 이럴 바에는 setter 주입 방식을 선택하는 것이 낫다.
* **해당 방식은 운영 환경에서는 사용하지 말아야 하며, 테스트 코드 또는 @Configuration 어노테이션을 명시하는 Java 설정 파일에만 한정적으로 사용**한다.

### 일반 메소드 주입
* 일반적인 메소드를 통해서도 의존 관계를 주입받을 수 있으며, 한 번에 여러 필드를 주입받을 수도 있다.
  * 원리적으로 생각해보면 메소드에 @Autowired 어노테이션을 할당하는 것이므로, setter 주입과 다를 것이 없다. 
  * 그러나 **실무에서는 잘 사용되지 않는 방식**이다.

### 의존 관계 주입 주의사항
* **의존 관계의 자동 주입은 당연히 스프링 컨테이너가 관리하는 스프링 빈에서만 동작**한다.
* 때문에 스프링 빈이 아닌 도메인 객체에 @Autowired 등 의존 관계 주입과 관련된 어노테이션을 명시하더라도 아무런 효과를 얻을 수 없다.