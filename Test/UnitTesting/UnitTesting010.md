# UnitTesting
## 2022-05-11 Wed

## 목 객체 사용하기
### HTTP 통신이 포함된 메소드를 테스트하기
* 메소드에서 HTTP 요청을 만드는 경우는 드문 일이 아니며, 언뜻 쉽게 테스트를 작성할 수 있을 것처럼 보인다.
* 그러나 이러한 메소드는 외부 시스템과 상호 작용이 필요하며, 다음과 같은 이유에서 단위 테스트를 어렵게 만들기 쉽다.
  1. **실제 REST 호출 및 응답을 대기하는 시간이 필요하므로, 나머지 대다수의 빠른 테스트에 비해 속도는 훨씬 느리다**.
  2. **외부 시스템의 가용성을 항상 보장할 수 없으므로, 테스트는 종종 외부 요인으로 인해 실패할 수 있다**.
* 메소드에 포함된 로직에서, 일반적으로 HTTP 통신을 수행하는 메소드는 널리 사용되므로 신뢰할 수 있다.
  * 따라서 **남은 것은 HTTP 호출을 준비하는 로직과, 호출에 대한 응답에서 생성되는 데이터를 후처리하는 로직을 테스트하는 것**이다.

### 번거로운 동작은 스텁으로 대체하기
* HTTP 호출로부터 반환되는 문자열을 JSON 으로 파싱하여 새로운 객체를 생성하는 시나리오를 가정한다.
* 기존의 get 호출을 담당하던 메소드가 테스트 용도로 하드코딩된 JSON 문자열을 반환하도록 수정한다.
  * 이렇듯 **테스트 용도로 하드코딩된 값을 반환하는 구현체를 스텁**이라고 한다.
  * **인터페이스를 구현하는 스텁의 경우 람다 또는 익명 내부 클래스를 활용하여 별도의 파일을 추가하지 않더라도 동적으로 쉽게 작성할 수 있다**.
  * 즉, 기존의 get 호출과 스텁은 동일한 인터페이스를 구현하지만, 스텁은 실제 통신을 수행하지 않고 하드코딩된 값만을 반환한다.
* **스텁을 정의한 경우, 이를 환경에 따라 실제 운영 코드를 구현하는 구현체와 교체할 수 있도록 의존성 주입 기법을 활용**한다.
  * **HTTP 통신을 수행하는 클래스의 생성자로부터 운영 코드와 스텁 구현체의 공통 인터페이스를 전달 받도록 수정**한다.
* **상술한 내용을 토대로 HTTP 통신을 수행하는 클래스에 스텁 구현체를 전달하는 방식을 활용하여 테스트를 작성**한다.
  * 테스트에서 생성되는 HTTP 통신 객체는 실제로 어떠한 구현체와 상호작용하는지 알지 못하며, 구현체가 get 메시지를 수신할 수 있다는 사실만 알게 된다.

### 테스트를 지원하기 위한 설계의 변경
* **상술한 시나리오에서 단지 단위 테스트를 지원하기 위해 스텁을 도입하였고, 이로 인해 시스템의 설계가 변경**되었다.
  * 이로 인해 HTTP 통신을 수행하는 클래스를 사용하는 클라이언트는 반드시 적절한 구현체를 생성하여 넘겨주도록 수정되어야 한다.
* 그러나 **시스템의 설계를 변경함으로써 테스트를 지원할 수 있다면, 이는 바람직한 변경에 해당**한다.
  * 간단한 방법으로 시스템이 기대한대로 동작함을 보여줄 수 있는 것이 더 중요하기 때문이다.
* 또한 **인터페이스를 활용한 의존성이 추가됨으로써 의존성은 깔끔한 방식으로 선언되며, 결합도 역시 조금은 낮아지는 이점이 수반**된다.
* 스텁을 주입하는 방식은 반드시 생성자 주입일 필요는 없으며, 다음과 같은 방식을 고려할 수 있다.
  1. 세터 메소드
  2. 팩토리 메소드
  3. 추상 팩토리
  4. 스프링 등의 서드 파티 DI 도구

