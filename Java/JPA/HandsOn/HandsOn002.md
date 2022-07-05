# HandsOn
## 2022-07-05 Tue

### 애플리케이션 아키텍쳐
* 고전적인 계층형 아키텍쳐의 경우, 다음과 같은 구조를 갖는다.
  1. 컨트롤러: 웹 계층을 담당한다.
  2. 서비스: 비즈니스 로직과 트랜잭션 처리를 담당한다.
  3. 리포지토리: JPA를 직접 사용하는 계층으로, 엔티티 매니저를 통해 데이터를 영속화하거나 조회한다.
  4. 도메인: 엔티티가 모여있는 계층이며, 모든 계층에서 사용된다.
* 예를 들어, 다음의 패키지 구조는 상술한 계층형 아키텍쳐를 따른다.
```
jpashop
├── domain
├── exception
├── service
├── repository
└── web
```
* 이 때, **컨트롤러에 요청된 기능이 매우 간단한 경우 컨트롤러에서 서비스를 호출하지 않고 직접 리포지토리를 호출해도 무방**하다.
* 오히려 항상 컨트롤러 > 서비스 > 리포지토리의 흐름을 유지하는 것은 때때로 서비스가 단순한 포워딩 역할만 수행하게 되므로, 유연성 측면에서 좋지 못할 수 있다.

### 회원 리포지토리 개발하기
* 회원에 대한 생성, 조회, 조건부 조회 등 간단한 동작은 다음과 같이 구현할 수 있다.
  * 아래의 예시에서 사용된 **JPQL의 문법은 SQL과 거의 유사하지만, 조회 대상인 from 절이 테이블이 아닌 엔티티라는 점에서 차이**가 있다.
```
@Repository
public class MemberRepository {

    @PersistenceContext
    private EntityManager em;

    public void save(Member member) {
        em.persist(member);
    }

    public Member findOne(Long id) {
        // 단건 조회이며, 각 매개변수로 엔티티 타입과 PK를 전달한다. 
        return em.find(Member.class, id);
    }

    public List<Member> findAll() {
        // 매개변수에 JPQL, 반환 타입 명시한다.
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findByName(String name) {
        // :컬렴명을 표기하여 where 절에 파라미터를 바인딩할 수 있다.
        return em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
    }
}
```
* 상술한 리포지토리 클래스는 컴포넌트 스캔의 대상이므로, 스프링 부트의 기본 동작에 의해 빈으로 등록된다.
  * **스프링 부트의 기본 동작은 @SpringBootApplication 어노테이션이 명시된 클래스의 패키지를 기준으로 해당 패키지와 하위 패키지는 모두 스캔**한다.
  * 이렇게 스캔된 컴포넌트는 모두 스프링 빈으로 자동 등록된다.
* **@PersistenceContext 어노테이션을 명시함으로써 스프링이 생성한 JPA 엔티티 매니저를 해당 리포지토리 클래스에 자동으로 주입**한다.
  * 해당 어노테이션을 활용하지 않고 순수 JPA로 개발하는 경우, 엔티티 매니저 팩토리로부터 직접 엔티티 매니저를 가져오는 과정이 필수적이다.
  * 그러나 @PersistenceContext 어노테이션을 명시하는 것만으로 이러한 번거로운 작업 없이 스프링이 필요한 의존성을 모두 자동으로 주입하게 된다.

### 회원 서비스 개발하기
* JPA와 관련된 모든 데이터 변경, 또는 로직들은 최대한 트랜잭션 안에서 실행되도록 적용하는 것이 바람직하다.
  * 이는 서비스 클래스에 @Transactional 어노테이션을 명시하는 것으로 간단하게 적용할 수 있다.
  * 이 경우, **서비스 클래스가 제공하는 모든 public 메소드의 동작에 기본적으로 트랜잭션이 적용**된다.
* **해당 어노테이션의 경우 javax와 springframework가 제공하는 두 종류가 존재하며, 이 중 스프링 프레임워크가 제공하는 어노테이션을 사용**한다.
  * 스프링 애플리케이션의 경우 이미 스프링에 종속되므로 상관이 없고, 무엇보다도 훨씬 많은 옵션을 제공하기 때문이다.
* 또한, **해당 어노테이션의 경우 `readonly = true` 옵션을 명시하면 JPA가 조회하는 부분에서의 성능을 최적화**한다.
  * 때문에 **읽기 동작의 경우 되도록이면 `readonly = true`를 명시하되, 쓰기 동작에는 절대로 작성하지 않아야** 한다.
* 만약 서비스가 전체적으로 읽기 동작에 집중하는 경우, 다음과 같이 작성해볼 수 있다.
  * **클래스 전체적으로 `readonly = true`옵션을 적용하되, 쓰기 동작에만 별도로 어노테이션을 명시**한다.
  * 반면, 클래스 전체적으로 쓰기 동작이 많은 경우에는 읽기 동작에만 `readonly = true`옵션을 명시할 수도 있다. 
```
@Service
@Transactional(readOnly = true)
public class MemberService {

    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Transactional
    public Long join(Member member) {
        validateDuplicateMember(member);
        // em.persist 시점에 영속성 컨텍스트에 의해 PK인 Id 값이 적용되는 것이 보장된다.
        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        List<Member> members = memberRepository.findByName(member.getName());
        if(!members.isEmpty()) {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        }
    }

    public List<Member> findMembers() {
        return memberRepository.findAll();
    }

    public Member findOne(Long id) {
        return memberRepository.findOne(id);
    }
}
```
* 상술한 코드의 join 메소드에는 중복된 사용자를 방어하는 로직이 포함되어 있으나, 이는 동시에 동일한 사용자가 가입하는 경우를 방어할 수 없다.
  * 때문에 **실무에서는 데이터베이스 자체에도 UQ 제약조건을 추가하여 동일한 사용자가 가입될 수 없도록 방어하는 것이 바람직**하다.

