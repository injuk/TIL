# Reactive
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

## 2024-01-01 Mon
### Reactor Sequence 부수 효과 Operator - doOnXXX()
* Reactor는 `Upstream Publisher`로부터 `Emit`된 데이터를 수정하지 않는, 오로지 부수 효과만을 위한 `Operator`들이 지원된다.
  * 또한, 이러한 연산자들은 일반적으로 `doOnXXX()` 형태로 명명되며 함수형 인터페이스를 인자로 전달 받아 아무 것도 반환하지 않는 식으로 동작한다.
  * 이러한 이유에서 `doOnXXX() Operator`들은 `Upstream Publisher`로부터 `Emit`된 데이터를 기반으로 내부 동작을 확인하는 등 디버그에 사용된다.

## 2024-01-02 Tue
### Reactor Sequence 에러 처리 Operator - error()부터 retry()까지
* `error()`의 경우, 인자로 전달된 예외로 종료하는 `Flux`를 생성한다.
  * 이는 마치 `throw` 키워드를 활용하여 예외를 의도적으로 던지는 것과 유사한 역할을 수행하며, 일반적으로 `checked exception`을 던지는 데에 활용된다.
* `onErrorReturn()`의 경우, 예외 이벤트가 발생한 경우 이를 `Downstream`으로 전달하는 대신 인자로 전달된 기본값을 `Emit`한다.
* `onErrorResume()`의 경우, 예외 이벤트가 발생한 경우 이를 `Downstream`으로 전달하는 대신 인자로 전달된 `Publisher`를 반환하는 식으로 동작한다.
* `onErrorContinue()`의 경우, 예외 이벤트가 발생한 경우 해당 데이터를 제거하여 `Upstream`으로부터 후속 데이터를 `Emit`하도록 지원한다.
* `retry()`의 경우, `Publisher`가 데이터를 `Emit`하는 과정에서 예외가 발생한 경우 인자로 전달된 횟수만큼 원본 `Flux`를 다시 구독한다.

## 2024-01-03 Wed
### Reactor Sequence 기타 Operator - elapsed() 부터 groupBy()까지
* `elapsed()`는 Reactor `Sequence`의 동작 시간을 측정하기 위해 사용되며, 측정된 시간을 `Downstream`에 `Emit`한다.
* `window()`는 `Upstream`으로부터 `Emit`된 첫 번째 데이터부터 인자로 전달된 수 만큼의 데이터를 포함하는 새로운 `Flux`를 생성한다.
* `buffer()`는 `Upstream`으로부터 `Emit`된 첫 번째 데이터부터 인자로 전달된 수 만큼의 데이터를 List 버퍼로 한 번에 `Emit`한다.
  * 반면, `bufferTimeout()`은 인자로 전달된 수 만큼 또는 `maxTime` 이내에 `Emit`된 모든 데이터를 List 버퍼로 한 번에 `Emit`한다.
* `groupBy()`는 `Emit`된 데이터를 `keyMapper`로 생성된 `key`를 기준으로 그룹화한 `Flux`를 반환한다.

## 2024-01-04 Thu
### Reactor Sequence 기타 Operator - publish() 부터 refCount()까지
* `publish()`는 구독이 발생하더라도 구독 시점에 즉시 데이터를 `Emit`하는 대신 `connect()`를 호출하는 시점에서야 데이터를 `Emit`한다.
  * 반면, `autoConnect()`는 인자로 지정한 숫자만큼의 구독이 발생하는 시점에 `Upstream`에 자동으로 연결되므로 `connect()`를 사용할 필요가 없다.
  * `refCount()` 역시 인자만큼의 구독 발생 시 `Upstream`에 연결되나, 구독이 모두 취소되거나 `Upstream`의 `Emit`이 종료될 때 연결을 해제한다.

## 2024-01-05 Fri
### Spring WebFlux란?
```
> Spring 진영의 WebFlux는 리액티브 웹 애플리케이션 구현을 위해 지원되는 리액티브 웹 프레임워크를 의미한다.
> Spring WebFlux는 요청부터 응답까지, Reactor의 두 타입인 Mono나 Flux의 Operator 체인으로 구성된 일종의 긴 Sequence라고 이해할 수 있다.
```
* 오랜 기간 사용되어 온 Spring MVC의 경우, 내부적으로는 서블릿 기반의 `Blocking I/O` 방식으로 구현되어 있다.
  * 이로 인해 하나의 요청을 처리하기 위해서는 하나의 스레드를 사용하고, 각 스레드의 작업이 종료될 때까지 해당 스레드는 블로킹된다.
