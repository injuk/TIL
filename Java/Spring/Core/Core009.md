# Core
## 2022-06-30 Thu

### 빈 스코프란?
* **빈 스코프란, 빈이 존재할 수 있는 범위**를 뜻한다.
  * 예를 들어, 싱글톤 스코프로 생성되는 스프링 빈은 스프링 컨테이너의 시작부터 종료까지 유지된다.
* 스프링이 지원하는 빈 스코프의 종류는 크게 다음과 같다.
  1. 싱글톤: **기본 스코프이며, 스프링 컨테이너의 시작부터 종료까지 유지되는 가장 넓은 범위의 스코프**이다.
     * 즉, 스프링 컨테이너와 같은 생명주기를 갖는 스코프이다.
  2. 프로토타입: **스프링 컨테이너가 빈 객체의 생성과 의존 관계 주입까지만 관여하고, 그 이후로는 관리하지 않는 매우 짧은 범위의 스코프**이다. 
     * **프로토타입의 빈은 요청이 들어왔을 때 빈을 생성하고, 의존 관계를 주입하고, 초기화 메소드까지는 호출한 후로는 전혀 관리**하지 않는다.
  3. 웹 관련 스코프 - request: 웹 요청이 들어와서부터 나갈 때까지 유지되는 스코프이다.
  4. 웹 관련 스코프 - session: 웹 세션이 생성되고 종료될 때까지만 유지되는 스코프이다.
  5. 웹 관련 스코프 - application: 웹 서블릿 컨텍스트와 같은 범위로 유지되는 스코프이다.
* 상술한 내용 중 웹 관련 스코프는 스프링 웹과 관련된 기능이 포함되어야만 사용이 가능하다.
* 이러한 **빈 스코프는 @Scope("스코프_이름") 어노테이션을 활용하여 명시**할 수 있다.
* **상술한 스코프 중 빈번히 사용되므로 상세하게 이해할 필요가 있는 항목은 싱글톤과 프로토타입, 그리고 request 스코프**이다.

### 싱글톤 스코프와 프로토타입 스코프
* 싱글톤 스코프의 빈을 조회할 경우, 스프링 컨테이너는 항상 같은 인스턴스의 스프링 빈을 반환한다.
  * 반면, **프로토타입 스코프 빈을 스프링 컨테이너에서 조회할 경우 컨테이너는 항상 새로운 인스턴스를 생성하여 반환**한다.
* 싱글톤 빈 요청 흐름은 크게 다음과 같다.
  1. 클라이언트는 임의의 싱글톤 스코프 빈을 스프링 컨테이너에 요청한다.
  2. 스프링 컨테이너는 자신이 관리 중인 빈 중 적절한 것을 찾아 반환한다.
  3. 이후 다른 클라이언트의 요청에 대해서도 스프링 컨테이너는 같은 객체 인스턴스의 스프링 빈을 반환한다.
* 그러나 프로토타입 빈 요청 흐름은 다음과 같으며, 싱글톤 스코프의 빈 요청 흐름과는 차이가 있다.
  1. 클라이언트가 프로토타입 스코프를 갖는 임의의 빈을 스프링 컨테이너에 요청한다.
  2. 스프링 컨테이너는 요청이 들어온 시점에 새로운 프로토타입 빈을 생성하고, 필요한 의존 관계를 주입한 후 초기화 메소드까지 호출한다.
  3. **완성된 빈 객체는 스프링 컨테이너가 클라이언트에게 반환하지만, 컨테이너 자신은 빈 객체 정보를 폐기하여 더 이상 관리하지 않는다**.
     * 즉, 요청된 빈을 생성하고 의존 관계를 주입하는 등의 과정은 수행해주지만 반환 이후에는 아무런 관리를 수행하지 않는다.
  4. 이후의 **동일한 요청에 대해, 스프링 컨테이너는 1. - 3. 과정을 반복하며 항상 새로운 프로토타입 빈을 생성하여 반환**한다.

