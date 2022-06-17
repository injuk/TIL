# Basics
## 2022-06-17 Fri

### JPA란?
* **JDBC와 비교하여 JDBC 템플릿은 반복적인 코드가 크게 줄어드는 장점이 있지만, SQL을 반드시 사용해야한다는 공통점이 존재**한다. 
* 반면, **JPA를 사용할 경우 SQL 쿼리까지도 JPA가 자동으로 처리**해주므로 개발 생산성을 크게 높일 수 있다.
  * 이는 마치 쿼리를 신경쓰지 않고 사용하던 인메모리 리포지토리와 유사하다.
  * JPA는 데이터를 삽입하는 등의 동작에 대해 알아서 데이터베이스에 쿼리를 대리한다.
* **JPA를 도입할 경우, 단순히 SQL을 작성하는 반복 작업에서 벗어나는 것 뿐만 아니라 보다 더 객체 지향적인 설계에 집중**할 수 있다.
  * 이는 SQL과 데이터 중심적인 설계에서 벗어나 객체 중심의 설계로 패러다임을 전환하는 것과 같다.
* **JPA는 기본적으로 인터페이스만을 제공하며, 이를 구현하는 실제 구현체 중 하나가 Hibernate**이다.
  * 즉, **JPA는 Java 진영에서 영속성을 위해 사용하는 표준**이며 실제 구현체는 여러 업체들이 정의한다.
* **JPA는 ORM이라고도 부르며, 객체와 관계형 데이터베이스를 간단한 어노테이션을 통해 매핑해주는 역할을 수행**한다.

### JPA 라이브러리 추가하기
* JPA는 build.gradle에 다음과 같은 라이브러리를 추가해야 한다.
  * 이 때, JPA는 기본적으로 JPA 뿐만 아니라 JDBC도 포함하므로 기존의 JDBC 관련 라이브러리 설정은 제거해도 무방하다.
```
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```
* 또한, h2를 사용하기 위해 application.properties에 다음과 같은 내용을 추가한다.
  * `show-sql`의 경우, JPA가 요청하는 쿼리를 보여주는 옵션이다.
  * `ddl-auto`의 경우, 활성화되어 있는 경우에 한해 도메인 객체의 형태를 보고 자동으로 테이블을 생성한다.
    * 이러한 기능을 활성화하고 싶다면 `ddl-auto=create`로 설정한다.
```
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
```

### 객체 매핑하기
* **도메인 객체를 JPA가 관리하는 엔티티로 정의하기 위해 @Entity 어노테이션을 명시**한다.
* 또한 객체의 기본 키로 사용될 필드에 @Id 어노테이션을 반드시 명시해야한다.
  * 또한, 예시와 같이 데이터베이스에 의해 자동 생성되는 값을 기본 키로 사용하는 경우 @GeneratedValue 어노테이션 역시 함께 매핑한다.
  * 이렇듯 **데이터베이스에 값을 삽입하면 데이터베이스가 Id를 자동 생성해주는 전략을 Identity 전략**이라고 한다.
```
@Entity
@Getter @Setter @ToString
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}
```
* 만일 도메인 객체의 멤버 변수 이름과 데이터베이스의 컬럼 명을 다르게 적용하고 싶은 경우, 다음과 같이 @Column 어노테이션을 활용할 수 있다.
  * 이렇듯 **JPA를 활용하면 단지 어노테이션을 명시하는 것만으로 객체와 관계형 데이터베이스를 손쉽게 매핑**할 수 있다.
```
@Column(name = "username")
private String name;
```

### JPA와 EntityManager 객체
* **JPA는 기본적으로 `EntityManager` 객체를 통해 모든 것이 동작하므로, 해당 리포지토리는 항상 `EntityManager`에 의존**하게 된다.
  * **`build.gradle`에 `implementation 'spring-boot-starter-data-jpa'`를 명시하였으므로, 해당 객체는 스프링 부트가 자동으로 생성**한다.
  * **스프링 부트는 설정에 명시된 데이터베이스와의 연결을 포함하여 `EntityManager`를 생성하므로, 리포지토리에서는 이를 주입 받아 사용**한다.
* **`EntityManager`는 내부적으로 `DataSource` 등을 포함하므로, 데이터베이스와의 연결 및 통신 등을 모두 내부에서 처리**한다.
  * 이로 인해 JPA의 사용에는 `EntityManager`의 주입이 필수적이다.
* 이러한 `EntityManager`를 설정 클래스에서 주입받는 경우, 스펙에서는 @PersistenceContext 어노테이션의 명시를 권장한다.
  * 그러나 **실제로는 아래와 같이 아무 것도 명시하지 않더라도 스프링 부트에 의해 자동으로 의존성이 주입**된다.
```
@Configuration
public class SpringConfig {

    private EntityManager em;

    public SpringConfig(EntityManager em) {
        this.em = em;
    }
    
    @Bean
    public MemberRepository memberRepository() {

        return new JpaMemberRepository(em);
    }
}
```

