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