# Spring Boot
## 2025-02-15 Sat
### 내장 톰캣 의존성
* `build.gradle`의 `dependencies` 블록을 다음과 같이 입력하는 것으로 내장 톰캣을 활용할 수 있다.
  * 해당 의존성을 명시할 경우, 애플리케이션 시작 시점에 해당 라이브러리에 접근하여 톰캣을 동작시킬 수 있다.
  * 즉, 별도의 톰캣 서버 설치 과정이 생략되고 `main()` 메소드로부터 톰캣을 실행할 수 있게 된다. 
```groovy
repositories {
  mavenCentral()
}

dependencies {
  // 스프링 MVC
  implementation 'org.springframework:spring-webmvc:6.0.4'
  
  // 내장 톰캣
  implementation'org.apache.tomcat.embed:tomcat-embed-core:10.1.5'
}
```
* 또한, 톰캣 의존성 내부에는 서블릿과 관련된 여러 코드가 포함되기에 서블릿과 관련된 기능을 바로 사용할 수도 있다.

## 2025-02-16 Sun
### 내장 톰캣을 코드로 제어하기
* 내장 톰캣은 톰캣 자체를 라이브러리로 포함하고, Java 코드로 이를 제어할 수 있게 하기에 아래와 같은 코드를 통해 톰캣을 명시적으로 제어할 수 있다.
```java
public class EmbedTomcatServletMain {
  public static void main(String[] args) throws LifecycleException {
    System.out.println("main");
    
    // 톰캣 설정하기
    Connector connector = new Connector();
    connector.setPort(8080); // 톰캣이 제공하는 커넥터를 활용하여 8080 포트에 연결한다.
    
    Tomcat tomcat = new Tomcat();
    tomcat.setConnector(connector);
    
    // 서블릿 등록하기
    Context context = tomcat.addContext("", "/");
    tomcat.addServlet("", "myServlet", new MyServlet());
    context.addServletMappingDecoded("/my-servlet", "myServlet"); // /my-servlet 경로 호출에 대해 myServlet을 실행한다.
    
    // 설정된 톰캣을 시작하기
    tomcat.start();
  }
}
```

## 2025-02-17 Mon
### 내장 톰캣 딥 다이브?
* 내장 톰캣을 코드로 제어할 경우, 톰캣을 위해 IDE로 복잡한 설정을 진행할 필요 없이 `main()` 메소드 호출만으로도 활용이 가능해진다.
  * 물론, 톰캣 서버를 별도로 설치하여 유지보수하는 작업 역시 불필요한 점에서도 큰 이점을 얻을 수 있다.
* 그러나 스프링 부트 차원에서 톰캣과 관련된 대부분의 작업을 자동화하므로, 실무에서 내장 톰캣을 위와 같이 직접 제어할 일은 사실상 없다고 볼 수 있다.
* 때문에 내장 톰캣의 세부적인 동작 원리까지 자세하게 이해할 필요는 없으며, 단순히 어떤 방식으로 동작하는지 개괄적으로 이해하기만 해도 무방하다.

## 2025-02-18 Tue
### 내장 톰캣과 스프링 컨테이너의 통합
* 상술한 코드와 앞서 다룬 프로그래밍적인 스프링 컨테이너 설정을 다음과 같이 조합할 경우 내장 톰캣 라이브러리와 스프링 컨테이너를 통합할 수 있다.
```java
public class EmbedTomcatWithSpringMain {
  public static void main(String[] args) throws LifecycleException {
    System.out.println("main");
    
    // 톰캣 설정하기
    Connector connector = new Connector();
    connector.setPort(8080); // 톰캣이 제공하는 커넥터를 활용하여 8080 포트에 연결한다.
    
    Tomcat tomcat = new Tomcat();
    tomcat.setConnector(connector);
    
    // 스프링 컨테이너 생성하기
    AnnotationConfigWebApplicationContext appCtx = new AnnotationConfigWebApplicationContext();
    
    // 스프링 컨테이너에 컨트롤러를 포함하는 Configuration 빈 등록하기
    appCtx.register(MyConfiguration.class);
    
    // 스프링 MVC 디스패처 서블릿 생성 및 스프링 컨테이너와 연결
    DistpatcherServlet dispatcher = DistpatcherServlet(appCtx);
    
    // 디스패처 서블릿 등록하기
    Context context = tomcat.addContext("", "/");
    tomcat.addServlet("", "dispatcher", dispatcher);
    context.addServletMappingDecoded("/", "dispatcher"); // / 경로 호출에 대해 디스패처 서블릿을 실행한다.
    
    // 설정된 톰캣을 시작하기
    tomcat.start();
  }
}
```
* 상술한 코드의 경우 서블릿 컨테이너 초기화 코드와 거의 유사한 것을 알 수 있다.
  * 다만, 큰 차이가 있다면 개발자가 명시적으로 `main()` 메소드를 실행하는지 또는 서블릿 컨테이너가 제공하는 초기화 메소드를 활용하는지 여부가 다르다.

