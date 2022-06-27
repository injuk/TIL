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

### 옵션 처리하기
* 간혹 주입할 스프링 빈이 없더라도 반드시 동작해야하는 경우가 있지만, @Autowired의 기본 동작은 `required=true`이므로 오류가 발생한다.
* 이 때, 자동 주입할 대상을 옵셔널하게 처리하는 방법은 크게 다음과 같다.
  1. @Autowired(required=false): **자동 주입 대상이 없는 경우, setter 메소드 자체가 호출되지 않는다**.
  2. @Nullable: **자동 주입 대상이 없는 경우, null이 입력**된다.
  3. Optional<>: **자동 주입 대상이 없는 경우, Optional.empty가 입력**된다.
* **@Nullable과 Optional<>은 스프링 전반에 걸쳐서 지원되는 개념이므로, 생성자 주입에서 특정 필드에만 사용해도 무방**하다.
  * 예를 들어, 생성자 매개변수 중 마지막 매개변수에 아무런 의존성이 주입되지 않았더라도 생성자가 호출되길 원할 수 있다.
  * 이러한 경우에 @Nullable을 유용하게 활용할 수 있다.

### 생성자 주입을 사용하기
* 기존에는 setter 주입과 필드 주입을 많이 사용했지만, **스프링을 포함하는 대부분의 DI 컨테이너는 생성자 주입 방식을 권장**한다.
* 대부분의 경우, 의존 관계의 주입은 한 번 발생한 후에 애플리케이션이 종료될 때까지 수정될 일이 없다.
  * 오히려 **대부분의 의존 관계는 애플리케이션이 종료되기 전까지 변하지 않는 불변 특성을 가져야만 한다**.
  * 생성자 주입은 객체 생성 시 단 1회만 호출되므로, 이후에 호출될 일이 없어 불변하게 만들 수 있다.
  * 또한 객체의 의존성을 위한 필드에 `final` 키워드를 삽입할 수 있다.
* setter 주입을 활용할 경우, 반드시 setter 메소드의 가시성을 public으로 두어야 한다.
  * 이렇듯 의존 관계를 가변적으로 둔 채 누구나 호출할 수 있도록 열어두는 것을 바람직한 설계 방식이 아니다.

### 순수 Java 코드를 활용한 테스트에서의 누락 가능성
* setter와 같은 메소드를 사용하여 의존 관계를 주입할 경우, 개발자의 실수로 인해 의존성 주입 자체를 누락하기 쉽다.
  * 또한 반드시 테스트 대상 빈 객체를 열어봐야 어떤 의존성을 사용하고 있는지 알 수 있다.
* **생성자를 활용한 의존 관계 주입의 경우, `final` 키워드를 적용할 수 있기 때문에 대부분의 누락을 미연에 방지**할 수 있다.
  * `final` 키워드를 사용하는 경우, 의존 관계의 누락을 컴파일 시점에 쉽게 인식할 수 있다.
  * 반면, 스프링의 빈 객체 주입 단계에 따라 setter 메소드는 컴파일 시점에 문제를 인식하지 못하기 쉽다.
  * 또한 setter 주입과 같은 방식의 의존 관계 주입은 모두 생성자 이후에 호출되므로 `final` 키워드를 사용할 수 없다.
    * **가장 빠르고 좋은 오류는 컴파일 오류임을 반드시 기억**하자. 
  * 또한, 해당 객체의 생성자에 의존성을 전달하는 코드가 명시적으로 드러나므로 내부적으로 어떤 의존 관계를 필요로하는지 쉽게 이해할 수 있다.

### 생성자 주입의 결론
* 생성자를 활용한 의존 관계 주입을 적용할 경우, 프레임워크에 의존하지 않고도 순수한 Java 언어의 특징을 잘 살릴 수 있다.
* 따라서 기본적으로는 생성자 방식을 사용하되, 필수 의존 관계가 아닌 경우에 한해 setter 주입 방식을 고려한다.
  * 또는 두 방식을 동시에 사용할 수도 있다.
* 가능한 한 **항상 생성자 주입을 선택하되, 간혹 추가적인 옵션이 필요한 경우에만 setter 주입을 사용**한다.
  * **필드 주입 방식은 고려조차 않는게 바람직**하다.

## 2022-06-28 Tue
### 롬복 적용해보기
* 실무에서는 빈 객체를 등록하는 과정이 대부분 비슷하며, 중복되는 코드도 많이 발생할 수 밖에 없다.
  * 특히 생성자 주입 방식은 다른 의존 관계 주입에 비해 코드가 길어진다.
* 롬복은 이러한 반복되는 코드의 작성을 크게 줄여주는 라이브러리이며, `build.gradle`에 다음과 같이 추가하여 사용할 수 있다.
```
configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

dependencies {
    // ...생략
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testCompileOnly 'org.projectlombok:lombok'
	testAnnotationProcessor 'org.projectlombok:lombok'
}
```
* **롬복 라이브러리는 어노테이션을 명시하는 것만으로 getter, setter 등의 보일러 플레이트 코드를 줄여주는 역할을 수행**한다. 
  * 롬복은 실무에서는 빼놓을 수 없는 라이브러리라고 생각해도 무방하다.
* 롬복이 제공하는 편리한 기능 중 하나인 @RequiredArgsConstructor를 사용하는 경우, final로 정의된 의존성을 위한 생성자를 제거할 수 있다.
  * 해당 어노테이션은 final로 정의된 필드를 전달받는 생성자를 자동으로 생성해준다.
  * 아래의 코드를 예로 들어, final로 정의된 DiscountPolicy와 MemberRepository 필드를 전달받는 생성자가 자동으로 생성된다.
  * **롬복은 Java의 어노테이션 프로세서를 활용하여 컴파일 시점에 생성자 코드를 자동으로 생성**해준다.
```
@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {
    private final DiscountPolicy discountPolicy;
    private final MemberRepository memberRepository;

    @Override
    public Order create(Long memberId, String productName, Long price) {
        Member member = memberRepository.findById(memberId);
        Long discountPrice = discountPolicy.discount(member, price);

        return new Order(memberId, productName, price, discountPrice);
    }
}
```
* 실무에서는 빈 객체로 등록할 대상의 생성자를 하나만 두고, @Autowired를 생략하는 방법을 활용한다.
  * 이 때, **롬복의 @RequiredArgsConstructor 어노테이션을 함께 사용하면 필요한 기능은 다 제공하면서도 코드를 깔끔하게 유지**할 수 있다.

### 조회 대상 빈 객체가 둘 이상인 경우
* **@Autowired 어노테이션은 의존성 주입을 위해 기본적으로 타입을 기반으로 조회**한다.
  * **타입으로 조회하므로, 마치 `ac.getBean(Type.class)`와 유사하게 동작**한다.
  * **스프링이 빈 객체를 조회할 때, 동일한 타입의 빈이 둘 이상 존재하는 경우에는 `NoUniqueBeanDefinitionException` 오류가 발생**한다.
* 이 경우 다시 하위 타입을 활용할 수도 있지만, 이는 DIP를 위배하면서 유연성을 떨어트리는 방식이다.
  * 무엇보다 이름만 다르고 완전히 같은 타입의 빈이 둘 이상 존재하거나 자식이 여럿인 경우에는 문제를 해결할 수 없다.
  * 즉, **하위 타입만으로는 이러한 문제를 완전히 해결할 수 없다**.
* **스프링 빈을 수동으로 등록하여 문제를 해결할 수도 있으나, 의존 관계를 자동 주입하는 과정 자체에서 해결하는 방법도 여럿 존재**한다.