### JPA를 활용하는 리포지토리 추가하기
* 예를 들어 JPA 리포지토리 클래스의 목록 조회 API의 경우 다음과 같이 정의할 수 있다.
```
@Override
public List<Member> findAll() {
    return em.createQuery("select m from Member m", Member.class)
            .getResultList();
}
```
* `em.createQuery` 메소드에 첫 번째 인자로 전달된 문자열은 JPQL이라고 하는 쿼리 언어의 형식을 따른다.
  * **일반적으로 테이블을 대상으로 쿼리를 요청하는 SQL과 달리, JPQL은 엔티티를 대상으로 쿼리를 요청**한다.
    * 이렇게 요청된 쿼리는 내부적으로 SQL로 번역된다.
  * `Member m`은 사실상 `Member as m`과 동일한 의미를 갖는다.
  * **`select m`은 SQL에서의 `select *` 또는 모든 컬럼을 명시하는 select 쿼리와 같은 기능을 수행**한다.
    * 정확히는 Member 객체 자체인 m을 select한다는 의미를 갖는다.
* 이렇듯 **JPA를 활용하는 경우, 쿼리로 요청받아온 데이터를 객체로 매핑하는 과정까지도 JPA가 대리**해주게 된다.

### JPA를 활용하는 리포지토리의 예시
* 다음은 JPA를 사용하도록 구현된 리포지토리 클래스의 예시이다.
  * **클래스 자체의 길이 또는 가독성이 이전의 리포지토리에 비해 크게 늘게되며, SQL 설계 역시 JPA에 의해 자동으로 수행**된다.
  * 그러나 **기본 키 기반 검색이 아닌 조회의 경우, JPQL을 작성**해주어야 한다.
```
public class JpaMemberRepository implements MemberRepository {

    private final EntityManager em;

    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }

    // 코드가 매우 간단해졌다!
    @Override
    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        return Optional.ofNullable(
                em.find(Member.class, id)
        );
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m  where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
        
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
}
```
* JPA를 사용하는 경우, 반드시 트랜잭션을 활용할 수 있도록 서비스 계층에 @Transactional 어노테이션을 명시해야 한다.
  * 언제나 데이터를 저장하거나 변경하는 경우 트랜잭션을 사용해야 한다.
  * **해당 어노테이션은 클래스 상단에 명시하여 클래스 전역에 적용할 수도 있지만, 트랜잭션이 반드시 필요한 생성 메소드에만 명시해주어도 무방**하다.
  * **JPA의 경우, 생성 동작에서 모든 데이터의 변경이 반드시 트랜잭션 안에서 실행되어야만 한다**.

### 통합 테스트 진행하기
* 통합 테스트를 진행하는 경우, 아래와 같은 로그를 통해 다음의 사실을 확인할 수 있다.
  1. 스프링 데이터 JPA는 기본적으로 하이버네이트 구현체를 적용한다.
  2. **데이터베이스와의 상호작용은 결국 SQL이 필수적이며, 이는 JPA에 의해 자동으로 생성되어 요청**된다.
```
Hibernate: select member0_.id as id1_0_, member0_.name as name2_0_ from member member0_ where member0_.name=?
Hibernate: insert into member (id, name) values (default, ?)
```

### 스프링 데이터 JPA
* 스프링 부트와 JPA만 조합하더라도 개발 생산성은 엄청나게 상승한다.
  * 여기에 더해 데이터 JPA를 활용하면 리포지토리에 구현 클래스 없이 인터페이스만으로도 개발을 완료할 수 있게 된다.
  * 또한, 반복적으로 작업해온 CRUD 기능 역시 스프링 데이터 JPA로부터 제공받을 수 있다.
* 이렇듯 **개발 생산성이 오르게 되면 단순하고 반복적인 코딩 작업은 줄어들고, 개발자는 핵심 비즈니스 로직을 개발하는 데에만 온전히 집중**할 수 있게 된다.
  * 이제는 실무에서도 관계형 데이터베이스를 사용하고 있다면 스프링 데이터 JPA를 사실상 필수적으로 적용해야 한다.
* 그러나 스프링 데이터 JPA는 JPA의 사용성을 향상시키는 기술이므로, JPA를 우선 학습한 후에 익히도록 해야 한다.
  * 즉, **스프링 데이터 JPA는 JPA를 편리하게 사용할 수 있도록하는 라이브러리**에 지나지 않는다.

### 스프링 데이터 JPA를 활용하는 리포지토리 구현하기
* 스프링 데이터 JPA를 활용하는 리포지토리는 `JpaRepository<>` 를 확장해야한다.
  * **스프링 데이터 JPA는 `JpaRepository<>`는 확장하는 인터페이스를 자동으로 구현하며, 스프링 빈에 자동으로 등록**한다!
  * 개발자는 이렇게 자동 등록된 스프링 빈을 그저 가져다 사용하기만 하면 된다.
* 기존 리포지토리를 스프링 데이터 JPA를 활용하도록 SpringConfig를 다음과 같이 수정한다.
  * 이 때, **생성자에 의해 주입되는 MemberRepository는 스프링 데이터 JPA가 자동으로 생성한 리포지토리의 구현체**가 된다.
  * **스프링 데이터 JPA는 해당 의존성을 자동으로 생성하여 스프링 빈에 등록**한다.
