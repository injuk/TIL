# Basics
## 2022-06-12 Sun

### 일반적인 웹 애플리케이션의 계층 구조
* 웹 애플리케이션은 일반적으로 다음과 같은 구성 요소를 갖는다.
  1. 컨트롤러: 웹 MVC 패턴에서의 컨트롤러 역할을 수행한다.
  2. 도메인: **회원, 주문 등 DB에 저장되어 관리되는 비즈니스 도메인 객체**를 가리킨다.
  3. 서비스: 예를 들어, 회원의 중복 가입 처리 등의 핵심적인 비즈니스 로직을 구현하는 계층이다.
     * 또는 **2.의 비즈니스 도메인 객체를 활용하여 핵심 비즈니스 로직이 동작하도록 구현한 계층**이 서비스이다.
  4. 리포지토리: **DB에 접근하여 도메인 객체를 저장하거나 조회하는 등의 기능을 수행하는 객체**이다.

### 리포지토리에서의 Optional 사용
* findById, findByName과 같은 메소드는 반환되는 결과가 null일 수도 있다.
* 이 경우, **null을 그대로 반환할 수도 있지만 최근에는 Java 8에서 추가된 Optional로 감싸는 방식이 선호**된다.
* 예를 들어, 인터페이스에 다음과 같이 작성한다.
  * findBy~ 메소드들은 null을 그대로 반환해도 무방하지만, 클라이언트 코드에서 여러 처리를 수행할 수 있도록 Optinal 클래스를 사용하는 것이 바람직하다.
```
public interface MemberRepository {
    Member save(Member member);
    Optional<Member> findById(Long id);
    Optional<Member> findByName(String name);
    List<Member> findAll();
}
```
* 이러한 리포지토리의 findBy~ 메소드는 다음과 같이 구현해볼 수 있다.
  * findById의 경우, `Optinal.ofNullable` 메소드를 활용하였다.
  * findByName의 경우, Map을 순회하며 일치하는 객체를 찾기 위해 Stream API를 활용하였다.
  * 이 때, 최종 연산인 findAny의 결과는 Optional이므로 findById 메소드처럼 Optional의 정적 메소드를 사용할 필요가 없다.   
```
@Override
public Optional<Member> findById(Long id) {
    
    return Optional.ofNullable(store.get(id));
}

@Override
public Optional<Member> findByName(String name) {
    // 결과는 Optional로 반환된다.
    // 탐색에 성공한 최초의 객체를 반환하고, 없으면 Optional에 null이 포함되어 반환된다.
    return store.values().stream()
            .filter(member -> member.getName().equals(name))
            .findAny();
}
```

### 테스트 케이스 작성하기
* 개발한 메소드의 동작성을 검증하는 경우, Java의 main 메소드를 활용하거나 컨트롤러가 제공하는 API를 직접 호출할 수 있다.
* 그러나 이러한 방법은 반복적으로 수행하기 어렵고, 모든 메소드를 한 번에 테스트할 수 없다.
  * junit과 같은 테스트 프레임워크는 이러한 문제를 해결하여 손쉽게 메소드를 테스트할 수 있도록 한다.
* 테스트 케이스를 작성하는 경우, 일반적으로 다음의 관례를 따른다.
  1. 테스트 케이스는 `src/test` 디렉토리에 작성하며, 테스트 대상 클래스와 같은 패키지 경로를 구성한다.
  2. **각 테스트 클래스의 이름은 `[테스트대상클래스명]Test.java`의 형태로 작성**한다.
* 테스트 클래스는 public일 필요가 없으며, default여도 무방하지만 테스트 메소드는 public으로 작성한다.
* 각 테스트 케이스에서 동작을 검증하는 로직은 `import static org.assertj.core.api.Assertions.*;`를 통해 아래와 같이 쉽게 표현할 수 있다.
```
@Test
public void save() {
    Member member = new Member();
    member.setName("ingnoh");

    repository.save(member);

    Member result = repository.findById(member.getId())
            .get(); // Optinal의 값은 get 메소드를 통해 빼낼 수 있다.
    assertThat(member).isEqualTo(result); // 동작 결과를 검증한다.
}
```
* 각 테스트 케이스는 메소드 작성 순서대로 호출되지 않으며, 호출 순서를 보장할 수 없다.
  * 때문에 **모든 테스트 메소드는 서로에게 독립적인 상태로 동작할 수 있도록 정의**해야 한다.
* **각 메소드의 독립성을 보장하기 위해 임의의 컬렉션 또는 맵을 테스트 후마다 초기화해야하는 경우, `@AfterEach` 어노테이션을 활용**할 수 있다.
  * 해당 어노테이션이 작성된 메소드는 마치 콜백 메소드처럼 각각의 테스트 케이스 실행 직후에 호출된다.
  * 예를 들어 아래의 afterEach 메소드는 매 테스트 케이스의 직후에 호출되며, 인메모리 Map 객체로 구현된 repository를 비우는 메소드를 호출한다.
  * 해당 테스트 클래스는 인메모리 리포지토리를 테스트하므로, repository 변수 선언에 굳이 인터페이스 형태로 정의할 필요가 없다.
```
MemoryMemberRepository repository = new MemoryMemberRepository();

@AfterEach
public void afterEach() {
    repository.clear();
}
```

### 테스트 주도 개발
* 상술한 방식은 우선 테스트 대상 인터페이스와 구현체를 작성한 후, 이에 맞춘 테스트 클래스를 작성하는 방식이다.
* 그러나 **이러한 순서를 뒤집어 우선 테스트 클래스와 테스트 케이스를 정의하고, 이에 맞추어 테스트 대상 인터페이스와 구현체를 작성**할 수도 있다.
  * 이렇듯 테스트 케이스가 개발 흐름을 주도해가는 방식을 테스트 주도 개발이라고 지칭할 수 있다.
* 테스트 케이스 없이 작성하는 애플리케이션은 개인 프로젝트인 경우에는 상관이 없을지도 모르지만, 실무에서는 필수적인 요소이다.
  * **프로젝트가 성숙해감에 따라 애플리케이션의 규모가 커질수록 테스트 케이스는 큰 역할을 하며, 테스트 케이스 없는 개발은 사실상 불가능**에 가깝다.