### 프로토타입 스코프 - 정리
* **중요한 것은 스프링 컨테이너가 프로토타입 빈을 생성하고 의존 관계를 주입한 후, 필요한 경우에 한해 초기화 콜백 메소드까지 호출한다는 점**이다.
* 이렇게 모든 처리가 끝난 프로토타입 빈은 클라이언트에게 반환되고, 스프링 컨테이너는 더 이상 해당 빈을 관리하지 않는다.
  * 즉, 반환된 시점 이후로 프로토타입 빈을 관리할 책임은 클라이언트에게 있다.
* 때문에 **프로토타입 스코프의 빈 객체는 소멸 전 콜백 메소드인 @PreDestroy 등이 호출되지 않는다**.

### 프로토타입 스코프 작성해보기
```
public class PrototypeTest {

    @Test
    void prototypeBeanTest() {
        // 얘는 싱글톤 스코프와 달리 빈 등록 로그도 안뜸. 프로토타입 빈 스코프 객체는 스프링 컨테이너가 아예 관리하지 않는다는 의미!
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);

        System.out.println("get prototypeBean1"); // 이 시점에서 프로토타입 빈이 생성된다.
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        System.out.println("get prototypeBean2"); // 이 시점에서 프로토타입 빈이 생성된다.
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);

        System.out.println("prototypeBean1 = " + prototypeBean1);
        System.out.println("prototypeBean2 = " + prototypeBean2);

        assertThat(prototypeBean1).isNotSameAs(prototypeBean2);

        // 관리 책임이 클라이언트에게 있으므로, 소멸 전 콜백 메소드를 호출해야하는 경우에는 해당 메소드를 직접 호출해야 한다.
        prototypeBean1.destroy();
        prototypeBean2.destroy();
        
        // 클로즈하여 스프링 컨테이너를 종료한다.
        ac.close();
    }

    // @Component는 붙이지 않아도 무방하다.
    // AnnotationConfigApplicationContext(PrototypeBean.class)로 지정하면, 인자로 전달된 클래스가 컴포넌트 스캔 대상이 된 것처럼 동작한다!
    // 즉, 아래의 코드는 컴포넌트 스캔처럼 등록하는 예시이다.
    @Scope("prototype")
    static class PrototypeBean {
        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init");
        }

        // 프로토타입 빈 스코프의 객체는 소멸 전 콜백 메소드가 호출되지 않는다.
        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```
* **싱글톤 빈은 스프링 컨테이너가 생성되는 시점에 초기화 메소드가 실행되지만, 프로토타입 빈은 스프링 컨테이너에서 해당 빈을 조회할 때 생성**된다. 
  * 때문에 프로토타입 빈은 2번 조회할 경우 2개가 생성된다.
* 싱글톤 빈은 스프링 컨테이너가 직접 관리하므로 스프링 컨테이너가 종료될 때 소멸 전 콜백 메소드가 호출된다.
* 그러나 프로토타입 빈은 스프링 컨테이너가 생성과 의존 관계 주입, 초기화 콜백 메소드 호출까지만 관여하고 그 이후로는 관리 책임을 클라이언트에게 넘긴다.
  * 때문에 프로토타입 빈은 스프링 컨테이너가 종료되는 시점에 소멸 전 콜백 메소드가 호출되지 않는다.
* 따라서 **프로토타입 빈 객체는 해당 빈을 조회한 클라이언트가 직접 관리해야 하며, 소멸 전 콜백 메소드 역시 클라이언트가 직접 호출**해야 한다.

### 싱글톤과 프로토타입 스코프의 혼용
* 싱글톤 빈은 프로토타입 빈과 함께 사용하는 경우, 의도대로 동작하지 않을 가능성이 있다.
* 이 때, **싱글톤 빈이 프로토타입 빈을 의존 관계 주입을 통해 주입받아 사용하고자 하는 경우가 발생할 수 있다**.
  1. 임의의 싱글톤 빈 객체는 일반적으로 스프링 컨테이너의 생성 시점에 함께 생성되며, 의존 관계도 함께 주입된다.
  2. 싱글톤 빈은 의존 관계 자동 주입을 사용하므로, 주입 시점에 스프링 컨테이너에 프로토타입 빈을 요청한다.
  3. 스프링 컨테이너는 프로토타입 빈을 생성하여 요청한 클라이언트인 싱글톤 빈에 반환한다.
  4. 이 시점부터 스프링 컨테이너는 더 이상 관여를 하지 않으며, 싱글톤 빈은 프로토타입 빈의 참조값을 내부 필드에 저장하여 보관한다.