* **Spring MVC는 오랜 기간 사용되어 온 검증된 방식이나, 대량의 요청 트래픽을 감당하지 못하는 치명적인 단점 역시 존재**한다.
* 이로 인해 **적은 수의 스레드로 많은 요청을 처리할 수 있는 비동기 처리 방식에 대한 필요성이 대두되었으며, `WebFlux`는 이러한 배경 속에서 탄생**했다.

## 2024-01-06 Sat
### Spring WebFlux의 기반 기술
* Spring MVC와 Spring WebFlux는 각각 기반이 되는 기술 스택을 가지며, 크게 다음과 같은 차이를 갖는다.
* Spring MVC는 서블릿 기반의 프레임워크이므로, 톰캣과 같은 서블릿 컨테이너 상에서 `Blocking I/O` 방식으로 동작한다.
  * 반면, **`WebFlux`의 경우 `Non-Blocking I/O` 방식으로 동작하는 Netty 등의 서버 엔진 상에서 동작**한다.
* Spring MVC는 서블릿 기반의 프레임워크이므로, 자연스레 서블릿 API를 사용한다.
  * 반면, `WebFlux`의 경우 서버 엔진 상에서 지원하는 리액티브 스트림즈 어댑터를 활용하여 리액티브 스트림즈를 지원한다.
* Spring MVC는 표준 서블릿 필터를 사용하는 Spring Security가 서블릿 컨테이너와 통합된다.
  * 반면, **`WebFlux`의 경우 `WebFilter`를 통해 Spring Security와 통합**된다.
* Spring MVC는 내부적으로 `Blocking I/O` 방식을 활용하는 Spring Data JDBC 또는 Spring Data JPA 등의 데이터 액세스 기술을 활용한다.
  * 반면, **`WebFlux`의 경우 데이터 액세스 계층까지 완벽하게 `Non-Blocking I/O`를 지원하는 Spring Data `R2DBC`를 활용**한다.

## 2024-01-07 Sun
### WebFlux의 요청 흐름
```
> Spring MVC와 마찬가지로, WebFlux 역시 정해진 흐름에 따라 각 컴포넌트가 클라이언트의 요청을 처리하게 된다.
```
* 클라이언트로부터의 요청이 최초로 발생했을 때, 요청은 Netty 등의 서버 엔진을 거쳐 `HttpHandler`에 전달된다.
  * 이 때, **`HttpHandler`는 Netty 등의 여러 서버 엔진에서 지원하는 서버 API를 사용할 수 있도록 서버 API를 추상화하는 역할을 담당**한다.
* 이를 통해 각 서버 엔진마다 주어지는 `ServerWebExchange`가 생성되고, 이는 `WebFilter` 체인에 전달된다.
  * 이 때, `ServerWebExchange`는 각 서버마다 주어지는 `ServerHttpRequest`와 `ServerHttpResponse`를 포함한다.
  * 이렇듯 **`ServerWebExchange`는 `WebFilter` 체인으로부터 전처리 과정을 거친 후 `WebHandler` 인터페이스의 구현체에게 전달**된다.
* **`DispatcherHandler`는 일반적인 `WebHandler`의 구현체이며, 이는 Spring MVC의 `DispatcherServlet`과 유사한 역할**을 담당한다.
  * 이 때, **`DispatcherHandler`에서는 `HandlerMapping List`를 원본 `Flux`의 소스로 전달**받는다.
* 이후 **`ServerWebExchange`를 처리할 핸들러를 조회한 후, 조회된 핸들러의 호출은 `HandlerAdapter`에 위임**된다.
  * 이윽고 `HandlerAdapter`는 `ServerWebExchange`를 처리하기 위한 핸들러를 호출한다.
  * 예를 들어, **해당 요청은 `Controller` 또는 `HandlerFunction` 형태의 핸들러에서 처리된 후에 응답 값을 반환하는 식으로 처리**된다.
