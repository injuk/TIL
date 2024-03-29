# Basics
## 2022-07-26 Tue

### JPA가 지원하는 쿼리 방식
* **데이터베이스에서 데이터를 특정 조건으로 조회하려면 결국 SQL을 사용해야하므로, JPA는 기본적으로 다음과 같은 다양한 쿼리 방식을 지원**한다.
  1. JPQL
  2. JPA Criteria
  3. QueryDSL
  4. 네이티브 SQL
  5. JDBC API, MyBatis, Spring JDBC Template, 등...
* 이렇듯 ORM 기술은 단순한 단건 조회, 단건 생성 등에 대해서 제공하는 메소드 이외에도 복잡한 쿼리를 지원할 수 있어야 한다.
* 이 중, JPA Criteria와 QueryDSL은 Java 코드를 통해 JPQL을 빌드할 수 있는 방법을 제공하는 제너레이터로 이해할 수 있다.
* **네이티브 SQL의 경우, JPA와 표준 SQL 만으로는 해결이 안되는 특정 데이터베이스 종속적인 쿼리를 지원하기 위해 JPA가 지원하는 기능**이다.
* **실무에서는 대부분 JPQL로 모든 요구사항을 충족시킬 수 있으며, 간혹 충족할 수 없는 경우 네이티브 SQL 또는 MyBatis, JDBC Template를 활용**한다.

### JPQL이란?
```
> JPQL이란, 객체를 대상으로 검색하는 객체 지향 SQL이다.
```
* JPA에서, 식별자를 활용한 가장 단순한 조회는 find 메소드를 통해 해결할 수 있다.
  * 반면, 이러한 방식은 조건부 검색 등 상대적으로 더 복잡한 쿼리를 요청해야하는 경우에 대응할 수 없다.
* 이렇듯 JPA를 활용하면 엔티티 객체를 중심으로 개발을 진행하게 되지만, 문제는 검색 쿼리를 요청하는 경우에 발생한다.
  * 이는 **검색 시에도 테이블 대신 엔티티 객체를 대상으로 검색하기 때문이지만, 모든 데이터베이스 데이터를 메모리 상 객체로 변환하여 검색할 수는 없다**.
  * 즉, **애플리케이션이 필요한 데이터베이스만 검색하려면 결국 검색 조건을 포함하는 SQL의 작성이 필수적**이다.
* JPA는 이러한 SQL을 추상화하는 객체 지향 쿼리 언어인 JPQL을 제공하며, 문법 역시 기존 SQL과 유사하게 사용할 수 있다.
* **SQL이 데이터베이스 테이블을 대상으로 쿼리하는 반면, JPQL은 엔티티 객체를 대상으로 쿼리를 수행**한다.
  * 결국 JPQL을 사용하면, 최종적으로는 SQL로 번역되어 요청된다.
* 이 때, 간단한 JPQL 쿼리의 예시는 다음과 같이 작성하여 요청해볼 수 있다.
  * 쿼리의 내용 중 `Member m`은 테이블이 아닌 엔티티 클래스로서의 Member를 의미한다.
  * JPA는 이러한 JPQL을 분석하여 적절한 SQL 쿼리를 생성하여 데이터베이스에 요청하는 방식으로 동작한다.
```
String query = "select m from Member m where m.name like '%ingnoh%'";
List<Member> result = em.createQuery(query, Member.class)
  .getResultList();
```
* **JPQL은 데이터베이스의 테이블이 아닌 객체를 대상으로 검색하며, SQL을 추상화하므로 임의의 데이터베이스 SQL에 의존하지 않도록 한다**.

### JPA Criteria란?
* **JQPL에서 사용되는 쿼리는 근본적으로 String에 불과하므로, 동적 쿼리를 작성하기에는 명확한 한계**가 있다.
  * 문자열을 더하는 연산은 가능하지만, 이는 휴먼 에러가 너무나도 발생하기 쉽다는 단점이 존재한다. 
  * **반면, MyBatis 등의 SQL 매퍼는 동적 쿼리를 쉽게 정의할 수 있다는 장점이 존재**한다.