* 즉, **싱글톤 빈이 프로토타입 빈의 참조값을 갖게 되므로 다음과 같은 이유에서 주입된 프로토타입 빈 역시 마치 싱글톤처럼 동작**하게 된다.
  1. 싱글톤 빈 내부적으로 프로토타입 빈이 제공하는 메소드를 활용하는 메소드가 있다고 하자.
  2. **프로토타입 빈은 요청 시마다 새로운 객체가 반환되지만, 이러한 경우 싱글톤 빈을 통한 간접 요청에 대해서는 새로운 객체가 생성되지 않는다**.
  3. **프로토타입 빈을 활용하는 싱글톤 빈의 메소드를 호출할 경우, 매번 같은 프로토타입 빈 객체가 요청에 응답**한다.
```
@Test
void singletonLikePrototype() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(SingletonBean.class, PrototypeBean.class);
    // SingletonBean은 싱글톤 빈이므로, 아래 두 줄은 모두 같은 객체를 참조한다.
    // 중요한 것은, 두 변수가 참조하는 동일한 객체에 포함된 프로토타입 빈 객체 역시 동일하다는 점이다.
    SingletonBean singletonBean1 = ac.getBean(SingletonBean.class);
    SingletonBean singletonBean2 = ac.getBean(SingletonBean.class);
   
    // // 단순히 프로토타입 빈에 대해 요청하는 메소드를 doSomething 메소드가 포함하고 있다고 해서 매 번 새로운 프로토타입 빈이 반환되지는 않는다. 
    int result1 = singletonBean1.doSomething();
    System.out.println("result1 = " + result1);
    assertThat(result1).isEqualTo(1);

    int result2 = singletonBean2.doSomething();
    System.out.println("result2 = " + result2);
    assertThat(result2).isEqualTo(2);
}
```
* 문제의 핵심은 스프링이 일반적으로 사용하는 싱글톤 빈이 참조하는 프로토타입 빈이 마치 싱글톤 빈처럼 동작하는 것이다. 
  * **싱글톤 빈은 빈 객체 생성 시점에만 의존 관계 주입을 받으므로, 최초 생성시에 주입받기 위해 생성한 프로토타입 빈과 함께 계속해서 유지**된다.
  * 만약 개발자의 의도가 프로토타입 빈에 새로운 요청을 전달할 때마다 새로운 객체가 반환되는 것을 기대했다면, 상술한 방식으로는 원하는 바를 얻을 수 없다.
  * 무엇보다도 빈 객체의 스코프를 프로토타입으로 설정한 것부터가 이러한 상황을 원치 않았다는 것을 반증한다.
* 반면, **여러 빈에서 동일한 프로토타입 빈을 주입받는 경우에는 주입 시점에 각각 서로 다른 프로토타입 빈이 생성**된다.
  * 이는 **스프링 컨테이너에 프로토타입 빈 객체에 대한 의존 관계 주입을 요청할 때마다 주입 시점에 새로운 객체가 생성**되기 때문이다.

### Provider 객체 활용하기
* 싱글톤 빈과 프로토타입 빈을 혼용하는 경우, 프로토타입 빈을 매 번 새로이 생성하기 위해서는 다음과 같은 방법을 고려할 수 있다.
  1. 클라이언트 빈 객체가 `@Autowired private ApplicationContext ac;`의 형태로 스프링 컨테이너를 주입받아 사용한다.
  2. ObjectFactory, ObjectProvider 활용하기
* 스프링 컨테이너를 활용하는 경우, `ac.getBean()`을 통해 항상 새로운 프로토타입 빈을 반환받을 수 있다.
  * 이렇듯 의존 관계를 외부로부터 주입하는 방식이 아닌, 필요한 의존성을 직접 조회하는 방식을 Dependency Lookup이라고도 한다.
