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

### 주문 엔티티 개발하기
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