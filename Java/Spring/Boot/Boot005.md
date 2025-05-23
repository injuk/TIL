# Spring Boot
## 2025-04-15 Tue
### 외부 파일 고려하기
> 상술한 모든 방식은 편리하지만, 필요한 값이 늘어날수록 사용이 번거로워지는 한계를 갖는다.
* 예를 들어, 실무의 경우 한 애플리케이션에 필요한 설정 값이 수십개가 될 수도 있으나 이를 모두 애플리케이션 실행 과정에서 전달하기에는 번거로울 수 있다.
* 대안으로 **외부 설정 값들을 파일 형태로 관리하는 방식이 고안되었으며, 해당 파일은 애플리케이션 실행 시점에 조회되어 적용**된다.
  * 예를 들어 아래와 같은 `.properties` 파일은 키-값 쌍의 설정 값을 관리하기에 매우 편리한 방식에 해당한다.
  * 이 때, 해당 파일은 소스 코드 내에 포함되지 않으며 코드 외부의 서버에 저장되어야 유연한 관리가 가능하다.
```properties
# application.properties
url=dev-db.injuk.ga
username=injuk
password=pw
schema=my_schema
```
* 이 때, 스프링 진영에서는 이러한 외부 설정용 파일을 설정 데이터(`Config Data`)라는 용어로 지칭한다.

## 2025-04-16 Wed
### 스프링과 외부 설정 파일
* 언뜻 상술한 제안에 따라 외부 파일을 설정 용도로 사용하는 경우, 이를 읽어들여 설정으로 적용할 수 있도록 프로그래밍적인 개발이 필요할 것처럼 보일 수 있다.
* 그러나 **스프링 부트는 이미 이러한 요구 사항에 맞는 구체 클래스를 개발해두었으므로, 개발자는 그저 적절한 위치에 외부 파일을 작성**만 하면 된다.
  * 이 경우 **스프링은 해당 파일을 조회하여 사용할 수 있도록하는 `PropertySource` 구현체를 제공**한다.
  * 물론, 이러한 **설정 데이터를 위한 `PropertySource` 역시 `Environment` 클래스에 의해 조회가 가능**하다.
* 추가적으로, 스프링 부트는 `.properties` 파일 이외에도 `.yaml` 외부 설정 파일을 지원하며 이 역시도 적절한 `PropertySource`에 의해 처리된다.

## 2025-04-17 Thu
### 외부 파일을 활용한 설정 정보 주입하기
* 상술한 내용은 다음과 같은 간단한 과정을 통해 확인할 수 있다.
  1. IntelliJ를 기준으로 했을 때, 임의의 스프링 부트 애플리케이션에서 `./gradlew clean build` 명령어를 통해 JAR 파일을 빌드한다.
  2. `build/libs` 위치로 이동한 후 `application.properties` 파일을 생성하고 키-값 쌍 형태의 설정 정보를 입력한다.
  3. `java -jar [애플리케이션명].jar` 명령어를 통해 애플리케이션을 실행할 경우, `.properties` 파일에 작성한 설정 정보가 적용된다.
* 이렇듯 스프링 부트 기반의 JAR 파일은 외부 설정 파일을 자동으로 조회하므로, 환경에 따라 설정 정보를 달리 관리하는 것으로 요구 사항을 충족할 수 있다.

## 2025-04-18 Fri
### 외부 파일을 활용한 설정 정보 주입 방식의 한계
* 반면, **외부 파일을 활용한 외부 설정값 주입 방식 역시 편리하나 다음과 같이 외부 설정 파일 관리 측면에서의 명확한 한계점**을 갖는다.
  1. 외부 설정을 별도 파일로 관리하게 되므로, 설정 파일 자체를 별도로 관리해야 한다.
  2. 서버의 개수가 늘어날수록 관리해야 할 설정 파일의 개수도 함께 늘어난다.
  3. 설정 파일은 별도로 관리되므로, 설정 값 자체의 변경 이력을 관리하기 어렵다.
* 단적으로 말했을 때, **상술한 단점들은 외부 설정을 위해 작성한 설정 파일이 애플리케이션 외부에 위치하는 데에서 기인**한다.
  * 중요한 것은 외부 설정 파일 작성 방식이 단점만 갖는 것이 아니며, 필요에 따라 사용을 고려할 수 있는 대안이라는 점을 인식하는 데에 있다.

