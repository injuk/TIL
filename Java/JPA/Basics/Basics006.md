# Basics
## 2022-07-21 Thu

### 상속 관계 매핑
* **관계형 데이터베이스는 구조적으로 상속 관계를 제공하지 않는다**.
    * 다만, 슈퍼타입과 서브타입 관계라는 모델링 기법이 객체의 상속과 유사성을 갖는다.
* **상속 관계 매핑이란, 객체의 상속 개념 및 구조를 데이터베이스의 슈퍼타입과 서브타입 관계라는 모델링 기법으로 매핑하는 것**이다.
* 이러한 상속 관계 매핑을 위해 슈퍼타입 및 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법은 크게 다음과 같이 구분된다.
    1. 조인 전략: 각각 별도의 테이블로 변환한다.
    2. 단일 테이블 전략: 하나의 통합 테이블에 모든 속성을 포함시킨다.
    3. 구현 클래스 별 테이블 전략: 서브타입 별로 테이블로 매핑하며, 공통 속성을 갖는 공통 테이블을 두지 않는다.
* 이렇듯 **논리 모델과 물리 모델을 구현하는 방식은 세 가지로 나뉘지만, 이를 사용하는 객체의 입장에서는 세 방식 모두 다를 바가 없다**.
    * 즉, 데이터베이스가 실제로 어떤 방식을 택하더라도 JPA에서 매핑이 가능하다.
* **JPA는 기본적으로 단일 테이블 전략을 적용하며, 상속 관계에 있는 모든 엔티티의 속성을 하나로 통합한 테이블을 정의**한다.
* **상속 계층에서 부모 엔티티를 단독으로 사용하지 않는 경우, 부모 엔티티는 추상 클래스로 정의**되어야 한다.

### 상속 관계 매핑과 어노테이션
* 상속 관계 매핑을 위해 `@Inheritance(strategy = InheritanceType.XXX)` 어노테이션을 사용하며, XXX에는 다음과 같은 속성이 명시될 수 있다.
    * JOINED: 조인 전략을 사용한다.
    * SINGLE_TABLE: 단일 테이블 전략을 사용한다.
    * TABLE_PER_CLASS: 구현 클래스 별 테이블 전략을 사용한다.
* `@DiscriminatorColumn(name = "DTYPE")` 어노테이션은 공통 테이블의 각 row가 어떤 엔티티로부터 영속화되었는지를 명시하는 컬럼을 의미한다.
    * 기본적으로 DTYPE이라는 명칭의 컬럼으로 적용되며, 원하는 경우 name 속성을 통해 컬럼 명을 변경할 수 있다.
    * 조인 전략에서는 해당 컬럼이 없더라도 조회가 불가능한 것은 아니지만, 단일 테이블 전략의 경우 해당 컬럼은 필수적이다.
* 또한, **상술한 모든 어노테이션은 상속 관계에서 부모 클래스의 위치에 명시**한다.
* 반면, `@DiscriminatorValue("XXX")` 어노테이션은 데이터베이스의 DTYPE 컬럼에 영속화될 값을 의미하며 자식 엔티티에 명시한다.
    * 예를 들어, 이를 명시하지 않은 경우 기본적으로 엔티티의 클래스 명으로 적용된다.
    * 사내 데이터베이스 규정에 따라 이를 변경하고자 하는 경우에 해당 어노테이션을 적용할 수 있다.
* 예를 들어 Item 엔티티의 자식 엔티티로 Album, Movie 엔티티가 존재하는 경우 DTYPE은 기본적으로 엔티티 클래스 명과 동일하게 적용된다.
    * 이를 Album은 A, Movie는 M 등의 글자로 축약하고 싶은 경우에 `@DiscriminatorValue` 어노테이션을 활용할 수 있다.

### 단일 테이블 전략
* 조인 전략이 필요 이상으로 복잡하게 느껴지고, 프로젝트 자체가 단순한 경우에는 단일 테이블 전략을 적용할 수 있다.
    * 이는 하나의 테이블에 모든 속성을 명시하되, DTYPE을 통해 엔티티를 구분하는 방식이다.
    * 이러한 **단순함으로 인해 삽입과 조회가 모두 한 번에 가능한 단일 테이블 전략은 다른 전략에 비해 가장 높은 성능**을 보인다.