## 2025-02-19 Wed
### 내장 톰캣을 활용하는 애플리케이션의 배포
* Java 애플리케이션의 `main()` 메소드를 실행하기 위해서는 우선 JAR 형태로 빌드할 필요가 있으며, `META-INF/MANIFEST.MF`에 이를 지정해주어야 한다.
  * 정확히는 해당 파일에 `main()` 메소드를 포함하는 클래스가 다음과 같이 명시되어야 한다.
```
Manifest-Version: 1.0
Main-Class: ga.injuk.embed.EmbedTomcatWithSpringMain
```
* 반면, 이러한 작업은 그래들에 다음과 같은 `task`를 명시한 후 `./gradlew clean buildJar` 명령어를 입력하는 것으로 손쉽게 진행할 수 있다.
```groovy
task buildJar(type: Jar) {
  manifest {
    attributes 'Main-Class': 'ga.injuk.embed.EmbedTomcatWithSpringMain'
  }
  with jar
}
```

## 2025-02-20 Thu
### JAR 파일의 한계
* 반면, 상술한 과정을 통해 만들어진 JAR 파일을 `java -jar [파일명].jar`로 실행하더라도 예외가 발생한다.
* 이를 확인하기 위해 우선 생성된 JAR 파일을 `jar -xvf [파일명].jar` 명령어로 unzip하여 분석해볼 필요가 있다.
  * 이 경우, unzip 결과에는 `META-INF`와 `ga` 폴더만이 존재하며 내장 톰캣이나 스프링 등과 같은 라이브러리와 관련된 코드는 존재하지 않는다.
  * 이는 기존에 WAR 파일을 unzip했을 때 `WEB-INF` 폴더 하위에 `classes`와 `lib`이 존재하여 이상 없이 실행되던 것과 대비된다.
* **WAR 파일과 달리 JAR 파일은 내부에 라이브러리 역할을 수행하는 JAR 파일을 포함할 수 없으며, 강제로 포함시키더라도 인식이 되지 않는다**.
  * 이는 JAR 파일이 갖는 스펙 차원의 한계이며, 그렇다고 WAS 상에서만 실행 가능한 WAR 파일을 사용하는 것 역시 권장되지 않는다.
* 가장 단순한 대안으로는 모든 JAR 파일을 구하여 `MANIFEST` 파일에 명시해주는 것이지만 이는 매우 번거롭고, 권장되지 않는 방법에 해당한다.

## 2025-02-21 Fri
### Fat JAR를 활용한 배포
* 상술한 JAR 파일의 한계를 극복하기 위해, JAR 안에 필요한 모든 클래스를 포함하는 방식을 고려해볼 수 있다.
  * 이러한 방식은 JAR가 JAR를 포함할 수 없지만 클래스는 얼마든지 포함할 수 있다는 점을 이용하며, `Fat JAR` 또는 `uber JAR` 라는 용어로 지칭한다.
* 예를 들어 라이브러리 용도로 사용되는 JAR 파일을 unzip한 결과인 클래스들을 새로운 JAR 파일에 모두 포함시킬 수 있다.
  * 결과물은 수많은 클래스를 포함하게 되며, `Fat JAR`라는 표현 역시 이로부터 기인한다.
