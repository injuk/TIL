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