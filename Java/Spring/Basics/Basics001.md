# Basics
## 2022-06-10 Fri

### 스프링 학습의 어려움
* 스프링은 실무에서 너무나도 빈번하게 사용되는 프레임워크이지만, 그 기능이 너무 방대하여 학습 과정에서 중도 포기하는 개발자가 많다.
  * 이는 처음부터 너무 이론적인 개념에 매몰되기 때문일 가능성이 크다.
* 정말 중요한 것은 `스프링을 왜 사용하고, 왜 배워야하는가`를 정확히 이해하는 것이다.
* 개발자는 근본적으로 직접 코딩하면서 실제 동작하는 애플리케이션을 만드는 과정에서 가장 빠르게 배울 수 있다.

### 스프링 프로젝트 시작하기
* `https://start.spring.io`를 활용하여 스프링 프로젝트를 쉽게 생성할 수 있다.
  * Spring Boot 항목에서, `SNAPSHOT` 또는 `M1` 등이 명시된 것은 아직 정식 릴리즈 버전이 아닌 경우에 해당한다.
* Project Metadata 항목의 경우, 일반적으로 다음과 같이 작성한다.
  1. Group: 일반적으로 기업 도메인을 작성한다.
     * 개인 사용자의 경우, 아무렇게나 작성해도 무방하다.
  2. Artifact: **빌드된 결과물을 가리켜 아티팩트라고 하며, 일반적으로 프로젝트의 이름으로 사용**된다.

### 프로젝트 구조 살펴보기
* `.idea` 폴더는 IntelliJ가 사용하는 설정 파일이 포함된다.
* `gradle` 폴더는 이름 그대로 그래들을 사용하기 위한 폴더이다.
* `src` 폴더의 하위에는 `main`과 `test` 폴더가 포함된다.
  * **최근에는 모든 프로젝트가 src/main과 src/test를 분리하며, 이는 사실상 표준**이 되었다.
* `main` 폴더는 다시 `java`와 `resources` 폴더로 분류된다.
  * `java` 폴더에는 실제 Java 파일들이 포함된다.
  * **`resources` 폴더에는 설정 파일 및 HTML 문서 등이 포함되며, 사실상 Java 파일을 제외한 모든 파일이 포함**된다.  
* **`build.gradle`의 경우 그래들 동작과 관련된 정보가 명시되며, 이를 토대로 그래들은 버전을 관리하고 라이브러리를 임포트**한다. 
  * 이전에는 `build.gradle`의 내용을 하나하나 작성해야했으나, 최근에는 `https://start.spring.io` 등을 통해 파일을 쉽게 작성할 수 있게 되었다.
* `.gitignore` 파일의 경우, Git에 의해 관리하지 않을 파일들을 명시한다.
  * **Git에는 소스 코드 이외의 파일을 올리지 않는 것이 바람직하며, 빌드 결과물 등을 올리지 않아야 한다**.
  * 이전에는 이러한 파일도 모두 개발자가 하나하나 작성해야했으나, `https://start.spring.io` 등을 통해 쉽게 설정 파일을 작성할 수 있게 되었다.

### build.gradle 살펴보기
* `plugins`의 경우, 애플리케이션 개발 과정에서 반복되는 Task 들을 묶어둔 개념이다.
* `group`의 경우, 일반적으로 기업 도메인 명이 명시된다.
* `version`의 경우, 기본적으로 0.0.1-SNAPSHOT 버전이 명시된다.
* `sourceCompatibility`의 경우, Java 버전이 명시된다.
* `repositories`의 경우, 라이브러리를 `dependencies`에 명시된 라이브러리를 다운로드하기 위한 패키지 관리자를 명시한다.
  * **주로 mavenCentral이라는 공개된 라이브러리 관리 사이트를 사용**하게 된다.
* `dependencies`의 경우, 프로젝트에서 사용할 라이브러리 목록이 명시된다.

### 프로젝트 실행하기
* src/java 디렉토리에는 다음과 같은 Java 파일이 이미 생성되어 있다.
```
@SpringBootApplication
public class HelloSpringApplication {
	public static void main(String[] args) {
		SpringApplication.run(HelloSpringApplication.class, args);
	}
}
```
* 해당 main 메소드를 실행할 경우, SpringBootApplication 애플리케이션이 시작된다.
  * 스프링 부트는 톰캣을 내장하므로, 자체적으로 톰캣을 띄우면서 스프링 부트 애플리케이션을 함께 실행한다. 

### 라이브러리 살펴보기
* 상술한 설정에서 임포트하는 라이브러리 중, 다음과 같은 라이브러리를 눈여겨볼 수 있다.
  1. `spring-boot-starter-web`: 내장 웹 서버인 `spring-boot-starter-tomcat`과 `spring-webmvc`를 포함한다.
  2. `spring-boot-starter-thymeleaf`: 템플릿 엔진인 타임리프를 포함한다.
  3. `spring-boot-starter`: 공통 라이브러리이며, 스프링 부트와 스프링 코어에 로깅 라이브러리를 포함한다.
     * `spring-boot`는 `spring-core`를 포함하여 임포트된다.
     * `spring-boot-starter-logging`은 스프링의 로깅 라이브러리이며, 사실 상의 로깅 표준인 `logback`과 `slf4j`를 포함한다.
  4. `spring-boot-starter-test`: 테스트용 라이브러리이며, 다음의 내용을 포함한다.
     * `junit`는 Java 진영의 테스트 프레임워크이다.
     * `mockito`는 테스트 모킹을 위한 라이브러리이다.
     * `assertj`는 테스트 코드를 더 편리하게 작성할 수 있도록 보조하는 라이브러리이다.
     * `spring-test`는 스프링의 통합 테스트를 지원하는 라이브러리이다.