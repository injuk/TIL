# HandsOn
## 2022-07-01 Fri

### JPA 애플리케이션 생성하기
* Spring.io를 통해 다음과 같은 의존성을 포함하는 프로젝트를 생성한다.
```
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```
* 이를 통해 import되는 라이브러리 목록을 확인하고 싶은 경우, 터미널에서 gradle wrapper를 활용하여 다음과 같이 입력한다.
  * 또는 인텔리제이 우측의 Gradle 탭을 활용할 수도 있다.
```
./gradlew dependencies --configuration compileClasspath
```

## 2022-07-02 Sat
### View 환경 설정
* 스프링 부트의 템플릿 엔진 중, 타임리프는 기본적으로 `resources:templates/${viewName}.html`경로의 뷰를 매핑한다.
* 예를 들어, 아래의 코드가 반환하는 것은 문자열이 아닌 `resource/templates` 디렉토리의 html 파일의 이름이다.
```
@Controller
public class HelloController {

    @GetMapping("/hello")
    // 컨트롤러는 Model에 데이터를 적재하여 View로 전달할 수 있다. 
    public String hello(Model model) {
        // View는 ingnoh라는 키에 매핑된 hello!!! 문자열을 전달받는다.
        model.addAttribute("ingnoh", "hello!!!");

        // 문자열 hellooooo가 아닌 resources/templates의 hellooooo.html로 리졸브한다.
        return "hellooooo";
    }
}
```
* 상술한 코드는 Model 객체를 통해 `[ingnoh, hello!!!]` 데이터를 뷰인 hellooooo.html에 전달한다.
  * 이렇게 전달된 data는 다음과 같이 꺼내어 사용할 수 있다.
```
<body>
    <p th:text="'안녕하세요. ' + ${ingnoh}">아무것도 없어요!</p>
</body>
```
* 상술한 코드의 경우, 순수한 HTML이라면 `아무것도 없어요!`가 출력되어야 한다.
  * 그러나 이 경우, 타임리프에 의해 서버사이드 렌더링되므로 `th:text`에 명시된 값인 `안녕하세요. hello!!!`가 출력된다.
* 컨트롤러에서 문자열 hellooooo만을 반환하는 데도 적절한 HTML 문서를 찾을 수 있는 이유는 스프링 부트와 타임리프가 조합되어 기본 설정을 따르기 때문이다.
  * 스프링 부트는 타임리프와 관련된 기본 경로를 `resource/templates`로 정의한다.
* 반면, 렌더링 없는 HTML 또는 이미지 파일 등 완전한 정적 이미지를 반환하고자 하는 경우 `static` 경로를 활용한다.
* **이렇듯 정적 파일을 제공하고자 하는 경우에는 static 경로를 사용하고, 템플릿 엔진을 활용한 뷰를 반환하고자 하는 경우에는 templates 경로를 활용**한다.

### application.yml 작성하기
* 설정이 길어질수록 application.properties보다 application.yml의 가독성이 좋을 수 있다.
* 작성한 애플리케이션이 로컬에서 동작하는 H2 데이터베이스와 상호작용할 수 있도록, datasource와 jpa 설정을 아래와 같이 작성한다.
```
server:
  port: 8079

spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/workspace/study/h2/test
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create  # 현재의 엔티티를 모두 제거한 후에 재생성하는 옵션이다.
    properties:
      hibernate:
#        show_sql: true # System.out으로 로그를 출력하므로, 운영 환경에서는 사용하지 않아야 한다.
        format_sql: true

logging:
  level:
    org.hibernate.SQL: debug  # logger를 활용하여 로그를 남긴다.
```

### 리포지토리 작성하기
* 리포지토리는 마치 DAO와 유사한 동작을 수행하며, 데이터베이스와 상호작용하기 위해 `EntityManager`를 필요로 한다.
* 이 때, **@PersistenceContext 어노테이션을 명시하는 것으로 별도의 설정 파일 없이도 `EntityManager`를 주입**받을 수 있다.
```
@Repository
public class MemberRepository {

    @PersistenceContext // Config 없이도 Spring Boot가 엔티티 매니저를 주입해준다.
    private EntityManager em;

    public Long save(Member member) {
        em.persist(member);

        // CQS 원칙을 준수하기 위해 ID 정도만을 반환한다.
        return member.getId();
    }

    public Member find(Long id) {
        return em.find(Member.class, id);
    }
}
```

