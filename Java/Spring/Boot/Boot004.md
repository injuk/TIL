# Spring Boot
## 2025-03-26 Wed
### ImportSelector 인터페이스의 필요성
* `@Import` 어노테이션을 통해 설정 정보를 추가하는 방법은 크게 다음과 같이 분류할 수 있다.
  1. 정적인 방식: `@Import([클래스명])`과 같이 입력하는 것으로 설정 정보를 추가할 수 있으나, 대상은 동적으로 변경이 불가능하다.
  2. 동적인 방식: `@Import([ImportSelector구현체])`와 같이 입력하는 것으로 설정 대상을 동적으로 결정할 수 있다.
* 즉, 정적인 방식의 경우 아래와 같은 코드를 통해 설정 정보인 `MyConfiguration` 클래스를 정적으로 추가할 수 있다.
```java
@Configuration
@Import(MyConfiguration.class)
public class ApplicationConfiguration {}
```
* 반면, 상술한 방식은 설정 정보가 임의의 조건에 따라 동적으로 결정되는 경우에 대응이 불가능하기에 스프링은 `ImportSelector` 인터페이스를 제공한다.

## 2025-03-27 Thu
### ImportSelector 인터페이스 구현하기
* 스프링은 임의의 조건에 따라 설정 정보가 동적으로 결정될 수 있도록 `ImportSelector` 인터페이스를 제공하며, 이는 기본적으로 다음과 같은 구조를 갖는다.
  * 즉, 상술한 방식과 같이 하드코딩 형태로 설정 정보를 명시하는 대신 적용될 설정 정보를 프로그래밍적으로 결정할 수 있도록 지원한다.
```java
public interface ImportSelector {
  String[] selectImports(AnnotationMetadata importingClassMetadata);
  // ...생략
}
```
* 이 때, `selectImports()` 메소드의 반환 값은 적용하고자 하는 설정 정보 클래스의 이름을 패키지명을 포함하여 작성한 문자열 배열이 된다.
  * 예를 들어 `ga.injuk.MyConfiguration` 클래스를 설정 정보로 적용하고자 하는 경우, 아래와 같이 `ImportSelector` 인터페이스를 구현할 수 있다.
  * 물론 이 경우에는 해당 메소드로부터 반환되는 문자열이 표현하는 경로에 해당하는 설정 정보가 적용된다.
```java
public class MyImportSelector implements ImportSelector {
  @Override
  String[] selectImports(AnnotationMetadata importingClassMetadata) {
    return new String[]{"ga.injuk.MyConfiguration"};
  }
}
```

## 2025-03-28 Fri
### 정적인 설정 정보 적용 방식
* 상술한 내용 중 정적인 방식의 경우, 다음과 같은 테스트 코드를 통해 그 동작을 확인할 수 있다.
```java
public class StaticTest {
  @Test
  void staticConfiguration() {
    AnnotationConfigApplicationContext appCtx = new AnnotationConfigApplicationContext(StaticConfiguration.class);

    MyBean bean = appCtx.getBean(MyBean.class);

    Assertions.assertThat(bean).isNotNull();
  }

  @Configuration
  @Import(MyConfiguration.class)
  public static class StaticConfig {}
}
```
* 이 경우, 스프링 컨테이너인 `ctx`에서 `MyConfiguration` 설정 정보가 명시하는 빈을 이름으로 조회할 수 있게 된다.
  * 즉, 어떠한 설정 정보에 포함된 빈이 필요한 경우 해당 설정 정보의 클래스 명을 `@Import` 어노테이션의 인자에 그대로 전달할 수 있다.

## 2025-03-29 Sat
### 동적인 설정 정보 적용 방식
* 반면 정적인 방식의 경우, 다음과 같은 테스트 코드를 통해 그 동작을 확인할 수 있다.
```java
public class DynamicTest {
  @Test
  void staticConfiguration() {
    AnnotationConfigApplicationContext appCtx = new AnnotationConfigApplicationContext(DynamicConfig.class);

    MyBean bean = appCtx.getBean(MyBean.class);

    Assertions.assertThat(bean).isNotNull();
  }

  @Configuration
  @Import(MyImportSelector.class)
  public static class DynamicConfig {}
}
```
* 이 경우, `@Import` 어노테이션의 인자로 전달된 `ImportSelector` 인터페이스 구현체의 `selectImports()` 메소드를 호출하게 된다.
  * 결과로는 문자열이 반환되며, 해당 문자열은 패키지를 포함한 설정 정보 클래스의 전체 경로로 취급된다.

## 2025-03-30 Sun
### @EnableAutoConfiguration 어노테이션의 동작 원리
* 상술한 과정에서 `@Import`와 `ImportSelector`를 이해했으므로, 다음과 같은 `@EnableAutoConfiguration` 어노테이션을 이해할 수 있게 된다.
```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
  // ...생략
}
```
* `AutoConfigurationImportSelector` 클래스는 `ImportSelector`의 구현체이므로, 설정 정보를 동적으로 선택할 수 있도록 지원한다.
  * 해당 클래스는 모든 라이브러리에 포함된 `org.springframework.boot.autoconfigure.AutoConfiguration.imports` 파일을 읽어들여 동작한다.
