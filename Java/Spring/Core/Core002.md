# Core
## 2022-06-20 Mon

### 프로젝트 설정
* `spring.start.io`에서 별도의 종속성 없이 프로젝트를 생성할 경우, 스프링 코어와 관련된 종속성만이 다음과 같이 포함된다.
```
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```
* 이 경우, `starter-web` 과 같은 웹 관련 종속성이 없으므로 프로젝트를 실행해도 exit code 0으로 종료된다.

### 확정되지 않은 요구사항
* 실무에서는 요구사항이 모두 확정되지 않은 상태에서 개발을 시작해야할 수도 있다.
  * 예를 들어, 주문 시스템의 할인 정책은 최대한 숙고하기 위해 확정이 늦춰질 수도 있다.
  * 최악의 경우에는 할인을 적용하지 않을 수도 있으나, 정책이 결정될 때까지 개발을 미룰 수는 없다.
* **인터페이스를 적절히 활용하여 역할과 구현을 성공적으로 분리할 경우, 이러한 상황 속에서도 필요한 다른 부분들을 개발하는 것이 가능**하다.

### 다이어그램 활용하기
* 요구사항을 토대로 도메인을 설계하는 경우, 기본적으로 다음과 같은 다이어그램을 활용해볼 수 있다.
  1. 회원 도메인 협력 관계 다이어그램: 회원 도메인에 포함되는 여러 객체 간의 협력을 개략적으로 묘사한다.
     * 해당 다이어그램은 기획자가 함께 공유할 수도 있다.
  2. 회원 클래스 다이어그램: 실제 구현 단위로 구현할 경우 어떠한 인터페이스와 구현체들이 필요한지 묘사한다.
     * 정적인 다이어그램이며, 개발자는 1.을 토대로 해당 다이어그램에 내용을 구체화한다.
  3. 회원 객체 다이어그램: 서버에서 애플리케이션이 동작할 때, 실제로 만들어진 객체 간의 참조 관계를 묘사한다.
* 회원 객체 다이어그램의 경우 동적이며, 서버에서 동작하는 애플리케이션에서 동적으로 결정되는 구현체를 한 눈에 보기 위해 작성한다.
  * 구현체는 동적으로 결정되므로, 회원 클래스 다이어그램만으로는 실제로 어떠한 객체와 협력하는지 판단하기 어려울 수 있기 때문이다.

### MemberRepository의 구현
* 인메모리 회원 리포지토리의 경우, 다음과 같이 구현할 수 있다.
  * 이 때, **인메모리 저장소 역할을 수행하는 `storage` 변수는 실무의 경우 동시성 이슈로 인해 ConcurrentHashMap을 사용**해야 한다. 
```
public class InMemoryMemberRepository implements MemberRepository {
    
    private static Map<Long, Member> storage = new HashMap<>(); 
    
    @Override
    public void save(Member member) {
        storage.put(member.getId(), member);
    }

    @Override
    public Member findById(Long id) {
        return storage.get(id);
    }
}
```

### MemberService의 구현
* 회원 서비스의 경우, 인터페이스인 `MemberService`를 구현하는 `MemberServiceImpl` 클래스를 다음과 같이 구현할 수 있다.
  * 이 때, **특정 인터페이스에 대한 구현체가 단 하나 뿐인 경우 관례상 `Impl`이라는 접미사를 붙여 작성**할 수 있다.