### 생성자를 활용한 EntityManager 주입
* 기준의 방식은 다음과 같이 EntityManager 의존성을 주입받아야 했다.
```
@Repository
public class MemberRepository {

    @PersistenceContext
    private EntityManager em;
    // ...생략
}
```
* 그러나 **롬복과 스프링 부트, 스프링 데이터 JPA를 활용하는 경우 EntityManager 역시 생성자를 통해 쉽게 주입**받을 수 있다.
```
@Repository
@RequiredArgsConstructor
public class MemberRepository {

    private final EntityManager em;
    // ...생략
}
```
* 기본적으로, EntityManager는 @PersistenceContext 어노테이션을 통해서만 주입받을 수 있다.
  * 그러나 **스프링 부트와 스프링 데이터 JPA를 활용할 경우 @Autowired를 통한 주입 또한 가능해지므로, 상술한 생성자 방식의 의존 관계 주입이 가능**하다.

### 테스트 작성하기
* 회원 가입 기능을 테스트하는 경우, 다음과 같은 테스트 코드를 작성해볼 수 있다.
```
@SpringBootTest
@Transactional
class MemberServiceTest {
  // ...생략
  @Test
  public void 회원가입() {
      Member member = new Member();
      member.setName("injuuk");
  
      Long id = memberService.join(member);
  
      assertThat(member).isEqualTo(memberRepository.findOne(id));
  }
}
```
* 이러한 **비교가 가능한 이유는, JPA는 같은 트랜잭션 내에서 PK 값이 같은 엔티티는 동일한 영속성 컨텍스트 내에서 하나로 관리되기 때문**이다.
  * 동일한 영속성 컨텍스트에서 같은 PK를 갖는 엔티티는 오로지 하나만 존재할 수 있다.
* 그러나 상술한 테스트 케이스를 실행하는 경우, 실제 insert 쿼리는 요청되지 않는 것을 확인할 수 있다.
  * 이는 JPA의 동작원리 때문으로, em.persist를 호출한다고 해서 insert 쿼리는 '기본적으로' 호출되지 않는다.
* **JPA는 데이터베이스 트랜잭션이 커밋되는 시점에 flush 되면서 insert 쿼리를 요청하는 방식으로 동작**하므로, 트랜잭션 커밋이 매우 중요하다. 
  * **데이터베이스 트랜잭션이 커밋되는 순간 flush를 통해 JPA 영속성 컨텍스트에 있는 엔티티 객체를 영속화하기 위한 insert 쿼리가 생성되어 요청**된다.
* 그러나 테스트 케이스에 명시된 @Transactional 어노테이션의 경우, 기본적으로 트랜잭션을 커밋하지 않고 롤백한다.
  * 이 경우, 트랜잭션 롤백 과정에서 데이터베이스에 변경사항을 저장하지 않고 모두 버리게 되므로 JPA 입장에서는 insert 쿼리를 요청할 필요 자체가 없다.
  * 정확히는 영속성 컨텍스트를 flush하지 않는다.
* 때문에 요청된 insert 쿼리를 눈으로 확인하고자 하는 경우, `회원가입()` 테스트케이스에 @Rollback(false) 어노테이션을 추가로 명시해야 한다.
  * **또는 EntityManager 의존성을 주입 받아 em.persist가 호출된 직후에 em.flush를 호출**한다.
  * **em.flush는 영속성 컨텍스트에 있는 신규 등록 또는 변경 사항을 즉시 데이터베이스에 반영**한다.
  * em.flush를 통해 insert 쿼리가 요청되더라도 @Transactional에 의해 트랜잭션이 롤백되므로 최종적으로는 데이터베이스에 변경이 반영되지 않는다.
* 이렇듯 기본적으로 트랜잭션을 롤백하는 동작은 테스트 케이스를 몇 번이든지 제한 없이 반복 실행할 수 있기 위한 장치이다.
  * 데이터베이스 관련 테스트가 지속적이고 반복 가능하기 위해서는 테스트 과정에서 데이터베이스에 아무런 변경 사항을 남기지 않아야 한다.

### 인메모리 H2 데이터베이스 활용하기
* 원칙적으로 운영 데이터베이스는 테스트에 사용하지 말아야 하며, 그렇다고 로컬에 데이터베이스를 설정하는 것도 번거로울 수 있다.
* 이 경우, H2가 자체적으로 제공하는 인메모리 데이터베이스 모드를 사용하면 간단하게 테스트를 진행할 수 있다.
  * 이는 H2는 Java로 작성되어 있어 JVM 상에서 동작할 수 있기 때문이다.
* 운영 환경에서 동작하는 애플리케이션은 `main/resources/application.yml`과 같은 구성 정보를 따른다.
  * 반면, 테스트 환경에서 동작하는 애플리케이션은 `test/resources/application.yml`과 같은 구성 정보를 우선적으로 따르므로 이를 활용한다.
```
# test/resources/application.yml
# ...생략

spring:
  datasource:
    url: jdbc:h2:mem:test
    
# ...후략
```
* **스프링 부트를 사용하는 경우, 별도의 설정이 없으면 스프링 부트는 기본적으로 H2 데이터베이스를 인메모리 모드로 동작**시킨다.
  * 때문에 스프링 부트를 사용한다면 상술한 설정이 모두 필요가 없어진다.