* 조인 전략의 경우, `@DiscriminatorColumn` 어노테이션이 엔티티에 명시되지 않았다면 DTYPE 컬럼은 추가되지 않는다.
* 반면, 단일 테이블 전략의 경우 해당 어노테이션이 명시되지 않았더라도 DTYPE 컬럼이 자동으로 생성된다.
    * 이는 단일 테이블 전략에서 DTYPE 컬럼이 존재하지 않는 경우, 각 row가 어떤 엔티티인지 알 방법이 전무하기 때문이다.

### 구현 클래스 별 테이블 전략
* 해당 전략은 공통된 속성을 갖는 부모 엔티티의 테이블을 사용하지 않으며, 각 자식 엔티티 별로 하나의 테이블을 만드는데에 그친다.
* 해당 전략의 경우, 각 row를 자식 엔티티 별로 구분할 필요가 없으므로 `@DiscriminatorColumn` 어노테이션을 명시하더라도 DTYPE 컬럼이 생성되지 않는다.
* **해당 전략은 값을 삽입할 때에는 효율이 좋지만, 값을 조회하는 경우에는 테이블의 모든 값을 확인해야하므로 성능 상 문제점이 존재**한다.
    * 예를 들어 ID로 조회하는 경우, 공통 속성을 갖는 엔티티가 별도로 존재하지 않으므로 자식 엔티티를 모두 조회해야 한다.
    * 상술한 Item의 예시로 들어, 코드 상에서 Item Id로 조회하더라도 Album과 Movie 테이블을 모두 조회해야 한다.

### 조인 전략의 장점과 단점
* 테이블이 가장 정규화되고, FK 참조 무결정 제약 조건을 활용할 수 있다.
    * 또한 저장 공간 역시 효율적으로 사용할 수 있다.
* 반면 **데이터 조회시 JOIN을 자주 사용하게 되므로 상대적으로 조회 성능이 떨어지고, 조회 쿼리가 복잡**해진다.
    * 또한 테이블이 정규화되어 있으므로 데이터를 영속화하는 과정에서 INSERT 쿼리가 두 번 요청된다.
* 그러나 **구조의 복잡성을 제외한 항목들은 실제로 치명적인 영향을 주는 정도의 단점들은 아니다**.
* **조인 전략은 정규화 측면에서도 강점이 있고, 객체에도 잘 맞을 뿐만 아니라 설계 자체도 깔끔하게 떨어지는 경우가 많은 정석적인 전략**이다.

### 단일 테이블 전략의 장점과 단점
* 해당 전략은 조인이 필요 없으므로 일반적인 조회 성능이 빠르고, 쿼리 역시 단순하다.
* 반면 **자식 엔티티가 매핑한 컬럼은 모두 nullable 이어야 하며, 테이블이 커질수록 많은 데이터가 저장되어 오히려 조회 성능이 떨어질 수 있다**.
    * 성능이 떨어질 정도로 테이블이 커지는 상황은 자주 일어나지 않지만, 많은 컬럼을 null 허용으로 만들기에 데이터 무결성 측면에서 치명적인 단점이 존재한다.

### 구현 클래스 별 테이블 전략의 장점과 단점
* 해당 전략은 서브 타입을 명확하게 구분하여 처리할 수 있고, not null 제약 조건을 설정할 수 있다.
* 그러나 여러 자식 테이블을 함께 조회하는 경우 UNION을 사용하므로 성능이 떨어지며, 자식 테이블을 통합하는 쿼리 역시 어려워진다는 치명적인 단점이 존재한다.
    * 이러한 이유에서 해당 전략은 시스템의 수정 사항에 매우 취약하다.
* **해당 전략은 데이터베이스 설계자와 ORM 개발자 모두가 추천하지 않는 방식이며, 실무에서 사용하지 않는 것이 바람직**하다.

### 상속 관계 매핑 전략 선택 기준
* 기본적으로는 조인 전략을 기본으로 선택하되, 요구 사항에 따라 단일 테이블 전략의 장 단점을 비교하여 결정한다.
    * 예를 들어 **요구 사항 자체가 너무 단순하고 데이터 역시 양이 많지 않으며, 확장 가능성이 없는 경우에는 단일 테이블 전략을 선택**할 수 있다.
    * 반면, 비즈니스적으로 복잡하고 중요한 요구 사항은 조인 전략을 선택할 수 있다.
    * 이렇듯 두 전략은 장점과 단점 측면에서 tradeoff가 분명한 편에 속하며, 선택이 어려운 경우에는 DBA와 소통을 통해 전략을 결정할 수도 있다.