* 핸들러로부터 **반환된 응답 값을 처리하기 위한 `HandlerResultHandler`가 조회되고, 조회된 객체는 응답 값을 처리한 후에 `response`로 반환**한다.
  * 반면, **엄밀히 말해 핸들러에서 처리되어 반환되는 응답 값은 응답 값을 포함하는 `Flux` 또는 `Mono Sequence`를 의미**한다.
  * 때문에 **반환된 Reactor `Sequence`는 반환된 즉시 어떠한 작업을 수행하는 식으로 동작하지는 않는 점에 유의**해야 한다.

## 2024-01-08 Mon
### WebFlux 핵심 컴포넌트 - HttpHandler
* `HttpHandler`는 서로 다른 유형의 HTTP 서버 API를 활용하여 요청과 응답을 처리하기 위해 추상화된 `handle()` 메소드만을 제공한다.
  * 이 때, 해당 메소드의 시그니쳐는 `Mono<Void> handle(ServerHttpRequest request, ServerHttpResponse response);`와 같다.
* 예를 들어, `HttpWebHandlerAdapter` 구현체는 전달받은 요청과 응답을 활용하여 `ServerWebExchange`를 생성한 후 `WebHandler`를 호출한다.

## 2024-01-09 Tue
### WebFlux 핵심 컴포넌트 - WebFilter
```
> WebFilter는 Spring MVC의 서블릿 필터와 유사하게 핸들러가 요청을 처리하기 전에 전처리 작업을 수행하는 역할을 담당한다.
> 때문에 WebFilter를 통해 보안 또는 세션 타임아웃 처리 등의 애플리케이션 전반에 공통적으로 적용될 전처리 작업을 처리할 수 있다.
```
* `WebFilter` 역시 `filter()` 메소드만을 정의하며, 인자로 `ServerWebExchange`와 `WebFilterChain`을 전달받는다.
  * 이 때, 전달받은 `WebFilterChain`을 활용하여 필터 체인을 형성하며, 원하는 만큼의 `WebFilter`를 추가할 수 있다.
* 상술한 바와 같이, **Spring `WebFlux`는 일종의 긴 Reactor `Sequence`이므로 `WebFilter` 역시 `Sequence`의 일부로 이해**할 수 있다.

## 2024-01-10 Wed
### WebFlux 핵심 컴포넌트 - HandlerFilterFunction
```
> HandlerFilterFunction은 함수형 기반의 요청 핸들러에 연결할 수 있는 필터를 의미한다.
```
* `HandlerFilterFunction`은 함수형 기반의 요청 핸들러에 대해 적용할 수 있는 필터이며, `Mono`를 반환하는 `filter` 메소드를 제공한다.
  * 이 때, `Mono<R> filter(ServerRequest request, HandlerFunction<T> next);` 형태로 정의되어 인자로 전달된 `next`에 연결된다.
* **`WebFilter`의 구현체는 Spring의 빈으로 등록되는 반면, `HandlerFilterFunction` 구현체는 함수 형태로 사용되므로 빈에 등록되지 않는다**.
  * 나아가 `WebFilter`는 정의된 모든 핸들러에 대해 공통으로 동작하는 특징이 있기에 어노테이션 기반 요청 핸들러와 함수형 기반 요청 핸들러 모두에 적용된다.
  * 그러나 `HandlerFilterFunction`는 함수형 기반의 핸들러에 대해서만 동작하므로, 이러한 특징에 따라 두 필터를 적시에 활용할 수 있어야 한다.

## 2024-01-11 Thu
### WebFlux 핵심 컴포넌트 - DispatcherHandler
```
> DispatcherHandler는 Spring 빈으로 등록되도록 설계되었으며, ApplicationContext로부터 요청 처리를 위한 위임 컴포넌트를 찾아 활용한다.
```
* `DispatcherHandler`는 `WebHandler`의 구현체로서 마치 Spring MVC의 `DispatcherServlet`처럼 먼저 중앙에서 요청을 수신하는 역할을 담당한다.
  * 이렇듯 **우선 요청을 수신한 이후에는 다른 컴포넌트에 적절한 요청 처리를 위임하는 식으로 동작**한다.
  * `DispatcherHandler`는 요청 처리를 위해 위임 컴포넌트인 `HandlerMapping`과 `HandlerAdapter`, `HandlerResultHandler`를 활용한다.
