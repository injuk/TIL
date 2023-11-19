# Reactive
## 2023-11-15 Wed
### Subscriber 개념 바로잡기
* **`Subscriber`는 정확히 말해 데이터를 처리하는 쪽만이 아닌, 모든 `Downstream Publisher`를 포함하는 개념**이다.
  * 이렇듯 `Upstream Publisher`로부터 데이터를 전달받아 처리하는 모든 `Publisher`는 `Downstream Consumer`라고도 지칭한다.
* **리액티브 프로그래밍의 경우, 원본 Flux에서 `Emit`된 모든 데이터는 각 `Operator`를 거칠 때마다 다시 Flux 또는 Mono로 반환**된다.
  * 각 **`Operator`가 반환하는 반환 값이 또 다른 `Publisher`인 Flux 또는 Mono이므로, `Subscriber`는 일부 `Publisher`를 포함**할 수 있다.

## 2023-11-16 Thu
### Backpressure란?
```
> Backpressure는 Publisher가 끊임없이 Emit하는 수많은 데이터를 적절히 제어하여 데이터 처리에 과부하를 주지 않는 것을 의미한다. 
```
* 예를 들어 `Publisher`가 데이터를 `Emit`하는 속도가 `Subscriber`의 처리 속도보다 너무 빠른 경우, 작업의 효율성은 떨어질 수 있다.
* **`Backpressure`는 이렇듯 `Publisher`가 끊임없이 `Emit`하는 데이터를 적절히 제어하기 위해 사용되는 개념**이다.
  * 이러한 **배압이 없을 경우 `Publisher`가 계속해서 데이터를 `Emit`하는 것으로 오버플로우가 발생하거나, 최악의 경우에는 시스템 다운**될 수 있다.

## 2023-11-17 Fri
### Backpressure 처리 유형 구분하기
* Reactor의 경우, 크게 다음과 같은 `Backpressure` 유형을 지원한다.
  1. 데이터의 개수를 제어
  2. `Backpressure` 전략 사용

### 데이터의 개수 제어하기
```
> 데이터의 요청 개수를 직접적으로 제어하기 위해서는 BaseSubscriber<T> 클래스를 활용할 수 있다.
```
* **해당 처리 유형은 `Subscriber`가 자신이 적절히 처리할 수 있는 수준의 데이터 개수만을 `Publisher`에게 요청**한다.
  * 다시 말해, `Subscriber`는 자신의 `request()` 메소드를 활용하여 `Publisher`에게 적절한 데이터의 개수를 요청한다.
* `Subscriber`가 데이터의 요청 수를 제어하기 위해서는 `BaseSubscriber<T>` 클래스를 활용할 수 있으며, 이는 다음과 같은 메소드를 제공한다.
  1. hookOnSubscribe: `Subscriber.onSubscribe()` 메소드를 대신하며, 구독 시점에 데이터의 최초 요청 개수를 제어한다.
  2. hookOnNext: `Subscriber.onNext()` 메소드를 대신하며, `Publisher`가 전달한 데이터를 처리한 후에 새로운 데이터의 요청 개수를 제어한다.
* 이 때, **hookOnSubscribe와 hookOnNext 메소드는 모두 내부적으로 `request()` 메소드를 활용하여 데이터의 요청 개수를 제어**한다.
* 또한, **해당 유형의 배압을 활용하기 위해서는 `Publisher.subscribe()` 메소드에 람다 표현식 대신 `BaseSubscriber<T>` 구현체를 전달**한다. 

## 2023-11-18 Sat
### Backpressure 전략 구분하기
* Reactor의 경우, 크게 다음과 같은 `Backpressure` 전략을 지원한다.
  1. IGNORE: 배압을 적용하지 않는 전략에 해당한다.
  2. ERROR: `Downstream`으로 전달하기 위한 데이터가 버퍼에 가득찰 경우, 예외를 던지는 전략에 해당한다.
  3. DROP: `Downstream`으로 전달하기 위한 데이터가 버퍼에 가득찰 경우, 버퍼 외부에서 대기 중인 데이터를 `Emit`된 순으로 버리는 전략에 해당한다.
  4. LATEST: `Downstream`으로 전달하기 위한 데이터가 버퍼에 가득찰 경우, 버퍼 외부에 있고 최근에 `Emit`된 데이터부터 버퍼에 채우는 전략에 해당한다.
  5. BUFFER: `Downstream`으로 전달하기 위한 데이터가 버퍼에 가득찰 경우, 버퍼 내부에 위치한 데이터부터 버리는 전략에 해당한다.

## 2023-11-19 Sun
### IGNORE 전략이란?
* 해당 전략은 말그대로 배압을 적용하지 않으며, `Downstream`에서 배압 요청이 무시되므로 예외가 발생하기 쉽다.

### ERROR 전략이란?
* 해당 전략은 `Downstream`의 데이터 처리 속도가 느려져 `Upstream`의 `Emit` 속도를 따라가지 못하는 경우, `IllegalStateException`을 던진다.
  * 이 때, 코드 상에서는 `onBackpressureError() Operator` 를 활용하여 해당 전략을 적용할 수 있다.
  * 이 경우, `Publisher`는 `Error Signal`을 `Subscriber`에게 전송한 후 삭제한 데이터를 폐기한다.