* 그래들의 경우 다음과 같은 사용자 정의 태스크를 명시한 후 `./gradlew buildFatJar` 명령어를 입력하는 것으로 `Fat JAR`를 간단히 생성할 수 있다.
```groovy
task buildFatJar(type: Jar) {
  manifest {
    attributes 'Main-Class': 'ga.injuk.embed.EmbedTomcatWithSpringMain'
  }
  duplicatesStrategy = DuplicatesStrategy.WARN
  from {
    configurations.runtimeClasspath.collect {
      it.isDirectory() ? it : zipTree(it)
    }
  }
  with jar
}
```

## 2025-02-22 Sat
### Fat JAR 방식의 장단점
* `Fat JAR` 파일을 `jar -xvf [파일명].jar` 명령어로 unzip할 경우, 모든 라이브러리에 포함된 수많은 클래스가 포함된다는 점을 확인할 수 있다.
  * 바꿔 말해, `Fat JAR` 자체는 그 이름값을 할 정도로 용량이 크다.
* `Fat JAR`를 사용할 경우 단일 JAR 파일에 필요한 모든 클래스를 내장하게 되므로 배포와 실행 과정이 모두 단순화될 수 있다.
  * 또한, 톰캣과 같은 WAS가 라이브러리 형태로 JAR 파일에 포함되기에 별도의 WAS 설치가 필요 없어진다.
  * 나아가 그래들로부터 톰캣의 버전 관리가 가능하며, `main()` 메소드만 실행하면 되기에 복잡한 WAS 설정은 필요 없어진다.
* 반면, `Fat JAR` 역시 은탄환이 아니며 다음과 같은 여러 단점들을 수반한다.
  * 어떤 라이브러리가 포함되어 있는지 확인하기 어려워진다.
  * 여러 **라이브러리에 포함된 파일 이름의 중복을 해결할 수 없으며, 클래스와 리소스 이름이 같은 경우 무언가를 포기할 수 밖에 없다**.