* 그러나 의존 관계 탐색 방식의 경우, 스프링 애플리케이션 컨텍스트 전체를 주입받으므로 스프링에 종속적인 코드가 된다.
  * 무엇보다 단위 테스트를 수행하기가 어려워진다!
* 지금 필요한 것은 프로토타입 빈을 컨테이너에서 대신 찾아주는 기능 뿐이며, ObjectFactory와 ObjectProvider는 이에 부합한다.

### ObjectFactory와 ObjectProvider
* **지정한 빈을 컨테이너에서 찾아 반환하는 DL 서비스를 제공하는 ObjectProvider는 기존 기능인 ObjectFactory에 여러 편의 기능을 추가한 클래스**이다.
  * ObjectFactory의 경우, 스프링에 의존적이지만 기능이 매우 단순하며 별도의 라이브러리를 필요로하지 않는다.
  * ObjectProvider의 경우, 스프링에 의존적이지만 스트림 처리 등 편의 기능이 많으며 별도의 라이브러리를 필요로하지 않는다.
    * 정확히는 getObject만을 제공하는 ObjectFactory를 ObjectProvider가 상속받아 기능을 확장한다.
* ObjectProvider를 활용할 경우, getObject 메소드를 통해 항상 새로운 프로토타입 빈을 반환받을 수 있다.
  * 실제로도 **ObjectProvider의 getObject 메소드가 호출되는 경우, 내부적으로는 스프링 컨테이너를 통해 해당 빈을 찾아 반환**한다.
  * 이렇듯 ObjectProvider는 DL만을 제공하며, 기능이 단순하므로 단위 테스트를 작성하거나 mock 객체를 만들기도 쉽다.
```
@Scope("singleton")
static class SingletonBean {
    // ObjectProvider는 제네릭을 제공한다.
    private final ObjectProvider<PrototypeBean> prototypeBeanObjectProvider;

    public SingletonBean(ObjectProvider<PrototypeBean> prototypeBeanObjectProvider) {
        this.prototypeBeanObjectProvider = prototypeBeanObjectProvider;
    }

    public int doSomething() {
        PrototypeBean prototypeBean = prototypeBeanObjectProvider.getObject();

        prototypeBean.addCount();
        int count = prototypeBean.getCount();

        System.out.println("prototypeBean.count = " + count);
        return count;
    }
}
```
* **ObjectProvider는 임의의 빈을 조회하는 DL 기능만을 제공하므로, 더 이상 DL을 위해 클라이언트에 ApplicationCotext를 주입할 필요가 없어진다**.
* 그러나 ObjectProvider의 용도는 프로토타입 빈 객체 전용으로 사용하기 위한 것은 아니다.
  * 오히려 **핵심은 스프링 컨테이너를 클라이언트가 직접 조회하는 대신 조회하는 대리자 역할**에 있다.
* **ObjectProvider와 ObjectFactory는 개발자가 직접 스프링 빈으로 등록하지 않더라도, 스프링이 자체적으로 생성하여 주입**해준다.

### JSR-330 Provider
* 상술한 방법 대신 `javax.inject.Provider`라는 Java 표준인 JSR-330을 활용할 수도 있다.
  * 이를 위해서는 `javax.inject:javax.inject:1` 라이브러리를 그래들에 추가해야 한다.
```
@Scope("singleton")
static class SingletonBean {
    private final Provider<PrototypeBean> prototypeBeanObjectProvider;

    public SingletonBean(Provider<PrototypeBean> prototypeBeanObjectProvider) {
        this.prototypeBeanObjectProvider = prototypeBeanObjectProvider;
    }

    public int doSomething() {
        PrototypeBean prototypeBean = prototypeBeanObjectProvider.get();

        prototypeBean.addCount();
        int count = prototypeBean.getCount();

        System.out.println("prototypeBean.count = " + count);
        return count;
    }
}
```
* Provider 객체는 get 메소드를 통해 스프링 컨테이너로부터 새로운 프로토타입 빈을 생성하여 반환하는 DL 기능을 제공한다.
  * Provider 역시 필요한 DL 기능 정도만을 제공한다.
