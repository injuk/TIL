# EffectiveCodes
## 2022-03-16 Wed

## 동시성
### 공유 중인 가변 데이터는 동기화하여 사용하기
* 동기화는 크게 다음과 같은 의의를 갖는다.
  1. 배타적 실행, 한 스레드가 변경하는 순간에 다른 스레드가 변경하지 못하도록 막는다.
  2. 스레드 간 통신, **동기화된 메소드 또는 블록에서 진행된 이전 수정 작업의 최종 결과만을 볼 수 있도록 보장**한다.
* Java 언어 명세상, long과 double 이외의 변수를 읽고 쓰는 동작은 원자적이다.
  * **그러나 하나의 스레드가 저장한 값이 다른 스레드에게 보이는지 보장하지는 않는다**.
  * 동기화를 적절히 처리하지 않는 경우, 하나의 스레드가 수정한 값을 다른 스레드가 언제 쯤 보게될지 보장할 수 없다.
* volatile 키워드는 배타적 수행과는 관계 없지만, 항상 가장 최근에 기록된 값을 읽어들일 수 있도록 스레드 간 통신을 지원한다.
* java.util.concurrent.atomic 패키지에는 락 없이도 스레드 안전한 개발을 지원하는 클래스들이 포함된다.
  * 예를 들어, **volatile은 통신만 지원하지만 해당 패키지는 배타적 실행까지 지원하며 성능도 우수**하다.
* 그러나 **가장 좋은 방법은 애초에 가변 데이터를 공유하지 않는 것**이다.
  * 대신 불변 데이터를 공유하거나, 아무것도 공유하지 않아야 한다.
  * 가변 데이터는 가능한 한 단일 스레드에서만 사용한다.

### 가변 데이터 동기화의 결론
* 여러 스레드가 가변 데이터를 공유한다면, 데이터를 읽고 쓰는 동작을 반드시 동기화한다.
* 배타적 실행은 필요 없고 스레드끼리의 통신만 필요한 경우, volatile 한정자만으로도 동기화할 수 있지만 올바로 사용하기가 어려울 수 있다.

### 과도한 동기화는 피하기
* 충분하지 못한 동기화에 문제가 있듯, 과도한 동기화 역시 문제가 생길 수 있다.
  * 과도한 동기화는 성능을 떨어트리고, 예측할 수 없는 동작으로 이어질 수도 있다.
* **응답 불가와 잘못된 계산 결과를 반환하는 안전 실패를 피하려면 동기화 메소드나 동기화 블록 안에서의 제어권을 절대 클라이언트 코드에 넘기지 말아야 한다**.
  1. 동기화된 영역 안에서는 재정의할 수 있는 메소드를 호출하지 않아야 한다.
  2. 클라이언트가 넘겨진 함수 객체를 호출하지 않아야 한다.
* **기본적으로 동기화 영역에서는 일을 가능한 한 적게 수행하는 것이 바람직**하다.
* 가변 클래스를 작성하는 경우, 다음과 같은 두 원칙 중 하나를 따르도록 해야 한다.
  1. 동기화를 전혀 하지 않고, 해당 클래스를 동시에 사용해야 하는 클래스가 외부에서 동기화하도록 한다.
     * 이는 java.util 패키지가 채택한 방식이다.
  2. 동기화를 내부에서 수행하여 스레드 안전한 클래스로 만들되, 1.의 방법보다 동시성이 월등히 개선되는 경우에만 적용한다.
     * 이는 java.util.concurrent 패키지가 채택한 방식이다.
* **두 원칙 중 적절한 것을 선택하기 어렵다면, 동기화하지 말고 문서에 스레드 안전하지 않은 클래스임을 명시해도 무방**하다.
* Java의 라이브러리에는 이러한 지침을 따르지 않은 클래스가 많다.
  * StringBuffer는 대부분의 경우 단일 스레드에서 사용되지만, 내부적으로 동기화를 수행한다.
    * 때문에 동기화하지 않는 StringBuffer인 StringBuilder가 추후에 추가되었다.
  * **java.util.Random은 스레드 안전하므로, 추후 동기화하지 않는 버전인 java.util.concurrent.ThreadLocalRandom으로 대체**되었다.

