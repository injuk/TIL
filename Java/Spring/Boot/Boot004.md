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

## 2025-04-03 Thu
### 외부 설정을 활용하기
* 외부 설정을 적절히 활용할 경우, 동일 소스로부터 애플리케이션을 한 번만 빌드하되 내부에 설정 값을 포함하지 않도록 하는 것으로 재활용성을 높일 수 있다.
  * 반면, 필요한 설정 값은 실행 시점에 환경에 따라 외부로부터 주입 받아 사용하게 된다.
* 이러한 방법을 활용하면 임의의 한 환경에서 검증된 후로는 다른 환경에서 안전하게 재사용할 수 있으며, 추후 다른 환경이 추가되어도 안전한 확장이 가능하다.
* 이렇듯 **유지보수하기 좋은 애플리케이션을 개발하기 위한 기본 원칙 중 하나는 변하는 것과 변하지 않는 것을 구분**하는 데에 있다.
  * **이러한 원칙은 `주입`이라는 행위의 근간을 이루는 기본적인 컨셉에 해당**한다.
  * 각 **환경에 따른 설정 값은 전형적인 `변하는 것`에 해당하므로, 변하지 않는 코드로부터 외부 설정의 형태로 분리하는 것이 바람직**하다.

## 2025-04-04 Fri
### 외부로부터 설정 값을 주입하는 여러가지 방법
* 스프링 부트로 작성된 애플리케이션을 실행하는 경우, 필요한 각 설정 값을 외부로부터 주입하는 방법은 크게 다음과 같이 구분할 수 있다.
  1. 환경 변수: 운영체제 차원에서 지원하는 외부 설정으로, 해당 운영체제 상에서 실행되는 모든 프로세스가 공유한다.
  2. Java 시스템 속성: Java 차원에서 지원하는 외부 설정으로, 해당 JVM 안에서만 사용된다.
  3. Java 실행 인자: Java 애플리케이션 실행시 CLI 인자로 전달하는 외부 설정이며, `main` 메소드의 `args` 인자로 사용된다.
  4. 외부 파일: 프로그램이 실행되는 과정에서 외부 파일에 직접 접근하여 사용하는 방식에 해당하며, 예를 들어 일종의 `txt` 파일을 만들어 사용할 수 있다.

## 2025-04-05 Sat
### 환경 변수로 외부 설정 값을 주입하기
* 상술한 바와 같이, 운영체제의 환경 변수는 해당 운영체제로부터 실행되는 모든 프로세스가 공유한다.
  * 즉, 다른 외부 설정 방식에 비해 적용 범위가 비교적 넓다.
  * 운영체제에 설정된 환경 변수는 `printenv` 등의 명령어를 통해 확인할 수 있다.
* Java의 경우, 다음과 같이 `System.getenv()` 메소드를 호출하는 것으로 운영체제에 적용된 환경 변수를 간단히 읽어들일 수 있다.
  * 해당 메소드의 반환 타입은 `Map<String, String>`이며, 각 환경 변수 키에 대응되는 값은 `System.getenv([키])` 형태로 조회할 수 있다.
```java
public class OsEnv {
  public static void main(String[] args) {
    Map<String, String> env = System.getenv();
    for (String key: env.keySet()) {
        log.info("env {}={}", key, System.getenv(key));
    }
  }
}
```
* 이렇듯 필요한 값을 운영체제의 환경 변수로 등록하는 것으로 외부 설정을 손쉽게 활용할 수 있으며, 실무에서는 서버 별로 환경 변수를 달리 설정하게 된다.
* 반면, 운영체제 환경 변수는 해당 운영체제에서 동작하는 모든 프로세스가 동일하게 사용하는 일종의 전역 변수로서 기능하게 된다.
  * 때문에 이러한 방식은 다른 프로세스에 영향을 줄 수 있으므로, 임의의 Java 애플리케이션에만 외부 설정을 적용하고자 하는 경우 다른 방법을 고려해야 한다.

