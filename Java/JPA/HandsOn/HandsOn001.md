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