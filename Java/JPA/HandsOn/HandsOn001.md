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