* Java 진영의 표준 기술이며, 기능이 매우 단순하므로 단위 테스트를 작성하거나 mock 코드를 만들기는 훨씬 쉬워진다.
  * Provider는 메소드를 get 하나만 제공하므로 매우 단순하고, 별도의 라이브러리도 필요 없다.
  * 또한, Java 진영의 표준이므로 스프링 이외의 컨테이너에서도 활용이 가능하다는 장점이 존재한다.
* 이렇듯 Provider는 라이브러리를 가져와야한다는 점을 제외하면 장점이 많은 방식이다.

### 결론
* **프로토타입 스코프의 빈 객체는 사용 시마다 의존 관계 주입이 완료된 새로운 객체가 필요한 경우에 사용**한다.
* 그러나 **웹 애플리케이션을 주로 개발하는 실무에서, 싱글톤 만으로도 대부분의 문제를 해결할 수 있으므로 프로토타입 빈을 사용하는 상황은 매우 드물다**.
* **ObjectProvider와 JSR-330의 Provider는 모두 프로토타입 빈 객체 뿐만 아니라 DL이 필요한 모든 상황에서 활용이 가능**하다.
  * 두 객체 중 무엇을 활용하느냐는 DI 컨테이너의 종류에 따르도록 한다.
  * 예를 들어, 자신이 개발한 애플리케이션 코드를 다른 컨테이너에서 사용할 가능성이 조금이라도 존재한다면 JSR-330 Provider를 선택하는 것이 바람직하다.
* 이렇듯 스프링을 활용하다 보면 여러 기능들이 Java 표준과 겹치는 것을 확인할 수 있으나, 대부분의 경우 스프링이 더 다양한 기능을 제공한다.
  * 때문에 **자신이 개발한 애플리케이션이 다른 DI 컨테이너를 사용할 가능성이 없는 경우, 기본적으로 스프링이 제공하는 기능을 선택하는 것이 바람직**하다.

### 웹 스코프
* 웹 스코프란, 웹 환경에서만 동작하며 스프링이 해당 스코프의 종료 시점까지 관리해준다는 특징이 있다.
  * 때문에 소멸 전 콜백 메소드 역시 호출된다.
* 웹 스코프는 크게 다음과 같이 분류할 수 있다.
  1. request: HTTP 요청 하나가 들어오고 나갈 때까지 유지되는 스코프로, 각 HTTP 요청마다 별도의 빈 객체가 생성되어 관리된다.
  2. session: HTTP 세션과 동일한 생명주기를 갖는 스코프이다.
  3. application: 서블릿 컨텍스트와 동일한 생명주기를 갖는 스코프이다.
  4. websocket: 웹 소켓과 동일한 생명주기를 갖는 스코프이다.

### request 스코프
* request 스코프는 HTTP 요청 당 할당되며, 다음과 같은 시나리오를 통해 이해할 수 있다.
  1. 클라이언트 A가 HTTP 요청을 통해 컨트롤러를 호출한다.
  2. 컨트롤러가 request 스코프로 설정된 로깅 객체를 사용하는 경우, 새로운 로깅 객체가 스프링 컨테이너로부터 생성되어 관리된다.
  3. 컨트롤러가 서비스를 호출하고, 서비스 역시 로깅 객체를 사용한다면 같은 request 범위이므로 동일한 로깅 객체가 사용된다.
  4. 이 때, 클라이언트 B가 HTTP 요청을 통해 컨트롤러를 호출하는 경우 별도의 request이므로 다른 로깅 객체가 생성되어 관리된다.
  5. 각 클라이언트의 요청이 처리되어 종료되면 로깅 객체는 소멸된다.
* 이렇듯 **request 스코프 빈의 경우 요청 클라이언트 별 전용 빈 객체를 생성하여 관리하고, 소멸시키는 것으로 이해**할 수 있다.

### request 스코프 객체를 사용하기 위해 웹 라이브러리를 추가하기
* 웹 스코프는 웹 환경에서만 동작하므로, `build.gradle`에 `'org.springframework.boot:spring-boot-starter-web'` 라이브러리를 추가한다.
  * 해당 라이브러리가 추가되면 스프링 부트는 내장 톰캣 서버를 활용하여 웹 서버와 스프링을 함께 실행시킨다.
