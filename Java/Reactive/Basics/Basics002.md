# Reactive
## 2023-10-13 Fri
### 리액티브 스트림즈란?
```
> 리액티브 스트림즈란, 리액티브 라이브러리를 구현하기 위해 준수해야 할 표준을 의미한다.
> 결국 리액티브 스트림즈는 Data Stream을 Non-Blocking하면서 비동기적으로 처리할 수 있도록 지원하는 리액티브 라이브러리의 표준 사양을 의미한다.
```
* **리액티브한 코드를 작성하기 위해서는 적절한 라이브러리의 지원을 받을 필요가 있으며, 이러한 라이브러리들은 리액티브 스트림즈라는 표준을 준수**한다.
  * 예를 들어 `RxJava`, `Reactor`, `Akka Streams`, `Java 9 Flow API` 라이브러리 등은 리액티브 스트림즈를 구현한 구현체에 해당한다.

## 2023-10-14 Sat
### 리액티브 스트림즈의 구성 요소
* 리액티브 스트림즈를 기반으로 반드시 구현해야 하는 API 컴포넌트는 크게 다음과 같다.
  1. `Publisher`: 데이터를 생성하고 이를 게시한다.
  2. `Subscriber`: `Publisher`를 구독하고, 게시된 데이터를 전달 받아 처리한다.
  3. `Subscription`: `Publisher`에 요청할 데이터의 개수를 지정하거나, 데이터의 구독을 취소한다.
  4. `Processor`: `Publisher`와 `Subscriber`의 기능을 모두 가진다.

## 2023-10-15 Sun
### 간단한 리액티브 스트림즈의 동작 과정
* 예를 들어 리액티브 스트림즈의 `Publisher`와 `Subscriber`가 데이터를 요청하고 전달 받는 과정은 다음과 같이 간략화할 수 있다.
  1. `subscribe`: `Subscriber`는 전달받을 데이터를 구독한다.
  2. `onSubscribe`: `Publisher`가 데이터를 게시할 준비가 되었을 때 이를 `Subscriber`에게 알린다.
  3. `Subscription.request`: `Subscriber`는 `Publisher`에게 전달 받고자 하는 데이터의 개수를 요청한다.
  4. `onNext`: `Publisher`는 `Subscriber`가 요청한 만큼의 데이터 개수를 전달한다.
  5. `onComplete`: 상술한 과정을 반복하는 과정에서, `Publisher`가 모든 데이터 개수를 전달한 경우 `Subscriber`에게 전송 완료를 알린다.
  6. `onError`: 반면, `Publisher`가 데이터를 처리하는 과정에서 예외가 발생한 경우에도 이를 `Subscriber`에게 알려야 한다.
* 이 때, **`Publisher`는 `Subscriber`에게 데이터를 일방적으로 전달하는 대신 우선 필요한 데이터의 수를 요청 받는 식으로 동작**한다.
  * 이는 **두 구성 요소가 일반적으로 같은 스레드가 아닌 다른 스레드에서 비동기적으로 상호작용하기 때문**이다.
  * 이는 **`Publisher`의 전달 속도가 `Subscriber`가 처리하는 속도가 빠른 경우에 데이터가 쌓여 시스템 부하로 이어지는 상황을 미연에 방지**한다.

## 2023-10-16 Mon
### Publisher 인터페이스
```
> 상술한 리액티브 스트림즈 컴포넌트들은 실제로는 인터페이스 형태로 정의되며, 이들을 구현하여 해당 컴포넌트를 사용하게 된다.
```
* 예를 들어, `Publisher` 인터페이스는 `Subscirber`를 등록할 수 있도록 `subscribe` 메소드를 제공한다.
  * 이렇듯 카프카와 같은 애플리케이션의 Pub/Sub 모델과 달리 `subscribe` 메소드를 `Publisher`가 갖는다.
* 카프카의 경우 중간에 위치한 메시지 브로커와, 메시지 브로커가 관리하는 여러 개의 토픽을 기반으로 느슨한 결합 구조를 갖는다.
  * 때문에 `subscribe`와 같은 구독 기능을 `Subscriber`가 수행하게 된다.
* 반면, **리액티브 스트림즈의 경우 `Publisher`가 `subscribe` 메소드를 기반으로 `Subscriber`를 등록하는 형태로 구독이 시작**된다.

## 2023-10-17 Tue
### Subscriber 인터페이스
* `Subscriber` 역시 리액티브 스트림즈에서는 인터페이스로 정의되며, 크게 다음과 같은 메소드들을 구현해야 한다.
  1. `onSubscribe`: 구독이 시작될 때 처리할 작업을 정의하며, 주로 `Publisher`로의 요청 개수를 지정하거나 구독 해지 등의 작업을 처리한다.
  2. `onNext`: `Publisher`가 전달한 데이터를 처리하기 위한 작업을 명시한다.
  3. `onError`: `Publisher`가 데이터를 전달하는 과정에서 예외가 발생한 경우, 이를 처리하기 위한 작업을 명시한다.
  4. `onComplete`: `Publisher`가 모든 데이터 전달을 완료했을 때, 후처리에 필요한 작업을 명시한다.

