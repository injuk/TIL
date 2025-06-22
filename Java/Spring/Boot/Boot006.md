# Spring Boot
## 2025-05-06 Tue
### 타입 안전한 설정 속성이란?
> 타입 안전한 설정 속성을 활용할 경우, 외부 설정 데이터를 Java 코드로 관리할 수 있게 된다.
* 스프링은 외부 설정의 일부를 객체로 변환하는 기능을 제공하며, 이는 `Type-safe Configuration Properties`라는 용어로 지칭한다.
  * 객체를 통해 타입 정보를 적용할 수 있으며, 이를 통해 외부 설정 데이터를 잘못된 속성으로 주입받는 문제를 해결하면서도 객체의 장점을 활용할 수 있게 된다.

## 2025-05-07 Wed
### @ConfigurationProperties 어노테이션 활용하기
* 상술한 타입 안전한 설정 속성 개념은 스프링의 `@ConfigurationProperties`를 통해 활용할 수 있으며, 다음과 같은 코드 형태를 갖는다.
  * `@ConfigurationProperties("my.datasource")`와 같이 작성하는 것으로 외부 설정 파일의 접두사를 적용할 수 있다.
```java
@Data
@ConfigurationProperties("my.datasource")
public class MyProperties {
    private String url;
    private Etc etc;
    
    @Data
    public static class Etc {
        private int maxConnection;
        private Duration timeout;
        private List<String> options;
    }
}
```
* `@ConfigurationProperties` 어노테이션이 명시된 경우 해당 객체가 외부 설정을 주입받을 것임을 선언하게 되며, 필요시 접두사를 인자에 전달할 수 있다.
  * 때문에 상술한 코드의 경우, `my.datasource`는 외부 설정 파일에 작성된 설정의 접두사를 의미한다.
* **해당 방식의 경우 외부 설정 데이터의 주입에 Java 빈 프로퍼티 방식을 활용하므로, 반드시 각 필드에 대한 `Getter`와 `Setter`가 정의**되어야 한다.

## 2025-05-08 Thu
### @ConfigurationProperties 기반의 외부 설정 데이터 활용하기
* 상술한 코드만으로는 동작하지 않으며, 아래와 같이 `@EnableConfigurationProperties` 어노테이션을 명시해줄 필요가 있다.
  * 해당 어노테이션은 인자에 전달 받은 클래스의 인스턴스를 생성하고, 이를 스프링 빈으로 등록하도록 한다.
```java
@Configuration
@EnableConfigurationProperties(MyProperties.class)
public class PropertiesConfiguration {
    private final MyProperties properties;
    
    public PropertiesConfiguration(MyProperties properties) {
        this.properties = properties;
    }
    
    @Bean
    public MyDataSource myDataSource() {
      return new MyDataSource(
              properties.getUrl(), 
              properties.getEtc().getMaxConnection(), 
              properties.getEtc().getTimeout(), 
              properties.getEtc().getOptions(),
      );
    }
}
```

## 2025-05-09 Fri
### @ConfigurationProperties과 타입 안전성
* 예를 들어 `@ConfigurationProperties` 어노테이션이 명시된 클래스의 정수형 프로퍼티에 문자열을 전달한 경우, 타입 변환에 실패하게 된다.
  * 이는 빈 등록 과정에서 실패하는 것이므로, 개발자는 잘못된 설정 값이 전달되었음을 사전에 확인하여 조치할 수 있다.
* 이렇듯 `@ConfigurationProperties` 어노테이션 방식은 잘못된 타입에 대해 오류를 발생시키므로, 타입 안전한 설정 속성으로 취급할 수 있다.
  * 즉, `@ConfigurationProperties` 어노테이션 기반의 외부 설정 데이터는 믿고 활용하는 것이 가능하다.

## 2025-05-10 Sat
### 참고 - 케밥 케이스와 카멜 케이스 간 전환
* `application.properties` 파일의 경우 일반적으로 케밥 케이스로 작성되지만, Java의 명명은 카멜 케이스를 사용하게 된다.
* 이에 스프링은 `@ConfigurationProperties` 어노테이션 기반의 외부 데이터 인스턴스 초기화 과정에서 각 프로퍼티 이름을 자동으로 변환하는 절차를 거친다.
  * 예를 들어 `max-connection`이라고 작성된 외부 데이터의 경우, 대응되는 Java 클래스의 `maxConnection` 필드에 초기화된다.