### 테스트 작성하기
* 다음과 같이 테스트를 작성하여 DB 상호작용을 확인할 수 있다.
  * 이 때, EntityManager를 활용한 모든 데이터 변경은 반드시 트랜잭션 안에서 이루어져야 하므로 @Transactional 반드시 어노테이션을 명시한다.
  * @Transactional 어노테이션은 스프링 프레임워크가 제공하는 어노테이션으로 적용한다.
```
@SpringBootTest
class MemberRepositoryTest {

    @Autowired MemberRepository memberRepository;

    @Test
    @Transactional
    public void saveTest() {
        // given
        Member member = new Member();
        member.setUsername("ingnoh");

        // when
        Long id = memberRepository.save(member);
        Member found = memberRepository.find(id);

        // then
        assertThat(found.getId()).isEqualTo(id);
        assertThat(found.getUsername()).isEqualTo(member.getUsername());
    }
}
```
* 그러나 상술한 테스트가 성공한 후, `ddl-auto: create` 옵션에 의해 H2 데이터베이스에 자동 생성된 MEMBER 테이블에서 추가된 항목을 확인할 수는 없다.
  * **이는 @Transactional 어노테이션이 테스트 케이스에 명시되면 테스트 완료 후 결과를 롤백하기 때문**이다. 
  * 이를 통해 동일한 테스트 케이스를 반복적으로 실행할 수 있게 된다.
  * 만일 롤백하지 않고 결과를 확인하고 싶은 경우, @Rollback(value = "false") 어노테이션을 추가로 명시한다.
* 이렇듯 JPA는 엔티티 정보를 확인하고 그에 맞추어 테이블을 자동으로 생성하거나 삭제할 수 있다.

### 동일한 영속성 컨텍스트 내에서의 동일성
* 상술한 테스트 케이스에 `assertThat(found).isSameAs(member);`를 추가하더라도 테스트는 성공한다.
  * 이는 데이터베이스에서 조회된 엔티티와 저장했던 엔티티가 동일하다는 것을 의미한다.
* **같은 트랜잭션 안에서 저장하고 조회할 경우, 동일한 영속성 컨텍스트를 갖는다**.
* **동일한 영속성 컨텍스트의 경우, 두 객체의 id 값이 같다면 동일한 엔티티로 식별**된다.
  * 즉, 영속성 컨텍스트 내에서 식별자가 같으면 같은 엔티티로 인식된다.
  * 이는 1차 캐시에서 영속성 컨텍스트에 의해 관리되는 엔티티를 활용하는 것으로, select 쿼리조차 요청하지 않고 캐싱된 객체를 가져온다.

## 2022-07-03 Sun
### 스프링 부트에 의한 설정 자동화
* 이렇듯 스프링 부트를 활용하면 복잡한 JPA 설정이 대부분 자동화된다.
  * 심지어 `persistence.xml` 파일 조차 작성할 필요가 없다.

### 쿼리 파라미터 로깅
* JPA를 활용하여 개발하는 경우, 쿼리 요청 또는 커넥션을 가져오는 시점이 언제인지 궁금한 경우가 많다.
* 때문에 application.yml에 `org.hibernate.type: trace` 옵션을 다음과 같이 추가하여 쿼리 파라미터를 확인할 수 있다.
```
logging:
  level:
    org.hibernate.SQL: debug
    org.hibernate.type: trace
```
* 또한, 더더욱 자세한 실제 쿼리를 확인하고자 하는 경우 `build.gradle`에 다음과 같은 종속성을 추가한다.
  * 스프링 부트가 관리하는 라이브러리의 경우, 각 스프링 부트 버전 별 필요한 라이브러리의 버전을 명시하므로 lombok 등은 굳이 버전을 명시할 필요가 없다.
  * 그러나 스프링 부트가 지원하지 않는 라이브러리의 경우 아래와 같이 버전 정보를 반드시 명시해야 한다.
```
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.8.0'
```
* 해당 종속성을 추가하여 그래들을 리로드한 후, 별도의 추가 설정 없이 테스트 코드를 실행시키면 쿼리를 확인할 수 있다.
  * 이렇듯 별도의 설정 과정 없이, auto configuration이 가능하다는 것은 스프링 부트의 커다란 장점이다.
* 그러나 이러한 라이브러리들은 성능을 크게 떨어트릴 수도 있으므로, 개발 환경이 아닌 운영 환경에서는 반드시 성능 테스트를 진행한 후 사용을 결정해야 한다.

### 도메인 분석
* 임의의 기능을 구현하는 경우, 일반적으로는 다음과 같은 흐름을 따른다.
  1. 요구사항 분석
  2. 도메인 모델과 테이블 설계
  3. 엔티티 클래스의 개발
