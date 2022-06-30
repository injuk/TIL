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