* 스프링 부트는 상술한 웹 라이브러리가 존재하지 않는 경우, 기본적으로 `AnnotationConfigApplicationContext`를 사용하여 애플리케이션을 동작시킨다.
  * 반면, 웹 라이브러리가 추가되면 웹과 관련된 추가 설정들이 필요하므로 `AnnotationConfigServletWebServerApplicationContext`를 사용한다.

### request 스코프 객체 추가하기
* 동시에 여러 클라이언트로부터 HTTP 요청이 수신된 경우, 정확히 어떠한 요청으로부터 발생한 로그인지 구분하기 어려울 수 있다.
  * 이 경우 여러 스레드로부터 로그를 남기게 되며, 구분하기에도 어렵다.
  * 이럴 때 사용하기 좋은 것이 request 스코프 빈 객체이다.
* 같은 클라이언트의 요청은 UUID를 통해 구분한다고 가정했을 때, 다음과 같은 공통 로그 모듈을 작성해볼 수 있다.
```
@Component
@Scope(value = "request")
public class MyLogger {
    
    private String uuid;
    private String requestUrl;

    public void setRequestUrl(String requestUrl) {
        this.requestUrl = requestUrl;
    }
    
    public void log(String message) {
        System.out.println(
                new StringBuilder("[")
                    .append(uuid)
                    .append("]")
                    .append("[")
                    .append(requestUrl)
                    .append("] ")
                    .append(message)
        );
    }
    
    @PostConstruct
    public void init() {
        // 글로벌하게 유니크한 uuid를 생성한다.
        // 이는 로또를 몇 번 맞을 정도의 확률과 비슷하게 동일한 uuid가 생성되므로, 안심해도 좋다!
        uuid = UUID.randomUUID().toString();
        System.out.println("[" + uuid + "] request scope bean created: " + this);
    }
    
    @PreDestroy
    public void close() {
        System.out.println("[" + uuid + "] request scope bean destroyed: " + this);
    }
}
```
* 해당 로그 클래스는 `@Scope(value = "request")`를 사용하여 request 스코프로 지정된다.
  * 또한 @Component이므로 해당 빈은 HTTP 요청 당 하나씩 생성되고, HTTP 요청이 종료되는 시점에 소멸된다.
  * **정확히는 스프링 컨테이너에 요청하는 시점에 해당 빈 객체가 생성**된다. 

### 컨트롤러 작성하기
* MyLogger 클래스를 활용하는 컨트롤러는 다음과 같이 작성할 수 있다.
  * 그러나 **아래의 코드는 에러로 인해 실행되지 않으며, 원인은 request 스코프인 myLogger를 스프링 컨테이너의 시작 시점에 주입 받으려하기 때문**이다.
```
@Controller
@RequiredArgsConstructor
public class LogDemoController {
    private final LogDemoService logDemoService;
    private final MyLogger myLogger;

    @RequestMapping("log-demo")
    @ResponseBody
    // 매개변수를 통해 Java에서 제공하는 표준 서블릿 규약에 의한 HTTP request 정보를 수신할 수 있다.
    public String logDemo(HttpServletRequest request) {
        String requestUrl = request.getRequestURL().toString();
        myLogger.setRequestUrl(requestUrl);

        myLogger.log("controller test");
        logDemoService.doSomething("testId");

        return "OK";
    }
}
```
* 또한 **`myLogger.setRequestUrl(requestUrl);`같은 로직은 공통 처리가 가능한 별도의 인터셉터 또는 서블릿 필터를 활용하는 것이 바람직**하다.
  * 이 때, 인터셉터란 HTTP request 요청이 컨트롤러를 호출하기 직전에 공통된 처리를 적용할 수 있는 기능이다.