### 과도한 동기화의 결론
* 멀티 코어가 만연한 현재는 과도한 동기화를 피하는 것이 무엇보다 중요하다.
* 동기화 영역 내부에서의 작업은 최소화하고, 절대 외부로부터 전달된 메소드를 호출하지 말아야 한다.
* 필요한 이유가 있을 때에만 클래스 내부에서 동기화하고, 동기화 여부는 반드시 문서에 명시한다.

### 스레드보다는 실행자, 태스크, 스트림을 사용하기
* **java.util.concurrent 패키지에는 인터페이스 기반의 유연한 태스크 실행 기능을 포함하는 실행자 프레임워크가 존재**한다.
* **필요한 실행자 대부분은 java.util.concurrent.Executors의 정적 팩토리로 생성**할 수 있다.
* **작업 큐를 직접 만들거나, 스레드를 직접 다루는 것은 일반적으로 지양**해야 한다.
  * 스레드를 직접 다루면 Thread가 작업 단위와 수행 메커니즘 두 역할을 모두 수행하게 된다.
* 실행자 프레임워크에서는 작업 단위와 실행 메커니즘은 분리된다.
  * 이 때, **작업 단위를 나타내는 핵심적인 추상 개념을 태스크라고 한다**. 
* **태스크는 다시 Runnable과 Callable로 나뉘며, 태스크를 수행하는 일반적인 메커니즘이 실행자 서비스**이다.
  * 즉, **마치 데이터의 모음을 컬렉션 프레임워크가 담당하듯 실행자 프레임워크가 전반적인 작업 수행을 담당**해줄 수 있다.

### wait와 notify 보다는 동시성 유틸리티를 사용하기
* Java 5에 도입된 고수준의 동시성 유틸리티 덕분에 이제는 wait와 notify를 직접 사용할 필요가 줄게 되었다.
  * **wait와 notify는 바르게 사용하기 어려우므로 고수준의 동시성 유틸리티를 사용하는 것이 바람직**하다.
* java.util.concurrent에 포함된 고수준 유틸리티는 다음과 같은 세 범주로 나누어볼 수 있다.
  1. 실행자 프레임워크
  2. 동시성 컬렉션
  3. 동기화 장치
* **동시성 컬렉션은 표준 컬렉션 인터페이스에 동시성을 추가하여 구현한 고성능의 컬렉션**이다.
  * 동시성 컬렉션은 높은 동시성을 보장하기 위해 동기화를 컬렉션 내부에서 수행한다.
  * **따라서 내부에서 처리되는 동시성을 무력화할 수 없으며, 외부에서 락을 추가로 명시하는 경우 오히려 성능이 떨어질 수 있다**.
* 동시성 컬렉션의 동시성을 무력화할 수 없으므로, 여러 메소드를 원자적으로 묶어 호출할 수 없다.
  * 때문에 **여러가지 기본 동작을 하나의 원자적 동작으로 묶는 상태 의존적 수정 메소드들이 추가**되었다.
  * 예를 들어, Map의 putIfAbsent 메소드는 키에 매핑된 값이 없을 경우에만 값을 추가하고, 기존 값의 존재 여부에 따라 null 또는 해당 값을 반환한다.
* **동시성 컬렉션은 동기화한 컬렉션을 사용할 필요를 없애주었다**. 
  * 예를 들어, 이제는 Collections.synchronizedMap보다 ConcurrentHashMap을 사용하는 것이 좋다.
* 동기화 장치는 스레드가 다른 스레드를 기다리도록 하며, 스레드 간 작업을 조율할 수 있도록 한다.
  * 가장 자주 사용되눈 CountDownLatch와 Semaphore가 있다.