## 2023-10-18 Wed
### Subscription 인터페이스
* `Subscription` 인터페이스는 크게 `request`와 `cancel` 메소드를 제공하며, 각각 다음과 같은 기능을 정의하게 된다.
  1. `request`: Long 타입 값을 인자로 받아 동작하며, `Subscriber`가 `Publisher`에게 요청할 데이터의 개수를 의미한다.
  2. `cancel`: `Publisher`에게 전달된 데이터의 요청 취소, 즉 구독을 해지하기 위한 기능을 정의한다.

## 2023-10-19 Thu
### Publisher와 Subscriber, Subscription의 상호 작용
* 상술한 세 구성 요소는 다음과 같은 흐름을 거쳐 상호 작용할 수 있다.
  1. `Publisher`가 제공하는 `subscribe` 메소드에 `Subscriber` 구현 객체를 전달하여 통신을 시작한다.
  2. `Publisher`는 내부적으로 참조하는 구독자인 `Subscriber`의 `onSubscribe` 메소드를 호출하여 구독을 의미하는 `Subscription`을 전달한다.
  3. `Subscriber`는 `Subscription` 구현 객체를 내부적으로 참조하며, `request` 메소드를 통해 `Publisher`에게 필요한 데이터의 개수를 요청한다.
  4. `Publisher`는 `Subscriber`의 `onNext` 메소드를 호출하여 전달받은 요청 개수만큼 데이터를 전달한다.
  5. `Publusher`는 요청 받은 개수 만큼의 데이터를 모두 통지한 경우, `Subscriber`의 `onComplete` 메소드를 호출하여 처리 종료를 전달한다.

## 2023-10-20 Fri
### Processor 인터페이스
* `Processor` 인터페이스는 상술한 구성 요소들과 달리 정의된 메소드를 갖지 않으며, 다음과 같이 단지 `Subscriber`와 `Publisher`를 확장한다.
  * 이는 단지 `Processor`가 두 구성 요소의 모든 기능을 갖기 때문이다.
```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}
```

## 2023-10-21 Sat
### 리액티브 스트림즈에서 사용되는 용어들 - Signal, Demand, Emit
* **`Signal`(이하 '시그널')은 `Publisher`와 `Subscriber`가 서로 주고 받는 모든 상호작용을 일종의 신호로 표현한 용어에 해당**한다.
  * 때문에 `onNext`와 같은 `Subscriber`의 메소드들에 더해 `request`와 같은 `Subscription`의 메소드 모두 시그널에 해당한다.
  * 이 때, `Subscriber`가 제공하는 메소드들은 실제로는 `Publisher`가 호출하게 되므로 `Publisher`로부터 `Subscriber`에 향하는 시그널이다.
  * 같은 논리에 의해 `Subscription`이 제공하는 메소드들은 `Subscriber`가 호출하므로 `Subscriber`가 `Publisher`에게 전달하는 시그널이다.
* **`Demand`는 `Subscriber`가 `Publisher`에게 요청했으나, 아직 `Publisher`가 전달하지 않은 데이터를 의미**한다.
* **`Emit`은 `Publisher`가 `Subscriber`에게 데이터를 전달하는 행위 자체를 의미**한다.
  * 특히, `Publisher`가 `Emit`하는 시그널 중 데이터를 전달하기 위한 `onNext` 시그널은 `데이터를 emit한다`고 표현할 수 있다.

## 2023-10-22 Sun
### 리액티브 스트림즈에서 사용되는 용어들 - Upstream/Downstream, Sequence, Operator, Source
* 다음과 같은 코드를 예로 들어, 구성되는 메소드 체인은 각각의 메소드 호출이 모두 `Flux` 객체를 반환하기 때문에 가능하다.
  * 이 때, 데이터 스트림의 관점에서 미루어 보았을 때 `just` 메소드로 반환된 `Flux`는 `filter`에 의해 반환된 `Flux`보다 상위에 위치한다.
  * 이렇듯 **메소드 호출 흐름 중 더 상위에서 반환된 `Flux`는 `Upstream`, 더 하위에 있는 `Flux`는 `Downstream`이라고 지칭**한다.
```kotlin
public fun doSomething() 
  = return Flux
    .just(1, 2, 3)
    .filter(n -> n % 2 == 0)
    .map(n -> n * 3)
    .subscribe(println)
```
* **`Sequence`는 `Publisher`가 `Emit`하는 데이터의 연속적인 흐름을 정의한 그 자체를 의미**한다.
  * 다시 말해, **`Sequence`는 다양한 `Operator`를 활용하여 데이터의 연속적인 흐름을 메소드 체인 형태로 정의한 것을 의미**한다.
  * 즉, 상술한 `doSomething()` 함수로부터 시작되는 메소드 호출 흐름 자체를 `Sequence`로 볼 수 있다.
* 상술한 `doSomething` 함수에서 정의된 **`just` 등의 메소드들은 `Operator`라는 용어로 총칭**할 수 있다.
  * **리액티브 프로그래밍은 `Operator`로 시작하여 끝난다고 해도 과언이 아니며, 이렇듯 `Operator`는 리액티브 프로그래밍의 핵심**과도 같다.
* **`Source`, 또는 `Original`은 마치 원본처럼 최초로 생성된 '무언가'를 의미**한다.
  * 예를 들어, 해당 용어는 `Data Source`나 `Source Flux`와 같은 방식으로 사용되곤 한다.