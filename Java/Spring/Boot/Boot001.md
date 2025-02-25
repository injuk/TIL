# Spring Boot
## 2025-01-16 Thu
### 스프링 부트의 특징
* 익히 다뤄온 스프링은 Java 진영의 사실상 표준이나, 그 오랜 역사로 인해 기능이 너무 많고 광범위한 특징을 갖는다.
  * 이는 새로운 개발자들의 진입장벽으로 기능하며, 너무 많은 유연성을 제공하므로 기술을 선택하는 것 자체가 어려워졌다.
* **스프링 부트는 무겁고 불편하다는 스프링의 인식을 개선하기 위해 고안되었으며, 스프링을 쉽고 편리하게 사용할 수 있도록 지원**한다.
  * 실무 관점에서도 최근 대부분의 프로젝트는 스프링 부트를 사용하게 되었으므로, 스프링은 스프링 부트를 통해 비로소 완성되었다고 볼 수도 있다.
* 스프링 부트는 크게 다음과 같은 핵심 기능을 제공한다.
  1. 내장 서버: 톰캣 등의 별도 서버 설치 없이도 손쉽게 웹 애플리케이션을 동작시킬 수 있도록 지원한다.
  2. 자동 라이브러리 관리: best practice를 기반으로 수많은 라이브러리들을 자동으로 선택하고 관리한다.
  3. 자동 구성: 복잡한 스프링 설정을 자동화하여 개발자들이 최대한 쉽고 빠르게 애플리케이션을 개발할 수 있도록 지원한다.
  4. 외부 설정: 개발 및 운영 환경 등 서로 다른 환경에서 애플리케이션을 손쉽게 개발할 수 있도록 편리한 외부 설정 조회 기능을 지원한다.  
  5. 모니터링 및 관리 기능: 애플리케이션이 제공하는 수많은 지표들을 자동으로 수집하고 관리할 수 있는 기능을 제공한다.
* 그러나 **스프링 부트는 편리하지만 여전히 너무 많은 기능을 제공하기에, 실무에 맞는 적절한 정도의 학습이 필수적**이다.
  * 즉, 너무 깊거나 너무 넓게 학습하지 않도록 주의를 기울일 필요가 있다.

## 2025-01-17 Fri
### 스프링의 등장
* 초기 Java 진영에는 표준 기술로 `Java Beans`, 즉 EJB라는 개념이 있었다.
  * EJB는 트랜잭션 관리와 분산 기술 등 이론적인 토대가 잡혀 있는 것처럼 보였고, 무엇보다 표준 기술이라는 점에서 큰 관심을 얻었다.
* 그러나 문제는 가격으로, 당시에 EJB를 사용하기 위해서는 필요한 소프트웨어 설치에 서버당 수천만원의 비용이 소모되었다.
* 또한, EJB는 복잡성이 매우 큰 기술이었고 무엇보다 애플리케이션이 EJB에 강결합된다는 단점을 수반했다.
  * 순수한 Java 코드의 사용을 권장하는 POJO 역시 이에 반발해서 나온 것으로 이해할 수 있다.
* 어찌되었건 EJB는 표준 기술이었기에 금융권이나 SI 시장에서 자주 채택되었고, 이에 반발한 SI 기업 출신의 개발자 로드 존슨이 스프링 프레임워크를 제안하는 데에 이르렀다.
  * 로드 존슨은 EJB 없이도 확장성 있는 고품질의 애플리케이션을 개발할 수 있음을 책으로 서술했고, 현재의 스프링 핵심 개념은 모두 해당 책에서 시작된 것으로 봐도 무방하다.
  * 결과적으로는 스프링이 EJB 컨테이너를 대체하게 되었고, 현재에는 스프링이 Java 진영의 사실상 표준으로 자리잡게 되었다.
* 스프링은 로드 존슨이 2002년에 출간한 책을 시작으로 2003년에 1.0 버전이 출시되었으며, 현재까지도 생태계를 키워오고 있다.

## 2025-01-18 Sat
### 스프링 프레임워크가 갖는 특징
* 스프링 프레임워크는 DI 컨테이너에 더해 MVC나 DB 접근 기술 등 수많은 기능을 제공하는 것으로 개발자들이 자주 접하는 다양한 문제의 해결에 직접적으로 기여한다.
  * 또한, 이 과정에서 다양한 라이브러리를 편리하게 사용할 수 있도록 통합해준다.
  * 때문에 스프링을 사용하는 개발자의 생산성은 크게 높아지고, 점차 넓은 범위에서 스프링이 사용되기에 이르렀다.
* 스프링 프레임워크는 현재에 이르러 Java 진영의 사실상 표준으로 자리잡았으며, 기존 표준 기술인 EJB를 완전히 대체하는데에 성공했다.
* 스프링은 지금까지도 계속해서 기능이 확장되어오고 있으며, 무엇보다도 스프링을 중심으로 한 스프링 시큐리티 등의 별도의 프로젝트가 추가되는 방식으로 그 생태계가 점차 거대해지고 있다.

