# Basics
## 2022-07-10 Sun

### JPA 도입의 이점
* JPA 이전에는 복잡한 JDBC 연결 설정과 쿼리를 모두 개발자가 하나하나 작성해야 했다.
* 이후에 JDBC template 또는 MyBatis 등의 SQL mapper가 등장하여 JDBC 관련 코드를 많이 줄여주었으나, 여전히 쿼리 작성은 개발자의 책임으로 남는다.
* **JPA는 이러한 배경에서 탄생하였으며, 이를 적절히 활용할 경우에는 JDBC 관련 코드는 물론 복잡한 쿼리를 작성할 필요도 없어지게 된다**.
  * JPA를 활용하면 마치 컬렉션 API를 사용하듯이 데이터베이스와 상호작용할 수 있게 된다.
  * 이를 통해 **JPA는 개발자 대신 적절한 쿼리를 생성하여 데이터를 조회하거나 영속화하는 작업을 수행**한다.
* 모든 데이터베이스 쿼리를 하나하나 손수 작성하는 것은 현재의 Java개발 환경 상에서는 생산성이 상대적으로 뒤떨어질 수 밖에 없다.
  * 반면, JPA를 사용할 경우 생산성은 물론 유지보수 측면에서도 큰 이점을 누릴 수 있다.

### JPA 도입의 어려움
* JPA를 도입하지 않아 데이터베이스의 연결 또는 SQL 쿼리들이 명시적으로 작성된 코드는 장황하지만, 반대로 말하면 그만큼 명시적이다.
* 반면, **JPA를 통해 많은 부분을 자동화한 경우에는 코드가 간결해지지만, 그만큼 많은 동작이 암시적으로 가려지게 되어 이해하기 어려울 수 있다**.
  * 심지어, 실무에서는 데이터베이스 테이블이 매우 많고 복잡하게 얽혀있어 JPA의 도입을 더 어렵게 한다.
* 또한 **JPA를 잘 활용하기 위해서는 객체와 테이블을 적절히 매핑하는 것이 무엇보다도 중요하지만, 이는 놓치기도 쉬울 뿐더러 어렵기도 한 작업**이다.

### JPA는 왜 사용해야하는가?
* 일반적으로 현대적인 애플리케이션을 작성하는 경우, Java와 같은 객체지향 언어와 관계형 데이터베이스를 사용하게 된다.
  * NoSQL이 부상하고 있으나, 여전히 영속성 계층을 주도하는 것은 Oracle 또는 MySQL과 같은 관계형 데이터베이스이다.
* 이는 다시 말해 **현대적인 애플리케이션은 객체를 관계형 데이터베이스에 저장하여 관리하는 식으로 동작한다는 결론**에 이르게 된다.
* 그러나 객체지향적인 프로그래밍을 진행해나간다고 한들, 관계형 데이터베이스가 이해할 수 있는 것은 SQL 뿐이므로 결국에는 SQL을 작성해야만하는 순간이 온다.
  * 즉, 싫어도 SQL 중심적인 개발을 해야만 한다.

### SQL 중심적인 개발의 한계
* 새로운 요구사항에 의해 하나의 테이블을 관계형 데이터베이스에 추가한 경우, 일반적으로 이와 관련된 CRUD 로직 역시 추가해야 한다.
  * 그러나 **SQL 중심적인 개발이라면 이에 맞는 쿼리를 반복적으로 작성해야하는 문제점이 존재**한다.
  * 시간이 지나 프로젝트가 성숙해 감에 따라 테이블이 수십개로 늘어나게 되면, 개발자는 그만큼 반복적인 작업을 하고 있다는 생각을 뿌리치기 힘들게 된다.
* 관계형 데이터베이스를 사용하는 이상, SQL에 의존적인 개발을 피하기는 어려울 수 밖에 없다.

### 관계형 데이터베이스와 객체지향 프로그래밍의 불일치
* 무엇보다도, **관계형 데이터베이스와 객체지향 프로그래밍이 각각 지향하는 바와 패러다임 자체가 다르다**.
  * 관계형 데이터베이스의 본질은 데이터를 잘 정규화하고, 영속화하는 데에 있다.
  * 반면 객체지향 프로그래밍의 본질은 데이터와 기능을 캡슐화하여 여러 이점을 누리는 데에 있다.
