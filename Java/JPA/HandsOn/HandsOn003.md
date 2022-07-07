# HandsOn
## 2022-07-06 Wed

### 비즈니스 로직을 처리하는 엔티티
* 상품 주문과 같은 요구사항에서, 재고 관리와 같은 비즈니스 로직은 엔티티에서 처리하도록 개발할 수 있다.
  * **도메인 주도 설계 관점에서, 엔티티 자체적으로 처리할 수 있는 기능은 엔티티 안에 위치한 비즈니스 로직으로 처리하는 것이 바람직**하다.
* 또한 **객체지향적으로 생각했을 때 각 데이터와 관련된 비즈니스 로직을 해당 데이터를 갖는 객체가 포함하는 것은 응집도 측면에서도 적절**하다.
* 이러한 이유에서 **세터를 무분별하게 추가하는 것보다는 적절한 핵심 비즈니스 로직을 처리하는 메소드를 통해 값을 수정하도록 하는 것이 이상적**이다.
  * 즉, 외부에서 객체의 값을 읽어들여 계산한 후 세터를 통해 설정하는 방식은 지양해야 한다. 
  * 대신 **객체 안에 비즈니스 로직을 추가하고, 필요한 검증을 수행하는 것이 더 객체지향적인 방식**이다.
```
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
@Getter
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();

    /* 비즈니스 로직 */
    public void addStock(int quantity) {
        this.stockQuantity += quantity;
    }
    /* 비즈니스 로직 */
    public void removeStock(int quantity) {
        if(this.stockQuantity < quantity) {
            throw new NotEnoughStockException("need more stock");
        }

        this.stockQuantity -= quantity;
    }
}
```

### 엔티티 생성 메소드 추가하기
* **객체의 생성 과정이 충분히 복잡한 경우, 객체 외부에서 new 연산자와 세터를 통해 객체를 구성하는 방식보다는 static 메소드를 제공하는 것이 바람직**하다.
  * 이를 통해 객체의 세터의 사용을 지양하고, 객체의 내부 구현을 가능한 한 외부로부터 격리시킬 수 있다.
```
public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems) {
    Order order = new Order();
    order.setMember(member);
    order.setDelivery(delivery);
    for (OrderItem orderItem : orderItems) {
        order.addOrderItem(orderItem);
    }
    order.setStatus(OrderStatus.ORDER);
    order.setOrderDate(LocalDateTime.now());

    return order;
}
```
* 그러나 여러 개발자와 협업하는 환경의 경우, 누군가는 생성자와 세터를 사용한 방식으로 개발을 진행할 수도 있다.
  * **이러한 상태가 지속되면 생성 방식이 일관화되지 않아, 생성 메소드의 내용이 변경되었을 때 프로젝트 전체에 이를 반영하기 매우 어려워질 수 있다**.
* 때문에 **엔티티의 생성 방식을 강제할 필요가 있으며, JPA는 스펙 상 엔티티의 protected 기본 생성자까지 지원하므로 이를 활용하는 것이 바람직**하다.
  * JPA를 사용하는 개발자 사이에서 protected 기본 생성자가 포함된 클래스는 기본 생성자를 사용하지 말라는 의미로 받아들여지는 점을 적극 활용할 수 있다.
```
protected OrderItem(){}
```
* 또는 동일한 기능을 다음과 같은 어노테이션을 통해 롬복으로부터 지원받을 수도 있다.
```
@NoArgsConstructor(access = AccessLevel.PROTECTED)
```
* 이렇듯 코드를 제약적인 방식으로 정의하는 것은 바람직하며, 이로 인해 다른 개발자에게 static 생성 메소드의 사용을 강제할 수 있게 된다. 
  * 보다 제약적인 구현 방식을 통해 더 좋은 설계와 유지보수가 가능한 방향으로 개발 방향을 이끌어갈 수 있다.