* Criteria는 이러한 동작 쿼리 문제를 해결하며, 여러 추가적인 장점을 제공하기 위한 대안으로 고안되었다.
  * 이를 통해 문자열이 아닌 Java 코드만으로 JPQL을 작성하는 JQPL 빌더 기능을 제공받을 수 있다.
* **Criteria를 사용하는 경우, 문자열이 아닌 Java 코드 상에서 쿼리를 동적으로 생성하므로 많은 오류를 컴파일 시점에서 인식할 수 있다는 장점**이 있다.
  * 그러나 **Criteria 역시 복잡한 쿼리를 생성하기에는 너무 어렵고 코드가 장황해진다는 단점 역시 존재**한다.
* **Criteria는 이렇듯 JPA 표준 스펙에서 제공하는 개념이지만, 유지보수성이 낮다는 이유로 인해 실무에서는 잘 사용되지 않는다**.
  * 그러나 여전히 **실무에서는 동적 쿼리가 자주 사용되며, Criteria와 같은 컴파일 시점의 SQL 문법 확인 기능 역시 제공받을 필요**가 있다.
  * 이러한 이유에서, 실무에서는 일반적으로 QueryDSL의 사용이 권장된다.

### QueryDSL이란?
* QueryDSL은 JPA Criteria와 마찬가지로 Java 코드를 기반으로 한 JPQL 빌더 역할을 수행하며, 컴파일 시점에 문법 오류를 찾을 수 있도록 한다.
* 그러나 **JPA Criteria와 달리 동적쿼리 작성이 편리하고, 단순하고 쉬워 유지보수성도 높으므로 실무에서 사용이 권장**된다.
  * QueryDSL은 사실상 JPQL과 일대 일 매칭되므로, JPQL을 정확히 이해한 경우 QueryDSL은 공식 문서만으로도 사용이 가능할 수 있다.
  * 이는 그만큼 QueryDSL의 공식 문서가 잘 작성되어 있기 때문이기도 하다.
* **실무에서는 95% 이상의 쿼리가 JPQL과 QueryDSL 조합으로 해결이 가능**하다.

### 네이티브 SQL이란?
* **JPA는 SQL을 직접 사용하는 기능을 네이티브 SQL 형태로 제공하며, 이는 JPQL만으로는 해결할 수 없는 데이터베이스 의존적인 기능에 활용**된다.
  * 예를 들어, Oracle의 `CONNECT BY` 등의 기능이 포함된다.
* **네이티브 SQL은 실제 SQL을 문자열 형태로 작성하고, 엔티티 매니저의 createNativeQuery 메소드를 호출하여 사용**할 수 있다.
```
String query = "SELECT * FROM MEMBER WHERE NAME = 'ingnoh'";
List<Member> results = em.createNativeQuery(query, Member.class)
  .getResultList();
```
* 그러나 **실무에서는 일반적으로 네이티브 SQL보다는 Spring JDBC Template을 사용하는 편**이다.

### JDBC 직접 사용, 또는 Spring JDBC Template 사용하기
* JPA를 사용하는 과정에서 JDBC 커넥션을 직접 사용하거나, Spring의 JDBC Template 또는 SQL 매퍼인 MyBatis를 함께 사용할 수 있다.
* 단, 반드시 영속성 컨텍스트를 적시에 강제로 flush 해 줄 필요가 있다.
  * 예를 들어, JPA를 우회하여 SQL을 실행하기 전에 영속성 컨텍스트를 직접 flush 한다.
* **기본적으로 트랜잭션이 커밋되는 시점이나 createQuery 또는 createNativeQuery 메소드가 호출되는 경우, JPA는 자동으로 flush를 호출**한다.
  * 이렇듯 JPA는 쿼리를 직접 요청하는 경우 암시적으로 flush를 호출하여 쿼리 결과를 그 때 그 때 데이터베이스에 반영한다.
  * 때문에 **이러한 동작을 JPA의 기본 동작으로 혼동하여 flush를 호출하지 않는 실수를 범하지 않아야 한다**.
* 실무에서 JPQL과 QueryDSL로 해결이 안되는 약 5%의 복잡한 통계성 쿼리는 Spring JDBC Template 등을 활용하여 쿼리를 직접 요청할 수 있다.
  * 그러나 이마저도 대부분의 경우에는 JPQL과 QueryDSL로 풀어낼 수 있다.