## 2025-05-25 Sun
### @ConfigurationPropertiesScan 어노테이션이란?
* 상술한 코드에서, 이미 `@ConfigurationProperties` 어노테이션을 할당했음에도 별도의 어노테이션을 다시 명시해야하는 것은 상당히 번거로운 축에 속한다.
  * 예를 들어, `PropertiesConfiguration` 클래스는 `@EnableConfigurationProperties` 어노테이션을 명시하고 있다.
* `@ConfigurationPropertiesScan` 어노테이션은 이를 위해 지원되는 기능으로, 마치 컴포넌트 스캔처럼 필요한 외부 데이터를 모두 빈으로 등록한다.
* 이렇듯 **임의의 외부 데이터 하나를 등록하는 경우와, 임의의 경로로부터 특정 범위에 포함된 모든 외부 데이터를 등록하는 방법을 선택하여 사용**할 수 있다.
  * 비유하자면 이는 빈을 직접 등록하는 것과, 컴포넌트 스캔을 기반으로 등록하는 차이와 유사하다.

## 2025-05-26 Mon
### Setter보다 생성자 방식을 활용하기
* 상술한 방식은 빈으로 등록되는 외부 데이터 클래스가 `Setter`를 갖기 때문에, 휴먼 에러에 의해 값이 변조될 가능성을 갖는다.
  * 사실 외부 데이터를 의미하는 필드 값들은 최초 한 번 초기화된 이후엔 변경되지 않는 것이 바람직하다.
* 때문에 **외부 데이터는 `Setter` 대신 생성자를 사용하는 방식이 권장되며, 이 경우 초기화된 이후 데이터가 변경될 만한 가능성을 미연에 방지**할 수 있다.
  * 이렇듯 좋은 프로그램은 다소의 불편한 제약이 따르는 프로그램으로 이해할 수 있다.

## 2025-05-27 Tue
### @ConfigurationProperties 어노테이션과 생성자 주입
* `@ConfigurationProperties` 기능은 `Getter`나 `Setter`를 사용하는 Java 빈 프로퍼티 방식 외에도 생성자 방식 역시 다음과 같이 제공한다.
```java
@Getter
@ConfigurationProperties("my.datasource")
public class MyProperties {
    private String url;
    private Etc etc;
    
    public MyProperties(String url, @DefaultValue Etc etc) {
        this.url = url;
        this.etc = etc;
    }
    
    @Getter
    public static class Etc {
        private int maxConnection;
        private Duration timeout;
        private List<String> options;
        
        public Etc(int maxConnection, Duration timeout, @DefaultValue("DEFAULT") List<String> options) {
            this.maxConnection = maxConnection;
            this.timeout = timeout;
            this.options = options;
        }
    }
}
```
* 상술한 외부 설정 데이터 클래스를 활용하기 위한 `@Configuration` 설정 클래스는 크게 바뀔 것이 없으며, 의존성을 잘 연결해주는 것만으로도 잘 동작한다.
  * 이는 외부 설정 데이터 클래스에 대한 인스턴스화 과정에서 Java 빈 방식 이외에도 생성자 주입 방식을 스프링 차원에서 잘 지원하기 때문이다.

## 2025-05-28 Wed
### @DefaultValue란?
> @DefaultValue 어노테이션은 대응되는 외부 설정 데이터 값을 찾을 수 없는 경우 정해진 규칙에 따른 기본값을 적용하기 위해 활용될 수 있다.
* `@DefaultValue`란 외부 설정 데이터 주입 과정에서 사용할 수 있으며, 임의의 프로퍼티에 대응되는 외부 설정이 존재하지 않는 경우 다음과 같이 동작한다.
  1. `@DefaultValue(값)`: 대응되는 외부 설정 값이 하나도 존재하지 않을 경우, 어노테이션의 인자에 전달된 값으로 초기화된다.
  2. `@DefaultValue`: 상술한 예시에서, `Etc`와 대응되는 외부 설정이 존재하지 않는 경우 객체를 생성하긴 하지만 내부 값은 모두 기본 값을 적용한다.