### 주문 기능 개발하기
* 주문을 저장하는 서비스 메소드는 다음과 같이 작성해볼 수 있다.
```
@Transactional
public Long order(Long memberId, Long itemId, int count) {
    Member member = memberRepository.findOne(memberId);
    Item item = itemRepository.findOne(itemId);

    Delivery delivery = new Delivery();
    delivery.setAddress(member.getAddress());

    OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);

    Order order = Order.createOrder(member, delivery, orderItem);

    orderRepository.save(order);
    return order.getId();
}
```
* 이 때, **중요한 것은 orderRepository.save()만을 호출하지 delivery와 orderItem을 영속화하기 위한 어떠한 코드도 작성하지 않았다는 점**이다.
  * **이는 Order 클래스의 delivery와 orderItems 구현부에는 `cascade = CascadeType.ALL` 옵션이 걸려있기 때문**이다.
  * cascade 옵션이 적용된 경우, Order 클래스에 대한 영속화 과정에서 사용되는 delivery와 orderItems 엔티티를 모두 함께 영속화한다.
```
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
private List<OrderItem> orderItems = new ArrayList<>();

@OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
@JoinColumn(name = "delivery_id")
private Delivery delivery;
```
* 그러나 **이러한 cascade 옵션은 무분별하게 사용하지 않아야 하며, order와 같은 private owner의 상황에서만 적용하는 것이 바람직**하다.
* 이 경우, orderItems와 delivery는 다른 엔티티로부터 사용되어지지 않고 order만이 참조하므로 order는 private owner이다.
  * 즉, orderItems와 delivery가 오직 order에 의해서만 참조되고 사용되어지는 경우에만 cascade 옵션의 도움을 받을 수 있다.
  * 반면 orderItems와 delivery가 다른 엔티티를 참조하는 것은 상관이 없다.
  * **중요한 것은 order만이 orderItems, delivery를 참조하고, 동일한 생명주기에서 관리되어도 무방한 경우에만 사용되어야한다는 점**이다.  
* 그러나 **비즈니스 상 delivery와 orderItems가 중요한 엔티티이기에 다른 엔티티들로부터 참조되는 상황이라면 cascade 옵션의 적용은 지양**해야 한다.
  * 그렇지 않은 경우, 단순히 order를 삭제했음에도 다른 엔티티로부터 참조되는 delivery 또는 orderItems가 삭제될 가능성이 존재한다.
  * **이러한 경우에는 엔티티 별로 별도의 리포지토리를 작성하여 persist를 따로 진행해주는 것이 바람직**하다.
* **상술한 예제는 order가 다른 두 엔티티의 private owner이며, 동일한 생명주기로 관리되어야 하는 조건을 충족하기 때문에 cascade 옵션이 적용**되었다.

### 주문 취소 기능 개발하기
* 주문 취소 기능의 경우, 다음과 같은 메소드를 서비스 클래스에 정의할 수 있다.
```
@Transactional
public void cancelOrder(Long orderId) {
    Order order = orderRepository.findOne(orderId);
    order.cancel();
}
```
* 상술한 바와 같이 코드가 매우 짧으며, 이는 비즈니스 로직을 엔티티에 구현한 데에 더해 JPA를 사용하기에 얻어지는 이점으로 볼 수 있다.
  * MyBatis 등 쿼리를 직접 작성하는 라이브러리를 사용하는 경우, 서비스 로직 상에서 모든 작업을 처리해줘야하기에 메소드가 길고 복잡해질 수 밖에 없다.
  * 그러나 **JPA는 객체 상태 변화에 따른 영속화를 지원하므로 코드가 지저분해지지 않으며, 이는 JPA의 엄청난 강점**이기도 하다.

### 도메인 모델 패턴과 트랜잭션 스크립트 패턴
* 상술한 주문과 주문 취소 기능의 경우, 대부분의 비즈니스 로직은 엔티티에 포함되도록 구현되어 있다.
  * 때문에 서비스 계층은 단순히 요청을 엔티티에 위임하는 형태를 따르게 된다.