## 2025-02-23 Sun
### 편리한 부트 클래스 작성하기
* 다음과 같은 부트 클래스는 이름 그대로 시작 과정을 처리해주는 역할을 담당하며, 코드 상 내장 톰캣 실행부터 스프링과의 연결까지 모든 것을 처리한다.
```java
public class MySpringApplication {
  public static void run(Class configClass, String[] args) {
    System.out.println("run args=" + List.of(args));

    // 톰캣 설정하기
    Connector connector = new Connector();
    connector.setPort(8080); // 톰캣이 제공하는 커넥터를 활용하여 8080 포트에 연결한다.

    Tomcat tomcat = new Tomcat();
    tomcat.setConnector(connector);

    // 스프링 컨테이너 생성하기
    AnnotationConfigWebApplicationContext appCtx = new AnnotationConfigWebApplicationContext();

    // 스프링 컨테이너에 컨트롤러를 포함하는 Configuration 빈 등록하기
    // 반면, **후술할 코드로부터 @ComponentScan 어노테이션이 할당된 클래스가 전달되므로 스프링 컨테이너 내부적으로 컴포넌트 스캔을 사용**하게 된다.
    appCtx.register(configClass);

    // 스프링 MVC 디스패처 서블릿 생성 및 스프링 컨테이너와 연결
    DistpatcherServlet dispatcher = DistpatcherServlet(appCtx);

    // 디스패처 서블릿 등록하기
    Context context = tomcat.addContext("", "/");
    tomcat.addServlet("", "dispatcher", dispatcher);
    context.addServletMappingDecoded("/", "dispatcher"); // / 경로 호출에 대해 디스패처 서블릿을 실행한다.

    // 설정된 톰캣을 시작하기
    try {
      tomcat.start();
    } catch (LifecycleException e) {
      throw new RuntimeException(e);
    }
  }
}
```
* 또한, 사용자 정의 어노테이션을 다음과 같이 작성한다.
  * 이 때, **`@ComponentScan`에 인자를 작성하지 않았으므로 해당 어노테이션이 명시된 클래스의 패키지와 모든 하위 패키지를 스캔 대상으로 적용**한다.
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ComponentScan
public @interface MySpringBootApplication {
}
```
* 이제 작성된 어노테이션을 다음과 같이 `main` 역할을 수행할 클래스에 할당한다.
  * **아래 코드로부터 `@MySpringBootApplication` 어노테이션이 할당된 클래스 자체를 설정으로 전달하므로 손쉬운 사용성을 제공**한다.
  * 때문에 **함께 협업하는 다른 개발자들 역시 동일한 방법을 활용하여 컴포넌트 스캔을 손쉽게 사용**할 수 있게 된다.
  * 이로 인해 내장 톰캣 실행과 스프링 컨테이너 생성, 그리고 디스패처 서블릿의 등록과 컴포넌트 스캔 모두 한 줄로 처리 가능하게 된다.
```java
@MySpringBootApplication
public class MySpringBootMain {
  public static void main(String[] args) {
    // 자기 자신을 설정으로 사용하며, @MySpringBootApplication 어노테이션은 내부적으로 @ComponentScan을 명시하므로 컴포넌트 스캔이 적용된다.
    MySpringApplication.run(MySpringBootMain.class, args);
  }
}
```

## 2025-02-24 Mon
### 스프링 부트의 편리함
* **상술한 모든 내용을 라이브러리화하여 배포할 경우, 그 자체가 스프링 부트**가 되는 것을 알 수 있다.
* 예를 들어, 스프링 부트의 메인 클래스는 일반적으로 다음과 같은 형태를 갖는다.
```java
@SpringBootApplication
public class MyApp {
  public static void main(String[] args) {
    SpringApplication.run(MyApp.class, args);
  }
}
```
* `@SpringBootApplication` 어노테이션은 `@ComponentScan`을 비롯한 여러 어노테이션을 내포한다.
  * 때문에 상술한 코드와 동일한 이유에서 `SpringApplication.run()` 메소드에 전달하여 스프링 설정 정보로 사용할 수 있도록 지원한다.
  * 다시 말해 **코드 한줄만으로 내장 톰캣과 스프링 컨테이너 생성 및 디스패처 서블릿의 연결과 컴포넌트 스캔까지 손쉽게 사용할 수 있도록 지원**한다.

## 2025-02-25 Tue
### 스프링 부트의 실행 과정
* 스프링 부트를 실행할 경우, 일반적으로 `main()` 메소드에서 `SpringApplication.run([설정 정보].class, args)` 형태의 메소드를 호출한다.
  * 이 과정에서 첫 번째 인자에 전달되는 것은 메인 설정 정보이며, 일반적으로 `@SpringBootApplication` 어노테이션이 할당된 현재 클래스를 전달한다.
  * 이를 통해 `@SpringBootApplication`에 포함된 `@ComponentScan` 기능을 손쉽게 사용할 수 있으며, 현재 패키지 및 하위 패키지를 스캔하게 된다.
* 이렇듯 **단순해보이는 코드 한 줄이지만 내부적으로는 수많은 과정이 처리되며, 대표적으로는 스프링 컨테이너의 생성과 WAS의 생성**을 꼽을 수 있다.
* 스프링 부트 내부적으로 처리되는 과정 중 스프링 컨테이너를 생성하는 코드는 `ServletWebServerApplicationContextFactory` 클래스에 포함된다.
  * 반면, 내장 톰캣을 생성하는 코드는 `TomcatServletWebServerFactory` 클래스에 포함된다.
  * 이렇듯 **스프링 부트는 앞서 직접 작성한 코드와 유사하게 스프링 컨테이너를 생성하고, 내장 톰캣을 생성한 후 둘을 연결하는 과정을 암시적으로 처리**한다.
* 반면, **스프링 부트는 너무 방대한 라이브러리이기에 이를 이해하기 위해 모든 코드를 하나하나 분석하는 것은 실용적이지 않을 수 있다**.
  * 대신 **스프링 부트가 어떻게 동작하는지 전체적인 흐름과 개념을 이해하고, 반드시 필요한 부분의 코드만 분석해도 무방**하다.

## 2025-02-26 Wed
### 스프링 부트 애플리케이션의 빌드 결과물
* `./gradlew clean build`와 같은 명령어로 빌드할 경우, `[앱 이름]-[버전]-SNAPSHOT.jar`와 같은 형태의 결과물이 생성된다.
  * `[앱 이름]-[버전]-SNAPSHOT-plain.jar`도 함께 생성되지만, 이는 라이브러리는 물론이고 `MANIFEST.MF` 파일조차 포함되지 않은 소스 코드 묶음이다.
* 생성된 JAR 파일의 압축을 해제할 경우, 크게 다음과 같은 세 개의 폴더가 포함된다.
  1. `META-INF`: 메인 클래스를 명시하기 위한 `MANIFEST.MF` 파일이 위치한다.
  2. `BOOT-INF`: 개발자가 직접 작성한 클래스들을 포함하는 `classes` 폴더와 JAR 형태의 외부 라이브러리를 의미하는 `lib` 폴더가 위치한다.
  3. `org/springframework/boot/loader`: 스프링 부트 `main()` 메소드를 포함하는 `JarLauncher.class`라는 이름의 클래스가 위치한다.
* 추가적으로, `BOOT-INF`의 경우 외부 라이브러리 경로를 의미하는 `classpath.idx`와 스프링 부트의 구조 경로를 의미하는 `layers.idx`를 포함한다.
* 이 때, **Java 언어 스펙상 JAR는 JAR를 포함할 수 없음에도 `BOOT-INF/lib`에 라이브러리에 해당하는 JAR 파일들이 포함되는 점에 주목**해야 한다.
  * 이러한 점에서 미루어 보았을 때, `Fat JAR`와는 다른 구조를 채택하는 것을 확인할 수 있다.

## 2025-02-27 Thu
### 스프링 부트의 실행 가능한 JAR 파일
* 앞서 다루었던 `Fat JAR`는 하나의 JAR 파일에 소스 코드는 물론 모든 라이브러리의 클래스를 포함했기에 여러 단점을 수반했다.
  * 예를 들어, 어떤 라이브러리가 포함되어 있는지 확인하기 어렵고 파일명 중복을 해결할 수 없다.
* **스프링 부트는 이를 해결하기 위해 JAR 파일 내부에 JAR 파일을 포함할 수 있는 특별한 JAR를 정의하고, 이를 실행할 수 있도록 했다**.
  * **Java 스펙상, 이렇듯 JAR 파일이 또 다른 JAR 파일을 품는 구조는 기본적으로 허용되지 않는다**.
  * 그러나 스프링 부트는 이를 가능하게 하는 `Executable JAR` 개념을 도입했고, 실행 가능한 JAR 파일을 생성할 수 있도록 지원한다.
* 스프링 부트의 실행 가능한 JAR는 `Fat JAR`의 문제를 각각 해결한다.
  1. JAR 파일 내부에 또 다른 JAR 파일이 포함되므로, 어떤 라이브러리가 포함되어 있는지 확인하기 쉽다.
  2. JAR 파일 내부의 여러 경로에 또 다른 JAR 파일이 있더라도 이를 인식할 수 있으므로, 파일명 중복 역시 해결 가능하다.
* 물론, 이러한 **실행 가능한 JAR는 Java 스펙 상의 표준은 아니며 스프링 부트에서 도입된 개념에 해당**한다.

## 2025-02-28 Fri
### 실행 가능한 JAR 파일의 내부 구조
* 스프링 부트의 빌드 결과물인 실행 가능한 JAR 파일은 크게 다음과 같은 구조를 갖는다.
  1. `META-INF`
  2. `org/springframework/boot/loader`
  3. `BOOT-INF`
* `META-INF`는 사실상 가장 중요하며, Java 진영의 특징 상 정해진 JAR 파일에서 필수적인 역할을 담당한다.
  * 예를 들어, **해당 폴더의 `MANIFEST.MF`는 Java 스펙 상 메인 클래스의 위치를 명시**한다.
  * 때문에 스프링 진영에서 무엇을 만들던 간에 해당 규격을 따라야 하며, 실행 가능한 JAR 역시 JAR 파일의 스펙을 준수하기 위해 해당 폴더를 포함하고 있다.
* `org/springframework/boot/loader` 폴더는 `JarLauncher.class` 파일을 포함하며, 스프링 부트의 실행 가능한 JAR 파일의 핵심이 된다.
* `BOOT-INF`는 다시 `classes` 폴더와 `lib` 폴더를 포함하며, 이외에도 `classpath.idx`와 `layers.idx` 파일을 포함한다.
  * `classes` 폴더의 경우, 개발자가 직접 작성한 소스 코드로부터 생성된 `class` 파일 및 리소스 파일이 포함된다.
  * `lib` 폴더의 경우, 해당 스프링 부트 애플리케이션이 의존하는 외부 라이브러리들의 JAR 파일을 포함한다.
  * `classpath.idx`의 경우, 외부 라이브러리의 모음으로 이해할 수 있다.
  * `layers.idx`의 경우, 스프링 부트의 구조 정보로 이해할 수 있다.

## 2025-03-01 Sat
### 실행 가능한 JAR 파일의 META-INF와 MANIFEST.MF 파일
* **`java -jar [파일명].jar` 명령어를 실행할 경우, Java 진영에서 결정한 스펙에 따라 다음과 같은 과정에 따라 기반으로 동작**하게 된다.
  1. 우선 `META-INF/MANIFEST.MF` 파일을 찾는다.
  2. 해당 파일에 포함된 `Main-Class` 정보를 찾아 메인 클래스 역할을 담당하는 클래스를 조회한다.
  3. 해당 클래스에 포함된 `main()` 메소드를 실행한다.
* 반면, 스프링 부트의 실행 가능한 JAR 파일을 확인할 경우 메인 클래스는 `org.springframework.boot.loader.JarLauncher`라고 명시되어 있다.
  * 이는 개발자가 직접 작성한 코드가 아니며, 오히려 실제 메인 클래스는 `Start-Class` 항목에 명시된다.
  * **해당 클래스는 스프링 부트가 빌드 시에 명시하는 정보로, 빌드된 JAR 파일에는 실제로도 `JarLauncher` 클래스가 포함**된다.
  * **스프링 부트는 스펙과 달리 JAR 파일 내부에 JAR 파일을 포함하므로 이를 조회하는 기능이 필요하며, 이러한 일을 `JarLauncher` 클래스가 담당**한다.
  * 때문에 우선 필요한 모든 JAR 파일을 조회한 후, `Start-Class`에 명시된 `main()` 클래스를 호출하는 방식으로 동작하게 된다.
* 반면, `MANIFEST.MF` 파일에 명시된 다른 정보들 역시 스프링 부트가 빌드 시에 명시하는 정보들이며, 각각 다음과 같은 의미를 갖는다.
  1. `Start-Class`: 개발자가 작성한 코드 중, 실제로 메인 클래스 역할을 수행할 클래스를 명시한다.
  2. `Spring-Boot-Version`: 스프링 부트의 버전 정보가 명시된다.
  3. `Spring-Boot-Classes`: 개발자가 직접 작성한 클래스들의 경로 정보가 명시된다.
  4. `Spring-Boot-Lib`: 라이브러리의 경로 정보가 명시된다.
  5. `Spring-Boot-Classpath-Index`: 외부 라이브러리 모음과 관련된 내용을 포함하는 파일의 경로 정보가 명시된다.
  6. `Spring-Boot-Layers-Index`: 스프링 부트의 구조와 같은 내용을 포함하는 파일의 경로 정보가 명시된다.
* 이 때, **`Main-Class`를 제외한 나머지 정보는 모두 Java 스펙에 명시된 표준이 아니며 스프링 부트가 임의로 사용하는 정보에 해당**한다.