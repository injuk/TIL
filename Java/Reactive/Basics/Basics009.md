# Reactive
## 2024-01-22 Mon
### R2DBC란?
```
> R2DBC는 관계형 DB에 리액티브 프로그래밍 API를 제공하기 위한 스펙이자, SPI에 해당한다.
> 기존에는 몇몇 NoSQL만 리액티브 클라이언트를 제공했으나, R2DBC로 인해 관계형 DB를 사용하는 완전한 Non-Blocking 애플리케이션의 구현이 가능해졌다.
```
* **`JDBC API` 자체가 `Blocking API`이므로, `R2DBC`가 탄생하기 이전에는 완전한 `Non-Blocking` 애플리케이션을 제작하기가 불가능**했다.

## 2024-01-23 Tue
### Spring Data R2DBC란?
* Spring Data `R2DBC`는 `R2DBC` 기반의 리포지토리를 더욱 쉽게 구현할 수 있도록 지원하는, Spring Data Family 프로젝트의 일부이다.
  * 때문에 Spring Data `R2DBC`는 Spring이 추구하는 여러 추상화 기법을 적용하며, 데이터 액세스 계층의 반복적인 코드를 크게 줄일 수 있도록 한다.
* 또한, **Spring Data `R2DBC`는 JPA 등의 프레임워크가 제공하는 캐싱 등의 특징이 제거되므로 더욱 단순한 사용이 가능**하다.

## 2024-01-24 Wed
### Spring Data R2DBC 초기 구성하기
* **Spring Data `R2DBC`는 JPA처럼 엔티티에 정의된 매핑 정보로 테이블을 자동 생성하지 않으므로, 테이블 생성용 SQL을 직접 작성할 필요**가 있다.
  * 이후에는 `application.yml` 등의 설정 파일에서 해당 SQL 파일을 조회하여 테이블을 생성하도록 추가적인 설정을 작성해줄 필요가 있다.
* 또한, Spring 기반의 애플리케이션이 `R2DBC`용 리포지토리와 Auditing 기능을 사용할 수 있도록 아래와 같은 어노테이션을 애플리케이션 진입점에 명시한다.
  1. `@EnableR2dbcRepositories`
  2. `@EnableR2dbcAuditing`

## 2024-01-25 Thu
### Spring Data R2DBC 리포지토리 정의하기
* Spring Data `R2DBC`는 여타 Spring Data Family 프로젝트와 마찬가지로, Spring에서 추상화한 데이터 액세스 기술을 쉽게 사용할 수 있도록 지원한다.
  * 이렇듯 **데이터 액세스 계층을 손쉽게 사용할 수 있도록 지원되는 개념이 `Repository` 인터페이스에 해당**한다.
* 이러한 Spring Data가 제공하는 리포지토리를 활용하여 사용자 정의 리포지토리를 정의하는 경우, 다음과 같은 코드를 작성할 수 있다.
```Java
public interface MyEntityRepository extends ReactiveCrudRepository<MyEntity, Long> {
    Mono<MyEntity> findByName(String name);
}
```
* 예를 들어 `MyEntity`에 대한 데이터 영속화를 활용하는 리포지토리는 상술한 바와 같이 작성할 수 있으며, 크게 다음과 같은 구성 요소를 갖는다.
  1. `ReactiveCrudRepository`: `Reactive` 사양을 지원하는 리포지토리 인터페이스에 해당한다.
  2. `Mono<MyEntity>`: **사용자 정의 메소드를 추가적으로 제공할 경우, 각 쿼리는 `Mono` 또는 `Flux` 유형의 반환형**을 갖는다.

## 2024-01-26 Fri
### R2dbcEntityTemplate 활용하기
* Spring Data Family 프로젝트에서 제공되는, 상술한 리포지토리 방식은 널리 알려진 자연스러운 방식에 해당한다.
  * 그러나 Spring Data `R2DBC`의 경우, 리포지토리 방식 이외에도 쿼리 문을 직접 작성하는 것과 유사한 `R2dbcEntityTemplate`을 함께 제공한다.
* 이러한 방식은 **마치 `Query DSL`과 같이 자연스러운 SQL 쿼리문을 작성하는 방식을 도입함으로써 코드의 가독성을 끌어올리도록 지원**한다.
  * 기존의 `JdbcTemplate` 역시 템플릿을 활용하는 것은 같으나, `JdbcTemplate`은 쿼리문을 실제로 템플릿에 전달했다는 점에서 명확한 차이가 있다.
  * 반면, **`R2dbcEntityTemplate`은 엔티티 객체와 쿼리 생성 메소드를 조합하여 템플릿에 전달하는 것으로 DB와 상호작용을 시도**한다.

## 2024-01-27 Sat
### WebFlux의 예외 처리 방식 - onErrorResume()
* Spring MVC에서 자주 사용되는 `@ExceptionHandler`나 `@ControllerAdvice` 방식은 `WebFlux`에서도 적용이 가능하다.
  * 반면, `WebFlux`는 이외에도 `onErrorResume()` 등의 `Operator`를 활용한 예외 처리 방식을 제공한다.
* `onErrorResume() Operator`의 경우, 예외가 발생했을 때 이를 `Downstream`으로 전파하는 대신 다음의 동작을 수행할 수 있도록 지원한다.
  1. 대체 `Publisher`를 통해 예외에 대한 대체 값을 `Emit`할 수 있다.
  2. 또는 예외 이벤트를 래핑한 후, 다시 또 다른 예외 이벤트를 발생시킬 수 있다.
* `onErrorResume() Operator`는 첫 번째 인자로 대응되는 예외 타입을, 두 번째 인자로는 대체 `Publisher Sequence`를 전달받아 동작한다.
  * 반면, 예외 타입 인자를 생략한 경우에는 모든 예외에 대해 활용될 대체 `Publisher Sequence`를 전달받아 동작한다.

## 2024-01-28 Sun
### WebFlux의 예외 처리 방식 - ErrorWebExceptionHandler
* 상술한 **`onErrorResume() Operator`의 경우, 사용법이 간단하지만 모든 `Sequence`에 대해 명시되어야 한다는 단점 역시 존재**한다.
* 이러한 단점을 보완하기 위해 `WebFlux`는 `Global Exception Handler`를 작성할 수 있도록 지원한다.
  * 이를 위해서는 **별도의 `@Configuration` 빈 클래스를 작성하고, 해당 클래스가 `ErrorWebExceptionHandler`를 확장하도록 구현**해주어야 한다.