## 2025-01-19 Sun
### 스프링 프레임워크의 확장과 관련된 문제
> 스프링에 의해 EJB 지옥에서 해방된 개발자들이 마주친 것은 아이러니하게도 스프링의 설정 지옥이었다.
* 상술한 바와 같이 스프링과 그 생태계가 점점 거대해짐에 따라 관련된 기능 역시 많아졌으며, 기저에 필요한 라이브러리도 크게 늘어났다.
* 즉, 스프링으로 신규 프로젝트를 시작할 때의 설정 과정은 점차 번거로워지고 있으며 이는 곧 스프링으로 신규 프로젝트를 시작하는 것이 점점 어려워지는 것과도 같다.
  * 극단적으로, 스프링은 자기 자신을 활용한 신규 프로젝트를 시작하기도 전에 포기하는 개발자가 점차 늘어나는 상황에 직면하게 된다.
  * 이러한 상황이 악화되던 2010년 무렵의 스프링은 개발자들에 의해 무겁고, 불편한 프레임워크라고 여겨지곤 했다.
  * 이는 스프링 프레임워크 자체적으로 개발자들에게 많은 선택의 여지를 남겨두기 때문으로, 숙련된 개발자에게는 유용하지만 일반적인 개발자들에게는 그 자체로 진입 장벽으로서 기능하곤 했다.

## 2025-01-20 Mon
### 스프링 부트의 등장
* 부트, 부팅은 최소한의 개입으로 완전히 동작하는 것을 의미하며 곧 무언가를 시작하기 전에 모든 준비를 마치겠다는 의미를 갖는다.
  * 다시 말해 스프링 부트 역시 시작을 위한 복잡한 설정을 직접 해결하며, 덕분에 개발자는 새로운 애플리케이션을 쉽고 빠르게 시작할 수 있게 된다.
  * 결국 스프링 부트는 스프링을 편리하게 사용할 수 있도록 지원하는 역할을 담당하지만, 최근에는 사실상 기본이 되었다.

## 2025-01-21 Tue
### 스프링 부트의 다섯 가지 핵심 기능
> 스프링 부트는 수많은 기능을 갖지만, 그 중 가장 핵심적인 것을 약 다섯 가지로 추려볼 수 있다.
* 스프링 부트는 톰캣과 같은 웹 서버를 내장하므로, 별도의 웹 서버를 설치하지 않고 WAS 역할을 수행할 수 있다.
* 스프링 부트는 손쉬운 구성을 위한 스타터 의존성을 제공하고, 스프링과 외부 라이브러리 간의 버전을 자동으로 관리해준다.
* 스프링 부트는 프로젝트의 시작에 필요한 스프링, 그리고 외부 라이브러리와 관련된 빈을 자동으로 등록한다.
* 스프링 부트는 환경에 따라 달라질 필요가 있는 외부 설정을 공통화한다.
* 스프링 부트는 애플리케이션의 상태를 모니터링하기 위한 여러 메트릭에 더불어 상태 확인 기능을 제공한다.

## 2025-01-22 Wed
### 스프링 기술 스택 결정하기
* 스프링 기반의 애플리케이션을 개발하는 경우, 필수적이라고 볼 수 있는 스프링과 스프링 부트의 조합에 더해 스프링 데이터나 스프링 세션 및 스프링 클라우드 등의 스프링 생태계의 여러 기술들을 선택적으로 고려할 수 있다.
* 중요한 것은 스프링 부트를 포함한 모든 기술 스택은 스프링을 중심으로 동작하며, 스프링을 선택적으로 사용할 수는 없다는 점에 있다.
  * 스프링 부트 역시 본질에 해당하는 스프링을 쉽게 사용하도록 지원하는 도구에 불과하다.
  * 다만, 단순한 도구로 취급하기엔 막강한 편의 기능을 제공하기에 사실상 필수로 취급하는 것이 권장된다.

## 2025-01-23 Thu
### 스프링 부트 학습의 중요성
* 스프링 부트로 인해 스프링 기반 애플리케이션 개발이 편리해진 것은 사실이지만, 문제는 스프링 부트가 너무 많은 것을 자동화한다는 점에 있다.
  * 즉, 어떠한 이슈가 발생했을 때 그 원리를 이해하지 못한 경우 트러블슈팅에 손을 못대는 경우가 발생할 수 있다.
* 최소한 스프링 부트가 어떠한 원리에 의해 동작하는지 이해하는 것은 필수적이며, 이를 통해 스프링 부트와 관련된 복잡한 이슈의 원인을 파악하고 트러블슈팅을 시도할 수 있게 된다.
* 또한, 스프링 부트는 그 자체로도 수많은 편의 기능을 제공하기 때문에 스프링 부트를 깊게 이해할수록 개발 시간을 단축하는 효과를 얻게 된다.
  * 즉, 바퀴를 재발명할 필요가 없다!  

