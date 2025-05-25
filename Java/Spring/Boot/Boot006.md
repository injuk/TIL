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

## 2025-05-24 Sun
### @ConfigurationPropertiesScan 어노테이션이란?
* 상술한 코드에서, 이미 `@ConfigurationProperties` 어노테이션을 할당했음에도 별도의 어노테이션을 다시 명시해야하는 것은 상당히 번거로운 축에 속한다.
  * 예를 들어, `PropertiesConfiguration` 클래스는 `@EnableConfigurationProperties` 어노테이션을 명시하고 있다.
* `@ConfigurationPropertiesScan` 어노테이션은 이를 위해 지원되는 기능으로, 마치 컴포넌트 스캔처럼 필요한 외부 데이터를 모두 빈으로 등록한다.
* 이렇듯 **임의의 외부 데이터 하나를 등록하는 경우와, 임의의 경로로부터 특정 범위에 포함된 모든 외부 데이터를 등록하는 방법을 선택하여 사용**할 수 있다.
  * 비유하자면 이는 빈을 직접 등록하는 것과, 컴포넌트 스캔을 기반으로 등록하는 차이와 유사하다.