* 특히, 두 번째 사용 방식의 경우 `Etc` 인스턴스가 생성되지만 `maxConnection`은 `int` 형의 기본 값인 `0`으로 초기화된 것을 확인할 수 있다.
  * 반면, 참조형 변수인 `timeout`에는 참조형의 기본 값인 `null`로 초기화된다.
  * 추가적으로 **`@DefaultValue` 어노테이션이 누락된 경우, 대응되는 외부 설정 데이터가 명시되지 않았다면 `MyProperties` 인스턴스화에 실패**한다.

## 2025-05-29 Thu
### @ConstructorBinding이란?
> @ConstructorBinding 어노테이션은 외부 설정 데이터 주입에 사용할 생성자를 명시하기 위해 활용될 수 있다.
* 스프링 3.0 버전 이전의 경우, 상술한 생성자 기반의 외부 설정 데이터 주입을 위해서는 반드시 `@ConstructorBinding` 어노테이션을 명시해야 했다.
* 그러나 스프링 3.0 버전 이후부터는 생성자가 하나인 경우에 한해 이를 생략할 수 있도록 개선되었다.
  * 반면, **생성자가 둘 이상이 존재하는 경우에는 여전히 외부 설정 데이터 주입에 사용할 생성자에 `@ConstructorBinding` 어노테이션을 명시**해야 한다.

## 2025-05-30 Fri
### 생성자 주입 방식의 장점과 한계
* 상술한 **생성자 기반의 외부 설정 데이터 주입 방식의 경우 `Setter`를 제공하지 않으므로, 런타임에서 값이 수정되는 상황을 미연에 방지**할 수 있다.
* 또한, 타입 안전한 방식에 속하기에 잘못된 형식의 외부 설정 데이터가 주입되는 등의 타입 문제 역시 상당 부분 해결될 수 있다.
* 그러나 정수 타입을 예로 들어, 반드시 양수만 허용해야하는 외부 설정 데이터에 대해 `0`이나 음수가 전달되는 경우를 방지할 수는 없는 명확한 한계를 갖는다.
  * 다시 말해, 상술한 외부 설정 데이터 중 `my.datasource.etc.max-connection`에 양수가 전달되지 않은 경우 예외를 발생시키고 싶을 수 있다.
  * 그러나 현재까지 다루어 온 방식에서는 이러한 문제를 해결할 수 없으며, 주입된 외부 설정 데이터를 검증할 수 있을만한 방법을 마련할 필요성이 있다.

## 2025-05-31 Sat
### @ConfigurationProperties를 활용하는 외부 설정 데이터의 유효성 검증
* **`@ConfigurationProperties`를 통해 주입된 데이터의 유효성을 검증하기 위해서는 다음과 같은 Java 빈 검증기의 도입을 고려**해볼 수 있다.
  * 이는 `@ConfigurationProperties` 어노테이션이 할당된 대상 역시 Java 객체이기에 스프링 차원에서 Java 빈 검증기를 지원하기 때문에 가능하다.
```groovy
implementation 'org.springframework.boot:spring-boot-starter-validation'
```
* 추가적으로 상술한 의존성을 주입할 경우, 다음과 같이 `jakartar`와 `hibernate` 기반의 빈 검증기가 프로젝트에 추가되는 것을 확인할 수 있다.
  1. `jakarta.validation.constraints`: Java 표준 검증기에서 지원하는 기능들이 포함된다.
  2. `org.hibernate.validator.constraints`: Java 표준 검증기가 아닌, 하이버네이트 검증기라는 표준 검증기 구현체에서 제공하는 기능에 해당한다.
* 이 경우 표준이 아닌 하이버네이트 검증기를 사용하는 것이 마음에 걸릴 수 있으나, 실무의 경우 대부분의 상황에서 해당 검증기를 사용하므로 문제가 되지 않는다.

## 2025-06-01 Sun
### Java 빈 검증기를 활용한 외부 설정 데이터의 검증
* 이 때, Java 빈 검증기를 적용하여 외부 설정 데이터의 유효성을 검증하는 코드의 예시는 다음과 같다.
```java
@Getter
@ConfigurationProperties("my.datasource")
@Validated
public class MyProperties {
    @NotEmpty
    private String url;
    private Etc etc;
    
    public MyProperties(String url, Etc etc) {
        this.url = url;
        this.etc = etc;
    }
    
    @Getter
    public static class Etc {
        @Min(1)
        @Max(999)
        private int maxConnection;
        @DurationMin(seconds = 1)
        @DurationMax(seconds = 60)
        private Duration timeout;
        private List<String> options;
        
        public Etc(int maxConnection, Duration timeout, List<String> options) {
            this.maxConnection = maxConnection;
            this.timeout = timeout;
            this.options = options;
        }
    }
}
```
* 이 경우, `@Validated`를 활용하여 Java 빈 검증기의 동작을 유도하고 `@NotEmpty` 등을 통해 Java 빈 검증 로직을 활용하는 점을 확인할 수 있다.

