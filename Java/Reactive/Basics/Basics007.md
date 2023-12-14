# Reactive
## 2023-12-13 Wed
### Context란?
```
> Reactor의 컨텍스트는 Operator 체인 상에서 전파되는 키-값 형태의 저장소를 지칭한다.
```
* **컨텍스트라는 용어는 프로그래밍 분야에서도 자주 사용되며, 어떠한 상황에서 문제를 해결하기 위해 필요한 정보 전반을 지칭**한다.
* **Reactor의 컨텍스트는 `Operator` 등의 구성요소 간에 전파되는 키-값 저장소에 해당**한다.
  * 덕분에 이를 통해 `Operator` 체인 상의 각 연산자는 해당하는 컨텍스트의 정보를 동일하게 활용할 수 있게 된다.
* **Reactor의 컨텍스트는 `Subscriber`와 매핑되며, 이로 인해 구독이 발생할 때마다 해당 구독과 연결되는 하나의 컨텍스트가 생성**된다.

## 2023-12-14 Thu
### Context에 쓰기
* Reactor의 컨텍스트에는 `contextWrite() Operator`를 활용하여 데이터를 쓸 수 있으며, 해당 연산자는 인자로 람다 표현식을 전달 받아 동작한다.
  * 이 경우, 해당 람다 표현식은 `Context`를 받아 `Context`를 반환하는 `Function` 타입의 함수형 인터페이스에 해당한다.
* 반면, 코드상에서 실제로 컨텍스트에 데이터를 쓰는 동작 자체는 `Context` API 중 `put()` 메소드를 활용한다.
  * 예를 들어, `Operator` 체인 상에서는 `contextWrite(ctx -> ctx.put("name", ""))`와 같은 형태의 코드로 작성된다.
* 또한, **Reactor는`context.put()` API로 컨텍스트에 데이터를 쓸 경우 매번 불변 객체를 다음 `contextWrite() Operator`에 전달**한다.
  * 이렇듯 **내부적으로는 불변 객체를 활용하는 Reactor의 특징으로 인해 컨텍스트에 데이터를 쓰는 동작은 스레드 안전성이 보장**된다.

## 2023-12-15 Fri
### Context로부터 읽기
* Reactor의 경우, 컨텍스트로부터 데이터를 조회하는 방식은 크게 다음과 같이 구분된다.
  1. 원본 데이터 소스 레벨로부터 읽기: `deferContextual() Operator`를 활용하여 조회한다.
  2. `Operator` 체인의 중간에서 읽기:
* 이 때, **`deferContextual() Operator`는 컨텍스트에 저장된 데이터와 원본 데이터 소스에 대한 처리를 지연**시킨다.
  * 이 경우, 상술한 `contextWrite()`와 달리 `deferContextual() Operator`는 `ContextView` 타입 객체를 인자로 받는 람다 표현식을 사용한다.
  * 이렇듯 Reactor는 **컨텍스트에 데이터를 쓸 때에는 `Context` 타입의 객체를 활용하되, 조회할 때에는 `ContextView` 타입 객체를 활용**한다.
* 데이터를 조회하는 경우 역시, 코드상에서는 `ContextView`가 제공하는 API 중 `get()` 메소드를 활용한다.
* 반면, **`Operator` 체인의 중간에서 컨텍스트의 데이터를 조회하는 경우에는 `transformDeferredContextual() Operator`를 활용**한다.
* 또한, `subscribeOn()`이나 `publishOn() Operator`와 혼용하는 경우 서로 다른 스레드들이 컨텍스트에 저장된 데이터에 접근할 수 있다.
  * 예를 들어, `Operator` 체인 상에서 `publishOn() Operator` 등을 활용하는 것으로 매번 다른 스레드가 컨텍스트의 데이터를 조회하도록 할 수 있다.