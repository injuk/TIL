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

### @Autowired, @Qualifier, @Primary
* 여러 빈이 선택된 경우, 해결 방법은 크게 세 가지로 분류할 수 있다.
  1. @Autowired의 필드 명을 매칭시킨다.
  2. @Qualifier 끼리 빈 이름을 매칭시킨다.
  3. @Primary 어노테이션을 사용한다.

### @Autowired에 필드 명을 매칭시키기
* **@Autowired 어노테이션은 우선 타입 매칭을 시도하고, 만약 여러 빈이 존재한다면 필드 이름 또는 파라미터 이름으로 빈 이름을 추가 매칭**한다.
  * 우선 타입으로 조회한 후에 필드명을 토대로 비교하고, 이름이 같은 빈을 찾아오므로, 동일한 타입이 있는 경우 같은 빈 이름의 객체를 가져오는 식으로 동작한다.
  * 당연히 조회 타입에 대해 유일한 빈 객체만이 존재하는 경우라면 이러한 추가 매칭 없이 바로 가져온다.

### @Qualifier 어노테이션 사용하기
* **해당 어노테이션은 추가적인 구분자를 명시하는 방법이며, 의존 관계 주입을 위한 추가적인 방법을 제공**한다.
  * 즉, 빈 이름 자체를 변경하지는 않는다.
* 해당 어노테이션을 통해 가져올 빈 객체의 구분자를 다음과 같이 명시할 수 있다.
```
@Component
@Qualifier("fixDiscountPolicy")
public class FixDiscountPolicy implements DiscountPolicy {
}
```
* 해당 의존성을 주입받는 생성자에는 다음과 같이 작성한다.
  * 물론, setter를 활용하여 의존성을 주입하더라도 작성이 가능하다.
```
@Autowired
public void showDependencies(@Qualifier("fixDiscountPolicy") DiscountPolicy discountPolicy, MemberRepository memberRepository) {
    System.out.println("OrderServiceImpl.showDependencies");
    System.out.println("discountPolicy = " + discountPolicy);
    System.out.println("memberRepository = " + memberRepository);
}
```
* 상술한 `@Qualifier("fixDiscountPolicy")`에서, fixDiscountPolicy 빈 객체를 찾지 못하는 경우아는 해당 이름의 스프링 빈을 추가로 조회한다.
  * 대부분의 경우, @Qualifier 어노테이션은 @Qualifier를 찾아내는 용도로만 사용하는 것이 바람직하다.
* @Qualifier는 직접 Java 코드를 작성하여 빈을 등록하는 경우에도 동일하게 사용이 가능하다.
* @Qualifier의 동작을 정리하면 크게 다음과 같다.
  1. @Qualifier끼리 우선 매칭해본다.
  2. 찾고자 하는 빈 객체가 없어 매칭에 실패했을 경우, 빈 이름을 기준으로 매칭한다.
  3. 2.의 과정까지 실패한 경우, NoSuchBeanDefinitionException이 발생한다

### @Primary 어노테이션 사용하기
* 해당 어노테이션은 우선 순위를 결정하는 방법이며, @Autowired 과정에서 여러 빈이 매칭된 경우에는 @Primary 어노테이션이 명시된 클래스가 우선권을 갖는다.
  * 해당 방법은 편리하고 빠르기에 실무에서도 자주 사용되는 방법이다.
  * @Qualifier는는 코드가 지저분해지므로 권장되지 않는 방식이다.
* 해당 어노테이션이 명시된 클래스가 존재하는 경우, 다른 코드는 전부 무시하고 @Primary가 명시된 클래스만이 우선 순위 최상위로 결정된다.

### @Qualifier와 @Primary 어노테이션
* @Qualifier의 경우, 주입이 필요한 모든 코드에 @Qualifier가 명시되어야 한다는 한계가 존재한다.
  * 반면, **@Primary를 활용할 경우 매 번 @Qualifier와 같은 형태의 어노테이션을 명시할 필요가 없다**.