### Java Persistence Query Language
* **JPQL은 객체 지향 쿼리 언어이며, 테이블을 대상으로 하지 않고 객체를 대상으로 쿼리를 수행**한다.
* JPQL은 SQL을 한 단계 더 추상화하므로, 특정한 데이터베이스에 의존하지 않는다.
  * 그러나 **JPQL은 매핑 정보와 방언을 조합하여 결국 SQL로 변환**된다.

### JPQL의 기본 문법
* **JPQL의 문법은 기본적으로 SQL과 같은 형태**를 띈다.
  * 예를 들어, select절과 from절에 이어 where절이나 groupBy, having, order by 절 등을 작성할 수 있다.
```
select m from Member as m where m.age > 13
```
* **엔티티와 속성은 모두 대소문자를 구분하지만, JPQL 키워드인 select나 from 절 등은 대소문자를 구분하지 않는다**.
* **JPQL은 엔티티를 대상으로 쿼리하는 객체 지향 쿼리 언어이므로, 테이블의 이름이 아닌 엔티티의 이름을 사용**한다.
  * 예를 들어, 상술한 쿼리의 요청 대상은 Member 엔티티이다.
* **쿼리에서 엔티티의 별칭은 필수이나, as 키워드를 생략이 가능**하다. 
  * JPA 표준 스펙에서는 as 키워드를 명시하지만, 하이버네이트에서는 이를 생략 가능하다.

### JPQL의 집합 및 정렬 연산
* JPQL 역시 SQL과 마찬가지로 SUM, COUNT, AVG 등을 호출할 수 있다.
  * 집합과 정렬을 위한 기능인 ORDER BY, GROUP BY, HAVING 역시 기존 SQL과 같은 방식으로 사용할 수 있다.

## 2022-07-27 Wed
### Type Query와 Query
* **Type query는 반환 타입이 명확할 때에만 사용할 수 있으나, Query는 반환 타입이 명확하지 않아도 좋다.**
* 예를 들어 쿼리 특성상 타입 정보를 명확히 전달할 수 있는 경우, createQuery 메소드의 반환 타입은 제네릭을 갖는 TypeQuery가 된다.
  * 아래 코드의 경우, createQuery 메소드의 두번째 인자로 Member.class라는 타입 정보를 명확히 전달한다.
```
TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);
```
* 그러나 타입 정보를 다음과 같이 명확히 명시할 수 없는 경우도 발생하며, 이 경우 createQuery 메소드의 반환 타입은 Query이다.
  * **문제는 select 절에서 가져오려는 정보가 String과 Integer가 혼합되었다는 점**에 있다.
  * 만일 m.name만 가져오는 쿼리의 경우, createQuery 메소드의 두 번째 인자로 String.class를 전달하여 TypedQuery를 반환받을 수 있다.
```
Query query = em.createQuery("select m.name, m.age from Member m");
```

### 쿼리를 활용한 결과 조회 API
* JPA에서 JPQL을 통해 쿼리하는 경우, 결과를 조회할 수 있는 메소드는 다음과 같이 분류된다.
  1. `query.getResultList()`: 결과가 하나 이상인 경우에 사용하며, List가 반환된다.
  2. `query.getSingleResult()`: 결과가 정확히 하나인 경우에 해당하며, 단일 객체를 반환한다.
* **getResultList 메소드의 경우, 결과를 발견하지 못했다면 빈 List가 반환**된다.
  * 때문에 **getResultList는 NPE로부터 안전**할 수 있다.
* 반면, **getSingleResult 메소드의 경우 결과가 없거나 둘 이상인 경우에 예외가 발생**한다.
  * 결과가 없는 경우, NoResultException 예외가 발생한다.
  * 결과가 둘 이상인 경우, NonUniqueResultException 예외가 발생한다.
* 덧붙여 getSingleResult 메소드에서, 결과가 없는 경우에 예외를 발생시키는 스펙은 굉장히 논란이 많은 부분이기도 하다.
  * 때문에 Spring Data JPA에서는 이를 한 번 감싸 추상화하는 과정에서 결과가 없는 경우 null 또는 Optional을 반환한다.