* **새로운 코드를 작성하는 경우, 언제나 wait와 notify 대신 동시성 유틸리티를 사용**한다.
* 레거시 코드를 수정해야 하는 경우, 다음과 같은 기본 규칙을 따라 작업한다.
  1. wait는 while 반복문 안에서 호출하며, 절대 반복문 외부에서 호출하지 않도록 한다.
  2. 일반적으로 notify보다 notifyAll을 사용한다.
  3. notify를 반드시 사용해야하는 경우라면 응답 불가 상태에 각별히 주의한다.

### 스레드 안전성 수준은 문서화하기
* 하나의 메소드를 여러 스레드가 동시에 호출하는 경우, 해당 메소드의 동작 방식은 클래스 자체와 클라이언트 코드와의 계약과도 같다.
* API 문서에서 이를 명시하지 않는 경우 사용자는 나름의 가정을 수립하여 개발하며, 이는 동기화를 충분히 하지 않거나 과도하게 하는 상황의 원인이 될 수 있다.
* API를 멀티스레드 환경에서도 안전하게 사용하려면 클래스가 지원하는 스레드 안전성 수준을 정확히 명시해야 한다.
* **스레드 안전성은 다음과 같이 분류**할 수 있다.
  1. 불변: 해당 클래스의 인스턴스는 마치 상수처럼 동작하며, 외부 동기화가 필요 없다.
  2. 무조건적 스레드 안전: 해당 클래스의 인스턴스는 수정될 수 있으나, 내부에서 적절히 동기화되어 외부 동기화가 필요 없다.
  3. 조건부 스레드 안전: 전반적으로 무조건적 스레드 안전과 같으나, 일부 메소드는 동시성을 위해 외부 동기화가 필요하다.
  4. 스레드 안전하지 않음: 해당 클래스의 인스턴스는 수정될 수 있으며, 동시에 사용하려면 각각의 메소드 호출을 클라이언트가 외부 동기화로 감싸야만 한다.
  5. 스레드 적대적: 해당 클래스는 모든 메소드 호출을 동기화하더라도 멀티스레드 환경에서 안전하지 않다.
* 스레드 안전성을 문서화하기 위해 직접 설명을 작성해도 좋지만, 상술한 스레드 안전성 분류에 대응하는 스레드 안전성 어노테이션을 사용할 수도 있다.
  * @Immutable, @ThreadSafe, @NotThreadSafe가 있으며, 무조건적 / 조건부 스레드 안전은 모두 @ThreadSafe의 분류에 속한다.
* 특히 조건부 스레드 안전한 클래스는 신중하게 문서화해야 한다.
  * 예를 들어, 어떤 순서로 호출할 때 외부 동기화가 필요한지 또는 해당 순서로 호출하기 위해 어떤 락을 얻어야하는지 명시해야 한다.
* 클래스의 스레드 안전성은 클래스의 문서화 주석에 기재하지만, 특이한 특성을 갖는 메소드는 해당 메소드의 주석에 기재해도 좋다.
* 클래스가 외부에서 사용할 수 있는 락을 제공하면 클라이언트에서의 일련의 메소드 호출을 원자적으로 수행할 수 있다.
  * 그러나 이러한 방식은 내부에서 처리하는 동시성 제어 메커니즘과 혼용할 수 없으며, 클라이언트가 락을 고의로 놓지 않는 서비스 거부 공격을 방지할 수 없다.
* 서비스 방지 공격을 방지하기 위해 synchronized 메소드 대신 비공개 락 객체를 사용할 수 있으며, 이는 무조건적 스레드 안전 클래스에서만 사용할 수 있다. 

## 2022-03-17 Thu
### 지연 초기화는 신중하게 사용하기
* **지연 초기화는 필드의 초기화 시점을 해당 값이 처음 필요할 때까지 늦추는 기법이지만, 다른 최적화 기법과 마찬가지로 필요할 때까지 고려하지 말아야** 한다.
  * 지연 초기화는 인스턴스 생성 비용을 줄여주는 대신, 해당 필드에 접근하는 비용을 키우는 방식이다.
  * 따라서 지연 초기화 대상 필드의 호출 빈도에 따라 성능은 떨어질 수 있다.