* 이렇듯 **엔티티가 비즈니스 로직을 포함하며 객체지향의 특성을 적극적으로 활용하는 패턴은 도메인 모델 패턴에 해당**한다.
* 반대로, **엔티티는 별도의 비즈니스 로직을 거의 갖지 않고 서비스 계층에서 대부분의 로직을 처리하는 것은 트랜잭션 스크립트 패턴에 해당**한다.
* 일반적으로 SQL을 직접 다루는 방식은 트랜잭션 스크립트 패턴을 따르기 쉽다.
  * 반면, JPA 등 ORM을 사용하는 경우에는 도메인 모델에 핵심 비즈니스 로직을 정의하는 도메인 모델 패턴을 따르기 쉽다.
* **두 패턴 사이에 우열이 있는 것은 아니며, 상황에 따라 어떤 패턴이 더 유지보수가 쉬울지 고려하여 도입하는 것이 이상적**이다.
  * 즉, 두 패턴은 문맥에 따라 트레이드오프가 존재한다.
  * 나아가 **하나의 프로젝트 안에서도 두 패턴은 양립할 수 있으므로 현재 개발 중인 문맥에 더 적절한 패턴을 적용하는 것이 바람직**하다.

## 2022-07-07 Thu
### 좋은 테스트 케이스
* 사실 스프링과 데이터베이스에 의존적인 테스트 케이스는 이상적인 테스트로 볼 수 없다.
* **정말 좋은 테스트 케이스는 어떠한 의존성도 없이 임의의 메소드의 동작성만을 테스트**할 수 있어야 한다.
  * 때문에 **느리고 많은 범위를 확인하는 통합 테스트 케이스 여럿보다는 작은 기능을 테스트하는 양질의 단위 테스트 케이스를 여럿 작성하는 것이 바람직**하다.
* 덧붙여 **도메인 모델 패턴의 엄청난 장점 중 하나는 리포지토리 등의 의존성 없이도 각 엔티티가 갖는 비즈니스 로직 자체만을 테스트할 수 있다는 점**이다.
  * 이는 도메인 모델 패턴에서의 엔티티가 자체적으로 핵심 비즈니스 로직을 갖기 때문에 가능한 특징이다.
* 이렇듯 **이상적인 단위 테스트는 데이터베이스, 리포지토리 등 외부 요인 없이 메소드의 동작성만을 검증할 수 있어야 한다**.

### JPA와 동적 쿼리
* 예를 들어 회원의 이름과 주문의 상태로 주문을 검색하는 경우, OrderSearch 클래스를 추가하여 다음과 같이 작성할 수 있다.
```
// OrderRepository.java

public List<Order> findAll(OrderSearch orderSearch) {
    return em.createQuery("select o from Order o join o.member m" +
            " where o.status = :status" +
            " and m.name like :name", Order.class)
            .setParameter("status", orderSearch.getOrderStatus())
            .setParameter("name", orderSearch.getMemberName())
            .setFirstResult(0) // offset
            .setMaxResults(20) // limit
            .getResultList();
}
```
* 그러나 **위의 코드는 orderSearch 객체에 항상 모든 파라미터가 존재하는 경우를 가정하며, 동적인 쿼리라고 볼 수는 없다**.
* 동적인 쿼리를 작성하기 위해서는 크게 다음과 같은 방법을 고려해볼 수 있다.
  1. 문자열 처리하기
  2. JPA Criteria 사용하기
* 그러나 상술한 두 방법 모두 동적 쿼리를 위해서 유지보수성이 낮은 코드를 만들게되는 단점이 수반된다.
  * 극단적으로 말해서, 두 방식 모두 실무에서는 사용할 수 없을 정도로 복잡성이 높다.
* 이에 대한 **대안으로 설계된 queryDSL 라이브러리를 사용할 수 있으며, 해당 라이브러리를 통해 동적 쿼리 문제를 간단하게 해결**할 수 있다.
  * 나아가, 정적 쿼리 역시 복잡해지는 경우 queryDSL을 통해 복잡도를 낮추고 장기적인 유지보수성을 높일 수 있다.