## 2025-01-24 Fri
### 전통적인 방식과 최근 방식의 배포
* **전통적인 방식으로 Java 웹 애플리케이션을 개발하는 경우, 우선 톰캣과 같은 WAS를 서버에 설치하는 것이 전제**되어야 했다.
  * 이후 애플리케이션이 WAS에서 동작할 수 있도록 서블릿 스펙에 맞추어 코드를 작성하고, WAR로 빌드한 결과물을 WAS에 배포하는 방식으로 개발을 진행했다.
* 반면, **최근 방식의 경우 스프링 부트가 자체적으로 톰캣을 내장 라이브러리 형태로 포함한다는 점에서 전통적인 방식과는 큰 차이**가 있다.
  * 즉, **이는 곧 애플리케이션 코드 상에 톰캣과 같은 WAS가 라이브러리 형태로 저장되는 것**과 같다.
  * 때문에 개발자는 단순히 코드를 작성한 후 JAR로 빌드하고, 이를 원하는 위치에서 실행시키기만 하는 것으로 WAS를 함께 실행할 수 있게 되었다.
  * 이는 극단적으로 바꿔 말해 개발자는 단지 `main()` 메소드만을 호출하며, WAS 설치 및 설정 등의 복잡한 작업을 더 이상 수행할 필요가 없음을 의미한다.

## 2025-01-25 Sat
### 전통을 체험하기
```
> 웹 애플리케이션 개발 과정에 있어 전통적인 프로세스를 따라보는 것은 스프링 부트의 현재를 더 깊이 있게 이해하는 데에 도움을 준다. 
```
* 스프링 부트를 최근에 학습한 개발자들은 WAR로 빌드하여 WAS에 배포하는 방식에 익숙하지 않지만, 이러한 방식을 알아둘 가치는 충분하다.
* 과거의 개발 프로세스를 이해하는 것은 곧 현재의 방식이 어떻게 발전해왔고, 이를 왜 사용해야하는지에 대해 더 깊이 있게 이해할 수 있도록 하는 초석이 된다.
  * 예를 들어, 서블릿 컨테이너를 설정하고 스프링 컨테이너와 디스패처 서블릿을 각각 생성하여 스프링 MVC와 연결하는 작업 등을 경험해볼 필요가 있다.

## 2025-01-26 Sun
### 톰캣용 서블릿 기반 웹 애플리케이션 작성하기
* 서블릿과 WAR를 활용하여 톰캣에서 동작할 수 있는 웹 애플리케이션을 개발하는 경우, 아래와 같이 gradle을 설정해줄 필요가 있다.
```groovy
plugins {
  id 'java'
  id 'war'
}

// ...중략

dependencies {
  implementation 'jakarta.servlet:jakarta.servlet-api:6.0.0' // 서블릿 의존성
}
```
* 이 떄, **웹 애플리케이션에서 사용할 메인 페이지인 `index.html`은 `src/main/webapp` 경로에 작성**한다.
* 반면, 아무리 간단한 HTML 파일이더라도 이를 톰캣과 같은 WAS의 서블릿에서 동작시키기 위해서는 서블릿 스펙에 맞추어 개발할 필요가 있다.
  * 이러한 **서블릿은 `src/main/java` 하위의 임의의 패키지에 `HttpServlet` 클래스를 아래와 같이 확장하는 것으로 작성**할 수 있다.
```java
/**
 * http://localhost:8080/test 호출시 아래 서블릿이 동작하도록 구현  
 */
@WebServlet(urlPatterns = "/test")
public class MyServlet extends HttpServlet {
  @Override
  protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    System.out.println("MyServlet.service");
    resp.getWriter().println("test"); // 클라이언트의 요청에 대해 'test' 라는 문자열로 응답한다.
  }
}
```

## 2025-01-27 Mon
### WAR의 구조
* 사용 중인 빌드 도구에 따라 애플리케이션을 WAR로 빌드해야 하며, 상술한 설정의 경우 gradle의 `war` 플러그인에 의해 빌드가 가능하다.
  * 예를 들어 래퍼를 사용하는 경우, `./gradlew build` 명령어를 통해 WAR를 빌드할 수 있다.
  * 이 때, 빌드 결과물은 IntelliJ를 기준으로 `build/libs` 폴더 하위에 `.war` 확장자를 갖는 WAR 파일로 생성된다.
  * 반면, gradle의 `war` 플러그인은 해당 프로젝트를 빌드할 때 결과물을 WAR 파일로 생성하도록 동작한다. 
* 생성된 WAR 파일은 필요한 경우 `jar -xvf [파일명].war` 명령어로 풀어볼 수 있으며, 다음과 같은 파일을 포함한다.
  1. `index.html`: 메인 페이지 역할을 담당할 HTML로, 앞서 `webapp` 폴더에 작성한 HTML과 같다.
  2. `META-INF`: Java의 설정과 관련된 파일들이 위치한다.
  3. `WEB-INF`: `WEB-INF/classes` 경로에 패키지 구조에 따라 바이트코드로 변환된 소스 코드가, `WEB-INF/lib`에는 라이브러리용 JAR가 위치한다.
