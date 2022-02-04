# CleanCode
## 2022-02-01 Tue

## 클래스
* 함수를 바르게 구현하고 함수가 서로 관련을 맺는 방법 이외에도 더 높은 단계까지 신경쓰지 않는다면 깨끗한 코드를 만들기 어렵다.
* 클래스를 정의하는 표준 자바 관례는 다음과 같다.
  1. 정적 공개 상수가 있다면 맨 처음에 작성한다.
  2. 그 다음으로 정적 비공개 변수를 작성한다.
  3. 그 다음으로 비공개 인스턴스 변수를 작성한다.
     * 공개 변수가 필요한 경우는 거의 없다.
  4. 변수 목록의 작성이 끝나면 공개 함수를 작성한다.
  5. 비공개 함수는 자신을 호출하는 공개 함수 다음에 작성한다.
* 이를 통해 추상화 단계는 순차적으로 내려가고, 코드는 신문 기사처럼 읽힐 수 있다.
* 변수와 함수는 가능한 공개하지 않는 것이 좋다.
  * 그러나 테스트 코드 등이 접근해야하는 경우, 접근제어자를 protected로 작성하여 접근을 허용하기도 하다.
  * 이러한 결정 이전에는 반드시 private 상태를 유지할 방안을 강구해야 한다.
    * **캡슐화를 풀어버리는 결정은 언제나 최후의 보루**이다.

### 클래스를 작게 만들기
* 클래스는 작아야 한다.
  * **함수는 물리적인 행의 수로 크고 작음을 비교했다면, 클래스에서는 클래스가 맡은 책임을 세어볼 수 있다**.
* **클래스의 크기를 줄이는 첫번째 관문은 작명이며, 클래스의 이름은 해당 클래스의 책임을 기술**해야만 한다.
  * 만약 간결한 클래스 이름이 떠오르지 않는다면?
    * **클래스의 크기가 너무 크다**.
  * 만약 클래스 이름이 너무 모호하다면?
    * **클래스가 맡은 책임이 너무 많다**.
* **클래스는 if, and, or, but을 사용하지 않고 25 단어 내외로 설명이 가능해야** 한다.
  * 이러한 단어들이 들어가야만 설명이 가능한 클래스는 맡은 책임이 너무 많은 상태이다.

### SRP - 단일 책임 원칙
* 클래스나 모듈을 변경할 이유는 단 하나뿐이어야 한다.
  * 따라서 클래스는 책임, 즉 변경할 이유가 하나뿐이어야 한다.
  * 책임(=변경의 이유)를 파악하다 보면 코드를 추상하기도 쉬워진다.
* SRP는 객체 지향 설계에서 매우 중요한 개념이며, 지키기도 쉽지만 자주 무시되는 규칙이다.
  * 대다수의 개발자는 단일 책임 클래스가 많아지면 전체를 이해하기 어렵다고 말한다.
  * 그러나 클래스의 크기는 시스템의 이해도에 영향을 주지 않는다. 
    * 큰 클래스 몇 개로 구성된 시스템이나, 작은 클래스 여러 개로 구성된 시스템이나 이해에 드는 비용은 비슷하다.
* **규모가 큰 시스템은 논리가 많고 복잡하므로, 이러한 복잡성을 다루기 위해서는 반드시 체계적으로 정리해야만 한다**.
  * 그래야만 시스템의 수정사항이 있을 때 변경이 필요한 컴포넌트만 이해하고 작업할 수 있다.
  * **커다란 클래스 몇 개로 구성된 시스템은 수정과 관련 없는 부분까지 모두 이해할 것을 강요**한다.
* 언제나**큰 클래스 몇 개로 구성된 시스템보다 작은 클래스 여럿으로 구성된 시스템이 바람직**하다.
  * 단일 책임을 갖는 작은 클래스는 변경의 이유도 하나이며, 다른 작은 클래스들과 협력하여 시스템에 필요한 동작을 수행한다.

### Cohesion - 응집도
* 클래스는 인스턴스 변수 수가 적어야 하며, 메소드는 인스턴스 변수를 하나 이상 사용해야 한다.
* 응집도는 메소드가 변수를 더 많이 사용할수록 높아진다.
  * 이러한 경우는 메소드와 클래스가 응집도가 높다라는 표현을 사용한다.
  * 예를 들어, 대부분의 메소드가 클래스의 인스턴스 변수를 모두 사용하는 경우가 있다.