* 이 때, `DispatcherHandler`는 크게 다음과 같은 메소드들을 제공한다.
  1. `initStrategies`: Spring의 `BeanFactoryUtils`를 활용하여 `ApplicationContext`로부터 필요한 빈을 검색한 후, 필요한 객체를 생성한다.
  2. `handle`: 내부적으로는 필요한 핸들러를 검색한 후, 핸들러 호출과 응답에 대한 처리를 각각 적절한 객체에 위임한다.

## 2024-01-12 Fri
### WebFlux 핵심 컴포넌트 - HandlerMapping
```
> HandlerMapping은 Spring MVC와 마찬가지로 요청과 핸들러 객체에 대한 매핑을 정의하는 인터페이스를 의미한다.
```
* 해당 인터페이스는 `RequestMappingHandlerMapping` 또는 `RouterFunctionMapping` 등의 구현체를 갖는다.
* 이 때, 제공되는 `getHandler` 메소드는 인자로 전달된 `ServerWebExchange`에 대응되는 핸들러 객체를 반환하는 방식으로 동작한다.

## 2024-01-13 Sat
### WebFlux 핵심 컴포넌트 - HandlerAdapter
* 해당 인터페이스는 `HandlerMapping`을 통해 반환된 핸들러를 직접 호출하며, 결과로 `Mono<HandlerResult>`를 반환하는 `handle` 메소드를 제공한다.
  * 즉, `handle` 메소드는 내부적으로 인자로 전달된 핸들러 객체의 핸들러 메소드를 호출하는 식으로 동작하게 된다.
* 또한, 해당 인터페이스는 인자로 전달된 핸들러 객체를 지원할 수 있는지 검증하는 `supports` 메소드를 함께 제공한다.

## 2024-01-14 Sun
### WebFlux와 Non-Blocking 프로세스
```
> Blocking I/O를 사용하는 Spring MVC는 요청을 처리하는 스레드가 차단될 수 있으므로, 기본적으로 대용량의 스레드 풀을 활용한다.
> 반면, Non-Blocking I/O를 사용하는 Spring WebFlux는 스레드가 차단되지 않으므로 적은 수의 고정된 스레드 풀을 사용하여 더 많은 요청을 처리한다. 
```
* **`WebFlux`는 내부적으로 이벤트 루프를 활용하여 요청을 처리하므로, 결과적으로 스레드의 차단 없이 더 많은 요청을 처리할 수 있게 된다**.
* 이러한 이벤트 루프 방식을 활용하는 `Non-Blocking Process`는 일반적으로 다음과 같은 흐름으로 동작한다.
  1. 클라이언트로부터 전달된 요청은 `RequestHandler`가 전달받는다.
  2. 전달받은 요청은 이벤트 루프에 푸시된다.
  3. **이벤트 루프는 NW, DB 등 비용이 많이 드는 IO 작업에 대한 콜백을 등록**한다.
  4. 상술한 **비용이 많이 드는 IO 작업이 완료된 경우, 이벤트 루프에 완료 이벤트가 푸시**된다.
  5. 등록된 콜백을 호출하여 처리 결과를 전달한다.
* 나아가 **이벤트 루프는 단일 스레드에서 계속해서 실행되며, 비용이 많이 드는 모든 IO작업은 이벤트로 처리**된다.
  * 또한, **이벤트가 발생할 경우 해당 이벤트에 대한 콜백을 등록함과 동시에 다음 이벤트 처리를 진행**할 수 있게 된다.
  * 이렇듯 `WebFlux`는 이러한 이벤트 루프 방식을 활용하는 것으로 더 적은 수의 스레드로 많은 요청을 `Non-Blocking Process`로 처리할 수 있게 된다.

## 2024-01-15 Mon
### WebFlux와 스레드 모델
* Spring MVC의 경우, 서블릿 컨테이너의 스레드 풀의 스레드가 사용자의 요청을 처리하고 작업을 완료한 이후에 스레드 풀에 반납되는 스레드 모델을 사용한다.
  * 즉, 클라이언트의 요청당 하나의 스레드를 사용하므로 결과적으로는 규모가 커질수록 많은 수의 스레드가 필요할 수 밖에 없다.
* **`WebFlux`의 경우 `Non-Blocking I/O`를 지원하는 Netty 등의 서버 엔진이 적은 수의 고정된 크기의 스레드를 생성하여 대량의 요청을 처리**한다.
  * 더 자세히는 일반적으로 CPU 개수만큼의 스레드를 생성하며, 예를 들어 Netty의 경우 최소 4개의 워커 스레드를 생성한다.