```
public class MemberServiceImpl implements MemberService {
    
    private final MemberRepository memberRepository = new InMemoryMemberRepository();
    
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

## 2022-06-21 Tue
### 도메인 객체 사용하기
* 다음과 같이 `member/MemberApplication.java`를 작성할 경우, 스프링과 관련된 무엇도 사용하지 않는 순수 Java 코드만으로 회원 도메인이 완성된다.
  * 그러나 **하기와 같은 클래스를 작성한 도메인 객체의 기능을 테스트하기 위해 작성한 것이라면, 대신 테스트 케이스를 작성하는 것이 바람직**하다.
  * 테스트 케이스를 작성하는 경우, 눈으로 모든 결과를 검증할 필요도 없으며 지속적인 코드의 유지보수가 가능하다.
  * **현대적인 애플리케이션을 작성하는 경우, 테스트 케이스의 작성은 선택이 아닌 필수**이다.
```
public class MemberApplication {
    public static void main(String[] args) {
        MemberService memberService = new MemberServiceImpl();
        Member member = new Member(1L, "member", Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.find(1L);
        System.out.println("member: " + member.getName());
        System.out.println("found: " + findMember.getName());
    }
}
```

### 회원 서비스 구현체가 갖는 문제
* 현재 회원 서비스는 다음과 같은 멤버 변수를 갖는다.
```
private final MemberRepository memberRepository = new InMemoryMemberRepository();
```
* 해당 멤버 변수는 다형성을 활용하지만, 추상화인 `MemberRepository`와 실제 구현체인 `InMemoryMemberRepository` 모두에 의존한다.
* 때문에 **의존성 역전 원칙을 위배하며, 구현체를 수정할 경우 회원 서비스 클래스 자체를 수정해야하므로 개방 폐쇄 원칙도 위배**하고 있다.

### 주문과 할인 도메인 설계
* 주문 및 할인과 관련된 요구사항은 다음과 같다.
  1. 회원은 상품을 주문할 수 있다.
  2. 회원 등급에 따라 할인 정책을 적용할 수 있다.
  3. 할인 정책은 추후에 변경될 가능성이 높다.
* 이러한 요구사항을 충족시키기 위한 도메인 객체들의 역할은 다음과 같을 것이다.
  1. 클라이언트는 새로운 주문을 만들기 위해 주문 서비스에 요청한다.
  2. 할인을 적용하기 위해 회원의 등급을 조회해야하므로, 주문 서비스는 회원 저장소에서 회원을 조회할 수 있다.
  3. 주문 서비스는 회원 등급에 따른 할인 여부의 결정을 할인 정책에 위임한다.
  4. 주문 서비스는 할인 결과를 포함하는 주문 결과를 반환한다.
* **이상적인 설계의 흐름은 요구사항을 기반으로 역할을 추출하고, 역할을 기반으로 구현체를 정의하는 것**이다.
  * 이를 토대로 구현 세부 사항을 최대한 늦게, 유연하게 변경하는 것이 가능하다.

### 주문 서비스 객체 구현하기
* 주문 서비스 객체의 경우, 주문 정책과 회원 리포지토리라는 두 가지 종속성을 갖는 상태에서 다음과 같이 작성될 수 있다.
```
public class OrderServiceImpl implements OrderService {

    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
    private final MemberRepository memberRepository = new InMemoryMemberRepository();

    @Override
    public Order create(Long memberId, String productName, Long price) {
        Member member = memberRepository.findById(memberId);
        Long discountPrice = discountPolicy.discount(member, price);

        return new Order(memberId, productName, price, discountPrice);
    }
}
```
* **`create` 메소드에서 확인할 수 있듯, 해당 클래스는 협력 상대로서의 `DiscountPolicy`를 가질 뿐 할인과 관련된 어떠한 세부사항도 알 필요가 없다**.
  * 즉, 단일 책임 원칙을 준수하므로 변경의 이유는 하나만이 존재하게 된다.
  * 때문에 추후 요구 사항의 변경으로 할인 정책의 내부 구현이 수정되더라도 주문 서비스 객체는 아무런 영향을 받지 않는다.
* 또한 할인 정책을 의미하는 객체인 DiscountPolicy에는 Member의 등급 정보만을 넘겨도 되며, 상술한 코드와 같이 Member 자체를 넘겨도 무방하다.
  * 이는 미래의 확장성을 고려하여 더 좋은 방식을 개발자가 선택해야 한다.

### 테스트 케이스 작성하기
* 통합 테스트가 아닌 경우, 단위 테스트 케이스는 수천개가 존재하더라도 수 초 이내에 완료된다.
  * 이는 **각 단위 테스트 케이스가 스프링이나 컨테이너의 도움 없이 Java 코드 자체적으로 실행되기 때문**이다.
* 때문에 각각의 단위 테스트 케이스를 잘 만드는 것은 중요하며, 이를 잘 유지보수해나가는 것 역시 중요하다.