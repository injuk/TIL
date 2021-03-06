# Basics
## 2022-07-24 Sun

### JPA의 데이터 타입 분류
* JPA는 데이터 타입을 크게 다음과 같이 분류한다.
  1. 엔티티 타입
  2. 값 타입
* **엔티티 타입이란 @Entity 어노테이션을 명시하는 객체이며, 데이터가 변하더라도 식별자를 통해 추적이 가능**하다. 
  * 예를 들어, 회원 엔티티의 이름 또는 나이 데이터가 변경되더라도 id와 같은 식별자를 통해 엔티티를 식별할 수 있다.
  * 극단적으로는 엔티티가 포함하는 모든 데이터 필드가 수정되었더라도 식별자를 통해 엔티티를 추적할 수 있다.
* 반면, **값 타입이란 int나 Integer와 같이 단순한 값으로 사용하는 Java의 기본 타입 또는 객체를 의미**한다. 
  * 100 또는 200 등과 같은 값 타입은 식별자가 존재하지 않으므로, 데이터가 변경된 경우에 추적은 불가능하다.
  * int를 예로 들어 **100이라는 데이터를 200으로 수정한 경우, 값은 완전히 다른 값으로 대체되는 것으로 이해**할 수 있다.

### 값 타입의 분류
* 값 타입은 다시 다음과 같이 분류할 수 있다.
  1. 기본 값 타입: int나 double과 같은 Java의 기본 값 타입과 래퍼 클래스, String 등을 의미한다.
  2. 임베디드 타입: 복합 값 타입이라고 지칭할 수 있으며, 사용자 정의 값 타입을 의미한다.
  3. 컬렉션 값 타입: Java가 제공하는 컬렉션 API를 통해 기본 값 타입 또는 임베디드 타입을 관리할 수 있다. 

### 기본 값 타입이란?
* **기본 값 타입은 String 또는 int와 같은 Java의 기본 데이터 타입을 의미하며, 생명 주기를 엔티티에 의존**한다.
  * 예를 들어, 회원 엔티티를 제거하면 이에 종속된 이름, 나이 등의 필드 역시 함께 삭제된다.
* **값 타입은 공유되지 않아야 한다**.
  * 예를 들어, 회원 A에 대한 기본 값 타입의 수정이 다른 회원 엔티티의 속성에 영향을 주지 않아야 한다.
  * 즉, 값의 관리 과정에서 다른 엔티티에 영향을 주는 사이드 이펙트가 발생하지 않아야 한다.
* Java의 경우 언어 자체적으로 기본 타입은 절대로 공유되지 않는다는 특징이 있으므로, 기본 값 타입을 사용하는 것은 안전하다.
  * 이는 Java의 기본 값 타입이 항상 값을 복사하는 식으로 동작하기 떄문이다.
* 반면, **래퍼 클래스 또는 String과 같은 특수한 클래스는 공유가 가능한 객체이지만 값을 변경할 수는 없는 불변 객체의 특징**을 갖는다.
* 결론적으로 Java의 기본 값 타입과 래퍼 클래스, String은 엔티티 간에 영향을 주는 사이드 이펙트가 발생하지 않는다.

### 임베디드 타입이란?
* **임베디드 타입은 복합 값 타입이라고도 하며, 새로운 값 타입을 개발자가 직접 정의할 수 있는 기능**을 말한다.
  * 복합 값 타입이라는 용어로 지칭하는 이유는 주로 int, String과 같은 기본 값 타입을 모아 새로운 타입을 정의하기 때문이다.
* 복합 값 타입 역시 int, String과 같은 값 타입으로 취급된다.
  * **복합 값 타입은 값 타입이지 엔티티 타입이 아니기 때문에 값의 추적이 불가능하며, 수정시 완전히 새로운 값으로 대체**된다.
* 복합 값 타입은 어려울게 없으며, 단지 엔티티에서 사용되는 값 중 공통되기 쉬운 속성을 새로운 값 객체로 추출한 것에 지나지 않는다.
* 이러한 이유에서 개발자에 의해 정의된 복합 값 타입을 JPA에서 사용하는 경우, 다음과 같은 어노테이션을 명시해야 한다.
  1. @Embeddable: 값 타입을 새로이 정의하는 클래스에 명시한다.
  2. @Embedded: 새로이 정의된 값 타입을 실제로 사용하는 위치에 명시한다.
* 이 때, **새로 정의한 복합 값 타입은 반드시 기본 생성자를 포함해야 한다**.