* 구현 클래스 별 테이블 전략은 고려하지 않도록 한다.

## 2022-07-22 Fri
### @MappedSuperclass란?
* 해당 어노테이션은 id, name 등 공통 매핑 정보가 필요할 때 사용할 수 있다.
  * 즉, **상속 관계 매핑과는 무관하게 사용되는 어노테이션**이다.
* 예를 들어, 많은 엔티티에서 id와 name 필드가 반복되는 시나리오를 생각해볼 수 있다.
  * 이 경우 각 엔티티는 부모 자식과 같은 동일한 상속 계층에 위치하지 않을 수 있지만, 코드의 중복은 계속해서 늘어만 간다.
  * 이 때 공통 속성을 갖는 모든 엔티티가 단일 클래스로부터 해당 공통 속성을 상속받도록 구현할 수 있다.
  * 즉, **공통된 속성을 갖는 BaseEntity를 두고 공통 속성을 사용하는 모든 엔티티가 이를 상속**받는다.
* 이 경우 **관계형 데이터베이스에 위치한 테이블의 형태가 완전히 다르지만, 순전히 객체의 사용성과 유지보수성을 높이기 위해 적용해볼 수 있는 기능**이다.
* 이렇듯 @MappedSuperclass 어노테이션이 명시된 클래스는 공통된 매핑 정보만을 갖는 클래스로서 기능한다.

### @MappedSuperclass 어노테이션 사용하기
* 공통 속성을 갖는 구체 클래스를 우선 정의한 후 해당 어노테이션을 명시한다.
  * 이 때, 게터와 세터 메소드를 함께 정의할 수 있다.
```
@Getter @Setter
@MappedSuperclass
public class BaseEntity {
  private String createdBy;
  private String lastModifiedBy;
  private LocalDateTime createdAt;
  private LocalDateTime lastModifiedAt;
}
```
* BaseEntity가 갖는 속성들을 공통으로 사용할 모든 엔티티에서 해당 클래스를 상속하는 것으로 구현은 완료된다.
```
@Entity
public class Member extends BaseEntity {
  // 구현
}
```
* BaseEntity로부터 공통 속성으로 사용될 필드들 역시 `@Column(name = "컬럼 명")` 어노테이션을 명시하여 대상 컬럼 명을 변경할 수 있다.
* **해당 기능은 실무 운영을 위해 매우 유용하게 사용되며, 정말 불변인 테이블을 제외하고는 모두 적용하는 것이 권장**된다.

### @MappedSuperclass의 특징
* **해당 어노테이션은 실제 테이블과는 관계가 없으며, 단순히 엔티티들이 공통으로 사용하는 매핑 정보를 모으기 위한 역할을 수행**한다.
  * 주로 등록자, 수정자, 등록일, 수정일 등과 같은 전체 엔티티에서 공통으로 사용할 정보를 모아두기 위해 사용한다.
* 해당 어노테이션은 상속 관계 매핑이 아니며, 관계도 없는 전혀 다른 기능이다.
* **해당 어노테이션이 명시된 클래스는 실제로는 엔티티가 아니므로, 관계형 데이터베이스에 테이블도 생성되지 않는다**.
  * **엔티티가 아니므로, 엔티티 매니저를 활용한 자식 타입의 엔티티를 조회하거나 검색하는 것 역시 불가능**하다.
  * 반면, **상속 관계 매핑의 경우 완전히 별도의 기능이므로 부모 타입으로 자식 타입의 엔티티를 조회하는 것이 가능**하다.
* 해당 클래스의 용도는 자신을 상속 받는 자식 클래스에 매핑 정보만을 제공하는 것이며, 직접 생성해서 사용할 일은 없다.
  * 때문에 **@MappedSuperclass를 명시할 클래스는 추상 클래스로 정의하는 것이 권장**된다.
* **@Entity 어노테이션이 명시된 엔티티 클래스는 같은 엔티티, 또는 @MappedSuperclass 어노테이션이 명시된 클래스만 상속이 가능**하다.