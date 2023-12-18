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

## 2023-12-16 Sat
### 자주 사용되는 Context API
* `Context`와 관련된 API는 여럿이 존재하지만, 그 중 특히 자주 사용되는 것은 크게 다음과 같다.
  1. `put`: 키-값 형태로 컨텍스트에 값을 쓴다.
  2. `of`: 최대 다섯 개의 데이터를 파라미터로 입력하여 여러 개의 값을 키-값 형태로 컨택스트에 쓴다.
  3. `putAll`: 현재 `Context`와, 파라미터로 전달한 `ContextView`를 병합한다.
  4. `delete`: 키를 인자로 전달 받아 현재 `Context`로부터 대응되는 값을 삭제한다.
  5. `readOnly`: `Context`를 `ContextView`로 변환한다.

## 2023-12-17 Sun
### 자주 사용되는 ContextView API
```
> 컨텍스트에 데이터를 쓰기 위해서는 Context API를 사용하듯, 데이터를 조회하기 위해서는 ContextView API를 사용해야 한다.
```
* `ContextView` 역시 관련된 API는 여럿이 존재하지만, 그 중 특히 자주 사용되는 것은 크게 다음과 같다.
  1. `get`: `ContextView`로부터 키에 대응되는 값을 반환한다.
  2. `getOrEmpty`: `ContextView`로부터 키에 대응되는 값을 `Optional`로 래핑하여 반환한다.
  3. `getOrDefault`: `ContextView`에서 키에 대응되는 값을 조회하되, 존재하지 않는 경우 디폴트 값을 반환한다.
  4. `hasKey`: `ContextView`에 인자로 전달된 키가 존재하는지 확인한다.
  5. `isEmpty`: `Context`가 비어 있는지 여부를 반환한다.
  6. `size`: `Context` 내부의 키-값 개수를 반환한다.
* 상술한 바와 같이, `Context`에 저장된 데이터를 조회하기 위한 `ContextView` API는 컬렉션 API의 `Map`과 사용방법이 크게 다르지 않다.

## 2023-12-18 Mon
### 주의해야 할 Context API의 특징
* **컨텍스트는 기본적으로 구독이 발생할 때마다 하나의 `Context`가 각각의 구독에 연결**된다.
* **컨텍스트는 `Operator` 체인 상에서, 아래에서 위로 전파**된다.
  * 이로 인해 동일한 키로 저장할 경우, 가장 위에 위치한 `contextWrite() Operator`가 저장한 값으로 덮어 씌워진다.
  * 바꿔 말해, 모든 `Operator`가 컨텍스트로부터 데이터를 조회할 수 있도록 하기 위해서는 `contextWrite()` 연산자를 체인의 맨 마지막에 두어야 한다.
* **`Inner Sequence` 내부에서는 외부 컨텍스트의 값을 조회할 수 있으나, 외부에서는 내부 컨텍스트에 저장된 값을 조회할 수 없다**.
* 상술한 특징으로 인해 **컨텍스트는 인증 정보 등 독립성을 갖는 정보를 관리하는 데에 적절**하다.