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

## 2023-12-19 Tue
### Reactor 디버깅의 어려움
* 일반적인 동기적 프로그래밍 방식의 경우, 예외가 발생했을 때 스택 트레이스를 활용하거나 체크포인트를 사용하여 쉽게 디버그를 진행할 수 있다.
* 반면, Reactor는 대부분의 작업이 비동기적으로 실행되며 선언형 프로그래밍 방식으로 코드를 작성하므로 디버깅이 쉽지 않다.
  * 이로 인해 Reactor는 디버그 과정을 지원할 수 있는 여러 방법을 제공한다.

### 디버그 모드 활용하기
* Reactor의 `Sequence`를 디버그하기 위해서는 `Hooks.onOperatorDebug()`와 같은 코드를 작성하여 Reactor의 디버그 모드를 활성화할 수 있다.
  * 해당 모드를 활성화할 경우, 복잡한 `Operator` 체인 상에서 예외가 발생했을 경우 발생 지점을 명확히 확인할 수 있다.
* 그러나 **해당 방식은 애플리케이션 내부의 모든 `Operator`로부터 스택트레이스를 캡쳐하는 등, 성능 상 단점이 적지 않다**.
  * 이는 예외가 발생할 경우, 캡쳐한 정보를 기반으로 예외가 발생한 어셈블리 스택트레이스를 원본 스택트레이스 중간에 삽입하기 때문이다.
  * 따라서 **단순히 예외의 원인만을 추적하기 위해 처음부터 디버그 모드를 활성화하는 것은 권장되지 않는 방식**임에 유의해야 한다.
* 때문에 운영 환경에서 디버깅하고자 하는 경우, 사용 중인 환경에 맞는 별도의 Java 에이전트를 고려하는 것이 바람직하다.
  * 예를 들어, `WebFlux` 기반의 애플리케이션의 경우 `ReactorDebugAgent`를 고려할 수 있다.
* 덧붙여 `Assembly`란 Reactor에서 자주 사용되는 용어로서, 임의의 `Operator`가 반환하는 새로운 `Mono` 또는 `Flux`가 선언된 지점을 지칭한다.
  * 이 때, 예외가 발생한 `Operator`의 스택트레이스를 캡쳐한 `Assembly` 정보를 `Traceback`이라는 용어로 지칭한다.
* 반면, 사용 중인 IDE가 IntelliJ인 경우 개발 환경 차원에서 디버그 모드를 활성화하는 방법 역시 제공된다.

## 2023-12-20 Wed
### checkpoint() Operator 활용하기
* 디버그 모드가 애플리케이션의 모든 `Operator`로부터 발생한 스택트레이스를 캡처한다면, 해당 연산자는 임의의 `Operator` 체인 내에서만 캡처를 시도한다.
* 이 때, `checkpoint() Operator`를 활용하는 방식은 크게 다음과 같이 분류할 수 있다.
  1. `Traceback` 출력하기: 에러가 발생한 `Assembly` 지점 또는 에러가 전파된 `Assembly` 지점의 `Traceback`을 추가한다.
  2. `Traceback` 출력 없이 식별자를 포함한 설명을 출력하여 에러 발생 지점을 에상하기: 이 경우, `checkpoint(description)` 형태로 사용해야 한다.
  3. `Traceback`과 `Description`을 모두 출력하기

## 2023-12-21 Thu
### log() Operator 활용하기
* 해당 연산자는 Reactor `Sequence` 동작을 로그 형태로 출력하며, 이렇게 출력된 로그를 기반으로 디버그를 시도할 수 있다.
  * 이 때, **해당 연산자는 `Subscriber`에 전달되는 결과 외에도 `onSubsribe()`나 `onNext()`와 같은 `Signal`을 함께 출력**한다.
  * **`checkpoint() Operator`가 특정 `Operator` 체인 내의 스택트레이스를 캡쳐한다면, 해당 연산자는 추가된 지점의 `Signal`을 출력**한다.
  * 또한, 연산자의 인자에 `log(식별자, 로그레벨)` 형태로 전달하여 적절한 로그 레벨을 설정할 수도 있다.
* 해당 연산자는 `Operator` 체인 상에서 사용되는 개수에 대한 제한이 없으므로 에러가 발생한 지점에 점진적으로 접근해나갈 수 있다.

## 2023-12-22 Fri
### Reactor 테스트 모듈
```
> Reactor의 가장 흔한 테스트 방식은 Flux나 Mono를 Sequence로 정의한 후, 구독 시점에 Operator 체인이 의도대로 동작하는지 검증하는 것이다.
```
* Reactor의 경우, 비동기적인 코드를 쉽게 테스트할 수 있도록 `reactor-test` 모듈을 제공한다.
  * 이는 그래들 등을 통해 `testImplementation("io.projectreator:reactor-test")`와 같은 형태로 사용할 수 있다.