* **응집도가 높은 클래스는 메소드와 변수가 서로 의존하며 논리적인 단위로 묶이는 클래스이며, 일반적으로 선호되는 형태**이다.
* 함수를 작게, 매개 변수 목록은 짧게하는 전략을 따르다보면 때때로 몇몇 메소드만이 많은 인스턴스 변수를 사용할 수도 있다.
  * **이는 새로운 클래스로 쪼개야 한다는 신호가 되어 준다**.
  * 이러한 경우에는 응집도가 높아지도록 변수와 메소드를 적절히 분리하고, 새로운 클래스 2-3개로 쪼개주어야 한다.
* **클래스의 응집도는 되도록 높게 유지되는 것이 바람직**하다.

### 응집도를 유지하기
* **응집도를 유지하면 작은 클래스 여러 개**로 쪼개어지게 된다.
* 예를 들어, 다음과 같은 경우를 확인해보자.
  1. 지역 변수를 매우 많이 사용하는 큰 함수로부터 작은 함수 하나를 메소드로 추출하고자 한다.
  2. 이 때, 작은 함수는 큰 함수의 네 개 변수를 사용한다.
  3. 네 개의 변수를 작은 함수의 인수로 넘기는 것보다 인스턴스 변수로 승격하는 것이 바람직하다.
     * 인수 목록은 적게 유지되는 것이 좋기 때문이다.
     * 그러나 이 경우에는 **두 함수만이 사용하는 인스턴스 변수 목록이 발생하므로, 응집도가 낮아진다**.
  4. 몇몇 함수만 사용하는 몇몇 변수가 존재한다면, 독자적인 클래스로 쪼갤 수 있다!
  * 따라서, **클래스의 응집도가 낮아지는 것은 클래스를 쪼개야한다는 신호가 되어준다**.
* **커다란 함수를 작은 함수 여럿으로 쪼개다보면 큰 클래스를 작은 클래스 여럿으로 쪼갤 기회가 생긴다**.
  * 이 과정에서 프로그램은 체계가 잡히고 구조가 투명해진다.

### 변경하기 쉬운 클래스
* 대부분의 시스템은 지속적으로 변경되며, 변경 시점마다 시스템이 의도한대로 동작하지 않을 위험성이 함께한다.
  * **깨끗한 시스템은 클래스를 체계적으로 정리하며 변경에 함께하는 위험성을 낮춰준다**.
* 새로운 기능을 추가하거나 기존 기능을 변경할 때 건드려야하는 코드가 최소인 시스템 구조가 가장 바람직하다.
* **이상적인 시스템은 새로운 기능을 추가할 때 시스템을 확장할 뿐, 기존 코드를 변경하지는 않는다**.
  * 이러한 시스템은 함수 하나를 수정해도 다른 함수가 망가지지 않는다.

### 변경으로부터의 격리
* 요구 사항은 변하기 마련이며, 이에 따라 코드도 계속해서 변해간다.
* 또한, 객체 지향의 입문에서 우리는 다음과 같은 두 사실을 배웠다.
  1. 구체적인 클래스는 상세한 구현(=코드)을 포함한다.
  2. 추상적인 클래스는 개념만을 포함한다.
* **상세한 구현, 즉 구체적인 클래스에 의존하는 클라이언트 클래스는 구체적인 클래스의 구현이 변경되면 위험에 빠진다**.
  * **때문에 우리는 인터페이스와 추상 클래스를 활용하여 구현과 구현의 변경이 미치는 영향을 격리한다**.
* 상세한 구현에 의존하는 코드는 테스트도 어렵다.
  * 예를 들어, 주가나 환율 등 실시간으로 응답이 변하는 외부 API에 의존하는 클래스는 테스트 결과값이 항상 변할 것이다.
  * 이러한 경우, **테스트할 대상 메소드를 포함하는 인터페이스를 정의**하고 외부 API를 호출하기 위한 클래스를 새로이 만들되, 이 인터페이스를 구현하도록 한다.
    * **이로 인해 우리는 이제부터 외부 API를 호출하는 클래스를 흉내내는 테스트용 클래스를 생성할 수 있다**.
      * 예를 들어, 해당 인터페이스를 구현하는 테스트용 클래스는, 항상 고정된 값을 반환하도록 할 수 있다.
      * 이를 통해 구체적인 구현이 아닌 추상적인 개념을 표현하는 인터페이스에 의존하며, 구체적인 사실은 모두 숨길 수 있게 된다.
  * 이러한 과정을 통해 **시스템의 결합도를 낮출 수 있다**.