### 조금 더 똑똑한 스텁 만들기
* 상술한 시나리오에서, 스텁은 get 메소드에 전달된 값과 무관하게 하드코딩된 정보를 활용하여 항상 동일한 문자열을 반환한다.
  * 이는 현재까지 작성된 단위 테스트의 구멍에 해당한다.
  * 이를 해결하기 위해 HTTP 통신을 수행하는 클래스의 통신 메소드와 스텁, 또는 운영 구현체의 get 메소드와 올바르게 상호작용하는지 검증할 수 있어야 한다.
* **스텁의 메소드에 전달된 인자를 검증하는 보호 절을 추가하는 것으로 스텁에 약간의 지능을 부여할 수 있으며, 전달된 인자를 검증하도록 수정할 수 있다**.
* 이렇게 **조금 더 똑똑해진 스텁은 목이라는 별도의 용어로 지칭**한다.
  * **목은 의도적으로 흉내 낸 동작을 제공하며, 수신한 인자가 모두 정상인지까지 검증하는 테스트 구조물**을 말한다.

### 목 도구를 활용한 테스트 단순화
* 똑똑한 스텁을 목으로 완전히 변환하기 위해 목에는 최소한 다음과 같은 일들이 추가되어야 한다.
  1. 테스트에 어떠한 인자를 기대하는지 명시
  2. get 메소드에 넘겨진 인자들을 잡아 저장하기
  3. get 메소드에 저장된 인자들이 기대하는 인자들이 맞는지, 테스트가 완료될 때 검증하는 능력을 지원하기
* 그러나 **이러한 모든 단계를 지원할 수 있는 목을 정의하는 것은 너무 과도한 처사**일 수 있다.
  * 반면, 같은 목 객체를 사용해야하는 추가적인 테스트를 작성할 필요가 있다면 실제로는 중복되는 코드를 줄여줄 수도 있다.
  * 결국 이렇게 과도한 목을 작성하기로 결정했다면, 다른 모든 의존성에 대해 더 많은 목을 구현하는 과정에서 코드의 중복을 발견할 수도 있다.
* 이렇듯 **복잡한 목 사이의 중복을 제거하기 위해, 목을 사용하는 테스트를 더욱 빠르게 정의할 수 있도록 하는 범용적인 도구의 도입을 고려**할 수 있다.
  * Java의 경우, Mockito 라이브러리가 이러한 일을 대리해줄 수 있다.
* Mockito를 활용하여 목을 사용하는 코드는 다음과 같다.
```
import static org.mockito.Mockito.*;
// 생략

Http http = mock(Http.class); // 1.
when(http.get(contains("~~~"))). // 2.
    thenReturn("~~~"); // 3.
    
AddressRetriever retriever = new AddressRetriever(http); // 4.
// 5. 테스트 진행
```
* 상술한 코드는 크게 다음과 같이 이해할 수 있다.
  1. mock 메소드를 활용하여 Mockito에 Http 인터페이스를 구현하는 목 인스턴스를 합성하라고 명시한다.
  2. when 정적 메소드를 호출하여 테스트의 기대 사항을 설정한다.
     * 이 경우, 테스트에 전달되어야 하는 인자를 명시하게 된다.
  3. thenReturn 메소드를 호출하여 기대사항이 충족된 경우의 처리를 정의한다.
     * 이 경우, 기대 사항이 충족되었다면 목은 하드코딩된 문자열 결과를 반환하게 된다.
  4. Mockito 목을 생성자를 통해 주입한다.
  5. **실제 테스트 코드는 Mockito 목과 상호작용하며, Mockito 목이 기대사항을 충족하고 있다면 하드코딩된 JSON 문자열을 반환**한다.
     * 기대사항을 충족하지 못한 경우, 테스트는 실패한다.
* 이러한 **테스트의 기대 사항 설정은 반드시 실제 테스트보다 먼저 정의**되어야 한다.

### 의존성 주입 도구
* 생성자를 활용하여 목을 대상 클래스에 넘기는 방식은 가능한 선택지 중 하나에 불과하다.
  * 그러나 **이 방식은 운영 코드에서 인터페이스를 변경하고, 내부 사항을 다른 클래스에 노출하게 되는 단점**이 있다.
  * 이러한 아쉬움은 스프링과 같은 DI 도구를 활용하여 해결할 수 있다.