## 2025-04-19 Sat
### 설정 데이터를 내부 파일로 관리하기
* 상술했듯, 설정 파일을 외부로 관리하는 방식은 설정 파일 자체적인 관리 측면에서 많은 번거로움을 수반한다는 명확한 한계를 갖는다.
* 이를 해결하기 위해서는 **설정 파일을 애플리케이션 내부에 포함하는 것으로, 애플리케이션이 빌드될 때 함께 빌드하는 방식을 고려**해볼 수 있다.
  * 다시 말해 **JAR 파일 자체에 설정 데이터를 포함하는 것으로, 이를 통해 애플리케이션 배포 시점에 설정 파일의 변경 사항도 함께 배포하는 것이 가능**하다.
* 예를 들어 개발 환경에서는 `application-dev.properties` 파일을, 운영 환경에서는 `application-prod.properties` 파일을 조회할 수 있다.
  * 이 경우, 두 파일은 각 서버 별로 위치하는 것이 아닌 모두 하나의 JAR 파일 내부에 위치하여 함께 배포된다.
* 물론 이러한 방식을 적용하는 경우에는 **실행 시점에 어떤 파일의 정보를 조회할지를 구분할 수 있도록 하는 구분 값이 추가될 필요**가 있다.
  * 예를 들어, `application-dev.properties` 파일을 조회하기 위해서는 이를 알릴 수 있는 별도의 설정 값이 필요하다.
  * 이를 위해 `profile`을 사용할 수 있으며, 예를 들어 `profile=dev` 설정을 사용할 경우 `application-dev.properties` 파일이 조회된다.
  * 이렇듯 **스프링은 이미 설정 데이터를 외부로 분리하고, 외부로부터 전달된 설정 값 중 `profile` 정보에 따라 각 파일을 조회하는 기능을 제공**한다.

## 2025-04-20 Sun
### 프로필이란?
* 스프링은 상술한 `환경 별로 어떤 설정 파일을 조회할지 결정하는 구분 값`을 `profile`이라는 개념으로 제공한다.
* 예를 들어, **`spring.profiles.active`라는 키를 갖는 외부 설정에 값을 작성할 경우 해당 프로필을 사용하는 것으로 판단**한다.
* 또한, `application-{profile}.properties` 형태의 규칙으로 작성된 설정 데이터는 각각 대응되는 프로필에 맞추어 조회한다.
  * 예를 들어 `spring.profiles.active=dev`가 작성된 경우 `application-dev.properties` 설정 데이터가 조회된다.
* 이 때, **`profile` 정보는 운영 체제의 환경 변수나 Java 시스템 속성 또는 커맨드 라인 옵션 인자 형태로 애플리케이션에 적용**될 수 있다.
  * 예를 들어, `-Dspring.profiles.active=dev` 형태의 Java 시스템 속성으로 `profile` 정보를 애플리케이션에 전달할 수 있다.

## 2025-04-21 Mon
### 내부 파일을 활용한 설정 정보 주입 방식의 한계
* 상술한 방식을 활용할 경우, 설정 데이터가 애플리케이션 내부 파일로 관리되므로 배포 수명 주기에 맞추는 등 편의성이 증대되는 효과를 얻을 수 있다.
* 반면, 해당 방식의 경우 **각 설정 파일이 환경 별로 별도의 파일로 작성되므로 이를 한 눈에 파악하기 어려운 한계점 역시 존재**한다.
  * 예를 들어 애플리케이션이 배포되는 파일이 많아질수록 설정 데이터 역시 많아지므로 환경 별 설정 데이터 파악은 점차 어려워질 수 있다.
* 때문에 **스프링은 이러한 불편함을 해소하기 위해 단일 물리 설정 파일에서 논리적인 영역을 분리하는 방법을 제공**한다.