### 쿼리 파라미터 바인딩
* JPQL에서는 쿼리에 파라미터를 작성할 수 있으며, 다음과 같이 파라미터 정보를 바인딩할 수 있다.
  * 이는 파라미터 명을 기준으로 한 바인딩에 해당한다.
```
List<Member> member = em.createQuery("select m from Member m where m.name = :name", Member.class)
  .setParameter("name", "memberName")
  .getResultList();
```
* 또는 다음과 같이 파라미터 위치를 기준으로 바인딩할 수도 있다.
  * 그러나 **위치를 숫자로 명기하는 해당 방식은 쿼리의 처음 또는 중간에 새로운 파라미터가 추가된 경우 에러가 발생하기 쉬우므로 유지보수성이 떨어진다**.
  * 이러한 이유에서 **위치 기반으로 파라미터를 바인딩하는 해당 방식은 실무에서 권장되지 않는다**.
  * 반면, 파라미터 명 기반 바인딩은 문자열을 사용하므로 위치 기반의 바인딩에서 발생할 수 있는 문제와는 관계가 없다.
```
List<Member> member = em.createQuery("select m from Member m where m.name = ?1", Member.class)
  .setParameter(1, "memberName")
  .getResultList();
```

### 프로젝션이란?
* **프로젝션이란 단순히 SELECT 절에서 조회할 대상을 지정하는 개념**이다.
* 이 때, JPA에서 지원하는 프로젝션 대상은 크게 다음과 같다.
  1. 엔티티
  2. 임베디드 타입: 개발자가 직접 정의한 복합 값 타입을 의미한다.
  3. 스칼라 타입: 숫자, 문자 등 기본 데이터 형을 의미한다.
* 반면, **관계형 데이터베이스의 경우 스칼라 타입만 선택이 가능**하다.
```
SELECT m FROM Member m // Member 엔티티를 조회한다.
SELECT m.team FROM Member m // Member 엔티티와 연관 관계를 맺는 Team 엔티티를 조회한다.
SELECT m.address FROM Member m // Member 엔티티가 갖는 복합 값 타입인 address를 조회한다.
SELECT m.name, m.age FROM Member m // Member 엔티티가 갖는 기본 값 타입을 조회한다.
```
* 또한, JPQL 역시 SELECT 절에 DISTINCT 지시어를 작성하여 데이터의 중복을 제거할 수도 있다.

### 엔티티 프로젝션과 영속성 컨텍스트
* **엔티티 프로젝션과 getResultList 메소드에 의해 조회된 대상은 다수일 수 있으며, 이는 모두 조회 시점부터 영속성 컨텍스트에 의해 관리**된다.

### 프로젝션에서 여러 값을 조회하는 경우
* 예를 들어 `SELECT m.name, m.age FROM Member m`과 같은 JPQL의 경우, 각각 String과 int 타입이 조회됨을 예측할 수 있다.
* 이러한 경우, 다음과 같은 방법 중 하나로 값을 조회할 수 있다.
  1. Query 타입으로 조회하기
  2. Object[] 타입으로 조회하기
  3. new 명령어와 Dto를 활용하여 바로 조회하기
* Query 타입으로 조회하는 경우, 반환 형은 Object를 포함하는 원시 List가 된다.
  * 이 때, 각각의 요소는 Object[] 형으로 적용된다.
  * **이러한 특징을 활용하면 List<Object[]> 형태로 제네릭을 적용할 수도 있다**.
```
List results = em.createQuery("select m.name, m.age from Member m")
  .getResultList();
Object obj = resultList.get(0);
Object[] result = (Object[]) obj;
System.out.println(result[0]);
System.out.println(result[1]);
```
* 특히, **Dto를 사용하는 경우 `SELECT new jpastudy.jpql.MemberDto(m.name, m.age) FROM Member m`과 같은 형태로 작성**할 수 있다.
  * 이 때, Dto 클래스는 반드시 패키지 명을 포함하는 full path로 작성해야 한다.
  * 또한, **해당 클래스에는 반드시 순서와 타입이 일치하는 생성자가 정의되어 있어야 한다**.
* 세 방식 중 가장 깔끔한 방식은 Dto를 활용하는 것이며, 이 경우의 코드는 다음과 같이 작성할 수 있다.
  * 그러나 해당 방식은 쿼리가 문자열이므로, 반드시 패키지 명을 모두 작성해야한다는 단점이 역시 존재한다. 
  * 이는 QueryDSL과 같은 라이브러리를 통해 상당 부분 해결할 수 있다.
