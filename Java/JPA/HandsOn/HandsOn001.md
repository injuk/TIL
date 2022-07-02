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