* 반면, **간단한 테스트의 경우 Mockito의 내장 DI 기능을 활용하여 의존성 주입을 처리할 수 있다**.
* Mockito DI를 사용하는 경우, 다음의 절차를 따르도록 한다.
  1. @Mock 어노테이션을 활용하여 목 인스턴스를 생성한다.
     * 상술한 시나리오의 경우, Http 필드를 선언하고 해당 어노테이션을 붙여준다.
     * 이는 목을 합성하고자 하는 객체를 의미한다.
  2. @InjectMock 어노테이션을 활용하여 대상 인스턴스 변수를 선언한다.
     * 상술한 시나리오의 경우, HTTP 통신을 수행하는 클래스에 어노테이션을 붙여준다.
     * 이는 목을 주입하고자 하는 대상을 의미한다.
  3. 대상 인스턴스의 인스턴스화가 완료된 후에 MockitoAnnotations.initMocks(this);를 호출한다.
     * 예를 들어 @Before 메소드에서 인스턴스화를 완료한 직후에 호출할 수 있다.
     * Mockito는 테스트 클래스로부터 @Mock 어노테이션이 붙는 각각의 필드에 대해 목 인스턴스를 합성한다.
     * 그 후, @InjectMocks 어노테이션이 붙은 모든 필드를 가져와 목 객체들을 주입한다.
* Mockito의 목 객체 주입의 경우 다음과 같은 우선순위에 따라 진행된다.
  1. 적절한 생성자
  2. 생성자가 없는 경우, 적절한 세터 메소드
  3. 세터 메소드도 없는 경우, 필드 타입과 일치하는 적절한 필드
* **Mockito가 제공하는 DI를 활용하는 경우, HTTP 통신을 수행하는 클래스의 생성자를 제거**할 수 있다.
  * 대신 필드 수준의 기본 구현을 제공하도록 클래스를 수정한다.

### 목을 올바르게 사용하기
* **테스트에 올바르게 목을 사용하는 경우, 테스트를 빠르게 읽고 이해하고 신뢰할 수 있다**.
* 이를 위해 **목을 활용하는 테스트는 진행하기를 원하는 내용을 분명하게 명시해야 한다**.
  * 테스트 코드를 읽는 다른 개발자가 코드를 깊이 볼 필요 없이 쉽게 파악할수록 코드는 더욱 좋아진다.
* 목은 실제 동작을 대신하므로, 이를 안전하게 사용하고 있는지 확인하기 위해 다음과 같은 내용을 자문해야 하며, 각각에 대해 추가 테스트가 필요할 수도 있다.
  1. 목이 운영 코드의 동작을 올바르게 묘사하는가?
  2. 운영 코드는 생각지 못한 다른 형식으로 반환하는가?
  3. 운영 코드가 예외를 던지거나 null을 반환하는가?
* **목을 사용하는 테스트의 경우, 테스트가 실제로는 목이 아닌 운영 코드를 실행하고 있지는 않는지 다음과 같은 방법을 통해 확인**해야 한다.
  1. 테스트를 실행할 때 약간 느려지는 것이 체감되는지 확인
     * 실행 시간이 느려지는 것이 확인되는 경우, 목이 아닌 운영 코드가 실행되는 것이다. 
  2. 임시로 운영 코드에서 예외를 던지도록 수정
     * 예외가 던져지는 경우, 목이 아닌 운영 코드가 실행되는 것이다.
     * 이 경우, 확인이 끝난 후 반드시 예외를 던지는 문장을 운영 코드에서 제거해야 한다.
  3. 테스트 데이터를 또 다른 테스트 데이터로 교체하였을 때 테스트가 실패하는지 확인
     * 테스트에 실패하는 경우, 목이 아닌 운영 코드가 실행되는 것이다.
* **가장 중요한 것은 목이 실제로 운영 코드를 직접 테스트하는 것이 아니라는 사실을 인지하는 것**이다.
  * 목을 도입하면 테스트 커버리지에서 커버하지 못하는 구멍을 만들게 된다.
  * 이러한 **구멍을 막기 위해서는 통합 테스트 등을 작성하여 실제 클래스의 종단 간 사용성을 보여주는 상위 테스트를 반드시 진행**해야 한다.  

## 결론
* 스텁과 목을 활용하여 의존하는 객체의 행동을 흉내낼 수 있다.
* 이로 인해 **테스트는 라이브 서비스나 파일, 데이터베이스 등 다른 번거로운 의존성과 실제로 상호작용할 필요가 없다**.
* Mockito와 같은 적절한 도구를 활용하여 목을 생성하고 주입하는 노력을 최소화할 수 있다.