* 이 때, `WEB-INF` 하위 경로를 더 자세히 분류하면 다음과 같다.
  1. `classes`: 실행 대상 클래스들이 위치하는 경로에 해당한다.
  2. `lib`: 라이브러리 용도의 JAR 파일들이 위치하는 경로에 해당한다.
  3. `web.xml`: 생략 가능하며, 웹 서버의 배치 설정 파일에 해당한다.
* 이렇듯 **`WEB-INF` 폴더의 하위 경로는 Java 클래스 및 관련된 라이브러리와 설정 정보가 위치하며, 그 외는 HTML 등의 정적 리소스를 위해 사용**한다.

## 2025-01-28 Tue
### JAR와 WAR의 차이
* Java의 경우, **여러 클래스 및 관련된 리소스를 묶은 단위인 JAR**라는 압축 파일을 생성할 수 있다.
  * 이러한 **JAR는 `java -jar [파일명].jar` 형태로 JVM 상에서 직접 실행되거나, 다른 곳으로부터 참조되는 라이브러리 형태로 사용**될 수 있다.
  * 반면, **직접 실행 가능한 JAR의 경우 `main()` 메소드가 필수적이며 `MANIFEST.MF` 파일에 메인 메소드를 포함하는 클래스를 명시**해야 한다.
* 반면, **WAR의 경우 WAS에 배포하기 위해 사용되는 파일로 JVM 상에서 실행되는 JAR와 달리 WAS 상에서 실행이 가능**하다.
  * 물론 WAS 역시 Java 환경에서 실행되며, WAR는 이러한 WAS 상에서 실행되는 것으로 이해할 수 있다.
  * 이러한 **WAR 파일은 정해진 구조를 반드시 준수해야 하며, WAS 상에서 실행되기 위한 정적 리소스 등을 모두 포함하기에 JAR보다 복잡한 구조**를 갖는다.

## 2025-01-29 Wed
### WAS에 WAR를 배포하기
* 톰캣과 같은 WAS에 WAR 파일을 배포하기 위해서는 우선 톰캣을 종료하고, 톰캣이 설치된 폴더에 위치한 `webapps` 하위 폴더를 모두 삭제해주어야 한다.
  * 이 때, 톰캣을 종료하기 위해서는 톰캣 폴더에 위치한 쉘 스크립트인 `shutdown.sh`을 활용한다.
  * 톰캣을 처음 설치한 경우, `webapps` 폴더에는 톰캣 측에서 준비한 기본적인 리소스가 포함되나 이는 제거해도 무방하다.
* 그 이후에는 빌드된 WAR 파일을 톰캣이 설치된 폴더에 위치한 `webapps`에 `ROOT.war`라는 이름으로 붙여넣는다.
  * 이 때, 옮겨진 WAR 파일의 이름은 반드시 대문자로 표기된 `ROOT`여야 한다.
* 모든 준비가 완료된 경우, 톰캣 폴더에 위치한 쉘 스크립트인 `startup.sh`을 활용하여 톰캣을 실행한다.
  * 이 때, 실행된 웹 애플리케이션의 로그 정보는 톰캣이 설치된 폴더의 하위에 위치한 `logs`에 `catalina.out` 형태로 저장된다.
  * 또한, 톰캣은 실행되는 과정에서 `ROOT.war`의 압축을 자동으로 해제하여 사용한다.
* 그러나 이러한 배포 과정은 운영 환경에서는 감내할 수 있으나, 개발 단계에서는 계속해서 상기 과정을 반복하는 것이 번거로울 수 있다.
  * 때문에 IntelliJ나 Eclipse와 같은 IDE는 이러한 번거로운 과정을 상당수 자동화하는 기능을 제공한다.

## 2025-01-30 Thu
### 서블릿 컨테이너 초기화의 필요성
* WAS를 실행하는 시점에 필터와 서블릿을 등록하거나, 스프링 관련 설정을 처리하는 등 여러 초기화 작업이 필요할 수 있다.
  * 예를 들어, 스프링을 사용하는 경우 스프링 컨테이너를 생성한 후 서블릿과 스프링을 연결하는 디스패처 서블릿을 등록해야 한다.
* WAS는 이러한 작업을 위해 초기화 기능을 제공하며, 이를 활용하는 것으로 WAS 실행 시점에 필요한 초기화 과정을 진행할 수 있다.
  * 기존에는 이러한 작업을 `web.xml`을 활용하여 진행하였으나, 최근에는 서블릿 스펙 자체적으로 Java 코드를 활용한 초기화 기능을 지원한다.

## 2025-01-31 Fri
### ServletContainerInitializer 인터페이스
* 서블릿은 `ServletContainerInitializer`라는 서블릿 컨테이너 초기화 기능을 위한 인터페이스를 다음과 같이 제공한다.
  * 이 때, 서블릿 컨테이너는 실행 시점에 초기화용 메소드인 `onStartup()`을 호출한다.
  * 이를 통해 애플리케이션에 필요한 기능들을 사전에 초기화하거나 등록할 수 있게 된다.