* `Operator` 체인을 테스트할 경우, `Signal`이나 데이터의 `Emit` 여부 등을 단계적으로 테스트하는 것이 일반적이다.
  * 이 때, **Reactor는 이러한 `Operator` 체인의 다양한 동작을 쉽게 테스트할 수 있도록 `StepVerifier API`를 제공**한다.

## 2023-12-23 Sat
### StepVerifier API 사용하기
```
> Reactor의 테스트는 verifyXXX() API를 통해 테스트 대상 Operator 체인을 구독한 후 expectXXX() API로 Emit된 데이터를 검증하는 과정이다.
```
* 해당 API의 경우, `Operator` 체인을 테스트하기 위해 다음과 같은 코드를 작성한다.
  1. `create()`: 테스트 대상이 될 `Sequence`를 생성한다.
  2. `expectNext()`: `Emit`된 데이터를 검증한다.
  3. `expectComplete()`: `onComplete Signal`을 검증한다.
  4. `verify()`: 상술한 테스트 스펙을 기반으로 실제 검증을 진행한다.
```java
public class Test {
    @Test
    public void reactorTest() {
        StepVerifier.create(
                Mono.just("테스트하기")
        )    
            .expectNext("테스트하기")
            .expectComplete()
            .verify();
    }
}
```
* 이렇듯 `StepVerifier API`를 활용한 테스트 방식은 `Sequence`를 생성한 후 `expectXXX() API`를 활용하여 `Signal`을 검증하는 식으로 진행된다.
  * 이 때, 전체 `Operator` 체인에 대한 실제 테스트는 `verify()` 연산자에 의해 시작된다.
* 나아가 `StepVerifier API`를 활용할 경우 시간 기반의 테스트는 물론 `Backpressure`와 `Context`에 관련된 내용 역시 테스트할 수 있다.
  * 또는 `recordWith() API`를 활용한 레코드 기반의 테스트를 진행할 수도 있다.

## 2023-12-24 Sun
### TestPublisher를 활용한 테스트
* `TestPublisher`는 Reactor의 테스트 모듈에서 지원하는 테스트 전용 `Publisher`로, 정상 동작 또는 오동작하는 `Publisher`를 가정하도록 지원한다.
  * 이 때, 정상 동작이란 `Emit`되는 데이터의 Null 여부나 `Emit`된 데이터의 개수 등 리액티브 스트림즈 스펙의 위반 여부를 미리 체크하는 것을 지칭한다.
  * 반면, 오동작이란 리액티브 스트림즈 스펙의 위반 여부를 사전에 체크하지 않는 경우이므로 테스트 과정에서 유효하지 않은 데이터를 `Emit`할 수 있다.
  * 때문에 오동작하는 `TestPublsher`를 명시적으로 생성하는 것으로 리액티브 스트림즈 스펙이 위배되는 상황 자체가 발생할 수 있는지 여부를 검증할 수 있다.
* 이러한 `TestPublisher`는 복잡한 로직이 포함된 메소드 또는 조건에 따라 `Signal`이 달라지는 등 특수한 상황을 테스트하는 데에 활용할 수 있다.

## 2023-12-25 Mon
### PublisherProbe를 활용한 테스트
* Reactor의 테스트 모듈이 제공하는 `PublisherProbe`를 활용할 경우, `Sequence`의 실행 경로를 테스트할 수 있다.
  * 때문에 `PublisherProbe`는 주로 **조건부로 `Sequence`가 분기되는 경우, 실행 경로를 추적하여 정상적으로 실행되었는지를 검증하기 위해 사용**된다.
  * 즉, **`PublisherProbe`는 `Publisher`가 `Emit`하는 데이터보다는 `Publisher`의 실행 경로 자체에 집중하는 테스트에 적합**하다.

## 2023-12-26 Tue
### Operator란?
```
> 리액티브 프로그래밍은 Operator에서 시작하고 Operator에서 끝난다.
```
* **Reactor는 수많은 `Operator`를 지원하며, 어떤 상황에 어떤 연산자를 활용할지에 대한 가이드 역시 제공**한다.
  * 예를 들어, 상술한 `just()`부터 시작하여 `map()`, `create()` 따위의 모든 것은 `Operator`에 해당한다.

## 2023-12-27 Wed
### Reactor Sequence 생성 Operator - justOrEmpty()부터 range()까지
* `justOrEmpty()`의 경우, `Emit`될 데이터가 null이더라도 NPE를 던지지 않고 `onComplete() Signal`을 전송한다.
  * 반면, `Emit`될 데이터가 null이 아니라면 해당 데이터를 `Emit`하는 `Mono`를 생성한다.