## 2025-04-22 Tue
### 내부 파일을 단일 물리 파일로 관리하기
* 상술한 바와 같이, 설정 파일을 여러 파일로 분리할 경우 가독성이 떨어질 수 있기에 스프링은 단일 물리 파일에 논리적인 영역을 나누는 방식을 지원한다.
* 예를 들어 개발 환경과 운영 환경에 `application-[환경].properties` 형태의 설정 파일을 관리한 경우, 다음과 같이 하나의 물리 파일로 병합할 수 있다.
  * 이 때, **파일 명은 `application.properties`로 명명하며 논리적인 환경 별 설정 구획을 `#---` 또는 `!---` 문자로 구분**한다.
  * 반면, `application.yaml`을 사용하는 경우 논리적인 환경 별 설정 구획은 `---` 문자로 구분한다.
```properties
spring.config.activate.on-profile=dev
url=dev.com
username=dev_user
password=dev_pw
#---
spring.config.activate.on-profile=prod
url=prod.com
username=prod_user
password=prod_pw
```
* 상술한 설정 파일의 경우 하나의 물리 파일로 작성되지만 내부적인 구분 문자에 따라 두 논리 영역으로 분리되며, `profile` 정보에 따라 다른 설정을 적용한다.
  * 예를 들어, `dev` 프로필이 활성화된 경우 구분 문자의 위 쪽에 위치한 설정 정보가 적용된다.
  * 이렇듯 **프로필에 따라 논리적으로 구분된 설정 데이터를 활성화하기 위해서는 `spring.config.activate.on-profile`을 명시**해주어야 한다.
  * 물론, 프로필을 의미하는 설정인 `spring.profiles.active`는 Java 시스템 속성이나 커맨드 라인 옵션 인자 중 어떤 방식을 사용하여 전달해도 무방하다.
* 이 때, **`#---`과 같은 구분 문자의 앞 뒤로는 `#`과 같은 문자열로 시작하는 주석이 작성되지 않아야 하는 점에 주의**를 기울여야 한다.

## 2025-04-23 Wed
### default 프로필이란?
* 상술한 바와 같이 **단일 파일에 여러 논리 영역을 구분했지만, 정작 `profile` 정보를 애플리케이션에 전달하지 않은 경우 값은 `default`로 설정**된다.
  * 즉, `--spring.profiles.active=dev`와 같은 설정이 누락되어 애플리케이션에 전달되지 않은 경우를 의미한다.
  * 또한, **`default`는 적절한 설정 값이 없어 프로필이 명시되지 않은 경우에 스프링이 기본적으로 활성화하는 프로필에 해당**한다.
* 이 경우, **`url`과 `username` 및 `password` 등의 값은 이를 설정하기 위한 프로필 정보가 없으므로 모두 `null`로 설정**된다.

## 2025-04-24 Thu
### 내부 파일을 활용하는 경우의 설정 정보의 기본 값 설정하기
> 스프링은 단순히 설정 파일을 위에서 아래로 읽어내려가며 필요한 설정 데이터를 적용한다.
* 예를 들어 로컬 환경에서 개발하는 경우, 이를 위한 별도의 논리 영역을 설정하는 것은 다소 번거로운 작업이 될 수 있다.
  * 또는 여러 환경에 대해 임의의 설정 값을 동일하게 적용하고자 하는 경우가 있을 수 있다.
* **스프링은 이를 위해 임의의 설정에 활성 프로필과 무관한 기본 값을 적용하는 방법을 제공하며, 이를 위해 다음과 같은 설정 파일을 작성**해볼 수 있다.
```properties
url=local.com
username=local_user
password=local_pw
#---
spring.config.activate.on-profile=dev
url=dev.com
username=dev_user
password=dev_pw
#---
spring.config.activate.on-profile=prod
url=prod.com
username=prod_user
password=prod_pw
```
* 스프링은 내부 설정 파일을 위에서부터 아래로 순차적으로 읽어들여 적용하는 반면, 논리 영역을 활성화하기 위한 프로필 정보가 없는 경우 최상단의 값이 적용된다.
  * 이렇듯 **프로필과 무관하게 항상 적용되는 설정 데이터를 기본 값이라는 용어로 지칭**한다.
  * **스프링의 설정 값 우선 순위 역시 이러한 방식을 사용하며, 이는 우선 기본 값을 적용하되 필요에 따라 각 설정 데이터를 덮어쓰는 것처럼 보이는 방식**이다.