* 예를 들어 코드에서 자주 사용되는 메인 데이터베이스의 커넥션을 획득하기 위한 빈이 있고, 코드 상 특별한 기능을 위해 간혹 사용되는 데이터베이스가 있다고 하자.
  * 이 경우, 자주 접근하는 메인 데이터베이스와 관련된 스프링 빈은 @Primary를 적용하여 깔끔하게 유지한다.
  * 반면, 서브 데이터베이스의 커넥션을 획득하기 위한 빈의 경우 @Qualifier를 적용하여 명시적으로 획득하는 것으로 코드를 깔끔하게 한다.
* **두 어노테이션 간의 우선 순위는 다음과 같은 이유에서 Qualifier가 더 높다**.
  1. @Primary는 마치 기본값처럼 동작하고, @Qualifier는 매우 상세하게 동작한다.
  2. 스프링은 자동 방식보다는 수동 방식이, 넓은 선택 범위보다는 좁은 범위의 선택권에 적용되는 우선 순위가 크가.
* **따라서, 이 경우에도 @Qualifier의 우선 순위가 더 높다**.

### 어노테이션 직접 작성하기
* @Qualifier의 형태로 명시한 경우, 컴파일시 타입 체크가 불가능한 단점이 존재한다.
  * 즉, **@Qualifier("mainDiscountPolicy")와 같은 형태로 명시한 경우 괄호 안의 값은 문자열이므로 컴파일시 타입 체크가 불가능**하다.
  * 때문에 `mainnDiscountPolicy` 와 같은 오타가 발생하더라도 컴파일 시에는 문제를 확인할 수 없다.
* 이러한 단점을 해소하기 위해, 다음과 같은 어노테이션을 직접 작성하여 활용할 수 있다.
```
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy {
}
```
* **상술한 어노테이션은 기존의 @Qualifier와 완전히 동일한 동작을 수행하지만, 어노테이션 명을 잘못 입력한 등의 상황에 대해서 컴파일 에러를 발생**시킨다.
  * 어노테이션을 기존 코드에 적용하는 경우, 다음과 같이 작성한다. 
```
@Component
@MainDiscountPolicy
public class RateDiscountPolicy implements DiscountPolicy {
}
```
* 이 때, 해당 빈 객체를 주입받아 사용하는 서비스의 생성자는 다음과 같이 작성한다.
  * **@MainDiscountPolicy 어노테이션 내부적으로 @Qualifier가 존재하므로, 기존 방식과 동일하게 동작**할 수 있다.
```
public OrderServiceImpl(@MainDiscountPolicy DiscountPolicy discountPolicy, MemberRepository memberRepository) {
    this.discountPolicy = discountPolicy;
    this.memberRepository = memberRepository;
}
```

### 어노테이션에 대해서
* **어노테이션에는 상속 개념이 존재하지 않는 대신, 여러 어노테이션을 모아 활용할 수 있도록 하는 것은 스프링이 지원해주는 기능**이다.
  * 이는 **Java가 제공하는 기능이 아니며, 단순히 스프링 컨테이너가 임의의 어노테이션을 확인했을 때 그 내부를 추가적으로 확인하는 것**이다.
  * 당연히 @Qualifier 뿐만 아니라 다른 어노테이션들도 함께 조합하여 활용할 수 있다.
* 심지어 **@Autowired 어노테이션도 재정의할 수 있으나, 스프링이 자체적으로 제공하는 기능을 명확한 목적 없이 무분별하게 재정의하는 것은 지양**해야 한다.
  * 이러한 방식은 오히려 다가올 수정과 유지보수를 방해하는 요인이 되기 쉽다.
* 대부분의 경우에 스프링이 제공하는 어노테이션만으로 해결이 가능하다.
  * 그러나 @Qualifier와 같이 문자열을 필요로 하는 어노테이션을 활용하는 것보다는 직접 정의한 어노테이션을 사용하는 것도 고려할 수 있다.
  * 어디까지나 무분별하게 사용하지 않는 선에서 애플리케이션의 가독성을 향상시킬 방법을 고민하자.

## 2022-06-29 Wed
### List와 Map 활용하기
* 애플리케이션 동작 과정에서, 정말 의도적으로 임의의 타입을 갖는 모든 스프링 빈이 필요한 상황이 발생할 수 있다.
  * 예를 들어, 할인 정책을 구현하는 여러 할인 서비스를 제공하는데 클라이언트가 이를 선택할 수 있는 시나리오가 있다.
  * **이러한 경우 가장 적절한 것은 전략 패턴을 적용하는 것으로, 스프링은 전략 패턴을 쉽게 구현할 수 있도록 지원**한다.