* 이렇듯 **패러다임이 다른 두 개념을 하나로 묶어 사용하게 될 경우, 그 불일치성으로 인해 여러가지 어려움을 마주칠 수 밖에 없다**.
  * 예를 들어 엔티티 간에 정의된 상속 관계를 데이터베이스에 1:1로 완벽히 매칭하여 밀어넣기 어렵다.
* 또한, **객체를 관계형 데이터베이스가 이해할 수 있는 정보로 변환하기 위한 SQL은 개발자가 직접 작성해야 한다.**
  * 결국 업무의 대부분의 시간을 소요하는 소모적이고 반복적인 작업을 위해 개발자가 사실상 SQL 매퍼같은 역할을 수행하게 된다.
* **패러다임의 불일치는 개발자가 객체지향적으로 사고하고, 객체지향적으로 개발할수록 SQL에 의존하기 위해 더 많은 매핑 작업을 필요로 하는 원인**이 된다.
  * 때문에 이러한 추가 작업을 피하기 위해 더더욱 SQL에 의존하고, raw query를 작성하게 되는 악순환이 발생한다.
  * **많은 개발자들은 마치 객체지향의 컬렉션을 사용하듯이 데이터베이스와 상호작용할 수 있는 방법을 고민하였고, JPA는 Java 진영이 내놓은 해답**이다.

## 2022-07-11 Mon
### Java Persistence API
```
> JPA는 Java 진영의 ORM 표준이다.
```
* ORM이란, 객체와 관계형 데이터베이스를 매핑하는 기술이다.
  * ORM을 적절히 활용할 경우 객체는 객체대로 설계하며, 데이터베이스는 데이터베이스대로 관리하지만 둘을 매핑할 수 있게 된다.
  * 이 때, **ORM 프레임워크는 중간에서 두 간극을 매핑하는 역할을 수행**한다.
  * 나아가 현존하는 대중적인 프로그래밍 언어는 대부분 ORM을 지원한다.
* Java 진영에서는 데이터베이스에 SQL을 요청하고, 결과를 반환받는 등 상호작용을 위해 JDBC API를 사용한다.
* 이 때, **JPA는 Java로 구현된 애플리케이션과 JDBC API 사이에 위치하여 동작**한다.
  * 때문에 **개발자는 JDBC API를 사용하기 위한 장황한 코드를 사용할 필요가 없으며, JPA에게 명령하여 JDBC API의 사용을 위임**한다.
* 예를 들어 JPA에게 엔티티의 저장을 명령할 경우, JPA는 전달받은 엔티티를 분석하여 적절한 INSERT 쿼리를 생성하고 JDBC API를 통해 이를 영속화한다.
  * **중요한 것은 이 과정에서 JPA에 의해 OOP와 RDB 간의 불일치성으로 인한 간극이 해소된다는 점**이다.
  * 조회 역시 마찬가지이며, JPA에게 적절한 식별자를 넘긴 경우 JPA는 자동 생성한 SELECT 요청의 결과인 ResultSet을 객체로 매핑하여 반환한다.

### 표준으로서의 JPA
* **JPA는 인터페이스의 모음으로 구성된 표준 스펙**이다.
  * 이 때, Hibernate, EclipseLink, DataNucleus 등은 JPA 표준을 구현한 실제 구현체이다.
  * 반면 실무에서는 9할 이상이 하이버네이트를 사용하여 개발한다고 이해해도 무방하다.

### JPA를 활용한 OOP와 RDB 사이의 불일치성 해결
* JPA를 도입하는 것으로 객체지향과 관계형 데이터베이스 사이의 다음과 같은 불일치성을 해결할 수 있다.
  1. JPA를 통해 상속 관계를 자연스럽게 표현할 수 있다.
  2. JPA를 통해 테이블 간의 연관관계를 자연스럽게 표현할 수 있다.
  3. JPA를 통해 객체 그래프 탐색을 수행할 수 있다.
  4. JPA를 통해 엔티티 간의 비교 기능을 자연스럽게 표현할 수 있다.
* 예를 들어, JPA는 객체가 포함하는 참조를 통한 객체 그래프 탐색을 시도할 경우 지연 로딩 기능을 통해 적절한 시점에 쿼리를 요청하는 식으로 동작한다.
* 또한, **JPA는 동일한 트랜잭션 내에서 조회한 엔티티에 대해 동일성을 보장해줄 수 있으므로 `==` 연산자를 활용한 비교를 신뢰**할 수 있다.