```
@Configuration
public class SpringConfig {
    private final MemberRepository memberRepository;

    public SpringConfig(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository);
    }
}
```
* 반면, 스프링 데이터 JPA를 활용하는 리포지토리는 다음과 같이 구현이 가능하다.
  * 스프링 데이터 JPA는 그저 JPA를 가져다 사용하는 것에 지나지 않으므로, 기존 방식과 동일하게 동작하는 것을 로그를 통해 확인할 수 있다.
  * **아래와 같이 인터페이스를 정의하는게 전부이며, 이를 실제로 구현할 필요조차 없다**!
  * 예를 들어, findByName의 구현부는 Java의 프록시와 리플렉션 기능을 활용하여 스프링 데이터 JPA에 의해 자동으로 정의된다.
```
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {

    @Override
    Optional<Member> findByName(String name);
}
```
* **스프링 데이터 JPA는 리포지토리 클래스에 명시된 클래스의 정보를 토대로 스프링 빈을 자동으로 생성**한다.
* 스프링 데이터 JPA에 의해 구현체가 자동으로 생성되는 것처럼 보이지만, 실제로는 `JpaRepository`에 이미 공통화된 메소드들이 여럿 오버로딩되어 있다.
  * 즉, **기본적으로 자주 사용되는 CRUD 동작에 대한 메소드가 `JpaRepository` 인터페이스에 이미 정의**되어 있다!
  * 때문에 **스프링 데이터 JPA를 효율적으로 사용하려면 리포지토리 클래스의 메소드 시그니쳐를 정확히 맞추어 정의**해두어야 한다.
    * 그래야만 이미 공통화된 메소드를 그대로 활용할 수 있기 때문이다.

### JpaRepository 인터페이스란?
* **데이터베이스에 대해 수행할 법한 동작을 최대한 추상화한 인터페이스이며, 개발자가 생각할 수 있는 대부분의 기능을 모두 공통화하여 제공**한다.
  * 즉, 개발자는 대부분의 경우에 이미 공통화된 메소드를 가져다 사용하기만 하면 된다!
* 거꾸로 말해, **공통화되지 않는 구체적인 메소드는 별도의 인터페이스를 정의한 후 `JpaRepository` 인터페이스와 함께 다중 상속**해야 한다.
  * 예를 들어, name이나 email 등 자신의 비즈니스 로직에 특화된 컬럼으로 조회하는 경우를 말한다.
  * 중요한 것은 자신의 구체적인 메소드를 정의할 때 아무런 이름으로 명명하는 것이 아닌, 최대한 공통화된 명명 규칙을 적용하는 것이다.
    * 이 경우에만 스프링 데이터 JPA의 이점을 온전히 누릴 수 있으며, 메소드를 구현할 필요가 없어진다.
* **개발 과정에서 사용되는 대부분의 쿼리 중 약 8할은 매우 단순한 쿼리이며, 나머지 2할이 아주 복잡한 쿼리가 필요한 경우에 해당**한다.
* 이 때, 스프링 데이터 JPA는 8할의 쿼리를 자동 구현해주어 개발자가 2할의 복잡한 쿼리에 집중할 수 있도록 지원한다.

### 스프링 데이터 JPA가 제공하는 것들과 한계점
* **스프링 데이터 JPA는 인터페이스를 활용한 기본적인 CRUD 메소드와, 메소드 이름만으로도 상세 조회를 가능하도록 지원**한다.
  * 나아가 페이징 역시 함께 지원받을 수 있다.
* 실무의 경우, JPA와 스프링 데이터 JPA를 기본적으로 사용하되 복잡한 동적 쿼리는 `Querydsl` 라이브러리를 활용한다.
  * `Querydsl`을 통해 복잡한 동적 쿼리를 Java 코드로 안전하게 작성할 수 있게 된다.
  * 이 세 가지 기술의 조합으로도 해결할 수 없는 쿼리는 JPA의 네이티브 쿼리를 활용하거나, `JdbcTemplate` 을 활용한다.

### 데이터베이스 접근 기술
* 스프링과 함께 사용할 수 있는 데이터베이스 접근 기술은 크게 다음과 같다.
  1. 순수 JDBC: 쿼리 하나를 요청하기 위해서 수많은 코드를 작성해야 한다. 
  2. JdbcTemplate: 순수 JDBC에서 필수적이었던 많은 반복적인 코드를 제거할 수 있으나, 여전히 쿼리를 직접 작성해야 한다. 
  3. JPA: CRUD에서 사용되는 대부분의 간단한 쿼리까지도 제거할 수 있으나, 조회 기능을 구현할 때 JPQL을 사용해야만 한다.  
  4. 스프링 데이터 JPA: 구현 클래스를 작성할 필요 조차 없으며, 인터페이스 정의만으로도 데이터베이스를 활용할 수 있게 된다.
     * JPA와 달리, 스프링 데이터 JPA에서는 조회 쿼리를 활용하는 메소드까지도 대부분의 경우에 구현부를 작성할 필요가 없어진다.