* 테스트가 가능할 정도로 시스템의 결합도를 낮추면 유연성과 재사용성이 높아진다.
  * **결합도가 낮다는 의미는 각 시스템의 요소가 다른 요소로부터, 그리고 다른 요소의 변경으로부터 잘 격리된다는 의미**이다.
  * 또한 시스템의 요소들이 서로 잘 격리된다면 각 요소를 이해하기도 쉽다.
* 결합도를 줄이려고 노력한다면, 또 다른 클래스 설계 원칙인 DIP를 따르는 클래스가 결과로 나오게 된다.
  * **본질적으로 DIP는 상세한 구현이 아닌 추상화에 의존해야한다는 원칙**이다.

### 개인 리마인드용 - 깨끗한 코드와 코드 길이
* 지저분한 코드를 깨끗한 코드로 리팩토링하는 과정에서 함수와 클래스를 분할하다보면 자연스레 코드의 길이는 길어진다.
* 이러한 현상의 원인으로는,
  1. 리팩토링 과정에서 길고 서술적인 명명을 사용하며,
  2. 코드에 주석을 추가하는 대신 함수와 클래스 정의를 활용하며,
  3. 가독성을 위해 공백을 추가하여 형식을 맞춰주기 때문이다.
* 안전하게 리팩토링하려면 우선 테스트 수트를 작성하고 한 번에 하나씩, 여러 번 조금씩 코드를 변경하며 테스트를 수행한다.

## 시스템
### 시스템의 제작과 사용의 분리
* **제작과 사용은 아주 다르다는 사실을 명심해야 한다**.
* **시스템의 준비 과정(객체 생성 및 의존성 연결)과 런타임 로직은 분리되어야** 한다.
* 일반적으로 준비 과정과 런타임 로직은 뒤섞이는 경우가 많다.
```
public Service getService() {
  // getService는 런타임 로직에서 호출되는 메소드일 것이다.
  // 그러나 아래의 if 블록은 시스템의 준비 과정에 해당하는 코드이다.
  if(service == null)
    service = new MyServiceImpl(...);
  return service;
}
```
* 상술한 getService 메소드는 MyServiceImpl 클래스와, 생성자에 전달되는 인수(... 부분)에 의존한다.
* 또한 getService가 사용되는 모든 상황에 MyServiceImpl이 적합하지 않을 수도 있다.
  * 현실적으로 하나의 객체 유형이 모든 문맥에 적합할 가능성은 적다.
* 위와 같은 방식은 초기화 지연 기법에 해당하지만, 이를 너무 자주 사용하면 모듈성이 낮아지고 중복이 심해지는 문제가 있다.
* 체계적이고 탄탄한 시스템은 설정 논리와 일반 실행 논리를 분리하고 모듈성을 높이는데에서 시작한다.
  * 또한, 의존성을 해결하기 위한 전반적이고 일괄적인 방식도 필요하다.

### Main의 분리
* Main 분리는 시스템 생성과 사용을 분리하는 하나의 방법이다.
  1. 시스템 **생성과 관련된 모든 코드를 main이나 main이 호출하는 모듈로 옮긴다**.
  2. 나머지 시스템은 모든 객체가 생성되었고, 모든 의존성이 연결되었다고 가정하고 실행된다.
* main 함수에서는 시스템에 필요한 객체를 생성하고, 이를 애플리케이션에 넘긴다.
  * 애플리케이션은 생성된 객체를 사용하기만 한다.
* 이를 통해 main은 애플리케이션에 의존하며, 애플리케이션은 main이나 객체가 생성되는 과정을 알 필요가 없다.
  * 단지 모든 객체가 적절히 생성된 것을 가정하고 실행된다.
* 만약 **객체가 생성되는 시점을 애플리케이션이 결정해야 하는 경우, 추상 팩토리 패턴을 사용할 수 있다**.
  * 이 경우, 객체의 생성 시점은 애플리케이션이 결정하지만 객체를 생성하는 코드는 알 수 없게 된다.