```
List<MemberDto> results = em.createQuery("select new jpql.MemberDto(m.name, m.age) from Member m", MemberDto.class)
  .getResultList();
```

## 2022-07-28 Thu
### 페이징 API란?
* JPA는 다음과 같이 페이징을 추상화하는 API를 제공한다.
  1. setFirstResult(): int를 인수로 전달하여 조회 시작 위치를 지정할 수 있으며, 기본값으로는 0이 설정된다.
  2. setMaxResults(): int를 인수로 전달하여 조회할 데이터의 수를 지정한다.
```
List<Member> results = em.createQuery("select m from Member m order by m.age desc", Member.class)
  .setFirstResult(10)
  .setMaxResults(50)
  .getResultList();
```
* 이렇듯 **JPA는 페이징 API를 매우 적절하게 추상화하고 있으므로, 이게 전부**이다!

### JPQL에서의 JOIN 활용
* **조인 역시 큰 틀에서 SQL 문법과 차이가 없지만, JPQL의 경우 엔티티를 대상으로 조인한다는 점에서 차이가 존재**한다.
* 내부 조인의 경우, 다음과 같이 작성할 수 있다.
  * 이 때, INNER 키워드는 생략이 가능하다.
  * 아래와 같은 내부 조인의 경우, member는 존재하지만 연관 관계에 있는 team이 존재하지 않는다면 데이터가 조회되지 않는다.
  * 즉, 내부 조인의 경우 member 정보가 조회되지 않을 수도 있다.
```
select m from member m [INNER] join m.team t
```
* 외부 조인의 경우, 다음과 같이 작성할 수 있다.
  * 이 때, OUTER 키워드는 생략이 가능하다.
  * 아래와 같은 외부 조인의 경우, member는 존재하지만 연관 관계에 있는 team이 존재하지 않는다면 team 관련 정보는 모두 null로 조회된다.
  * 즉, 외부 조인의 경우 member 정보는 항상 조회된다.
```
select m from member m left [OUTER] join m.team t
```
* 세타 조인의 경우, 다음과 같이 작성할 수 있다.
  * **세타 조인은 아무런 관계가 없는 두 엔티티의 공통점을 기반으로 조회**한다.
  * 이 때, **조회되는 데이터의 결과는 카테션 곱**이 된다.
```
select m from member m, team t where m.name = t.name
```

### JOIN과 ON 절
* ON 절을 활용하여 조인하는 경우, 조인 대상을 필터링이 가능하다.
  * 예를 들어, 회원과 팀을 조인하는 과정에서 팀 이름이 특정한 조건을 만족하는 경우에만 조인할 수 있다.
  * 즉, **실제 조인을 수행하기 전에 ON 절의 조건을 통해 팀의 데이터 수를 우선 줄여둔 후에 조인을 수행**한다.
  * 이렇듯 ON 절은 조인에서 사용되는 조건절을 의미한다.
* 무엇보다, **해당 방식을 통해 연관 관계가 전혀 없는 엔티티 간의 외부 조인이 가능**해진다.
  * 말이 안되는 시나리오이나, 회원의 이름과 팀의 이름이 같은 대상만을 외부 조인하기 위해 ON 절을 사용할 수 있다.
  * 이렇듯 요구 사항에 맞추어 전혀 연관 관계가 없는 두 엔티티를 LEFT JOIN하고자 하는 경우에 ON 절을 사용할 수 있다.

## 2022-07-29 Fri
### 서브 쿼리란?
* JPA의 서브 쿼리는 일반적인 SQL에서 말하는 서브 쿼리와 다르지 않다.
  * 예를 들어, 쿼리 내부에서 쿼리를 또 만들어낼 수 있다.
```
select m from Member m where m.age > (select avg(m2.age) from Member m2)
```
* 상술한 예제의 경우, 같은 엔티티인 Member에 대해 쿼리하지만 서브 쿼리에서 m이 아닌 m2를 대상으로 명시한다.
  * 이를 통해 **메인 쿼리와 서브 쿼리 사이의 연관성을 끊어내며, 서브 쿼리는 이러한 방식으로 설계되어야 성능이 좋다**. 