```java
package jakarta.servlet;

public interface ServletContainerInitializer {
	public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException;
}
```
* 이 때, `ServletContainerInitializer` 인터페이스의 `onStartUp()`메소드의 인자는 각각 다음과 같은 의미를 갖는다.
  1. `Set<Class<?>> c`: 더 유연한 초기화 기능을 제공하며, `@HandleTypes` 어노테이션과 함께 사용된다.
  2. `ServletContext ctx`:  서블릿 컨테이너 자체의 기능을 제공하며, 이를 통해 필터 또는 서블릿 등을 등록할 수 있다.

## 2025-02-01 Sat
### ServletContainerInitializer 등록하기
> ServletContainerInitializer 인터페이스를 구현하는 초기화 클래스를 생성했더라도 톰캣은 인식할 수 없으므로, 이를 등록해줄 필요가 있다.
* `src/main/resources/META-INF/services` 경로에 `jakarta.servlet.ServletContainerInitializer` 라는 이름의 파일을 생성한다.
  * 이 때, `jakarta.servlet.ServletContainerInitializer`은 `ServletContainerInitializer`의 패키지 경로를 의미한다.
  * 이러한 명명법은 사실상 규약이라고 이해할 수 있으며, 때문에 `META-INF`나 `services` 등의 경로에 대해 오타가 있을 경우 톰캣은 이를 인식할 수 없다. 
* 생성된 파일에는 다음과 같이 개발자가 직접 작성한 `ServletContainerInitializer`의 이름을 패키지 경로와 함께 작성한다.
```text
ga.injuk.study.container.MyInitializerV1
```
* 톰캣이 실행될 때 해당 경로의 파일을 읽어들이며, 개발자가 작성한 `ServletContainerInitializer` 구현체의 `onStartup()` 메소드를 호출한다.

## 2025-02-02 Sun
### 프로그래밍 방식의 서블릿 등록
* 서블릿 컨테이너는 `애플리케이션 초기화`라는 용어로 지칭할 수 있는, 더 유연한 초기화 기능을 지원한다.
  * 이 때, 해당 용어는 정식 용어가 아님에 주의해야 한다.
* 이러한 애플리케이션 초기화 기능을 활용하기 위해서는 아래와 같은 인터페이스의 작성이 필수적이다.
```java
public interface ApplicationInit {
    void onStartup(ServletContext servletContext);
}
```
* 이제 상술한 인터페이스를 기반으로 아래와 같은 구현체 클래스를 작성하는 것으로 서블릿을 수동으로 등록한 후 적절히 매핑할 수 있다.
```java
public class ApplicationInitServlet implements ApplicationInit {
    @Override
    public void onStartup(ServletContext servletContext) {
        // 순수 서블릿 코드 등록하기
        ServletRegistration.Dynamic myServlet = servletContext.addServlet("myServlet", new MyServlet());
        myServlet.addMapping("/my-servlet");
    }
}
```

## 2025-02-03 Mon
### 서블릿을 등록하는 두 가지 방법
* 서블릿을 등록하는 방법은 크게 다음과 같은 두 가지 방식으로 나뉘며, 각기 다른 장단점을 갖는다.
  1. `@WebServlet` 어노테이션을 활용한 등록
  2. `ServletContext`를 활용한 프로그래밍 방식의 등록
* 어노테이션 방식은 편리하다는 장점을 갖지만, 마치 하드코딩된 것처럼 동작하기에 애플리케이션에 유연성을 부여하기 어렵다.
* 반면, 프로그래밍 방식의 등록은 더 많은 코드 작성이 필요하므로 번거롭고 불편하지만 유연성 측면에서 훨씬 더 큰 이점을 갖는다.
  * 예를 들어, 서블릿의 매핑 경로 등록하는 과정에서 필요에 따라 외부 설정을 읽어들이거나, `if` 분기를 활용한 매핑이 가능하다.

## 2025-02-04 Tue
### 프로그래밍 방식의 서블릿 컨테이너 초기화 로직 호출하기
* 상술한 바와 같이 `ApplicationInit` 인터페이스를 구현한 경우, 아래와 같이 `@HandleTypes` 어노테이션을 활용하여 구현체를 인자로 전달받을 수 있다.
  * 아래 예시의 경우, 인자 `c`에 `ApplicationInit` 인터페이스를 구현한 모든 클래스의 정보가 전달된다.
  * 이 때, 클래스 정보가 전달될 뿐 실제 인스턴스가 전달되는 것이 아님에 주의해야 한다.
```java
@HandleTypes(ApplicationInit.class)
public class MyContainerInitializer implements ServletContainerInitializer {
    @Override
    public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException {
        for (Class<?> clazz : c) {
            try {
              ApplicationInit applicationInit = (ApplicationInit) clazz.getDeclaredConstructor().newInstance();
              applicationInit.onStartup(ctx);
            } catch (Exception e) {
                throw e;
            }
        }
    }
}
```
* 또한, 이렇게 작성한 `MyContainerInitializer` 클래스의 이름은 `jakarta.servlet.ServletContainerInitializer` 파일에 추가해주어야 한다.