* 예를 들어, 간단한 쇼핑몰이 다음과 같은 기능을 필요로 한다고 하자.
  1. 사용자는 회원으로 가입할 수 있으며, 가입된 회원 정보를 확인할 수도 있다.
  2. 사용자는 상품을 등록할 수 있으며, 등록된 상품을 수정하거나 목록을 확인할 수도 있다.
  3. 사용자는 상품을 주문할 수 있으며, 주문 내역을 확인하거나 취소할 수도 있다.
     1. 이 때, 상품 주문시 배송 정보를 입력할 수 있다.
  4. 상품은 재고 관리가 가능하며, 카테고리 별로 구분할 수 있다.

### 엔티티 클래스 개발하기
* **양방향 연관관계의 경우 연관관계의 주인을 설정해줄 필요가 있으며, 일반적으로는 FK와 가까운 엔티티를 주인으로 설정**한다.
  * 예를 들어 주문과 회원의 관계에서 주문 테이블은 member_id를 FK로 가지므로, 연관관계의 주인은 주문 테이블이 되는 것이 바람직하다.
```
// Order.class
@ManyToOne
@JoinColumn(name = "member_id")
private Member member;

// Member.class
@OneToMany(mappedBy = "member")
private List<Order> orders = new ArrayList<>();
```
* 엔티티 클래스에 **enum 타입을 사용하는 경우, 반드시 다음과 같이 `@Enumerated(EnumType.STRING)` 어노테이션을 명시**한다.
  * 작성하지 않는 경우, 기본 EnumType은 ORDINAL로 적용된다.
  * 이는 0부터 정수로 매핑하는 방식이며, 이후 새로운 enum 값이 READY와 COMP 사이에 추가되었을 때 오작동을 일으킬 가능성이 매우 높다. 
```
@Enumerated(EnumType.STRING)
private DeliveryStatus status; // READY, COMP
```
* 일대 일 관계인 경우, FK는 두 테이블 중 어디에 두어도 상관이 없다.
  * **실무의 경우, 편의성을 위해 더 자주 접근되는 테이블에 두는 것이 일반적**이다. 
  * 예를 들어 **주문은 배송을 참조할 일이 많지만 배송으로 주문을 조회하는 경우는 없다고 한다면, 주문 테이블에 FK를 두는 것이 바람직**하다. 
```
// Order.class
@OneToOne
@JoinColumn(name = "delivery_id") // Order 테이블은 Delivery 테이블의 delivery_id 컬럼을 기준으로 매핑한다.
private Delivery delivery;

// Delivery.class
@Id @GeneratedValue
@Column(name = "delivery_id")
private Long id;

@OneToOne(mappedBy = "delivery")  // Delivery 테이블은 Order 테이블의 delivery 필드를 기준으로 매핑된다.
private Order order;
```

## 2022-07-04 Mon
### 다대 다 관계 매핑하기
* M:N 관계는 실무에서 한계점이 명확하므로 사용을 지양해야 하지만, 어쩔수 없는 경우에는 다음과 같이 작성한다.
  * 이 때, 연관 관계의 주인에는 반드시 @JoinTable 어노테이션과 관련된 설정을 명시해야 한다.
  * **이는 M:N 관계를 매핑하기 위해 관계형 데이터베이스에 중간 테이블이 필요하기 때문으로, 각 설정은 중간 테이블의 내용을 결정짓는 속성**이 된다.
```
// Category.java
@ManyToMany
@JoinTable(
    name = "category_item",
    joinColumns = @JoinColumn(name = "category_id"),
    inverseJoinColumns = @JoinColumn(name = "item_id")
)
// 이는 중간 테이블이 필요하기 때문으로, 객체는 컬렉션을 활용한 다대 다 관계 구현이 가능하지만 관계형 데이터베이스는 컬렉션 관계를 양 측에 가질 수 없다.
// 때문에 1:N, N:1 로 풀기 위한 중간 테이블이 필수적이다.
private List<Item> items = new ArrayList<>();

// Item.java
@ManyToMany(mappedBy = "items")
private List<Category> categories = new ArrayList<>();
```