### 의존성 주입
* 의존성 주입은 사용과 제작을 분리하는 강력한 메커니즘이다.
  * 의존성 주입은 제어 역전 기법을 의존성 관리에 적용한다.
* 객체는 의존성 자체를 인스턴스화하는 책임은 지지 않고, 이러한 작업을 전담하는 메커니즘에 맡긴다.
  * **객체가 의존성 클래스를 인스턴스화하는 것은 어떻게 보면 자신의 기능과 인스턴스화, 두가지 책임을 갖는 것으로 이해할 수 있다**.
    * 즉, 시스템 설정과 시스템 사용을 모두 담당하게 되므로 SRP를 위반한다.
  * 이러한 초기화 작업은 시스템 전체에서 필요하므로, 이를 전담하기 위해 main 루틴이나 특수한 컨테이너를 사용한다.
    * Spring을 예로 들어, 의존성 주입만을 위한 DI 컨테이너 기능을 제공한다.
* **진정한 의존성 주입에서, 클래스는 수동적이며 스스로 의존성을 해결하지 않는다**.
  * 대신 클래스는 의존성을 주입하는 데에 활용할 수 있을 setter나 생성자 인수, 또는 둘 모두를 제공한다.
  * DI 컨테이너는 요청이 들어올 때 의존성 클래스를 인스턴스화하고, 이미 제공되는 생성자 인수나 setter를 통해 주입한다.
    * 이 때, DI 컨테이너는 실제로 생성되는 객체의 유형을 제어하지 않는다. 
    * 이러한 객체 유형은 설정 파일에 지정되거나 특수 생성 모듈에서 코드로 명시한다.
* 대부분의 DI 컨테이너는 필요할 때까지 객체를 생성하지 않고, 계산 지연이나 비슷한 최적화 기법에 활용할 수 있도록 팩토리 호출 또는 프록시 생성을 제공한다.

### 개인 리마인드용 - 의존성 주입과 단일 책임
* 어떤 클래스가 메소드에 사용하기 위해 다른 클래스를 사용하는 경우, 이는 두가지 문제를 갖는다.
  1. 클래스 의존성
  2. 단일 책임 원칙 위배: 클래스가 '의존성의 인스턴스화'와 '자신이 담당하는 기능' 두 가지 책임을 가짐.
* 때문에 클래스의 보조 책임인 '의존성의 인스턴스화'를 전담하는 클래스 또는 메커니즘에 맡길 필요가 있다.
* 이 때, 의존성의 인스턴스화 및 주입을 위한 메커니즘으로 DI 컨테이너가 있으며, 의존성을 능동적으로 해결해야하는 책임만을 갖는다. 
  * 반면 이러한 메커니즘은 실제로 반환되는 의존성 인스턴스의 유형을 제어하지 않는다.
* 상술한 예제에서 시스템의 설정과 생성을 함께하던 코드는 단점이 있지만, 초기화 지연으로 인한 장점도 갖고 있다.
  * 때문에 일반적인 DI 컨테이너는 초기화 지연과 같은 장점을 포기하지 않기 위한 최적화 메커니즘을 제공한다.

## 2022-02-02 Wed
### 확장
* **처음부터 확장된 시스템을 올바르게 만들 수 있다는 믿음은 미신**이다.
```
> 우리는 오늘 주어진 사용자 스토리에 맞추어 시스템을 구현해야만 한다.
> 내일은 내일 있을 새로운 스토리에 맞추어 시스템을 조정하고 확장하면 된다.
> 이것이 애자일 방식의 핵심이다!
```
* **TDD, 리팩토링, 그리고 깨끗한 코드는 코드 수준에서 시스템을 조정하고, 확장하기 쉽게 만들어준다**.
* **시스템은 상황이 약간 다르며, 관심사를 적절히 분리하여 관리해야 점진적으로 발전할 수 있다**.
  * **소프트웨어는 수명이 짧다는 본질로 인해 아키텍쳐의 점진적인 발전이 가능**하다.
* 시스템 아키텍쳐는 필요한 관심사를 적절히 분리할 수 있어야 유기적으로 성장할 수 있다.
  * 예를 들어, 시스템의 설정 단계에서 객체를 준비하는 것 역시 관심사이다.
  * 우리는 앞서 **시스템의 설정이라는 관심사를 의존성 주입을 통해 분리**하였다.