```
select m from Member b where (select count(o) from Order o where m = o.member) > 0
```
* 반면, 상술한 쿼리는 서브 쿼리에서 메인 쿼리의 m을 사용하므로 상대적으로 성능은 뒤떨어진다.

### 서브 쿼리 지원 함수
* EXISTS 키워드는 서브 쿼리에 결과가 존재하는 경우에 참으로 판정된다.
* ALL 키워드는 조건을 모두 만족하는 경우에 참으로 판정된다.
* ANY, SOME 키워드는 조건을 하나라도 만족하는 경우에 참으로 판정된다.
* IN 키워드는 서브 쿼리의 결과 중 하나라도 같은 것이 있는 경우에 참으로 판정된다.
* 상술한 함수 키워드들은 모두 다음과 같이 키워드(서브 쿼리) 형태로 사용한다.
```
select m from Member m where m.team ANY (select t from Team t)
```

### JPA 서브 쿼리의 한계점
* JPA 표준 스펙에서는 where와 having 절에서만 서브 쿼리를 사용할 수 있으나, 하이버네이트의 경우 select 절에서도 가능하다.
* 반면, **from 절의 서브 쿼리는 현재 JPQL에서는 사용이 불가능**하다.
  * 이러한 요구 사항은 **대부분의 경우 조인으로 풀어낼 수 있으므로 조인을 사용하되, 그 조차 불가능하다면 포기**해야 한다.
  * 또는 좋은 접근은 아니지만 네이티브 SQL을 활용하거나, 메인 쿼리와 서브 쿼리를 각각 요청하는 방식을 고려할 수도 있다.

### JPQL의 타입 표현
* JPQL에서 사용 가능한 타입 표현은 크게 다음과 같다.
  1. 문자형: 'Hello'와 같은 형태로 표현한다.
  2. 숫자형: 10L, 10D, 10F와 같은 형태로 표현한다.
  3. 불린형: TRUE, FALSE와 같은 형태로 표현한다.
  4. 열거형: 패키지명을 포함하여 `study.MemberType.Admin`과 같은 형태로 표현한다.
     * 이 경우, 열거형 클래스의 이름이 MemberType이 된다.
     * 이렇듯 **쿼리에 풀 패키지 경로를 하드코딩하지 않기 위해, 일반적으로는 setParameter를 활용하여 JPQL을 작성하는 것이 권장**된다.
  5. 엔티티 타입: TYPE(m) = Member와 같이 표현하며, 상속 관계에서 사용한다.
* setParameter를 활용한 열거형 적용은 다음과 같은 코드로 표현할 수 있다.
```
em.createQuery("select m.name from Member m where m.type = :userType", Member.class)
  .setParameter("userType", MemberType.ADMIN)
  .getReulstList();
```
* 객체 지향적으로 개발하는 경우, 상속 관계를 활용하는 방식은 다음과 같은 코드로 표현할 수 있다.
```
em.createQuery("select i from Item i where type(i) = Book", Item.class)
  .getResultList();
```

### SQL 문법을 따르는 JPQL 표현식의 예시
* JPQL은 다음과 같은 키워드를 제공하며, SQL 표준과 동일한 문법을 지향한다.
  1. EXISTS
  2. IN
  3. AND, OR, NOT
  4. =, >, <, >=, <=, <>
  5. BETWEEN, LIKE, IS NULL

## 2022-07-30 Sat
### 조건식 - CASE 식
* JPQL의 조건식은 다음과 같이 작성해볼 수 있다.
```
select
  case when m.age <= 10 then '학생'
       when m.age >= 60 then '경로'
       else '일반'
  end
from Member m
```
* 또는 이를 다음과 같이 단순화하여 작성할 수도 있다.
```
select
  case m.name
       when 'A' then '학생'
       when 'B' then '경로'
       else '일반'
  end
from Member m
```
* COALESCE 키워드는 값을 하나씩 조회하여 null이 아닌 경우에만 반환한다.
  * 예를 들어, 아래의 쿼리는 사용자의 이름이 없는 경우에 anonymous를 반환한다.