## 2025-06-02 Mon
### @ConfigurationProperties 방식의 장점
* 앞서 다룬 바와 같이, `@@ConfigurationProperties` 어노테이션을 활용할 경우 타입 안전한 외부 설정 데이터를 편리하게 사용할 수 있다.
  * 덕분에 외부 설정 데이터를 표현력이 좋은 객체 형태로 편리하게 변환하여 사용할 수 있게 된다.
* 추가적으로, Java 빈 검증기를 활용하여 쉽고 편리하게 설정 정보를 검증할 수도 있다.
  * **좋은 예외는 언제나 컴파일 시점에서 발견 가능한 예외이며, 반대로 나쁜 예외는 서비스 도중에 발생하는 런타임 예외인 점을 기억하는 것이 바람직**하다.

## 2025-06-03 Tue
### YAML 설정 파일 사용하기
* 스프링은 `application.properties` 뿐만 아니라 `application.yml`과 같은 YAML 형식의 설정 데이터 역시 지원한다.
  * 이 때, YAML은 사람이 읽기 쉬운, 가독성 그 자체를 목표로 하는 데이터 구조를 의미한다.
* YAML 형태로 작성된 설정 데이터는 사람이 읽기 쉽도록 계층 구조를 이루며, 스프링은 이를 `properties` 파일처럼 평탄화하여 읽어들인다.
  * 한 편, YAML 작성시 계층 구조는 일반적으로 공백 두 글자를 사용한다.
* 반면, 하나의 스프링 애플리케이션에 `properties`와 `yml` 확장자로 작성된 설정이 동시에 존재할 경우 `properties` 파일이 우선권을 갖는다.
  * 물론 **실무 관점에서 이를 혼용하는 것은 일관성이 깨지기에 권장되지 않으며, 일반적으로는 가독성이 좋은 YAML 설정 파일이 자주 사용**된다.

## 2025-06-04 Wed
### @Profile 어노테이션이란?
* 상술한 과정을 기반으로 프로필과 외부 설정 기능을 조합하면 환경마다 서로 다른 설정 값을 적용할 수 있다.
  * 반면, 나아가 설정 값 뿐만 아니라 환경 별로 생성되는 빈 자체를 달리하고자 하는 요구 사항이 발생할 수 있다.
* `@Profile` 어노테이션은 현재 애플리케이션에 적용된 프로필 값에 따라 서로 다른 빈을 다음과 같이 등록하도록 할 수 있다.
```java
@Configuration
public class MyConfig {
    @Bean
    @Profile("default") // 프로필을 설정하지 않은 경우 default 프로필이 활성화되므로, 해당 빈이 등록된다.
    public MyClient myClient() {
        return new MyClient();
    }

    @Bean
    @Profile("prod")
    public MyProdClient myProdClient() {
      return new MyProdClient();
    }
}
```

## 2025-06-05 Thu
### @Profile 어노테이션의 내부 동작 원리
* 상술했듯, `@Profile` 어노테이션은 활성 프로필에 따라 서로 다른 빈을 등록하도록 지원하는 기능을 담당한다.
* 이는 `@Conditional` 어노테이션의 동작 방식과 유사하게 느껴지며, 실제로도 `@Profile`은 내부적으로 `@Conditional` 어노테이션을 기반으로 동작한다.
  * 즉, 스프링 자체적으로도 `@Conditional` 어노테이션을 재사용하는 것으로 이해할 수 있다.
  * 이로 인해 환경 별 외부 설정 데이터를 분리하는 데에 그치지 않고, 스프링 빈까지도 격리하여 활용하는 것이 가능할 수 있다.