### 서비스 작성하기
* 컨트롤러로부터 호출되는 서비스 계층은 비즈니스 로직을 포함하며, 다음과 같이 작성할 수 있다.
```
@Service
@RequiredArgsConstructor
public class LogDemoService {

    private final MyLogger myLogger;

    public void doSomething(String id) {
        myLogger.log("service id = " + id);
    }
}
```
* 만약 request 스코프를 MyLogger에 적용하지 않았다면, 메소드 또는 생성자에 많은 파라미터를 추가하여 지저분해질 수 밖에 없어진다.
  * **가장 큰 문제는 requestUrl과 같은 웹 관련 정보가 웹과 관련이 없는 서비스 계층을 오염시킨다는 점**이다.
  * **웹과 관련된 부분은 컨트롤러 영역에서까지만 사용해야 하며, 서비스 계층은 웹에 종속되지 않고 순수하게 유지하는 것이 유지보수성 측면에서 바람직**하다.
* 반면 상술한 예제는 MyLogger 클래스를 request 스코프로 지정함으로써 불필요한 정보를 파라미터로 넘기지 않고, 각 코드와 계층을 깔끔하게 유지한다.

### 스코프와 Provider 객체
* 상술한 예제는 MyLogger가 request 스코프로 정의되었으나, 스프링 컨테이너 생성 시점에 의존 관계를 주입받으려 하기에 오류가 발생하며 실행되지 않는다.
* 이를 해결하는 방법 중 하나는 아래의 코드와 같이 ObjectProvider와 같은 Provider 객체를 사용하는 것이다.
  * **ObjectProvider를 통해 MyLogger를 생성 시점에 주입 받는 것이 아닌, MyLogger를 DL을 통해 조회할 수 있는 Provider 객체가 주입**된다.
  * 이렇듯 **ObjectProvider는 의존 관계 주입 시점에 주입 받을 수 있는 객체**이다.
```
@Service
@RequiredArgsConstructor
public class LogDemoService {

    // DL을 위해 ObjectProvider를 추가한다.
    private final ObjectProvider<MyLogger> myLoggerProvider;

    public void doSomething(String id) {
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.log("service id = " + id);
    }
}
```
* **상술한 방식은 ObjectProvider의 getObject 메소드를 호출하기까지 request scope 빈의 생성을 지연하기 때문에 가능**하다.
* **컨트롤러에 요청이 수신되었다는 것은 HTTP 요청의 생명주기에 진입한 것이므로, request 스코프의 빈 객체를 주입받을 수 있게 된다**. 
  * 이 때, MyLogger 클래스는 컨트롤러의 `myLoggerProvider.getObject();` 로직에서 최초로 생성된다.
  * **핵심은 동시에 여러 요청이 수신되더라도 각 요청에 대해 별도의 빈 객체를 관리해준다는 것**이다.
* 결국 **동일한 HTTP 요청에서 컨트롤러, 서비스 등 여러 객체에서 request 스코프 빈을 주입받더라도 동일한 객체를 반환**받게 된다.

### 스코프와 프록시
* ObjectProvider 방식조차 길다면, 아래와 같이 @Scope 어노테이션에 프록시를 적용하는 것으로 코드의 중복을 더 줄일 수 있다.
```
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {
  // ...생략
}
```
* 이 경우, 컨트롤러와 서비스에서 ObjectProvider를 사용한 코드를 모두 기존 방식과 같은 생성자 주입 방식으로 수정하여도 오류가 발생하지 않는다.
  * 이렇듯 **proxyMode를 적절히 사용할 경우, request 스코프의 빈 객체조차 생성자 주입 방식을 사용할 수 있게 된다**.

### ScopedProxyMode
* ScopedProxyMode는 적용 대상에 따라 다음과 같이 다른 값을 명시해야 한다.
  1. 적용 대상이 구체적인 클래스인 경우, TARGET_CLASS를 명시한다.
  2. 적용 대상이 추상적인 인터페이스인 경우, INTERFACES를 명시한다.
* 이를 통해 **@Scope가 명시된 클래스의 가짜 프록시 클래스를 미리 생성해두고, HTTP request의 실재 여부에 관계 없이 다른 빈에 미리 주입**해둘 수 있다.
* 컨트롤러에서 myLogger.getClass()를 적절히 작성할 경우, 가짜 프록시 클래스가 사용되는 것을 로그를 통해 확인할 수 있다.
  * 스프링 빈 객체에 등록되는 실체는 스프링에 의해 조작되어 생성된 새로운 객체이며, 해당 객체가 마치 Provider 객체를 사용한 것처럼 동작하게 된다.