* 같은 논리로, `--spring.profiles.active=dev`와 같이 활성 프로필을 명시한 경우에는 다음과 같은 순서에 의해 적절한 설정 정보가 적용된다.
  1. 설정 파일을 상단부터 조회하므로, 우선 기본 값을 적용한다.
  2. 그 다음 논리 영역을 조회하는 과정에서, 활성 프로필 정보가 명시되었으므로 `spring.config.activate.on-profile=dev`에 의해 값은 덮어씌워진다.
  3. 그 다음 논리 영역을 조회하는 과정에서, 명시된 정보와 활성 프로필 정보가 다르므로 무시하여 값이 덮어씌워지지 않는다.
* 덧붙여, `spring.profiles.active`에서 알 수 있듯 `--spring.profiles.active=dev,prod` 형태로 복수의 프로필을 적용할 수도 있다.
  * 물론, 실무에서는 잘 사용되지 않는 방식이며 일반적으로 설정 파일은 기본 값을 적용한 후 환경 별로 다른 값을 사용하는 설정들만 명시하게 된다.

## 2025-04-25 Fri
### 외부 설정 데이터 적용 우선 순위
* 스프링은 동일한 애플리케이션 코드를 유지하면서도 환경 별 동작을 달리할 수 있도록 여러 외부 설정 방식을 지원하며, 이들은 다음과 같은 우선 순위를 갖는다.
  1. 내부 설정 파일
     1. JAR 파일 내부의 `application.properties` 설정 파일
     2. JAR 파일 내부의 활성 프로필에 따른 `application-[환경].properties` 설정 파일
     3. JAR 파일 외부의 `application.properties` 설정 파일
     4. JAR 파일 외부의 활성 프로필에 따른 `application-[환경].properties` 설정 파일
  2. 운영 체제 환경 변수
  3. Java 시스템 속성
  4. 커맨드 라인 옵션 인자
  5. `@TestPropertySource` 어노테이션
* 이들은 **위에서부터 아래로 더 높은 우선 순위를 갖게 되며, 예를 들어 내부 설정 파일의 정보는 커맨드 라인 옵션 인자에 전달된 동일한 설정에 덮어씌워진다**.

## 2025-04-26 Sat
### 외부 설정 데이터 적용 우선 순위 이해하기
* 앞서 다룬 바와 같이, 외부 설정 데이터의 적용 우선 순위는 다음과 같은 상식적인 원칙에 따라 이해할 수 있다.
  1. 더 유연한 것이 우선 순위가 높다.
  2. 더 좁은 범위에 적용되는 것이 더 높은 우선 순위를 갖는다.
* 첫 번째 원칙에 따라, JAR 파일을 다시 빌드해야하는 내부 설정 파일 방식보다 실행시 값을 전달할 수 있는 Java 시스템 속성의 우선 순위가 더 높다.
  * 즉, **Java 시스템 속성에 의한 외부 설정 데이터 전달은 내부 설정 파일 방식보다 유연**하다.
* 두 번째 원칙에 따라, 운영 체제의 환경 변수보다 더 좁은 범위에 적용되는 Java 시스템 속성의 우선 순위가 높다.
  * 같은 원리로, JVM 전체적으로 적용되는 Java 시스템 속성보다 임의의 애플리케이션 하나에만 적용되는 커맨드 라인 옵션 인자의 우선 순위가 더 높다.

## 2025-04-27 Sun
### Environment 클래스의 관점에서 본 외부 설정 데이터의 병합
* `Environment`를 통해 외부 설정 데이터를 조회하는 관점에서, 각 설정 데이터들은 마치 계속 추가되거나 우선 순위에 따라 값을 덮어쓰는 것처럼 보인다.
  * 물론 실제로 값을 덮어쓰는 방식으로 동작하는 것은 아니며, 어디까지나 정의된 우선 순위에 따라 값을 적용하게 된다.
* 이로 인해 내부 설정 파일에 `a=b`를 명시하고, Java 시스템 속성으로 `-Dc=d`를 전달했다면 `Environment`에서는 `a`와 `c`를 모두 조회할 수 있다.
  * 반면, `-Da=e`를 추가로 전달했다면 우선 순위에 따라 내부 설정 파일에 명시된 `a` 설정의 값이 `e`로 덮어씌워진다.
* 즉, **`Environment`클래스를 사용하는 입장에서는 모든 외부 설정 데이터가 하나로 병합되며, 동일한 설정은 우선 순위에 따라 덮어씌워지는 것처럼 보인다**.