* 결국 스프링 부트 자동 구성의 동작 방식은 다음과 같은 순서로 정리해볼 수 있다.
  1. `@SpringBootApplication` 어노테이션
  2. `@EnableAutoConfiguration` 어노테이션
  3. `@Import(AutoConfigurationImportSelector.class)` 어노테이션
  4. `resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 파일
  5. 4.의 파일에 명시된 설정 정보가 조회되어 스프링 컨테이너에 등록

## 2025-03-31 Mon
### 자동 구성 설정 주의 사항
> @AutoConfiguration 어노테이션이 명시된 자동 구성 설정 파일은 컴포넌트 스캔 대상이 되지 않아야 한다.
* 스프링 부트의 자동 구성을 직접 구현하는 경우, `@AutoConfiguration`에 자동 구성 순서를 명시할 수 있다.
* `@AutoConfiguration`는 내부적으로 `@Configuration`을 포함하므로, 이 역시 일종의 설정 파일로 이해할 수 있다.
  * 그러나 **이러한 설정 파일은 일반적인 스프링 설정과는 다른 생명 주기를 가져야하기에 파일에 명시해야 하며 컴포넌트 스캔의 대상은 되지 않아야 한다**.
  * 때문에 스프링 부트의 컴포넌트 스캔 과정은 `@AutoConfiguration` 설정 파일을 제외하는 `AutoConfigurationExcludeFilter`를 포함한다.
```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
  // ...생략
}
```
* 자동 구성은 내부적으로 컴포넌트 스캔을 사용하지 않아야하는 반면, `@Import`를 활용한 설정 정보 파일 사용은 가능하다.
  * 즉, `@AutoConfiguration` 어노테이션이 명시된 클래스에 `@ComponentScan`도 함께 명시하지 않아야 하는 반면 `@Import`는 사용이 가능하다.

## 2025-04-01 Tue
### 자동 구성은 언제 사용해야 할까?
* `AutoConfiguration`과 같은 **자동 구성은 라이브러리 제작에 사용되며, 그 외의 경우에는 거의 고려되지 않는다**.
  * 이는 일반적으로 필요한 빈은 모두 컴포넌트 스캔을 사용하거나, 직접 등록하기 때문으로 이해할 수 있다.
  * 그러나 라이브러리를 제작하는 경우 자동 구성은 매우 유용하며, 실제로도 수많은 외부 라이브러리들이 자동 구성 기능을 함께 제공한다.
* 일반적으로 라이브러리를 사용하기만 하기 때문에 자동 구성을 이해할 필요가 없을 것으로 오인할 수 있으나, 이러한 배경 지식은 트러블슈팅에 큰 도움이 된다.
  * 예를 들어, 개발 과정에서 임의의 빈이 어떤 원리로 등록된 것인지 확인해야하는 경우가 발생할 수 있다.
  * 이러한 경우 스프링 부트가 제공하는 자동 구성 기능을 이해하고 있다면 원인을 더 빠르게 분석하고, 문제에 대처하기도 매우 쉬워지게 된다.
  * 이렇듯 자동화는 편리하지만 트러블슈팅의 난이도를 높일 수 있으므로, 유사시에 대비하여 자동 구성과 관련된 원리를 이해해두는 것이 바람직하다.

## 2025-04-02 Wed
### 외부 설정의 필요성
* 임의의 애플리케이션을 서로 다른 여러 환경에서 사용해야할 수 있으며, 예를 들어 개발 및 운영 환경 등으로 구분되는 서버가 있다.
* 이 경우, 일반적으로 각 환경 별로 서로 다른 설정 값을 적용해야하는 일이 빈번히 발생한다.
  * 예를 들어, 개발 환경과 운영 환경은 서로 다른 DB를 사용하므로 접속 정보 역시 서로 달라지게 된다.
* 이를 해결하기 위한 가장 쉬운 방법은 애플리케이션을 용도에 맞추어 빌드하는 것으로, 상술한 예를 들어 `dev.jar`와 `prod.jar`를 각각 빌드할 수 있다.
  * 즉, 환경 별로 적절한 정보를 포함하도록 빌드된 서로 다른 JAR 파일을 각각 배포하게 된다.
* 그러나 이러한 방식은 빌드를 더 자주 해야하는 번거로움이 수반되며, 무엇보다 환경 별로 정말 동일한 소스 코드로부터 빌드되었는지 보장할 방법이 없다.
  * 이는 각 환경 별로 서로 다른 빌드 결과물을 사용하기 때문으로, 예를 들어 개발용 빌드와 운영용 빌드 사이에 누군가에 의한 코드 수정이 발생할 수 있다.
  * 또한 각 빌드 결과물은 반드시 적절한 환경에서만 사용할 수 있기에 유연성이 떨어지고, 새로운 환경이 추가될 때마다 새로운 빌드 결과물을 생성해야 한다.
* 이를 해결하기 위해 **빌드는 한 번만 처리하되, 환경 별로 필요한 설정 정보는 실행 시점에 외부로부터 주입 받는 외부 설정 방식이 제안**되었다.