## 2025-06-06 Fri
### 프로덕션 준비 기능이란?
```
> 서비스를 운영하는 개발자에게 있어 운영 상의 장애는 언제나 발생할 수 있는 일이지만, 그렇다고 해서 모니터링을 소홀히하지는 않아야 한다.
```
* 개발자는 애플리케이션을 개발하는 과정에서 기능적인 요구 사항만을 처리하는 것이 아니며, 서비스를 모니터링하고 감시하는 활동 역시 중요하다.
  * 다시 말해, 서비스가 운영 단계까지 이르게 되면 서비스 자체적인 문제가 없는지 지속적으로 확인할 필요가 있다.
* 이렇듯 **서비스를 운영할 때 필요한 여러 기능들을 프로덕션 준비 기능이라고 지칭하며, 이는 운영 환경에 준비되어야 하는 비기능적인 요소들을 포함**한다.
  * 에를 들어 지표와 추적, 감사와 모니터링 등의 비기능적 요소를 준비할 필요가 있다.
  * 이를 통해 애플리케이션의 동작 여부와 필요한 로그의 출력, 커넥션 풀의 가용 수준 등을 확인할 수 있어야 한다.

## 2025-06-07 Sat
### 스프링 부트와 프로덕션 준비 기능
```
> 액추에이터는 시스템을 움직이거나 제어하는 데에 활용될 수 있는 기계 장치를 의미한다.
```
* 스프링 부트는 프로덕션 준비 기능을 편리하게 사용할 수 있도록 여러 편의 기능을 제공하며, 이는 액추에이터라는 이름으로 지칭된다.
  * 예를 들어 액추에이터로 인해 자동 등록되는 여러 엔드포인트를 활용하여 애플리케이션의 상태나 로그들을 확인할 수 있도록 지원한다.
  * 나아가 실무에서 자주 사용되는 마이크로미터나 프로메테우스, 그라파나와 같은 모니터링 시스템과 쉽게 연동할 수 있도록 지원하는 기능 역시 제공한다.

## 2025-06-08 Sun
### 프로젝트에 액추에이터 적용하기
* 액추에이터가 제공하는 여러 프로덕션 준비 기능을 활용하기 위해서는 아래와 같은 의존성을 추가해야 한다.
```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```
* 스프링 웹 MVC 기반의 애플리케이션을 예로 들어, 상술한 의존성에 의해 애플리케이션을 실행시킨 후 `/actuator` 경로로 GET 요청을 전달할 수 있게 된다.
* 상술한 요청에 의해 반환되는 경로는 크게 다음과 같으며, 각각의 경로 역시 GET 요청을 처리할 수 있다.
  * `/actuator/health`: `{"status":"UP"}`과 같은 응답이 반환되며, 이는 단순히 현 서버의 정상 동작성을 표현한다.
  * `/actuator/health/{*path}`
* 이러한 **기본 기능은 단순히 애플리케이션의 정상성만을 검증하므로, 액추에이터의 다양한 기능을 활용하기 위해서는 추가적인 설정이 필요**하다.

## 2025-06-09 Mon
### 액추에이터의 추가 기능을 노출하기
* 액추에이터는 다양한 기능을 제공하는 반면, 이들은 기본적으로 웹에 노출되지 않으므로 다음과 같은 설정을 명시하여 기능을 노출시켜야 한다.
```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```
* 이는 액추에이터가 사용하는 설정이며, 애플리케이션을 재실행한 후 다시 `/actuator` 경로에 GET 요청을 전송할 경우 수많은 경로가 반환된다.
* **액추에이터가 제공하는 수많은 기능을 엔드포인트라는 용어로 지칭하며, 예를 들어 `beans`는 등록된 빈 목록을 반환**한다.
  * 이렇듯 액추에이터의 다양한 기능은 `/actuator/{엔드포인트명}` 형태의 경로를 통해 접근하는 것이 가능하다.

## 2025-06-10 Tue
### 액추에이터 엔드포인트 설정하기
```
> 액추에이터의 엔드포인트를 사용하기 위해서는 각 엔드포인트를 활성화하고, 노출하는 과정이 먼저 진행되어야 한다.
```
* 임의의 기능에 대한 엔드포인트를 활성화한다는 것은 해당 기능 자체의 사용 여부를 `on` 또는 `off`로 설정하는 것을 지칭한다.
* 반면, **엔드포인트를 노출하는 것은 상술한 과정에서 활성화된 엔드포인트를 HTTP 또는 JMX 중 어디에 노출할지 선택하는 것을 의미**한다.
  * 예를 들어 두 위치 중 하나에만 노출하거나, 두 위치에 모두 노출하는 식으로 노출 위치를 명시할 수 있다.
  * 물론, 임의의 엔드포인트에 대한 활성화 자체가 이루어지지 않았다면 노출 역시 되지 않는다.