* Map을 활용할 경우, 키에 스프링 빈 객체의 이름을 넣어 임의의 타입으로 조회한 모든 스프링 객체를 주입한다.
* List를 활용할 경우, 임의의 타입으로 조회한 모든 스프링 빈 객체를 주입한다.
  * 이 때, **해당하는 타입의 스프링 객체가 존재하지 않는 경우 빈 컬렉션 또는 Map을 주입**한다.
```
static class DiscountService {
    private final Map<String, DiscountPolicy> policyMap;
    private final List<DiscountPolicy> policies;

    public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
        this.policyMap = policyMap;
        this.policies = policies;

        System.out.println("policyMap = " + policyMap);
        System.out.println("policies = " + policies);
    }

    public Long discount(Member member, Long price, String discountPolicy) {
        DiscountPolicy policy = policyMap.get(discountPolicy);
        return policy.discount(member, price);
    }
}
```
* 상술한 코드에서, 별다른 Map과 List를 통해 동일한 타입의 모든 스프링 빈을 쉽게 가져올 수 있다.
  * 특히 policyMap에 DiscountPolicy 형태의 모든 스프링 빈인 fixDiscountPolicy와 rateDiscountPolicy가 주입된다.
* discount 메소드는 할인액을 계산하기 위해 Map으로부터 할인 정책을 가져온 후, 할인액을 계산하여 반환한다.
  * 이 과정에서 다형성이 적용되었으므로, discount에 문자열 fixDiscountPolicy 또는 rateDiscountPolicy를 넘길 때마다 반환 값은 달라진다.
  * 이렇듯 Map과 List를 활용하면 매우 간단하게 다형성을 활용한 전략 패턴의 적용이 가능하다.
  * 실무에서도 동적으로 빈 객체를 선택해야하는 경우, Map으로 주입받는 것으로 다형성을 유지하면서도 쉬운 구현이 가능하다.

### 자동 설정과 수동 설정의 올바른 운영 기준
* 스프링의 의존성 주입은 컴포넌트 스캔과 자동 주입을 활용하는 자동 설정과 수동으로 빈을 등록하는 방식으로 나뉘어진다.
* **기본적으로는 편리한 자동 설정을 주로 사용**하며, 실제로도 스프링은 시간이 지남에 따라 점점 더 자동 설정을 선호하는 추세이다.
  * 현재의 스프링은 @Component 뿐만 아니라 서비스, 컨트롤러, 리포지토리 등의 계층 별 애플리케이션 로직을 자동으로 스캔할 수 있도록 지원한다.
  * 추가적으로 최근의 스프링 부트는 아예 컴포넌트 스캔을 기본으로 사용한다.
* **직접 작성한 설정 정보를 기반으로 애플리케이션의 구성부와 동작부를 구분하는 것은 이상적이지만, 이는 개발자 입장에서는 상당히 부담스러운 작업**이다.
  * 예를 들어, 스프링 빈을 추가하기 위해 어노테이션 하나를 명시하는 것과 설정 정보 파일을 손수 수정하는 것은 그 부담이 다르다.
  * 이러한 방식은 관리할 스프링 빈이 많아질수록 설정 파일 자체를 관리하는 것 역시 어려워진다.
  * **무엇보다, 자동 설정 방식을 사용하더라도 OCP와 DIP를 준수하는 데에는 아무런 문제가 없다**!

### 수동 설정을 사용하지 말아야 할까?
* 애플리케이션의 로직은 크게 다음과 같은 두 종류로 분류할 수 있다.
  1. 업무 로직
  2. 기술 지원 로직
* 때문에 빈 역시 업무 로직 빈과 기술 지원 로직 빈으로 분류할 수 있다.
  1. 업무 로직 빈: 컨트롤러, 서비스, 리포지토리 등은 모두 업무 로직에 해당한다.
     * **업무 로직 빈은 일반적으로 비즈니스 요구사항을 개발하기 위해 추가되거나 변경**된다.
  2. 기술 지원 로직 빈: 기술적인 문제나 공통된 관심사인 AOP를 처리하기 위해 주로 사용된다.
     * **기술 지원 로직 빈은 일반적으로 데이터베이스 연결이나 공통 로그 처리 등, 업무 로직을 지원하기 위한 하부 기술 또는 공통 기술들**이다.