### JPA와 성능 최적화 기법
* JPA와 같이 출발지와 목적지 사이에 위치한 버퍼를 사용하는 경우, 다음과 같은 두 기법을 사용할 수 있다.
  1. 버퍼링: 필요한 정보를 모아서 한 번에 처리한다.
  2. 캐싱: 읽어들인 정보를 저장해두어 재활용한다.
* **JPA 역시 이러한 중간 계층의 역할을 충실히 실행하므로, JPA 최적화를 잘 구현했을 경우에는 오히려 성능 상의 이점**을 누릴 수 있다.
  * 1차 캐시: 같은 트랜잭션 안에서는 동일한 엔티티를 반환하므로, 약간의 성능 향상을 누릴 수 있다.
    * 그러나 **엄밀히 말해, 이는 한 클라이언트의 요청을 처리하는 트랜잭션 안에서의 캐싱이므로 생각보다 큰 성능 향상 효과를 누릴 수는 없다**.
  * 트랜잭션을 지원하는 쓰기 지연: **트랜잭션을 커밋할 때까지는 INSERT 쿼리를 모아둔 후, JDBC Batch SQL을 통해 한 번에 요청**한다.

### JPA의 지연 로딩과 즉시 로딩
* JPA에서 지연 로딩과 즉시 로딩은 다음과 같이 구분된다.
  1. 지연 로딩: 코드 상에서 객체가 실제로 사용될 때 쿼리를 요청한다.
  2. 즉시 로딩: 연관된 객체는 최초 요청 시점에 JOIN 쿼리를 통해 모두 요청한다.
* **두 방식에 우열은 없으며, 애플리케이션의 비즈니스 로직에 따라 적절한 로딩 방식을 사용**해야 한다.
  * 예를 들어, 엔티티가 포함한 객체를 항상 사용하는 경우에는 네트워크 IO를 여러 번 사용하는 지연 로딩보다는 즉시 로딩을 활용한다.
  * 또한, 이러한 로딩 방식은 간단한 설정을 통해 정의할 수 있다.
* **실무에서는 우선 지연 로딩을 도입하여 개발을 진행한 후, 최적화가 필요한 경우에만 용도를 분석하여 즉시 로딩을 적용하는 방식을 선택**할 수 있다.

### JPA 사용시의 마음가짐
* JPA와 같은 ORM 기술은 객체지향과 관계형 데이터베이스라는 두 기둥을 근간으로 구성되는 기술이다.
* 때문에 **JPA를 효율적으로 사용하기 위해서는 OOP와 RDB 두 기술 모두에 대한 이해가 반드시 선행되어야 한다**.

### JPA 프로젝트 시작하기
* JPA는 가장 간단한 기본 동작을 위해서도 몇 가지의 설정 파일을 작성할 필요가 있다.
  * 예를 들어, `/META-INF/persistence.xml` 과 같은 설정 파일이 있다.
* JPA에 연결하기 위해 persistence.xml에는 다음과 같은 내용을 입력할 수 있다.
  * 이는 **JPA가 데이터베이스에 연결하기 위한 접근 정보와 관련된 설정**이다.
```
<property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
<property name="javax.persistence.jdbc.user" value="sa"/>
<property name="javax.persistence.jdbc.password" value=""/>
<property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/workspace/study/h2/test"/>
<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
```

### JPA 방언
* 상술한 코드 중, `hibernate.dialect`는 데이터베이스 방언이라고도 한다.
* JPA는 임의의 데이터베이스에 종속되지 않도록 설정되었으므로, 극단적으로는 MySQL을 사용하다가 OracleDB를 사용하더라도 아무런 문제가 없어야 한다.
  * **그러나 각 데이터베이스마다 제공하는 SQL 문법과 함수의 형태는 조금씩 다른 문제점**이 있다.
  * 이 때, **데이터베이스 방언이란 이렇듯 SQL 표준을 준수하지 않는 특정 데이터베이스만의 고유 기능을 의미**한다.
  * 구조적으로는 JPA에 Dialect 클래스가 포함되며, 데이터베이스 별로 Dialect 클래스를 상속받는 방언 클래스가 존재한다.
* 이러한 방언 설정은 `org.hibernate.dialect.${DB_VENDOR}` 형태로 persistence.xml에 작성할 수 있다.
* 또한 하이버네이트는 40가지 이상의 데이터베이스 방언을 제공하며, 이는 사실상 실무에서 사용 가능한 거의 모든 데이터베이스를 의미한다.