## 2025-04-06 Sun
### Java 설정으로 외부 설정 값을 주입하기
* **Java 시스템 속성은 실행한 JVM 내에서 접근 가능한 외부 설정 값이며, 이 중에는 Java 차원에서 미리 설정해두고 사용하는 속성들 역시 존재**한다.
* Java 시스템 속성의 경우, 실행시 `java -Dstage=dev -jar myapp.jar` 형태와 같이 VM 옵션인 `-D` 인자를 통해 전달한다.
  * 이 경우 **순서에 주의해야 하며, `-jar` 인자 앞에 `-D` 옵션을 부여**해야 한다.
* Java의 경우, `System.getProperties()` 메소드를 호출하는 것으로 JVM에서 접근 가능한 외부 설정 값 목록에 접근이 가능하다.
  * 해당 메소드의 반환 타입은 `Properties` 타입이며, 다음과 같이 `System.getProperty()` 메소드를 통해 각 설정 값에 접근이 가능하다.
```java
public class JavaSystemProperties {
  public static void main(String[] args) {
    Properties props = System.getProperties();
    for (Object key: props.keySet()) {
        log.info("prop {}={}", key, System.getProperty(String.valueOf(key)));
    }
  }
}
```
* 이렇듯 **`System.getProperties()` 메소드는 `Map`의 자식 타입인 `Properties`를 반환하며, 이를 통해 Java 시스템 속성에 접근이 가능**하다.
* 참고로, `System.setProperty("myprop", "my-value")` 형태로 코드 상에서 시스템 속성을 설정하는 것 역시 가능하다.
  * 그러나 이 경우 코드 상에 시스템 속성을 결정하는 내용이 작성되므로, 외부 설정을 분리하는 이점을 누릴 수 없음에 주의해야 한다.

## 2025-04-07 Mon
### 실행 인자로 외부 설정 값을 주입하기
* 메인 메소드 호출 형태을 예로 들어, 인자로 주입되는 `args`와 같이 Java 애플리케이션 실행 시 전달되는 인자를 통해 외부 설정 값을 주입받을 수 있다.
  * 예를 들어, `java -jar myapp.jar paramA paramB`와 같은 형태의 실행에서 `paramA`와 `paramB`가 실행 인자에 해당한다.
* 이는 Java의 기본 문법에 해당하므로, 다음과 같은 간단한 메인 클래스를 통해 실행 인자 정보를 확인하는 것이 가능하다.
```java
public class CommandLineV1 {
  public static void main(String[] args) {
    for (String arg : args) {
        log.info("arg {}", arg);
    }
  }
}
```
* 반면, 일반적으로는 사용성을 위해 실행 인자에 `key=value` 형태를 전달하게 되나 이를 직접 파싱해줘야하는 한계가 존재한다.
  * 예를 들어, 실행 인자에서 `endpoint=dev.com`을 전달했다고 하면 `=` 기호를 기준으로 좌항을 키 / 우항을 값으로 매핑하는 로직이 작성되어야 한다.

## 2025-04-08 Tue
### 스프링과 커맨드 라인 옵션 인자
* 상술한 내용과 같이, Java의 실행 인자로 전달된 문자열은 모두 키-값 쌍으로 매핑하는 작업이 필요하며 이는 번거롭고 에러 발생 가능성도 높은 축에 속한다.
* 때문에 **스프링 진영에서는 이를 쉽게 사용할 수 있도록 스프링만의 표준을 정립하였고, 이를 `커맨드 라인 옵션 인자`라는 용어로 지칭**한다.
* **`커맨드 라인 옵션 인자`는 `--` 기호를 사용하며, `--key=value` 형태의 인자는 아래와 같이 자동으로 키-값 쌍으로 인식하여 매핑**한다.
  * 이 때, `--username=me --username=you`와 같이 동일한 키에 대해 여러 값을 매핑할 수도 있다.
```java
public class CommandLineV2 {
  public static void main(String[] args) {
    for (String arg : args) {
        log.info("arg {}", arg);
    }
    
    ApplicationArguments appArgs = new DefaultApplicationArguments(args);
    log.info("SourceArgs = {}", List.of(appArgs.getSourceArgs()));
    log.info("NonOptionArgs = {}", appArgs.getNonOptionArgs());
    log.info("OptionNames = {}", appArgs.getOptionNames());
    
    Set<String> names = appArgs.getOptionNames();
    for (String name : names) {
      log.info("option arg {} = {}", name, appArgs.getOptionValues(name));
    }
    
    // 아래의 경우, myArg 인자를 전달하지 않았거나 --를 붙이지 않고 myArg=42 형태로 전달했다면 커맨드 라인 옵션 인자로 취급되지 않아 null이 반환된다.
    // List<String> values = appArgs.getOptionValues("myArg");
  }
}
```
* 이러한 **`커맨드 라인 옵션 인자`는 Java 표준이 아니며, 스프링 진영에서 개발 편의성을 위해 제공하는 기능으로 이해하는 것이 바람직**하다. 