* 그러나 액추에이터의 각 엔드포인트는 기본적으로 활성화되어 있으므로, 개발자는 설정을 통해 어떤 엔드포인트를 노출할지 결정해야 한다.
  * 일반적으로 JMX는 잘 사용되지 않으므로, HTTP에 어떤 엔드포인트를 노출할지 결정하게 된다.

## 2025-06-11 Wed
### 모든 엔드포인트를 HTTP에 노출하기
* 상술한 내용을 기반으로, 다음과 같은 설정은 활성화된 모든 엔드포인트를 HTTP로 노출한다는 것을 의미한다.
```yaml
management:
  endpoints:
    web: # HTTP로 노출한다.
      exposure:
        include: "*"
```
* 반면, `shutdown`이라는 엔드포인트는 기본적으로 활성화되어 있지 않기에 상술한 설정을 통해서도 노출되지 않는다.
  * 결국 **임의의 액추에이터 기능을 활용하기 위해서는 대응되는 엔드포인트가 활성화되고, 노출되도록 설정되어야 함**을 알 수 있다.

## 2025-06-12 Thu
### 새로운 엔드포인트를 활성화하고 노출하기
* 액추에이터가 제공하는 다양한 기능 중, `shutdown` 기능을 활성화하는 설정은 다음과 같이 작성할 수 있다.
  * 이렇듯 임의의 엔드포인트를 활성화하는 경우, `management.endpoint.[엔드포인트명].enabled`를 `true`로 설정해주어야 한다.
```yaml
management:
  endpoint:
    shutdown:
      enabled: true
  endpoints:
    web:
      exposure:
        include: "*"
```
* 해당 설정을 통해 `shutdown` 기능은 명시적으로 활성화되며, `management.endpoints.web.exposure.include`가 `*`로 설정되어 있기에 노출된다.
  * 해당 기능을 활용하고자 하는 경우, 서버의 동작을 변경하는 것이므로 노출된 `/actuator/shutdown` 엔드포인트를 `POST` 메소드 호출해주어야 한다.

## 2025-06-13 Fri
### 임의의 엔드포인트를 선택적으로 노출하기
* 활성화되어 있는 여러 엔드포인트를 모두 노출하는 대신 선택적인 기능만을 외부에 노출하고자 하는 경우, 설정을 다음과 같이 작성할 수 있다.
```yaml
management:
  endpoints:
    jmx:
      exposure:
        include: "health,info" # health와 info 엔드포인트만을 노출한다.
```
* 반면, 임의의 엔드포인트를 제외하고 모두 노출하고자 하는 경우의 설정은 다음과 같이 `exclude`를 사용한다.
```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
        exclude: "env,beans" # env와 beans 엔드포인트를 제외한 모든 엔드포인트를 노출한다.
```

## 2025-06-14 Sat
### 액추에이터의 다양한 엔드포인트
* 여러 액추에이터 기능에 대응되는 엔드포인트를 외부로 노출하는 것으로, 개발자는 애플리케이션 내부의 많은 기능을 관리하거나 모니터링할 수 있다.
* 액추에이터가 제공하는 기능 중 자주 사용되는 것은 크게 다음과 같이 정리할 수 있다.
  1. `beans`: 스프링 컨테이너에 등록된 빈을 조회할 수 있다.
  2. `conditions`: `condition` 기능을 통해 빈을 등록하는 경우의 평가 조건에 부합하는지 여부를 조회할 수 있다.
  3. `configprops`: `@ConfigurationProperties`를 조회할 수 있다.
  4. `env`: `Environment`와 관련된 정보를 조회할 수 있다.
  5. `health`: 애플리케이션의 헬스 정보를 조회할 수 있다.
  6. `httpexchanges`: HTTP 호출에 대한 응답 정보를 조회할 수 있는 반면, `HttpExchangeRepository`를 구현하는 빈을 등록해주어야 한다.
  7. `info`: 애플리케이션의 정보를 조회할 수 있다.
  8. `loggers`: 애플리케이션에 설정된 로거 정보를 조회할 수 있으며, 이를 변경하는 것 역시 가능하다.
  9. `metrics`: 애플리케이션의 메트릭 정보를 조회할 수 있다.
  10. `mappings`: `@RequestMapping`에 의해 설정된 정보를 조회할 수 있다.
  11. `threaddump`: 애플리케이션의 쓰레드 덤프를 실행하여 이를 조회할 수 있다.
  12. `shutdown`: 애플리케이션을 종료하는 반면, 해당 기능은 기본적으로 비활성화된다.

