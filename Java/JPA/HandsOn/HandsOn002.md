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