## 2025-02-05 Wed
### 서블릿 컨테이너의 초기화 과정
* 상술한 코드에서, `@HandleTypes` 어노테이션에 애플리케이션 초기화 인터페이스를 명시하는 것으로 필요한 모든 구현체의 클래스 정보를 활용할 수 있다.
  * 예를 들어, `Set<Class<?>> c` 인자에는 해당 애플리케이션 초기화 인터페이스를 구현하는 모든 구현체의 클래스 정보가 전달된다.
  * 이렇듯 서블릿 컨테이너 초기화를 위한 `ServletContainerInitializer`는 필요한 클래스 정보를 모두 인자로 전달 받는 것을 기반으로 동작한다.
* 이 때, 전달되는 정보는 구현체 인스턴스가 아닌 클래스 정보에 불과하므로 의도한 동작을 위해서는 반드시 명시적인 인스턴스화 과정이 전제되어야 한다.
  * 상술한 코드에서, `clazz.getDeclaredConstructor().newInstance()`는 리플렉션을 활용한 객체 생성 과정에 해당한다.
* 상술한 코드에서, `applicationInit.onStartup(ctx)`와 같이 애플리케이션 초기화 코드를 명시적으로 호출하는 과정에서 `ctx` 인자를 함께 전달한다.

## 2025-02-06 Thu
### WAS의 실행과 서블릿 컨테이너 초기화, 애플리케이션 초기화
* WAS가 실행되는 과정에서 서블릿 컨테이너가 초기화되며, 이 때 초기화되는 기준은 다음의 조건을 모두 충족하는 구현체에 해당한다.
  1. `ServletContainerInitializer` 인터페이스를 확장하는 클래스
  2. `resources/META-INF/services/jakarta.servlet.ServletContainerInitializer`에 작성된 클래스
* 또한, 이 과정에서 `@HandleTypes` 어노테이션에 명시된 인터페이스의 모든 구현체가 조회되어 상술한 조건에 맞는 구현체에 전달된다.
  * 일반적으로, 이렇게 전달된 구현체를 활용하여 애플리케이션 초기화에 필요한 기능들을 호출하도록 애플리케이션을 개발하게 된다.

## 2025-02-07 Fri
### 서블릿 컨테이너 초기화와 애플리케이션 초기화의 분리
* 언뜻 보기에 서블릿 컨테이너만으로 모든 것을 해결할 수 있으며, 애플리케이션 초기화는 오버 엔지니어링에 의해 설계된 불필요한 개념처럼 보일 수 있다.
* 반면, 애플리케이션 초기화는 인터페이스를 기반으로 구현된 구현체를 모두 자동 등록하므로 편리성에서 상대적인 이점을 갖는다.
  * 에를 들어, 서블릿 컨테이너 초기화는 `ServletContainerInitializer` 인터페이스를 구현하는데에 더해 `META-INF` 내의 파일도 수정해주어야 한다.
  * 때문에 **애플리케이션을 최초로 개발하는 과정에서 한 번의 수고를 들여둔다면, 추가적인 초기화 로직은 개방 폐쇄 원칙을 준수하며 확장**해나갈 수 있다.
* 또한, 애플리케이션 초기화를 위한 인터페이스는 서블릿 컨테이너라는 기술과 완전히 분리된 별도의 개념으로 개발될 수 있다는 점에서의 장점 역시 무시할 수 없다.
  * 상술한 예시의 `ApplicationInit` 인터페이스는 `onStartup()` 메소드로부터 서블릿이라는 구체적인 기술에 의존한다.
  * 반면, 원하는 경우 인자를 제거하여 서블릿 등의 구체적인 기술에 의존하지 않는 애플리케이션 초기화 인터페이스 역시 얼마든지 정의할 수 있다. 

## 2025-02-08 Sat
### WAS와 스프링 컨테이너 통합하기
* 상술한 서블릿 컨테이너 초기화 및 애플리케이션 초기화를 활용할 경우, 스프링 컨테이너와 간단히 통합하는 것 역시 가능하다.
  * 이 경우 스프링 컨테이너를 생성한 후 스프링 컨트롤러를 이에 등록하고, 스프링 컨트롤러를 호출하기 위한 디스패처 서블릿을 서블릿 컨테이너에 등록하게 된다.
* 이러한 기능을 위한 애플리케이션 초기화 로직은 상술한 `ApplicationInit` 인터페이스를 구현하는 구현체를 아래와 같이 추가하는 것으로 작성할 수 있다.
  * 이렇듯 애플리케이션 초기화 기능을 서블릿 컨테이너 초기화 기능과 분리하는 것으로 개방 폐쇄 원칙을 준수하기 쉬워진다.