## 2025-04-28 Mon
### 실무에서의 외부 설정 데이터 활용
* 실무 관점에서는 주로 `application.properties`와 같은 내부 설정 파일을 사용하되, 필요에 따라 Java 시스템 속성과 커맨드 라인 옵션 인자를 혼용한다.
  * 또는 특수한 환경에서는 JAR 외부 설정 파일을 적용하는 것을 고려할 수도 있다.
* 중요한것은 우선 순위에 따라 설정 값이 병합되거나 덮어씌워지는 것처럼 보인다는 특징이며, 이러한 방식은 개발자에게 있어 편리하면서도 유연한 구조를 제공한다.

## 2025-04-29 Tue
### 스프링이 제공하는 다양한 외부 설정 조회 방식
* 외부 설정 파일 및 운영체제 환경 변수, Java 시스템 속성 등은 앞서 다루었던 `Environment` 클래스를 활용하여 일관된 방식으로 조회 가능하다.
* 나아가, 스프링은 `Environment` 클래스를 활용하여 개발자로 하여금 더욱 편리하게 외부 설정을 조회할 수 있도록 지원하는 방법을 다음과 같이 제공한다.
  1. `Environment` 클래스
  2. `@Value` 어노테이션을 활용한 설정 데이터 주입
  3. `@ConfigurationProperties` 어노테이션을 활용한 타입 안전 데이터 주입

## 2025-04-30 Wed
### Environment 클래스를 활용한 외부 설정 데이터 주입
* 예를 들어 `Environment` 클래스를 활용하여 외부 설정 데이터를 임의의 빈 클래스에 주입하는 경우, 다음과 같은 `@Configuration`을 작성할 수 있다.
  * 이 때, 필요한 설정 데이터는 모두 `application.properties` 파일에 작성되어 있다고 가정한다.
```java
@Configuration
public class EnvironmentConfiguration {
    private final Environment env;
    
    // Environment 클래스는 별도의 작업 없이도 스프링에 의해 빈으로 등록되므로 자동으로 주입받을 수 있다.
    public EnvironmentConfiguration(Environment env) {
        this.env = env;
    }
    
    @Bean
    public MyDataSource myDataSource() {
        String url = env.getProperty("my.datasource.url");
        int maxConnection = env.getProperty("my.datasource.etc.max-connection", Integer.class); // 타입 정보를 전달할 수 있다.
        Duration timeout = env.getProperty("my.datasource.etc.timeout", Duration.class); // 5000ms와 같이 입력한 데이터는 Duration 클래스로 매핑이 가능하다.
        List<String> options = env.getProperty("my.datasource.etc.options", List.class); // 쉼표로 구분된 데이터는 List로 매핑이 가능하다.
      
        return new MyDataSource(url, maxConnection, timeout, options); 
    }
}
```
* `MyDataSource`라는 별도의 빈 클래스를 정의했다고 했을 때, 상술한 바와 같이 `Environment` 클래스를 활용하여 일관성 있는 외부 설정 조회가 가능하다.
  * 예를 들어, **외부 설정이 어떠한 방식으로 주입되었건 간에 `Environment` 클래스만으로 접근이 가능**하다.
* 또한, **`Environment.getProperty([설정 키], [클래스]);` 형태의 메소드를 호출하는 것으로 임의의 설정을 원하는 타입으로 변환하는 것이 가능**하다.
  * 이 경우, **내부적으로 사전 정의된 다양한 타입들에 대해 기본적으로 적용되는 스프링의 타입 변환이 적용**된다.

## 2025-05-01 Thu
### Environment 클래스를 활용하는 경우의 장단점
* 상술한 방식을 활용할 경우, 외부 설정 방식이 달라지더라도 애플리케이션의 코드를 그대로 유지할 수 있다는 장점을 얻을 수 있다.
  * 예를 들어, `application.properties`와 같은 외부 설정 데이터를 사용하다 커맨드 라인 옵션 인자를 적용할 수 있다.
