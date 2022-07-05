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