## 2022-07-12 Tue
### JPA 동작 방식
* JPA는 일반적으로 다음과 같은 흐름으로 동작한다.
  1. Persistence 클래스에서 시작한다.
  2. Persistence 클래스는 META-INF/persistence.xml 파일로부터 설정 정보를 조회하여 EntityManagerFactory를 생성한다.
  3. EntityManagerFactory는 EntityManager 객체들을 필요에 따라 생성한다.
* 이 때, EntityManager가 완성된 시점부터는 데이터베이스와 연결과 그 이후의 작업을 수행할 수 있는 것으로 이해할 수 있다.
* 상술한 동작 순서를 코드로 묘사하면 다음과 같다.
  * 아래의 코드에서, Persistence.createEntityManagerFactory에 전달하는 PersistenceUnitName은 persistence.xml의 내용이다.
```
public static void main(String[] args) {
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
    EntityManager em = emf.createEntityManager();

    // em을 가져온 상태에서 실제 데이터베이스와 상호작용하는 코드를 작성할 수 있게 된다.
    // 저장, 조회, 등...
    
    em.close();
    emf.close(); // 모든 동작이 완료되어 애플리케이션이 종료되면 emf를 닫아줘야 한다. 
}
```
* **EntityManagerFactory는 애플리케이션 로딩 시점에 단 하나만 만들어져 유지**되어야 한다.
* 반면, 데이터베이스와 상호작용하기 위한 트랜잭션 단위마다 EntityManger를 만들어주어야 한다.
  * 여기서 데이터베이스와의 상호작용은 **데이터베이스 커넥션을 얻어 쿼리를 요청한 후에 종료되는 단위 동작을 의미**한다.

### Entity 작성핟기
* 간단한 엔티티는 다음과 같이 추가할 수 있다.
  * 이 때, **@Entity 어노테이션은 반드시 명시홰야 해당 클래스를 JPA가 로딩되는 시점에 인식하고 관리**할 수 있다.
```
@Entity
@Getter @Setter
public class Member {
    
    @Id
    private Long id;
    private String name;
}
```
* 이를 활용하여 다음과 같이 영속화를 시도하는 경우, 에러가 발생하며 진행되지 않는다.
```
public static void main(String[] args) {
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
    EntityManager em = emf.createEntityManager();

    // em을 가져온 상태에서 실제 동작하는 코드를 작성할 수 있게 된다.
    // 저장, 조회, 등...
    Member member = new Member();
    member.setId(1L);
    member.setName("ingnoh");
    em.persist(member);

    em.close();
    emf.close(); // 모든 동작이 완료되어 애플리케이션이 종료되면 emf를 닫아줘야 한다.
}
```
* **JPA에서는 트랜잭션이 매우 중요하며, JPA를 통해 발생하는 모든 데이터의 변경 작업은 반드시 트랜잭션 안에서 처리되어야 한다**.
  * 상술한 코드 역시 이로 인해 실행되지 않는 것으로 이해할 수 있다.
* JPA에서는 엔티티매니저의 getTransaction 메소드를 호출하여 새로운 트랜잭션을 요청할 수 있으며, begin과 commit을 다음과 같이 수행할 수 있다.
```
public static void main(String[] args) {
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
    EntityManager em = emf.createEntityManager(); // 데이터베이스 커넥션을 하나 받아온 것으로 간단히 이해할 수도 있다.

    // 새로운 트랜잭션을 요청한 후 이를 시작한다.
    EntityTransaction tx = em.getTransaction();
    tx.begin();

    Member member = new Member();
    member.setId(1L);
    member.setName("ingnoh");
    em.persist(member);

    // 모든 요청이 완료된 후에 커밋한다.
    tx.commit();

    em.close();
    emf.close(); // 모든 동작이 완료되어 애플리케이션이 종료되면 emf를 닫아줘야 한다.
}
```
* 해당 코드는 정상 동작하며, 코드 상에서 어떠한 SQL 구문을 작성하지 않았음에도 데이터베이스에 영속화된 것을 확인할 수 있다.
  * 즉, **JPA는 매핑 정보를 분석하여 적절한 쿼리를 생성한 후에 데이터를 영속화**한다.
  * 이 경우, 데이터가 영속화될 테이블 정보를 명시하지 않았음에도 JPA의 관례에 따라 MEMBER 테이블에 데이터를 저장한다.
  * 반면 데이터가 영속화될 테이블명과 클래스의 이름이 상이한 경우에는 반드시 클래스에 @Table(name = "TABLE_NAME") 어노테이션을 명시해야 한다.