## 2025-06-15 Sun
### 액추에이터를 활용한 헬스 정보 확인하기
```
> 액추에이터와 같은 도구를 활용하여 애플리케이션의 헬스 정보를 확인할 수 있다면 문제 발생시 이를 빠르게 인지할 수 있다.
``` 
* **헬스 정보는 단순히 애플리케이션이 요청에 응답할 수 있는지 여부를 떠나 DB의 응답과 디스크 사용량 등의 다양한 정보를 포함**한다.
* 이 때, 액추에이터의 경우 헬스 정보를 더욱 자세히 표현하기 위해서는 다음과 같은 설정을 명시할 필요가 있다.
```yaml
management:
  endpoint:
    health:
      show-details: always
```
* 특히, DB의 경우 JDBC 스펙에서 제공되는 기능을 활용하여 정상성을 확인할 수 있다.
  * 오래된 방식으로는 우선 더미 쿼리를 DB에 요청하고, 실제 응답이 반환된 경우 DB의 상태가 정상이라고 판단하는 것이 있다.
* 상술한 설정을 통해 연결된 DB와 서버의 디스크 사용량, Ping 정상성 여부를 종합하여 상태를 결정한다.
  * 이는 **세 상태를 모두 종합하는 최종 상태이므로, 셋 중 하나라도 DOWN 상태라면 전체 상태 역시 DOWN으로 노출**된다.

## 2025-06-16 Mon
### 간소화된 헬스 정보 확인하기
* 반면, 너무 상세한 정보를 노출하고 싶지 않은 경우 외부 설정 데이터를 다음과 같이 설정할 수 있다.
```yaml
management:
  endpoint:
    health:
      # show-details: always
      show-components: always
```
* 이 경우, 상세한 정보 없이 단순히 DB와 서버의 디스크 사용량 및 Ping 각각의 항목에 대한 UP 또는 DOWN만을 노출시킨다.

## 2025-06-17 Tue
### 액추에이터와 헬스 정보 - 참고
* **액추에이터의 경우 `db` 뿐만 아니라 `mongo`와 `redis` 등의 주요한 헬스 기능을 기본으로 제공**한다.
  * 단지 **액추에이터 의존성을 주입하는 것만으로도 각각의 헬스 기능이 자동으로 등록되어 사용자의 요청시 반환되는 정보에 포함**된다.
* 추가적으로, 개발자가 원하는 경우 필요한 헬스 기능은 직접 구현하여 등록한 후 활용하는 것 역시 가능하다.
* **실무의 경우, 항상 엔드포인트를 호출하여 JSON 데이터를 직접 확인하는 방식보다는 이를 별도의 서비스를 통해 주기적으로 모니터링하는 방식**을 택한다.
  * 이를 직접 구현하는 경우 적절한 대시보드의 도입을 고려하고, 문제가 발생한 경우 서버에 연결하여 트러블슈팅을 시도할 수 있다.
  * 반면, 이미 작성된 모니터링 시스템을 활용하는 경우 DOWN으로 전환되는 등의 문제 발생시 개발자가 알림을 받도록 설정하는 것 역시 가능하다.
  * 중요한 것은 **이를 통해 문제 발생시 트러블슈팅 시간이 크게 줄어든다는 것으로, 개발자는 더욱 더 빠르게 문제에 대처할 수 있게 된다**.

## 2025-06-18 Wed
### 액추에이터를 활용한 애플리케이션 정보 확인하기
* 액추에이터가 기본적으로 제공하는 `info` 엔드포인트를 활용할 경우, 다음과 같은 애플리케이션의 기본 정보를 확인하는 것이 가능하다.
  1. `java`: Java 런타임 정보를 확인할 수 있다.
  2. `os`: 운영체제 정보를 확인할 수 있다.
  3. `env`: `Environment` 클래스가 제공하는 정보 중 `info.` 접두사를 갖는 정보를 확인할 수 있다.
  4. `build`: 애플리케이션의 빌드 정보를 제공하며, 해당 기능은 `META-INF/build-info.properties` 파일이 작성되었음을 전제한다.
  5. `git`: 애플리케이션의 `git` 정보를 제공하며, 해당 기능 역시 `git.properties` 파일이 작성되었음을 전제한다.