### 자기 자신과 매핑하기
* 예를 들어, 카테고리 테이블은 엔티티 간에 1:N의 부모 자식 관계를 가질 수 있다고 하자.
* 이는 **같은 엔티티 간의 연관관계 매핑을 통해 해결할 수 있으며, 마치 다른 테이블과 연관관계를 매핑하듯이 작성하는 것으로 해결이 가능**하다.
```
// Category.java
// 동일한 엔티티 내에서 양방향 관계를 매핑할 수 있으며, 어렵게 생각할 것 없이 다른 엔티티와 매핑하는 경우와 동일하게 작성하면 된다.
@ManyToOne
@JoinColumn(name = "parent_id")
// 해당 필드는 자식 역할의 카테고리가 사용하며, 자식 카테고리 관점에서는 하나의 부모 카테고리만을 갖는다.
// 1:N 관계에서 자식 카테고리가 N이 되므로, 자식 카테고리가 사용하는 해당 필드가 연관관계의 주인이 된다.
private Category parent;

@OneToMany(mappedBy = "parent")
// 해당 필드는 부모 역할의 카테고리가 사용하며, 부모 카테고리 관점에서 여러 자식 카테고리를 가질 수 있다.
// 때문에 해당 필드는 mappedBy로 연관관계의 주인이 아님을 명시하다.
private List<Category> child = new ArrayList<>();
```

### 엔티티 클래스 개발시 참고사항
* **이론적으로는 게터와 세터를 모두 제공하지 않고 캡슐화하는 것이 바람직하지만, 실무에서는 데이터를 조회할 일이 많으므로 게터를 허용하는 것이 편리**하다.
  * 정말 데이터를 조회하는 역할만을 수행하는 게터의 경우, 수백 번 호출하더라도 사이드 이펙트가 발생하지 않는다.
  * 그러나 **세터의 경우 인스턴스의 값을 변경하므로 제공하지 않는 것이 바람직하며, 대신 명확한 변경 지점을 갖는 별도의 메소드를 제공**해야 한다.
* 엔티티 식별자의 경우, Java 코드 상에는 id라고 적더라도 `@Column(name="member_id")` 형태와 같은 어노테이션을 명시하는 것이 바람직하다.
  * 이는 Java 코드 상에서는 `member.id`와 같은 형식으로 접근되므로 의미를 파악하기 쉽지만, 클래스 정보가 없는 데이터베이스에서는 이를 구분하기 어렵다.
  * 또한 **데이터베이스 테이블 관리와 관련된 관례상으로도 `테이블명_id`와 같은 형태를 적용하는 경우가 많다**. 
  * 반면, **Java 클래스 상에서는 id 대신 memberId와 같은 명명을 적용해도 무방하지만 되도록 엔티티 클래스 간의 일관성을 유지하는 것이 바람직**하다.
* **실무에서는 @ManyToMany 어노테이션을 적용하는 M:N 연관관계는 절대로 사용하지 말아야 한다**.
  * **M:N 관계는 언뜻 편리해보이지만, 생성되는 중간 테이블에 등록일 또는 수정일 등의 컬럼을 추가할 수 없는 치명적인 단점이 존재하며, 운영도 어렵다**.
  * 때문에 **M:N 관계는 1:N과 N:1 관계로 풀어낼 수 있도록 중간 엔티티 역할을 수행할 테이블을 추가하는 것이 바람직**하다.

### 값 타입의 사용
* 상술한 예제의 경우, 주소를 의미하는 정보는 다음과 같이 별도의 클래스로 정의되고 있다.
  * 이렇듯 **도메인에서 사용되는 값을 의미하기 위한 클래스를 값 타입**이라고 부를 수 있다.
* 이 때, **JPA의 경우 스펙상 엔티티와 임베디드 타입(@Embeddable)에 대해 리플렉션 또는 프록시와 같은 기술을 사용하기 위해 기본 생성자를 필요**로 한다.
  * **기본 생성자를 public으로 설정해도 무방하지만, 이 경우 의도치 않게 호출될 가능성이 존재하므로 JPA 스펙은 protected 생성자까지 허용**하고 있다.
```
@Embeddable
@Getter
public class Address {

    private String city;
    private String street;
    private String zipcode;
    
    // JPA는 기본 생성자를 필요로 한다.
    protected Address() {}

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }    
}
```
* 이 때, **값 타입은 반드시 변경 불가능한 불변적인 클래스로 설계**되어야 한다. 
  * 때문에 세터를 제공하지 않고, 반드시 생성 시점에만 값을 설정하는 것이 좋은 설계이다.
  * 이로 인해 값 타입은 변경 불가능해지며, 변경을 위해서는 새로운 값 객체를 생성하는 식으로 사용되어야 한다.