## 2024-01-16 Tue
### WebFlux의 한계와 보완
* 상술한 바와 같이, `WebFlux`를 활용할 경우 Netty와 같은 서버 엔진이 제공하는 스레드 풀을 기반으로 많은 수의 요청을 효과적으로 처리할 수 있다.
* 그러나 **요청을 처리하는 과정에서 CPU 집약적인 작업 등, 스레드가 차단되는 지점이 존재한다면 오히려 성능이 떨어지는 명확한 한계 역시 존재**한다.
  * **Reactor의 경우, 이러한 성능 저하에 대해 요청을 처리할 때 서버 엔진의 스레드 풀이 아닌 별도의 스레드 풀을 사용하는 `Scheduler`를 지원**한다.

## 2024-01-17 Wed
### Spring MVC 기반의 컨트롤러
* 임의의 리소스에 대한 관리 기능을 제공하는 컨트롤러가 Spring MVC를 사용하는 경우, 크게 다음과 같은 특징을 갖게 된다.
  1. `@RequestBody` 어노테이션을 명시하는 것으로 클라이언트의 요청 본문을 전달받는다.
  2. `ResponseEntity` 타입의 인스턴스를 활용하여 응답을 클라이언트에게 반환한다.

### Spring WebFlux 기반의 컨트롤러
* 상술한 Spring MVC 방식과 달리, `WebFlux`는 크게 다음과 같은 두 가지 프로그래밍 방식을 제공한다.
  1. 어노테이션 기반 모델
  2. 함수형 기반 모델
* 이 중 어노테이션 기반의 프로그래밍 모델은 Spring MVC와 유사한 형태를 띄는, 상대적으로 익숙한 방식에 해당한다.
  * 또한, **어노테이션 기반의 `WebFlux` 컨트롤러는 `ResponseEntity` 대신 `Mono<T>`를 반환한다는 점 외에는 변경점이 거의 필요 없다**.
  * 이렇듯 `WebFlux`의 어노테이션 기반 방식을 활용할 경우, Spring MVC 방식의 컨트롤러를 쉽게 `Non-Blocking I/O` 방식의 핸들러로 수정할 수 있다.

## 2024-01-18 Thu
### 객체 매핑 과정에서 발생할 수 있는 약간의 블로킹
```
> Spring WebFlux는 Spring MVC에서 지원하는 @RequestBody나 ResponseEntity를 그대로 지원한다.
> 반면, Spring MVC와는 달리 WebFlux는 리액티브 타입을 활용하므로 데이터를 비동기적으로 ResponseBody에 렌더링하는 방식으로 동작한다.
```
* **`@RequestBody` 로 전달된 DTO나, 서비스에서 반환된 엔티티를 응답 DTO로 매핑하는 과정은 약간이지만 스레드를 차단하는 블로킹 요소에 해당**한다.
  * 이는 무시할 수 있을만한 작업이지만, 그럼에도 `WebFlux`를 사용하는 이상 블로킹 요소는 최대한 제거하는 것이 바람직하다.
* 예를 들어, 클라이언트의 요청을 그대로 `@RequestBody`에 매핑하는 방식 대신 `@RequestBody Mono<T> body`와 같이 매핑할 수 있다.
  * 이렇듯 **`WebFlux`는 인자로 `Reactive` 타입을 지원하므로, 모든 매핑 과정까지도 `Mono`나 `Flux` 내에서 처리되도록 구현하는 것이 권장**된다.

## 2024-01-19 Fri
### 함수형 엔드포인트란?
* `WebFlux`는 익숙한 방식인 어노테이션 기반 방식과 함께 함수형 엔드포인트 기반 방식을 제공한다.
  * **클라이언트의 요청을 처리하기 위해 어노테이션 매핑 기법을 사용하는 방식과 달리, 함수형 엔드포인트는 모든 과정을 하나의 함수 체인으로 처리**한다.