* 이 때, **`info` 엔드포인트와 관련된 여러 기능 중 `env`와 `java` 및 `os`는 기본적으로 비활성화**된다.

## 2025-06-19 Thu
### java와 os 애플리케이션 정보 확인하기
* 액추에이터를 활용하여 Java 런타임 및 운영체제 정보를 확인하고자 하는 경우, 외부 설정 데이터 파일에 다음과 같은 내용을 작성해줄 필요가 있다.
  * 이 때, `management.info` 형태이며 `management.endpoint.info` 형태가 아님에 주의해야 한다.
```yaml
management:
  info:
    java:
      enabled: true
    os:
      enabled: true
```
* 해당 액추에이터 설정을 적용한 후, 애플리케이션을 시작하면 `/actuator/info` 경로를 통해 필요한 애플리케이션 정보를 확인하는 것이 가능하다.

## 2025-06-20 Fri
### env 애플리케이션 정보 확인하기
* 상술한 `env` 설정의 경우, 다음과 같은 외부 설정 데이터 파일을 작성하게 된다.
```yaml
management:
  info:
    env:
      enabled: true

info:
  app:
    name: my-awesome-application
    alias: maa
```
* 이 경우, **`/actuator/info` 엔드포인트를 호출할 경우 `info` 경로 하위에 작성한 `app.name`과 `app.alias`가 결과에 포함**된다.
  * 즉, `env`를 활용할 경우 애플리케이션에 자신만의 정보를 설정하고 이를 외부에서 확인하는 것이 가능하다.

## 2025-06-21 Sat
### build 애플리케이션 정보 확인하기
* `build` 애플리케이션 정보를 확인하기 위해서는 사전 준비해야할 파일이 있으나, 이는 그래들을 다음과 같이 활용하여 손쉽게 준비할 수 있다.
  * `build.gradle`과 같은 빌드 설정 파일에 아래와 같은 설정을 추가하고 애플리케이션을 빌드할 경우, 빌드 결과물에 필요한 파일이 모두 자동으로 포함된다.
```groovy
// ...생략
springBoot {
  buildInfo()
}
```
* 반면, **`build`의 경우 기본적으로 활성화된 기능이므로 상술한 파일이 빌드 결과물에 포함되어 있기만 하다면 이를 바로 확인**할 수 있다.
  * 같은 원리에서, `git` 애플리케이션 정보  역시 필요한 플러그인을 설치한다면 빌드 결과물에 자동 포함되는 파일을 통해 기능을 활용하는 것이 가능하다.

## 2025-06-22 Sun
### 액추에이터 로거 엔드포인트 활용하기
* `loggers` 엔드포인트를 통해 로깅과 관련된 정보를 확인하거나, 이를 실시간으로 변경하는 것이 가능하다.
* 반면, 액추에이터와 무관하게 스프링 부트 차원에서 임의의 패키지 및 하위 패키지들의 로깅 레벨을 수정하기 위해서는 다음과 같은 설정을 작성할 수 있다.
```yaml
logging:
  level:
    ga.injuk.controller: debug # 해당 패키지 및 하위 패키지 모두의 로깅 레벨을 debug로 적용한다!
```
* `/actuator/loggers` 엔드포인트로 확인할 경우, 상술한 설정에 의해 로깅 레벨이 설정된 패키지를 제외하고는 기본 설정이 적용되는 것을 확인할 수 있다.
  * 이 경우, `ROOT`의 `configuredLevel` 정보가 `INFO`로 설정된 것을 확인할 수 있으며 해당 정보가 기본 설정에 해당한다.
  * 즉, **로깅 레벨을 별도로 설정하지 않는다면 스프링 부트의 `ROOT` 설정에 의해 로깅 레벨은 기본으로 `INFO`가 적용**된다.
  * 반면 **상술한 설정을 작성할 경우, `ga.injuk.controller` 패키지 및 하위 패키지들은 예외적으로 `DEBUG` 로깅 레벨이 적용**된다.