* 횡단 관심사: 보안, 트랜잭션, 영속성 등은 모듈화와 캡슐화가 가능하지만, 그 특성 상 코드가 여기 저기로 흩어지는 경향이 있다. 
  * 이 때, 여기저기 흩어진 코드에서 찾아낸 같은 관심사가 횡단 관심사이다.
  * 횡단 관심사는 현실적으로 캡슐화와 모듈화가 어려운 특징이 있다.
* **AOP는 횡단 관심사에 대처해 모듈성을 확보하기 위해 고안된 일반적인 방법론**이다.
* AOP에서, 관점이라는 모듈 구성 개념은 다음을 명시한다.
  1. 특정 관심사를 지원하려면, 시스템에서 특정한 지점들이 동작하는 방식을 일관성 있게 바꾸어야 한다!
  2. 이 때, 명시는 간결한 선언이나 프로그래밍 메커니즘을 활용한다.
* Java에서는 관점(또는 관점과 유사한) 메커니즘을 다음과 같이 지원한다.
  1. Java 프록시: 단순한 상황에 적합하며, 예를 들어 개별 객체나 클래스에서의 메소드 호출을 감싸는 경우가 있다. 
     * 프록시는 깨끗한 코드를 작성하기 어렵다.
     * 진정한 AOP에 필요한 '시스템 단위로 명시된 실행 지점 메커니즘'을 지원하지 않는다.
  2. 순수 Java AOP 프레임워크: Spring AOP 등이 있으며, 모든 정보가 어노테이션 속에 포함되므로 코드가 깔끔해진다.
     * 이러한 방식은 관점이 필요한 상황 중 80 - 90%를 만족시킨다.
  3. AspectJ: 관심사를 관점으로 분리하는 가장 강력한 도구이다.
     * 언어 차원에서 관점을 모듈화 구성으로 지원하는 Java의 확장이다.
     * 관점을 분리하는 강력하고 풍부한 도구 집합을 제공하나, 러닝커브가 존재하는 단점이 있다.

### 테스트 주도 시스템 아키텍쳐를 구축하기
* 관점(또는 유사한 개념)으로 관심사를 분리하는 방식은 위력이 엄청나다.
* 애플리케이션 도메인 논리를 POJO로 작성할 수만 있다면 진정한 테스트 주도 아키텍쳐 구축이 가능하다.
  * 즉, 코드 수준에서 아키텍쳐 관심사를 분리할 수 있는 경우이다.
* 소프트웨어 구조가 관점을 효과적으로 분리한다면 아키텍쳐는 극적인 변화도 가능하다.
  * 프로젝트의 초기에 범위, 목표, 일정, 결과가 될 일반적인 시스템의 구조도 생각해야 하지만, 변경에 대처할 수 있는 능력도 고려해야 한다.
* **최선의 시스템 구조는 각기 POJO 객체로 구현되는 모듈화된 관심사 영역인 도메인으로 구성**된다.
  * 서로 다른 도메인은 해당 도메인 코드에 최소의 영향을 미치는 관점(또는 유사한 도구)로 통합해야 한다.

### 의사 결정의 최적화
* 최대한 많은 정보를 모아 최선의 결정을 내리기 위해, 가능한 마지막까지 결정을 미루는 것이 최선의 방법이다.
* 관심사를 모듈로 분리한 POJO 시스템을 이를 위한 기민함을 제공한다.
  * 이상적인 시스템은 최신 정보를 기반으로 최선의 시점에 최적의 결정을 내리기 쉽도록 지원한다.

### 결론
* 시스템 또한 깨끗해야 하며, 그렇지 못한 아키텍쳐는 도메인 논리를 흐려 기민성을 떨어트린다.
* 기민성이 떨어지는 아키텍쳐는,
  1. 제품 품질이 떨어지며,
  2. 스토리의 구현이 어려워지고,
  3. 생산성이 낮아진다.
* 모든 추상화 단계에서의 의도는 명확히 표현되어야 한다.
  * 이를 위해 POJO를 작성하고, 관점(또는 유사한 메커니즘)을 통해 구현 관심사인 도메인을 분리해야 한다.
* 시스템이든 개별 모듈이든, 설계시에는 실제로 돌아가는 가장 단순한 수단을 사용해야만 한다.