## 2025-04-09 Wed
### 스프링 부트의 커맨드 라인 옵션 인자 지원
* **스프링 부트는 커맨드 라인 자체를 포함하여 커맨드 라인 옵션 인자를 활용할 수 있도록 `ApplicationArguments`를 빈으로 등록**해둔다.
  * 이 때, `ApplicationArguments`는 인터페이스이므로 스프링이 등록하는 것은 구현체에 해당한다.
  * 해당 빈 내부에 커맨드 라인으로 전달된 데이터를 저장하므로, 이를 주입 받을 수 있는 클래스라면 커맨드 라인을 통해 전달된 외부 설정을 사용할 수 있다.

## 2025-04-10 Thu
### 외부 설정 주입과 스프링의 추상화
* 상술한 과정에서 활용했던 운영 체제의 환경 변수와 Java 시스템 속성, 커맨드 라인 옵션 인자는 모두 외부 설정을 키-값 쌍으로 사용할 수 있도록 한다.
* 그러나 **키-값 쌍 형태를 갖는 외부 설정이라는 공통점이 있음에도, 그 유형에 따라 이를 사용하는 방법이 모두 다르다는 단점이 존재**한다.
  * 예를 들어, 운영 체제의 환경 변수로 주입된 외부 설정과 Java 시스템 속성으로 주입된 외부 설정을 읽는 방법이 모두 다르다.
  * 이러한 단점은 임의의 방식을 또 다른 방식으로 변경하고자 하는 수정 사항이 발생했을 때 애플리케이션 전역적인 수정을 유발할 잠재적인 위험성을 갖는다.
* 외부 설정이 어디에 위치하든지와 관계 없이 이를 일관성 있게, 유연하게 조회할 수 있다면 더 편리하고 견고한 애플리케이션 작성이 가능하다.
  * **스프링 진영에서는 이러한 문제를 해결하기 위해 `Environment`와 `PropertySource`라는 추상화 계층을 제공**한다.

## 2025-04-11 Fri
### PropertySource란?
* `PropertySource`는 스프링 코어에 포함되는 추상 클래스이며, 외부 설정의 종류에 따라 구현된 `XXXPropertySource` 구현체를 갖는다.
  * 예를 들어, `CommandLinePropertySource`나 `SystemEnvironmentPropertySource` 등이 있다.
* **스프링은 스프링 기반 애플리케이션이 로딩되는 시점에 필요한 `PropertySource` 구현체들을 생성하며, `Environment`에 연결하는 방식으로 동작**한다.
  * 덕분에 `Environment`는 여러 `PropertySource` 구현체에 접근하여 사용하는 것이 가능하다.

## 2025-04-12 Sat
### Environment란?
> 모든 외부 설정은 Environment만으로 접근이 가능하다.
* **`Environment` 역시 스프링 코어에 포함되며, 임의의 키-값 형태의 외부 설정에 종속되지 않고 일관된 방식으로 접근할 수 있도록 지원**한다.
  * 예를 들어, `env.getProperty([키])` 메소드 호출을 통해 임의의 키에 대응되는 값에 접근할 수 있다.
* **`Environment`는 내부적인 동작 과정에서 여러 `PropertySource`에 접근이 가능하며, 동일한 키 쌍이 존재할 경우 미리 정해진 우선 순위**를 따른다.
  * 다시 말해, 임의의 외부 설정 키에 대해 조회할 경우 `Environment`는 자신이 접근 가능한 `PropertySource`에 대해 우선 순위에 따라 조회한다.
* 익히 알려진 `application.properties` 역시 외부 설정에 해당하므로 `PropertySource`에 추가되어 `Environment`에 의한 접근이 가능하다.