* 반면, `Environment` 클래스를 활용하는 방식은 `env.getProperty()` 메소드가 반복적으로 호출되어야 한다는 한계를 갖는다.
  * 또한, 외부 설정의 키는 문자열로 전달하므로 휴먼 에러에도 취약하다는 단점이 수반된다.
  * 이에 스프링은 `@Value` 어노테이션을 활용하여 외부 설정 값을 주입 받을 수 있도록 하는 개선된 기능을 제공한다.

## 2025-05-02 Fri
### @Value 어노테이션을 활용한 필드 주입
* 스프링은 외부로부터 주입 받은 설정 데이터를 편리하게 사용할 수 있도록 `@Value` 어노테이션을 지원하며, 이는 다음과 같이 사용이 가능하다.
  * 이 때, **`@Value` 어노테이션 역시 내부적인 동작 과정에서는 `Environment` 클래스를 활용**한다.
```java
@Configuration
public class ValueAnnotationConfiguration {
    @Value("${my.datasource.url}")
    private String url;
    @Value("${datasource.etc.max-connection}")
    private int maxConnection;
    @Value("${my.datasource.etc.timeout}")
    private Duration timeout;
    @Value("${my.datasource.etc.options}")
    private List<String> options;
    
    @Bean
    public MyDataSource myDataSource() {
        return new MyDataSource(url, maxConnection, timeout, options); 
    }
}
```
* **생성자를 통해 주입했던 `Environment` 클래스 방식과 달리, `@Value` 어노테이션을 활용하는 경우 각 값이 외부로부터 주입**된다.
  * 이렇듯 `@Value("${프로퍼티_키}")` 형태의 어노테이션을 활용하는 것으로 외부 설정 데이터를 손쉽게 주입 받는 것이 가능하다.
* 반면, **`@Value` 어노테이션 역시 외부 설정 데이터를 원하는 클래스 형태로 캐스팅해주는 기능을 지원**한다.

## 2025-05-03 Sat
### @Value 어노테이션을 활용한 인자 주입
* 상술한 바와 같이 **`@Value` 어노테이션은 필드 주입에 활용할 수 있으나, 아래와 같이 작성할 경우 인자 주입에도 활용이 가능**하다.
```java
@Configuration
public class ValueAnnotationConfiguration {
    @Bean
    public MyDataSource myDataSource(
        @Value("${my.datasource.url}") String url,
        @Value("${datasource.etc.max-connection}") int maxConnection,
        @Value("${my.datasource.etc.timeout}") Duration timeout,
        @Value("${my.datasource.etc.options}") List<String> options         
    ) {
        return new MyDataSource(url, maxConnection, timeout, options); 
    }
}
```

## 2025-05-04 Sun
### @Value 어노테이션에 기본값 명시하기
* `@Value` 어노테이션을 통해 임의의 외부 설정 데이터를 주입 받고자 했으나, 대응되는 데이터가 존재하지 않는다면 아래와 같이 기본 값을 명시할 수 있다.
  * 이 경우, 기본 값은 `:` 문자 뒤에 명시하게 되며 해당 프로퍼티 키에 대응되는 데이터가 존재하지 않는 경우에만 적용된다.
```java
@Configuration
public class ValueAnnotationConfiguration {
    @Bean
    public MyDataSource myDataSource(
        @Value("${my.datasource.url:local.db.com}") String url, // : 문자 뒤에 기본 값을 명시한다.
        @Value("${my.datasource.etc.max-connection}") int maxConnection,
        @Value("${my.datasource.etc.timeout}") Duration timeout,
        @Value("${my.datasource.etc.options}") List<String> options         
    ) {
        return new MyDataSource(url, maxConnection, timeout, options); 
    }
}
```

## 2025-05-05 Mon
### @Value 어노테이션의 한계
* 해당 방식은 편리하지만, 대응되는 외부 설정의 키 값을 하나하나 손수 입력하여 주입 받아야 한다는 명확한 한계가 존재한다.
  * 때문에 해당 방식 역시 오타로부터 발생할 수 있는 휴먼 에러의 가능성을 방지할 수 없다.
* 또한, 상술한 예시와 같이 `my.datasource` 형태로 반복적으로 나타나는 접두사는 불필요한 중복에 해당한다.
  * 이를 제거하거나 임의의 객체로 변환할 수 있다면 사용성이 증대될 수 있을 것으로 보이며, 이에 스프링은 `@ConfigurationProperties`를 제안하였다.