```
myLogger = class hello.core.common.MyLogger$$EnhancerBySpringCGLIB$$fd973e1
```
* **이렇게 주입된 MyLogger는 MyLogger와 유사한 가짜를 미리 주입해둔 것이며, 실제 객체의 호출이 필요한 시점에는 요청을 포워딩하는 식으로 동작**한다.
  * 즉, 내부적으로는 Provider 객체와 유사하게 동작한다.

### CGLIB과 프록시 객체
* 상술한 예시는 CGLIB 라이브러리를 통해 원본 클래스를 상속받는 가짜 프록시 객체를 만들어 주입하는 식으로 동작한다.
  * @Scope와 proxyMode를 설정한 경우, 스프링 컨테이너는 바이트 코드 조작 라이브러리인 CGLIB을 통해 가짜 프록시 객체를 생성한다.
  * 이후, **원본과 동일한 이름인 myLogger라는 이름으로 스프링 컨테이너에 가짜 프록시 객체를 등록**한다.
  * 때문에 ac.getBean 메소드를 활용하더라도 프록시 객체가 조회되며, 의존 관계 주입에서도 해당 프록시 객체가 주입된다.
* 이렇게 생성된 가짜 프록시 객체에는 요청이 수신될 경우 내부적으로 원본 빈 객체를 호출하는 위임 로직이 포함된다.
  * 즉, 가짜 프록시 빈 객체는 원본 빈 객체를 찾는 방법을 알고 있다.
  * 또한, 가짜 프록시 객체는 원본을 상속 받아 생성되므로 해당 객체를 사용하는 클라이언트는 원본과의 차이를 알지 못한 상태에서 동일하게 사용할 수 있다.
* **가짜 프록시 객체는 실제로 적용되어야 할 request 스코프와는 전혀 무관하며, 사실상 싱글톤처럼 동작하며 내부적인 위임 로직을 활용**한다. 

### 프록시 객체 - 정리
* 프록시 객체를 통해 클라이언트는 마치 싱글톤 빈을 사용하듯이 편리하게 request 스코프를 활용할 수 있다.
* **핵심은 Provider를 사용하든, 프록시를 사용하든 request 스코프가 적용된 실제 빈 객체의 조회를 적시까지 지연한다는 점**이다.
  * 결국 실제 HTTP 요청이 수신될 때까지는 가짜 객체로 버티는 것이며, 클라이언트가 실제 HTTP 요청을 송신한 시점에 실제 객체를 찾아 메소드를 호출한다.
* 이렇듯 **어노테이션의 설정 변경만으로도 원본 객체를 프록시 객체로 대체할 수 있으며, 이는 다형성과 DI 컨테이너가 갖는 가장 큰 장점**이기도 하다.
  * AOP 역시 가짜 객체를 사용하는 유사한 방식으로 동작한다.
  * 그러나 **중요한 것은 실제 클라이언트 코드에는 전혀 영향을 주지 않는다는 것이며, 이는 스프링 컨테이너가 갖는 엄청나게 큰 장점**이기도 하다.
* 꼭 **웹 스코프가 아니더라도 프록시를 사용할 수 있다**.

### 프록시와 주의사항
* 프록시를 사용하는 것은 마치 싱글톤을 활용하는 것처럼 느껴지지만, 실제로는 다르게 동작하므로 주의해야할 부분이 있을 수 밖에 없다.
  * 예를 들어, **결국에는 매 HTTP 요청마다 request 스코프 빈 객체가 생성된다는 점을 간과하지 않아야 한다**.
* 이렇듯 **특별한 유형의 빈 스코프는 반드시 필요한 곳에만 최소한으로 사용해야하며, 무분별하게 적용할 경우 유지보수성을 크게 떨어트릴 수 있다**.
  * 무엇보다, 이러한 유형의 빈 스코프 객체들은 테스트하기도 까다롭다.
  * 때문에 실무에서도 특별한 유형의 빈 스코프는 대놓고 사용하기보다는 백그라운드에서 사용하는 경우가 많다.