### 임베디드 타입은 왜 사용하는가?
* 복합 값 타입을 적절히 사용할 경우, 다음과 같은 이점을 얻을 수 있다.
  1. 기간 등의 클래스를 복합 값 타입으로 정의할 경우, 시스템 전체적으로 공유하여 코드의 재사용성을 높일 수 있다.
  2. 새로이 정의된 복합 값 클래스 내부적으로 높은 응집도를 유지할 수 있다.
  3. 기간을 Period 라는 복합 값 타입으로 정의한 경우, `Period.isWork()`와 같이 해당 값 타입에 사용할 수 있는 의미 있는 메소드를 추가할 수 있다.
  4. **복합 값 타입을 포함하는 모든 값 타입은 값 타입을 소유하는 엔티티에 의존적으로 생명 주기가 관리**된다.
     * 값 타입은 기본적으로 엔티티가 제거되면 연관된 값 타입은 모두 함께 제거되며, 엔티티가 새로이 생성되는 시점에 종속된다.
* 반면, **복합 값 타입은 어디까지나 객체지향 프로그래밍에 도움을 주는 개념이므로 관계형 데이터베이스의 테이블 입장에서는 바뀌는 점이 없다**.
  * 때문에 주소 또는 기간을 각각 Address, Period 클래스라는 복합 값 타입으로 추출하였다고 해도 테이블에는 추출된 클래스의 속성 명이 컬럼으로 매핑된다.
  * **테이블은 객체가 복합 값 타입을 사용하든 사용하지 않든 똑같이 단일 테이블로 유지되며, 대신 약간의 매핑 작업만 처리**해주면 된다.

### 임베디드 타입 정의 예시
* 복합 값 타입은 상술한 @Embeddable과 @Embedded 어노테이션을 명시하여 쉽게 구현할 수 있다.
  * 둘 중 하나만 명시해도 정상 동작하지만, 둘 모두 명시하는 것이 권장된다.
```
// Member.java
public class Member {
  // ...생략
  @Embedded
  private Period workPeriod;
  // ...생략
}

// Period.java
@Embeddable
public class Period {
  private LocalDateTime startTime; 
  private LocalDateTime endTime; 
  // 게터와 세터를 명시한다.
}
```

### 임베디드 타입과 테이블 매핑
* **복합 값 타입은 엔티티의 값 중 하나일 뿐이므로, 복합 값 타입을 도입하기 전과 후의 테이블 매핑 구조는 달라지지 않는다**.
* 객체와 테이블을 아주 세밀하게 매핑할 수 있으며, 잘 설계된 ORM 애플리케이션은 매핑 대상 테이블 수보다 매핑이 될 클래스의 수가 더 많은 특징을 갖는다.
  * 복합 값 타입을 잘 활용하면 공통으로 사용될 값들을 하나로 모으는 데에 그치지 않고, 유용한 메소드를 추가로 정의할 수도 있다.
  * 또한, **엔티티가 매핑을 위해 많은 속성을 필요로 한다고 해도 코드 상에서는 명시적인 이름을 갖는 클래스의 형태를 갖기에 가독성과 유지보수성도 높아진다**.
  * **실무에서는 값 타입을 찍어내듯이 만들지는 않지만, 적절하게 생성된 값 타입은 용어와 코드를 공통화하는 큰 장점이 존재**한다.
* 이 때, **복합 값 타입 자체가 null로 초기화된 경우 매핑된 모든 컬럼의 값 역시 null**이 된다.
  * 예를 들어, 회원 엔티티의 주소를 의미하는 복합 값 타입인 `private Address address;` 필드 자체가 null로 초기화된 경우를 의미한다.

### 임베디드 타입과 연관 관계
* **복합 값 타입은 다른 복합 값 타입을 포함하거나, 심지어는 엔티티를 포함할 수도 있다**.
  * 이 경우, 사실 복합 값 타입 내부적으로 포함할 엔티티에 대한 FK 값만을 포함하는 것이므로 생각보다 어렵지 않게 구현이 가능하다.

### @AttributeOverride 어노테이션
* 하나의 엔티티에서 동일한 값 타입을 사용하는 경우, 관계형 데이터베이스의 테이블의 컬럼 명은 중복될 가능성이 존재한다.
  * 예를 들어, 주소 정보를 Address라는 복합 값 타입으로 추출하여 정의하였으나 회원 엔티티가 집 주소와 회사 주소를 동시에 갖게 될 가능성이 있다.
* 따라서 **@AttributeOverrides 또는 @AttributeOverride 어노테이션을 명시하여 컬럼 명 속성을 재정의**할 수 있다.
  * 이 경우, 재정의 대상 컬럼이 단 하나일 경우에는 @AttributeOverride 어노테이션만 사용해도 무방하다.
```
public class Member {
  // ...생략
  @Embedded
  private Address home; 
  
  @Embedded
  @AttributeOverrides(
    @AttributeOverride(name = "city", column = @Column(name = "work_city"),
    @AttributeOverride(name = "street", column = @Column(name = "work_street"),
    @AttributeOverride(name = "zipcode", column = @Column(name = "work_zipcode")
  )
  private Address work;
  // ...생략
}
```
* 그러나 **실무에서는 해당 어노테이션이 잘 사용되지는 않는다**.