* 해당 클래스의 필드 중 지연 초기화 필드를 사용하는 빈도가 낮으며, 해당 필드의 초기화 비용이 큰 경우에 지연 초기화를 고려할 수 있다.
* **멀티스레드 환경에서는 지연 초기화가 까다로우며, 지연 초기화 필드를 둘 이상의 스레드가 공유한다면 반드시 동기화**해야 한다.
* **대부분의 상황에서는 일반적인 초기화가 지연 초기화보다 좋다**.
* 성능 때문에 반드시 지연 초기화를 사용해야 하는 경우, 적절한 지연 초기화 기법을 반드시 사용해야 한다.
  1. 인스턴스 필드에는 volatile 키워드와 함께 이중검사 관용구를 사용한다.
  2. 정적 필드에는 지연 초기화 홀더 클래스 관용구를 사용한다.
  3. 반복하여 초기화해도 무방한 인스턴스 필드에는 단일검사 관용구를 고려할 수 있다.
  * 단일검사 관용구의 경우, long과 double이 아닌 기본 타입이라면 volatile 한정자를 생략해도 무방하다.

### 프로그램의 동작을 스레드 스케쥴러에 기대지 않기
* 여러 스레드가 실행되는 경우, 운영체제의 스레드 스케쥴러는 각 스레드의 실행 시간을 결정한다.
  * 이러한 스레드 스케쥴러의 정책은 운영체제마다 다를 수 있다.
* 잘 작성된 프로그램은 운영체제에 종속되는 스레드 스케쥴러 정책에 의존해서는 안된다.
  * 정확성이나 성능이 스레드 스케쥴러의 정채겡 의존한다면, 다른 플랫폼에 이식하기 어려운 프로그램이 된다.
* **견고하고 이식성이 좋은 프로그램을 작성하는 좋은 방법은 실행 가능한 스레드의 평균적인 수를 프로세서 수보다 과도하게 많아지지 않도록 하는 것**이다.
  * 이 때, 실행 가능한 스레드 수와 전체 스레드 수는 구분되어야 한다.
  * 전체 스레드 수는 훨씬 많을 수 있으며, 대기 중인 스레드는 실행 가능한 스레드가 아니다.
* **실행 가능한 스레드 수를 적게 유지하는 좋은 방법은 각 스레드가 작업을 완료한 후에는 다음 작업이 생길 때까지 대기하는 것**이다.
  * 스레드는 당장 처리해야할 작업이 없다면 실행되지 않아야 한다.
* **실행자 프레임워크의 예시로, 스레드 풀 크기는 적절히 설정하고 작업은 짧게 유지**하도록 한다.
  * 단, 작업이 너무 짧아진다면 컨텍스트 스위칭 비용이 성능을 되려 떨어트릴 수도 있다.
* **특정한 스레드가 다른 스레드들보다 충분한 CPU 시간을 획득하지 못해 간신히 동작하는 프로그램을 보더라도 Thread.yield를 활용하여 고치지 말아야 한다**.
  * Thread.yield는 테스트할 수단도 없다.
* 같은 이유에서 스레드 우선 순위를 조절하는 방법도 사용하지 않아야 한다.
  * **스레드 우선 순위는 Java에서 이식성이 가장 나쁜 특성**에 속한다.
  * 스레드 몇 개의 우선 순위를 조절하여 프로그램의 품질을 높일 수도 있지만, 이러한 경우는 드물고 이식성도 떨어진다.
  * **무엇보다 스레드 우선 순위로 멀티스레드 프로그램의 문제를 고치려는 시도는 절대 하지 않아야 한다**.

### 스레드 스케쥴러의 결론
* 프로그램의 동작은 스레드 스케쥴러와 Thread.yield, 스레드 우선 순위에 의존하지 않아야 하며, 이는 프로그램의 견고성과 이식성을 떨어트린다.
  * 이러한 기능들은 스레드 스케쥴러에 제공되는 힌트에 지나지 않는다.
* 스레드 우선 순위는 잘 동작하는 프로그램의 품질을 높이기 위해 사용할 수도 있지만, 간신히 동작하는 프로그램을 고치는 용도로는 절대 사용하지 말아야 한다.