```java
public class SpringInitializer implements ApplicationInit {
    @Override
    public void onStartup(ServletContext servletContext) {
        // 명시적인 스프링 컨테이너 생성
        AnnotationConfigWebApplicationContext appCtx = new AnnotationConfigWebApplicationContext();
        
        // 스프링 컨테이너에 컨트롤러 등록 로직을 포함하는 Configuration을 등록
        appCtx.register(MyConfiguration.class);
        
        // 스프링 MVC 디스패처 서블릿을 생성하고, 스프링 컨테이너 연결
        DispatcherServlet dispatcher = new DispatcherServlet(appCtx);
        
        // 디스패처 서블릿을 서블릿 컨테이너에 등록
        ServletRegistration.Dynamic servlet = servletContext.addServlet("myDispatcherV1", dispatcher);
      
        // /spring/* 요청은 이제 디스패처 서블릿에 의해 처리된다.
        servlet.addMapping("/spring/*");
    }
}
```
* **서블릿은 필터 등의 특수한 용도 클래스를 제외하고 WAS 환경에서 사용자의 요청을 최초로 수신하는 역할을 담당**한다.
  * 특히, **디스패처 서블릿은 상술한 코드에서 확인할 수 있듯 애플리케이션 컨텍스트를 인지하고 있으므로, 빈으로 등록된 컨트롤러를 찾아 매핑**할 수 있다.

## 2025-02-09 Sun
### 프로그래밍 방식의 WAS와 스프링 컨테이너 통합 코드 해설
* 상술한 코드에서, `AnnotationConfigWebApplicationContext`가 스프링 컨테이너에 해당한다.
  * 해당 클래스의 조상인 `ApplicationContext` 인터페이스를 구현하며, 이름 그대로 어노테이션 기반 설정과 웹 기능을 지원하는 컨테이너에 해당한다.
  * 이렇게 명시적으로 생성된 스프링 컨테이너에는 `appCtx.register(MyConfiguration.class)` 형태로 스프링 설정을 명시적으로 등록할 수 있다.
* 또한 `DispatcherServlet`의 생성자에 스프링 컨테이너를 전달하는 것으로 둘을 연결지을 수 있다.
  * 이로 인해, 이후 디스패처 서블릿에 HTTP 요청이 전달될 경우 해당 스프링 컨테이너 상의 컨트롤러 빈을 호출하게 된다.
* 이 시점에서 디스패처 서블릿과 스프링 컨테이너가 연결되었으므로, 이를 서블릿 컨테이너에 등록해줄 필요가 있다.
  * 상술한 코드의 경우, 명시적으로 인스턴스화된 디스패처 서블릿은 `/spring` 하위 경로의 모든 요청을 처리하게 된다.
  * 이 때, **서블릿을 등록하는 과정에서 중복된 이름을 명시할 경우 오류가 발생한다는 점에 주의**를 기울일 필요가 있다.
* 이후 **`/spring` 하위 경로를 향해 HTTP 요청이 발생한 경우, 디스패처 서블릿이 실행되고 디스패처 서블릿은 적절한 컨트롤러에 처리를 위임**한다.
  * 이 때, 디스패처 서블릿은 `/spring` 이후의 경로를 기반으로 매핑된 컨트롤러를 조회하여 실행한다.
  * 예를 들어, **`/spring/my-api`를 호출한 경우 디스패처 서블릿은 `/my-api`를 처리하는 컨트롤러의 메소드를 호출**한다.

## 2025-02-10 Mon
### 스프링 MVC의 서블릿 컨테이너 초기화 지원
* 상술한 과정은 `ServletContainerInitializer` 인터페이스에 대한 구현체를 직접 작성하는 등 복잡하고 번거로운 부분이 있다.
  * 이러한 서블릿 컨테이너 초기화 과정은 번거롭고, 복잡성이 높기에 휴먼 에러가 발생하기 쉬운 축에 속한다.
* 스프링 MVC는 이러한 상황에 대비하여 서블릿 컨테이너 초기화 과정을 자동화하며, 개발자는 이러한 과정을 생략하고 애플리케이션 초기화 코드에만 집중할 수 있다.
* 이 때, 스프링이 지원하는 애플리케이션 초기화 인터페이스는 다음과 같은 `WebApplicationInitializer`에 해당한다.
```java
public interface WebApplicationInitializer {
    void onStartup(ServletContext servletContext) throws ServletException;
}
```

## 2025-02-11 Tue
### WebApplicationInitializer 구현하기
* `WebApplicationInitializer` 인터페이스를 사용하더라도 기존의 스프링 컨테이너 통합 코드와 유사한 코드를 작성할 수 있다.
  * 반면, 기존 코드와의 충돌을 감안하여 디스패처 서블릿의 이름을 다르게 등록해주어야 하는 점에 유의해야 한다.
