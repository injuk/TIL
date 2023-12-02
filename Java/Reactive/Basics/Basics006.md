# Reactive
## 2023-11-26 Sun
### Sinks란?
```
> Sinks는 리액티브 스트림즈의 Signal을 프로그래밍적인 방식으로, 명시적으로 전송할 수 있도록 지원하는 요소에 해당한다. 
```
* 리액티브 스트림즈의 구성 요소 중 하나인 `Processor`는 `Publisher`와 `Subscriber`의 모든 특징을 갖는다.
* 그러나 최근 Reactor에서는 `Processor`의 기능을 개선한 `Sinks`가 지원되고 있으며, `Processor`와 관련된 API는 제거된다.
* 앞서 다룬 내용은 모두 Flux나 Mono가 `onNext() Signal` 등을 내부적으로 전송하는 방식에 해당한다.
  * 반면, **`Sinks`를 활용하는 것으로 `Signal`은 명시적으로 전송**될 수 있다.

## 2023-11-27 Mon
### Sinks와 Operator 방식의 비교
```
> Sinks는 프로그래밍적인 방식으로 Signal을 전송하면서도 멀티스레드 기반 환경에서 스레드 안전성을 보장받을 수 있도록 지원한다.
```
* Reactor에서 프로그래밍 방식으로 `Signal`을 전송하는 가장 일반적인 방식은 크게 다음과 같다.
  1. `generate() Operator`
  2. `create() Operator`
* 이 때, 상술한 **두 `Operator`는 싱글스레드 기반 환경에서 `Signal`을 전송하기 위해 사용되던 전통적인 방식에 해당**한다.
* 반면, **`Sinks`는 멀티스레드 방식으로 `Signal`을 전송하면서도 그 과정에서 스레드 안전성이 보장되도록 지원**한다.
  * 즉, `Sinks`는 상술한 두 `Operator`를 완전히 대체하려는 개념은 아니다.
* 또한 **`Sinks`는 공유 자원에 대한 동시 접근을 감지하고, 접근하려는 스레드 중 하나가 빠르게 실패할 수 있도록하는 것으로 스레드 안전성을 보장**한다.

## 2023-11-28 Tue
### Sinks의 종류
* Reactor에서 `Sinks`를 활용하여 `Signal`을 전송하는 방식은 크게 다음과 같이 구분할 수 있다.
  1. `Sinks.One`: `Sinks.one()` 메소드를 활용하여 한 건의 데이터를 전송하는 방법을 정의하는 기능 명세에 해당한다.
  2. `Sinks.Many`: `Sinks.many()` 메소드를 활용하여 여러 건의 데이터를 여러 방법으로 전송하는 기능을 정의한 기능 명세에 해당한다.

## 2023-11-29 Wed
### Sinks.One이란?
```
> Sinks.One은 Mono 방식으로 Subscriber가 데이터를 소비하도록 지원하는, Sinks 클래스 내부에 인터페이스로 정의된 Sinks의 사양으로 이해할 수 있다. 
```
* `Sinks.One`은 하나의 데이터를 프로그래밍적인 방식으로 `Emit`하는 역할 또는 상술한 스펙 역할을 수행한다.
  * 즉, **`Sinks.one()` 메소드를 호출하여 하나의 데이터를 `Emit`하기 위한 기능 명세를 반환**받을 수 있다.
* **`Sinks.One` 사양을 활용할 경우, 아무리 많은 수의 데이터를 `Emit`한다 해도 처음 이외의 데이터들은 모두 버려진다**.
  * 때문에 버려지는 데이터를 처리하기 위한 `FAIL_FAST` 등의 `EmitFailureHandler` 구현체를 활용할 수 있다.
  * 예를 들어, `FAIL_FAST`는 에러가 발생한 경우 재시도하지 않은 대신 즉시 실패 처리하여 빠르게 실패할 수 있도록 동작한다.
  * 이러한 빠른 실패 처리는 교착 상태 등을 미연에 방지하며, 결과적으로는 스레드 안전성을 보장하는 수단이 된다.

## 2023-11-30 Thu
### Sinks.Many란?
* **`Sinks.Many`는 `Sinks.Many`가 아닌 `ManySpec` 인터페이스를 반환**한다.
  * `Sinks.One`의 경우 단순히 하나의 데이터만을 `Emit`하므로 별도의 사양을 정의하는 대신 디폴트 스펙을 활용한다.
  * 반면, `Sinks.Many`의 경우 데이터를 `Emit`할 수 있도록 여러 기능을 정의하는 `ManySpec`을 반환한다.
* `Sinks.Many`에 의해 반환되는 `ManySpec`은 크게 다음과 같은 세 가지 기능을 별도의 스펙으로 정의한다.
  1. `UnicastSpec`: **단 하나의 `Subscriber`에게만 데이터를 `Emit`하는 경우에 사용**할 수 있다.
  2. `MulticastSpec`: **하나 이상의 `Subscriber`에게 데이터를 `Emit`하고자 하는 경우에 사용**할 수 있다.
  3. `MulticastReplaySpec`: **하나 이상의 `Subscriber`에게 데이터를 `Emit`하며, 이미 `Emit`된 데이터를 다시 `Emit`할 수 있게 지원**한다.
* 이 때, 상술한 **각자의 스펙은 모두 스펙 유형의 인스턴스를 반환하고 최종적으로 스펙에 정의된 기능을 사용**한다.
  * 예를 들어, `Sinks.many().unicast()`를 호출할 경우 `UnicastSpec` 인스턴스가 반환된다.

## 2023-12-01 Fri
### MulticastSpec과 MulticastReplaySpec의 특징
* 또한, **`Sinks`는 `Publisher` 역할을 수행하는 경우 기본적으로 `Hot Publisher`와 같이 동작**한다.
  * 때문에 `MulticastSpec`을 활용하더라도 구독 시점 이전에 이미 `Emit`된 데이터는 전달받을 수 없다.
  * 대신, `MulticastReplaySpec`이 지원하는 `limit()` 또는 `all()` 등의 메소드를 활용하여 이미 `Emit`된 데이터를 다시 전달받을 수 있다.
  * 이렇듯 **`MulticastReplaySpec`은 이미 `Emit`된 데이터 중 임의의 시점으로 되돌린 시점의 데이터부터 `Emit`하는 역할을 수행**한다.

## 2023-12-02 Sat
### Sinks - 결론
* `Sinks`는 `Publisher`와 `Subscriber`의 기능을 모두 갖는, 일종의 향상된 `Processor`와 같은 기능을 제공한다.
* `Sinks`가 데이터를 `Emit`하는 사양을 정의한 것은 크게 `Sinks.One`과 `ManySpec`로 구분할 수 있다.
* `Sinks.Many`의 `MulticastReplaySpec`은 이미 `Emit`된 데이터 중 임의의 시점으로 되돌린 시점의 데이터부터 `Emit`하는 역할을 수행한다.
  * 반면, `MulticastSpec`은 단지 하나 이상의 `Subscriber`에게 데이터를 `Emit`하는 역할을 수행한다.