* **업무 로직의 경우 숫자도 많고, 개발할 때마다 유사한 패턴이 존재하는 경향이 있으므로 자동 설정 기능을 적극적으로 활용하는 것이 바람직**하다.
  * 일반적으로, 업무 로직의 경우 문제가 발생했을 때에도 문제의 발생 지점을 파악하기 쉽다.
* 반면 기술 지원 로직의 경우 그 수는 매우 적지만, 애플리케이션에 광범위한 영향을 미친다.
  * 또한, 기술 지원 로직은 업무 로직과 달리 문제가 발생한 경우 이를 파악하기 어렵다.
  * 때문에 **이러한 기술 지원 로직들은 수동 설정을 활용하여 명확하게 명시하는 것이 바람직**하다.
* 즉, **애플리케이션에 광범위한 영향을 주는 기술 지원 객체들은 수동 설정을 통해 설정 정보 파일에 명시적으로 드러내는 것이 유지보수에 유리**하다.

### 수동 설정을 적극적으로 고려해야하는 경우
* **비즈니스 로직 중 다형성을 적극적으로 활용하는 경우에는 오히려 수동 설정을 적극적으로 고려**해야 한다.
* 예를 들어, 상술한 Map 주입 예시의 경우 다른 개발자와 협업하는 상황에서는 코드 상에서 의도가 명확하게 드러나지 않는다.
  * 이는 자동 설정을 사용하고 있기 때문에 발생하는 문제이며, 코드의 역할을 파악하기 위해 많은 코드를 찾아봐야 하는 수고를 강제한다.
* 이러한 경우에는 다음과 같은 방법 중 하나를 적용하여 한 눈에 파악 가능하도록 가독성을 향상시켜볼 수 있다.
  1. 수동 설정을 통해 스프링 빈으로 등록한다.
  2. 또는 특정한 패키지에 함께 묶어 둔 후에 자동 설정을 적용한다.
* 예를 들어, 수동 설정 파일을 활용하면 다음과 같이 임의의 타입에 대한 모든 스프링 빈이 명확하게 드러난다.
  * **해당 설정 파일만 보더라도 스프링 빈의 이름은 물론 어떠한 빈들이 주입될지 한 눈에 알아볼 수 있다**.
```
@Configuration
public class DiscountPolicyConfig {
    
    @Bean
    public DiscountPolicy fixDiscountPolicy() {
        return new FixDiscountPolicy();
    }
    
    @Bean
    public DiscountPolicy rateDiscountPolicy() {
        return new RateDiscountPolicy();
    }
}
```
* **그럼에도 빈 자동 등록을 적용하고자 하는 경우, 대신 DiscountPolicy와 관련된 구현체들을 하나의 패키지에 모아 관리하는 방법을 고려**할 수 있다.
* 중요한 것은 수동 설정을 사용하든지, 컴포넌트 스캔을 사용하든지 간에 비즈니스 로직에서 다형성을 적극적으로 활용하는 경우에는 의도가 명확해야한다는 점이다.

### 스프링과 스프링 부트의 자동 등록 빈 객체
* **스프링과 스프링 부트가 자동 설정으로 등록하는 빈들은 예외이며, 이러한 유형의 빈은 스프링 자체를 잘 이해하고 의도대로 활용하는 것이 중요**하다.
  * 스프링 부트를 예로 들어 DataSource는 데이터베이스 연결에 사용하는 기술 지원 로직까지 내부적으로 자동 등록한다.
  * 이러한 부분은 무리하게 수동 설정으로 드러낼 필요 없이, 매뉴얼에 따라 스프링 부트의 의도대로 활용하는 것이 바람직하다.
* 반면, 스프링 부트가 아닌 개발자 자신이 등록한 기술 지원 객체를 빈으로 등록할 경우 수동으로 등록하여 명확히 드러내는 것이 바람직하다.

### 결론
* 자동 설정 기능을 주로 사용한다.
* 직접 개발한 기술 지원 객체의 경우, 수동 설정을 활용한다.
  * 나아가 다형성을 적극적으로 활용하는 비즈니스 로직 역시 수동 설정을 고려해야한다.