```java
public class SpringMvcApplicationInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        // 명시적인 스프링 컨테이너 생성
        AnnotationConfigWebApplicationContext appCtx = new AnnotationConfigWebApplicationContext();
        
        // 스프링 컨테이너에 컨트롤러 등록 로직을 포함하는 Configuration을 등록
        appCtx.register(MyConfiguration.class);
        
        // 스프링 MVC 디스패처 서블릿을 생성하고, 스프링 컨테이너 연결
        DispatcherServlet dispatcher = new DispatcherServlet(appCtx);
        
        // 디스패처 서블릿을 서블릿 컨테이너에 등록
        ServletRegistration.Dynamic servlet = servletContext.addServlet("myDispatcherV2", dispatcher);
        
        // 모든 요청이 디스패처 서블릿에 의해 처리된다.
        servlet.addMapping("/");        
    }
}
```
* 이렇듯 **각자 다른 경로를 처리하는 여러 서블릿을 등록할 경우, 사용자의 요청을 더 구체적으로 처리할 수 있는 서블릿이 우선 동작**한다.
  * 예를 들어, `/spring` 형태의 접두사를 갖는 경로는 여전히 이전의 서블릿이 처리한다.
* 반면, 상술한 예시와 같은 코드를 작성하는 것으로 동일한 웹 애플리케이션에 스프링 컨테이너와 디스패처 서블릿이 각각 둘 씩 존재하게 된다.
  * 그러나 **일반적으로는 스프링 컨테이너와 디스패처 서블릿 각각 하나씩 생성하여 연결하고, 디스패처 서블릿에 `/` 경로를 매핑하여 모든 요청을 처리**한다.

## 2025-02-12 Wed
### 스프링 MVC가 제공하는 서블릿 컨테이너 초기화 과정
* 스프링 역시 상술한 바와 같이 서블릿 컨테이너의 요구사항을 모두 충족하는 코드를 미리 작성해두어 애플리케이션 초기화를 지원하는 것에 지나지 않는다.
  * 예를 들어, `spring-web` 라이브러리에는 상술한 `META-INF` 하위 경로에 이미 서블릿 컨테이너 초기화를 위한 등록 파일이 포함되어 있다.
  * 특히, `jakarta.servlet.ServletConainerInitializer` 파일에 `SpringServletContainerInitializer` 인터페이스 경로를 포함한다.
* 정확히는 스프링은 `ServletContainerInitializer`를 구현하는 `SpringServletContainerInitializer` 클래스를 활용한다.
  * 또한, 해당 클래스는 `@HandleTypes` 어노테이션에 `WebApplicationInitializer`를 명시한다.
  * 이로 인해 상술한 `SpringMvcApplicationInitializer` 구현체가 자동으로 등록되어 애플리케이션 초기화 로직을 호출할 수 있게 된다.

## 2025-02-13 Thu
### 스프링 부트와 내장 톰캣
* 상술한 과정을 통해 서블릿 컨테이너를 초기화하여 필요한 서블릿을 등록하거나, 스프링 컨테이너와 디스패처 서블릿을 연결해볼 수 있다.
* 반면, 이러한 방식은 모두 서블릿 컨테이너 상에서 동작하는 방식이므로 언제나 서블릿 컨테이너 위에 배포를 해야 애플리케이션을 동작시킬 수 있다.
  * 즉, 물리 서버 상에 톰캣을 구성한 후 서블릿 컨테이너 스펙에 맞춘 WAR 기반 애플리케이션을 개발한 후에야 배포가 가능했다.
* 이러한 복잡하고 번거로운 배포 절차는 스프링 부트가 제안되고, 톰캣이 스프링 부트에 포함됨에 따라 상당 부분 개선되어 더욱 편리한 사용성을 지니게 되었다.

## 2025-02-14 Fri
### WAR 배포 방식의 단점과 내장 톰캣
* WAR를 사용하는 경우, 웹 애플리케이션을 배포하기 위해서는 반드시 WAS를 설치하고 WAR를 빌드하여 배포하는 과정이 수반된다.
  * 즉, 웹 애플리케이션을 위해 서버를 별도로 설치해야하는 구조이며 과거에는 이렇듯 WAS와 WAR가 분리되어 있는 것을 당연시했다.
* 그러나 이러한 방식은 크게 다음과 같은 단점이 수반되어 개발 생산성을 크게 떨어트리게 된다.
  1. 톰캣과 같은 WAS의 설치 및 톰캣 버전 관리 등의 유지보수 난이도 증대
  2. 개발 환경 설정의 복잡성 증대
  3. 배포 과정의 복잡성 증대
* 이러한 불편한 방식을 개선하기 위해 톰캣을 마치 일종의 라이브러리처럼 제공하는 내장 톰캣의 아이디어가 고안되었다.
  * 이는 마치 `main()` 메소드만 호출하면 애플리케이션이 시작되는 것과 유사하며, 톰캣을 라이브러리 형태로 애플리케이션에 포함시키게 된다.
  * 덧붙여 톰캣 역시 Java를 기반으로 작성되었기에 이러한 방식이 가능했다고도 볼 수 있다.
  * 이를 활용할 경우 WAS에 빌드된 WAR를 배포하는 것이 아닌, 톰캣 자체를 일종의 라이브러리로 취급하는 JAR를 빌드하여 활용할 수 있게 된다.

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