* **`WebFlux`의 경우, 함수형 엔드포인트는 클라이언트의 요청을 처리하기 위해 다음과 같이 `HandlerFunction`이라는 함수형 핸들러를 사용**한다.
```Java
@FunctionalInterface
public interface HandlerFunction<T extends ServerResponse> {
    Mono<T> handle(ServerRequest request);
}
```
* 상술한 바와 같이, `HandlerFunction`은 함수형 인터페이스이므로 `handle()` 메소드 하나만을 제공하며 다음과 같은 타입을 활용한다.
  1. `ServerRequest`: **`HandlerFunction`에 의해 처리되는 HTTP 요청을 표현하며, 헤더나 HTTP 메소드 등의 정보를 포함**한다.
  2. `ServerResponse`: **`HandlerFunction` 또는 `HandlerFilterFunction`에서 반환되는 HTTP 응답을 표현**한다.
* 특히, `ServerRequest`의 경우 HTTP 요청 본문에 접근하기 위한 `body()`나 `bodyToMono()` 및 `bodyToFlux()` 등의 메소드를 제공한다.

## 2024-01-20 Sat
### RouterFunction이란?
* **`RouterFunction`은 클라이언트의 요청을 적절한 `HandlerFunction`으로 라우팅하며, `@RequestMapping`과 동일한 역할을 수행**한다.
  * 이 때, `RouterFunction` 역시 단일 메소드를 제공하는 함수형 인터페이스이며 `ServerRequest`를 받아 적절한 `HandlerFunction`을 반환한다.
* `RouterFunction`을 사용하는 경우, 일반적으로는 `@Configuration`을 활용하여 `RouterFunction`을 빈으로 등록하여 활용하게 된다.
  * 이 경우, **빈으로 등록될 해당 `RouterFunction`은 `@Component`로 등록된 요청 핸들러를 주입받아 동작**하게 된다.

## 2024-01-21 Sun
### 함수형 엔드포인트 방식에서의 요청 유효성 검증
* 함수형 엔드포인트를 활용할 경우, 크게 다음과 같은 방식 중 하나로 요청 본문에 대한 유효성을 검증할 수 있다.
  1. Custom Validator: Spring의 Validator 인터페이스를 구현한 Custom Validator를 구현하여 유효성 검증을 처리한다.
  2. 표준 Bean Validation: Spring MVC에서 사용하던 표준 Bean Validation을 활용하여 유효성 검증을 처리한다.
* 이 중, **Custom Validator 방식은 비즈니스 로직과 유효성 검증 코드가 뒤섞이며 중복이 발생하기 쉬워 바람직하지 않은 방식에 해당**한다.
* 반면, 표준 Bean Validation의 경우 Spring에서 지원하는 Validator 또는 javax에서 지원하는 Validator 인터페이스 중 하나를 활용한다.
  * 다시 말해, **두 인터페이스 중 하나를 활용하여 Validator 구현체를 주입받아 요청 본문를 표현하는 DTO 객체에 대한 유효성 검증을 진행**하게 된다.
  * **두 방식은 단지 어떤 인터페이스를 활용하여 유효성 검증 로직을 구현하느냐에 대한 차이만 존재하므로, 둘 중 익숙한 방식을 선택해도 무방**하다.
  * 실제로는 두 방식 모두 내부적으로는 유효성 검증을 처리하기 위해 Validator 구현체인 Hibernate Validator를 사용한다.

## 2024-01-22 Mon
### R2DBC란?
```
> R2DBC는 관계형 DB에 리액티브 프로그래밍 API를 제공하기 위한 스펙이자, SPI에 해당한다.
> 기존에는 몇몇 NoSQL만 리액티브 클라이언트를 제공했으나, R2DBC로 인해 관계형 DB를 사용하는 완전한 Non-Blocking 애플리케이션의 구현이 가능해졌다.
```
* **`JDBC API` 자체가 `Blocking API`이므로, `R2DBC`가 탄생하기 이전에는 완전한 `Non-Blocking` 애플리케이션을 제작하기가 불가능**했다.

## 2024-01-23 Tue
### Spring Data R2DBC란?
* Spring Data `R2DBC`는 `R2DBC` 기반의 리포지토리를 더욱 쉽게 구현할 수 있도록 지원하는, Spring Data Family 프로젝트의 일부이다.
  * 때문에 Spring Data `R2DBC`는 Spring이 추구하는 여러 추상화 기법을 적용하며, 데이터 액세스 계층의 반복적인 코드를 크게 줄일 수 있도록 한다.
  * 또한, **Spring Data `R2DBC`는 JPA 등의 프레임워크가 제공하는 캐싱 등의 특징이 제거되므로 더욱 단순한 사용이 가능**하다.