* `fromIterable()`의 경우, 임의의 이터러블로부터 데이터를 `Emit`하는 `Flux`를 생성한다.
* `fromStream()`의 경우, 임의의 스트림에 포함된 데이터를 `Emit`하는 `Flux`를 생성한다.
* `range()`의 경우, 첫 번째 인자로 전달된 n부터 하나씩 증가되는 수를 두 번째 인자로 전달된 m개만큼 `Emit`하는 `Flux`를 생성한다.
  * 이러한 특징으로 인해, 해당 연산자는 고전적인 for 반복문처럼 특정 횟수만큼 작업을 반복하고자 하는 경우에 유용하다.

## 2023-12-28 Thu
### Reactor Sequence 생성 Operator - defer()부터 create()까지
* `defer()`의 경우, 연산자를 선언한 시점이 아닌 구독하는 시점에 데이터를 `Emit`하는 `Mono` 또는 `Flux`를 생성한다.
  * 해당 연산자의 경우, 데이터의 `Emit`을 지연시키는 것으로 적시에 데이터를 처리하는 식으로 불필요한 작업을 줄일 수 있다.
* `using()`의 경우, 인자로 전달받은 리소스를 `Emit`하는 `Flux`를 생성한다.
* `generate()`의 경우, 프로그래밍 방식으로 `Signal`을 전송하므로 동기적으로 데이터를 하나씩 `Emit`하고자 하는 경우에 유용하다.
* `create()`의 경우, `generate()`와 유사하지만 한 번에 여러 개의 데이터를 비동기적으로 `Emit`할 수 있다.

## 2023-12-29 Fri
### Reactor Sequence 필터링 Operator - filter()부터 next()까지
* `filter()`의 경우, `Upstream`으로부터 `Emit`된 데이터 중 조건에 일치하는 데이터만 `Downstream`으로 전달한다.
  * 즉, 해당 연산자의 인자로는 `Predicate`가 전달된다.
* `skip()`의 경우, `Upstream`으로부터 `Emit`된 데이터 중 인자로 전달된 숫자만큼을 건너 뛴 후 나머지 데이터만을 `Downstream`으로 전달한다.
* 반면, `take()`의 경우 `Upstream`으로부터 `Emit`된 데이터 중 인자로 전달된 숫자만큼의 데이터만 `Downstream`으로 전달한다.
* 이 때, `skip()`과 `take()` 모두 인자로 시간을 지정하여 시간 내에 `Emit`된 데이터만을 `Downstream`으로 건너 뛰거나 전달할 수도 있다.
* `next()`의 경우, `Upstream`으로부터 `Emit`된 데이터 중 첫 번째 데이터만을 `Downstream`으로 전달한다.
  * 이 과정에서 `Upstream`으로부터 `Emit`된 데이터가 비어 있는 경우, `Downstream`에는 `Empty Mono`가 전달된다.

## 2023-12-30 Sat
### Reactor Sequence 변환 Operator - map()부터 merge()까지
* `map()`의 경우, `Upstream`으로부터 `Emit`된 데이터를 임의의 매핑 함수로 데이터를 변환한 후에 `Downstream`으로 전달한다.
* `flatMap()`의 경우, `Upstream`으로부터 `Emit`된 데이터를 `Inner Sequence`에서 평탄화하여 하나의 `Sequence`로 병합한다.
* `concat()`의 경우, 인자로 전달된 `Publisher`들의 `Sequence`를 연결한 후 순차적으로 데이터를 `Emit`한다.
* `merge()`의 경우, 인자로 전달된 `Publisher`들의 `Sequence`를 인터리빙 방식으로 병합한다.
  * 다시 말해, 각각의 `Publisher`로부터 `Emit`되는 데이터들은 시간 순서대로 배치된다.

## 2023-12-31 Sun
### Reactor Sequence 변환 Operator - zip()부터 collectMap()까지
* `zip()`의 경우, 인자로 전달된 `Publisher`들이 `Emit`하는 데이터를 하나씩 결합하여 튜플 형태의 데이터를 `Emit`한다.
* `and()`의 경우, 임의의 `Mono`로부터 발생한 완료 `Signal`과 인자로 전달된 `Publisher`의 완료 `Signal`을 결합한 `Mono<Void>`를 반환한다.
  * 즉, 해당 연산자를 활용할 경우 임의의 `Mono`와 인자로 전달된 `Publisher`의 `Sequence`가 모두 종료되었음을 `Subscriber`에게 전달할 수 있다.
* `collectList()`의 경우, `Flux`가 `Emit`한 데이터를 모아 `List`로 변환한 후 이를 `Emit`하는 `Mono`를 반환한다.
  * 이 때, `Upstream Sequence`가 비어 있는 경우에는 빈 `List`를 `Downstream`으로 `Emit`하게 된다.
* `collectMap()`의 경우, `Flux`로부터 `Emit`된 데이터를 기반으로 키-값 형태의 `Map`을 `Emit`하는 `Mono`를 반환한다.