### 즉시 로딩이란?
* 즉시 로딩의 경우, 연관된 테이블을 한 번에 조회하는 것을 말한다.
  * 즉, 조회 대상을 로딩하는 시점에 연관된 모든 테이블을 함께 조회한다.

### 엔티티 설계시 주의사항
* 엔티티에는 가능한 한 세터 적용을 지양한다.
  * 과도한 세터는 변경 가능한 지점을 늘리므로, 유지보수를 어렵게한다.
* **모든 연관관계는 지연 로딩으로 설정**한다.
  * `EAGER`와 같은 즉시 로딩은 예측이 어렵고, 어떠한 SQL이 실행될지 추적하기도 쉽지 않다.
    * 특히, JPQL을 실행하는 과정에서 `N+1` 문제가 자주 발생한다.
  * **실무에서는 모든 연관 관계를 `LAZY`와 같은 지연 로딩으로 설정**해야 한다.
  * **`XToOne`관계는 기본적으로 즉시 로딩이 적용되므로, 직접 지연 로딩을 설정**해야 한다.
  * 연관된 엔티티를 함께 DB에서 조회해야하는 경우, fetch join 또는 엔티티 그래프 기능을 사용한다.
* **컬렉션은 필드에서 초기화하는 것이 null 문제로부터 안전**하다.
  * 예를 들어, **`private List<OrderItem> orderItems = new ArrayList<>();`와 같은 초기화 방식은 best practice**이다.
  * **하이버네이트는 엔티티를 영속화하기 위해 컬렉션을 감싸는 내장 컬렉션을 사용**한다.
  * 이 때, getOrders 등 임의의 메소드로부터 컬렉션을 잘못 생성하는 경우 하이버네이트의 내부 메커니즘 상 문제가 발생할 수 있다.
    * 즉, **하이버네이트가 필요에 의해 내장 컬렉션으로 바꾸어둔 참조를 다른 컬렉션으로 바꾸어버리는 경우에 문제가 발생**한다.
  * 상술한 이유에서 필드 단위로 컬렉션을 생성하는 것이 가장 안전하며, 코드도 간결하다.
  * **가장 안전한 것은 한 번 생성해둔 컬렉션을 바꾸는 등의 수정작업을 수행하지 않는 것**이다.
* 하이버네이트의 경우, 기본 매핑 전략에 의해 테이블과 컬럼 명을 모두 `snake_case`로 변경한다.

### cascade 설정
* 예를 들어 `private List<OrderItem> orderItems = new ArrayList<>();`의 경우, 기본적으로 OrderItem 엔티티 마다 별도로 영속화해야 한다.
  * 예를 들어, Order 엔티티에 포함된 orderItems는 컬렉션에 포함된 각각의 OrderItem 엔티티를 우선 persist 메소드를 통해 영속화해야 한다.
  * 그 후, order를 영속화해야 JPA는 의도한대로 동작한다.
* **이러한 경우에 `@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)`와 같이 명시하는 것으로 영속화를 1회만 수행**할 수 있다.
  * 이 경우, Order 엔티티만을 persist 메소드를 통해 영속화하더라도 연관된 OrderItem 엔티티도 함께 영속화된다.

### 연관관계 편의 메소드
* **양방향 연관관계를 설정하는 경우, 연관관계 편의 메소드를 정의하는 것이 사용성을 더 높일 수 있다**.
  * 예를 들어, 회원과 주문 사이의 연관관계에서 회원이 주문하였을 때 회원과 주문 양 측에 필요한 값을 설정해주는 것이 편의성을 높여준다.
    * **이를 통해 member.getOrders() 또는 order.getMember()를 통해 필요한 객체를 오가며 개발을 진행할 수 있게 된다**.
  * 연관관계 편의 메소드는 이러한 경우에서 세터 형식의 원자적인 메소드를 통해 두 엔티티를 설정한다.
```
// Order.java
// 연관관계 편의 메소드
public void setMember(Member member) {
    this.member = member;
    member.getOrders().add(this);
}
// 이러한 연관관계 편의 메소드의 경우, 핵심적으로 제어권을 갖는 쪽에 정의하는 것이 바람직하다.
public void addOrderItem(OrderItem orderItem) {
    orderItems.add(orderItem);
    orderItem.setOrder(this);
}
// delivery도 마찬가지로 order와 양방향 연관관계이다.
// 양방향 연관관계에서는 연관관계 편의 메소드를 정의하는 것이 바람직하다.
public void setDelivery(Delivery delivery) {
    this.delivery = delivery;
    delivery.setOrder(this);
}
```