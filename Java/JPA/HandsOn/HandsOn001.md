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