```
select coalesce(m.name, 'anonymous') from Member m
```
* NULLIF 키워드는 두 값이 같으면 null을 반환하고, 다른 경우에는 첫 값을 반환한다.
  * 예를 들어, 아래의 쿼리는 사용자의 이름이 admin인 경우에만 null을 반환한다.
```
select NULLIF(m.name, 'admin') from Member m
```
* 상술한 함수는 모두 표준 함수이므로, JPA가 지원하는 모든 데이터베이스에서 활용이 가능하다.

### JPQL의 함수
* JPQL이 지원하는 함수는 크게 다음과 같이 분류된다.
  1. 기본 함수: JPQL이 제공하는 기본 함수이며, 데이터베이스와 상관 없이 사용이 가능하다.
  2. 사용자 정의 함수: 기본 함수만으로는 처리할 수 없는 상황에는 개발자가 직접 함수를 정의하여 사용이 가능하다.
     * 그러나 사용자 정의 함수는 그냥 사용할 수는 없으며, 사용하는 데이터베이스의 방언을 상속받아 사용자 정의 함수를 등록해주어야 한다.
  3. 하이버네이트가 기본적으로 포함하는 함수: registerFunction에 의해 등록된 데이터베이스 종속적인 함수이다.
     * **데이터베이스 종속적이므로, 데이터베이스가 변경되는 경우에는 사용할 수 없을 가능성이 높다**.
     * 그러나 **대부분의 데이터베이스가 제공하는 함수들은 미리 등록되어 있으므로, 편리하게 사용이 가능**하다.
     * 반면, 등록되어 있지 않는 함수의 경우 데이터베이스 벤더들이 제공하지 않는 함수이므로, 개발자가 직접 등록하여 사용해야 한다.

### JPQL 기본 함수
* CONCAT은 인자로 전달된 문자열들을 더하여 반환한다.
  * **하이버네이트를 사용하는 경우, CONCAT 함수 대신 `'a' || 'b'`와 같이 `||` 연산자를 사용할 수도 있다**.
* SUBSTRING은 첫 번째 인자로 전달된 문자열에 범위를 지정하여 잘라 반환한다. 
* TRIM은 공백을 제거하여 반환한다.
* LOWER, UPPER는 대, 소문자를 바꾸어 반환한다.
* LENGTH는 문자열의 길이를 반환한다.
* LOCATE는 전달 된 문자열에 포함된 임의의 문자열의 시작 인덱스를 반환한다.
* ABS, SQRT, MOD는 여러 수학 연산에 활용되는 함수이다.
* SIZE, INDEX는 JPA 용도로 사용되며, 각각 엔티티 컬렉션의 길이와 컬렉션에 포함된 요소의 인덱스를 반환한다.
  * 그러나 **이 두 함수의 경우, 컬렉션이 중도에 변경될 가능성이 높으므로 사용을 지양하는 것이 바람직**하다. 

### JPQL 사용자 정의 함수
* 사용자 정의 함수의 경우, 데이터베이스에 함수를 추가한 후에 다음과 같이 데이터베이스 방언 클래스를 확장할 필요가 있다.
  * 이러한 사용자 정의 함수의 등록 방식은 암기할 필요는 없으며, 필요한 경우 Dialect 클래스를 참고하여 작성할 수 있다.
```
public class MyDialect extends H2Dialect {
  public MyDialect() {
    registerFunction("my_func", new StandardSQLFunction("my_func", StandardBasicTypes.STRING));
  }
}
```
* 상술했듯 **데이터베이스 방언을 확장하는 클래스를 정의한 경우, persistence.xml에 다음과 같이 작성하여 해당 방언을 사용할 것임을 명시**한다.
```
<property name="hibernate.dialect" value="dialect.MyDialect"/>
```
* 이러한 과정을 통해 등록한 사용자 정의 함수를 실제로 사용하는 경우, 다음과 같은 코드를 작성할 수 있다.
  * 이렇듯 사용자 정의 함수를 호출하는 경우 `function('함수이름', 인자)`의 형태로 쿼리를 작성해야 한다.
  * 반면, 하이버네이트를 사용하는 경우에는 이를 `함수이름(인자)` 형태로 간소화할 수 있다.
```
em.createQuery("select function('my_func', m.name) from Member m", Member.class)
  .getResultList();
```