* 상술한 코드는 에러 발생시의 롤백 처리가 전혀 안되어 있으며, 엔티티 매니저를 잘 닫는다는 보장도 없다.
  * 때문에 다음과 같은 try - catch - finally 구문을 활용하여 코드를 수정하는 것이 바람직하다.
```
public static void main(String[] args) {
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
    EntityManager em = emf.createEntityManager(); // 데이터베이스 커넥션을 하나 받아온 것으로 간단히 이해할 수도 있다.

    EntityTransaction tx = em.getTransaction();
    tx.begin();

    try {
        Member member = new Member();
        member.setId(1L);
        member.setName("ingnoh");
        em.persist(member);

        tx.commit();    
    } catch (Exception e) {
        tx.rollback();
    } finally {
        em.close();
    }
    emf.close(); // 모든 동작이 완료되어 애플리케이션이 종료되면 emf를 닫아줘야 한다.
}
```
* 특히 **엔티티 매니저는 내부적으로 데이터베이스 커넥션을 갖고 동작하므로, 유사시에도 항상 close될 수 있도록 코드를 작성하는 것이 매우 중요**하다.

### 정말 이렇게 해야할까?
* **상술한 방식은 이미 스프링에 의해 대부분 자동화되므로, 개발자는 em.persist 정도만 호출해도 데이터를 영속화하는 데에는 아무런 어려움을 겪지 않는다**.

### 중간 정리
* **엔티티 매니저 팩토리는 애플리케이션의 로딩 시점에 단 하나만 생성되고, 애플리케이션과 같은 생명 주기를 가져야 한다**.
  * 웹 서비스의 경우, 웹 서버가 동작을 시작하는 과정에서 단 하나만 생성되어야 한다.
  * 또한 웹 서버가 종료될 때 엔티티 매니저 팩토리가 닫혀야 커넥션 등의 리소스가 릴리즈될 수 있다.
* 반면, **엔티티 매니저는 고객의 요청마다 생성하여 사용한 후에 close하는 식으로 코드를 작성**해야 한다.
  * **엔티티 매니저는 절대로 스레드 간에 공유되지 않아야 하며, 반드시 사용하고 버리는 식으로 구현**해야 한다.
  * 이는 마치 데이터베이스 커넥션을 최대한 빠르게 사용하고 반환해야하는 것과 같다.
* **JPA의 모든 데이터 변경은 반드시 트랜잭션 안에서 실행**되어야 한다.
  * 반면, 단순 조회는 상관이 없다.
  * **이는 JPA의 한계가 아니며, RDB 자체가 데이터의 변경을 반드시 트랜잭션 안에서 실행하도록 설계되었기 때문**이다.

### JPQL이란?
```
> JPQL은 객체를 대상으로 하는 객체 지향 쿼리이다.
> JPQL의 목적은 SQL을 추상화하여 데이터베이스 별 방언에 의존하지 않는 것에 있다.
```
* JPA를 사용하면 자연스레 엔티티 객체를 중심으로 개발을 진행하게 되지만, 검색 요구사항은 처리가 어려울 수 있다.
* **쿼리 역시 JPA의 사상에 따라 테이블보다는 엔티티를 대상으로 검색해야 했으며, JPQL은 객체지향과 RDB 사이의 불일치성을 해결하기 위해 도입**되었다.
* 이 때, 데이터베이스의 모든 데이터를 객체로 변환하여 검색하는 것은 현실적으로 불가능하다.
  * 때문에 애플리케이션이 필요로하는 데이터만 데이터베이스로부터 추려서 가져오기 위해서는 결국 검색 조건을 작성할 수 있는 SQL의 도입이 필요할 수 밖에 없다.
* 그러나 **실제 SQL을 사용하게 되면 결국 각 데이터베이스의 방언에 종속되는 설계로 이어지므로, JPA는 엔티티를 대상으로 쿼리하는 JPQL을 도입**하였다.
* **JPQL은 실제 SQL과 유사한 문법을 지원하며, 테이블을 대상으로 쿼리하는 SQL과 달리 엔티티 객체를 대상으로 쿼리한다는 차이점이 존재**한다.
  * 이러한 JPQL의 도입으로 인해 애플리케이션이 사용하는 데이터베이스가 바뀌어 방언이 변경되더라도 JPQL을 변경할 필요가 없어진다.
  * 이렇듯 **JPA 입장에서는 실재하는 테이블을 대상으로 절대 코드를 작성하지 않으며, 대신 엔티티 객체를 대상으로 코드를 작성**한다.