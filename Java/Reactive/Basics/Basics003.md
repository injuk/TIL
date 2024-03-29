# Reactive
## 2023-10-25 Wed
### 블로킹 IO란?
```
> 운영체제의 관점에서, IO는 기본적으로 시스템이 외부의 입출력 장치들과 데이터를 주고 받는 것을 의미한다.
```
* 웹 애플리케이션은 일반적으로 동적인 데이터를 처리하기 위해 DB IO나 NW IO 등 어떠한 형태로든 IO가 발생할 수 밖에 없다.
* 이 때, **블로킹 IO로 구성된 애플리케이션은 IO 작업을 처리하는 과정에서 요청 스레드가 차단되어 대기하는 상황이 발생**하게 된다.
    * 예를 들어, NW IO를 사용하는 웹 애플리케이션이 사용자의 요청을 처리하기 위해 외부 서버를 호출할 경우, 응답이 반환될 때까지 요청 스레드는 블로킹된다.
    * 이렇듯 **임의의 스레드가 단지 IO 작업을 위해 차단되어 대기하는 현상을 `Blocking I/O`라는 용어로 지칭**한다.

## 2023-10-26 Thu
### 블로킹 IO와 멀티스레딩 기법
* 상술한 블로킹 IO 방식의 문제점을 보완하기 위해서는 멀티스레딩을 도입하여 추가 스레드를 할당하는 것으로 시간을 최대한 효율적으로 활용할 수 있다.
* 그러나 이렇듯 **CPU의 코어 수 대비 많은 수의 스레드를 할당하는 멀티스레딩 기법은 다음과 같은 문제점**을 갖는다.
  1. 컨텍스트 스위칭으로 인한 스레드 전환 비용: **프로세스의 정보를 PCB에 저장하고, 또 다른 프로세스 정보를 PCB로부터 로드하는 데에 시간이 소요**된다.
  2. 과다한 메모리 사용으로 인한 오버헤드: JVM은 새로운 스레드를 위해 스택의 일부를 할당하므로, 스레드가 늘어날수록 물리적인 메모리 사용량 역시 증가한다.
  3. 스레드 풀로부터의 응답 지연: 스레드 풀에 유휴 스레드가 존재하지 않을 경우, 가용한 스레드가 확보되고 사용 가능한 상태로 전환되기까지 지연이 발생한다.

## 2023-10-27 Fri
### 멀티스레딩 기법의 단점 살펴보기
* 우선, **프로세스의 정보를 PCB에 저장하고 로드하는 동안 CPU는 다른 작업을 수행하지 못하고 대기하기에 컨텍스트 스위칭의 횟수는 성능에 큰 영향**을 준다.
  * 심지어 멀티스레딩 기법은 프로세스 당 다수의 스레드를 실행하므로, 스레드 간에도 컨텍스트 스위칭이 유발된다.
* 다음으로 **서블릿 컨테이너 기반의 Java 웹 애플리케이션은 요청당 하나의 스레드를 생성하므로, 추가 작업을 위한 스레드의 할당은 큰 오버헤드를 유발**시킨다.
  * 예를 들어, 요청 별 작업을 처리하기 위해 스레드를 추가적으로 할당할 경우 메모리 오버헤드는 걷잡을 수 없이 늘어날 수 있다.
* 마지막으로 **`Spring Boot`는 자체적으로 톰캣을 내장하며, 톰캣은 서블릿 컨테이너로서 요청을 효율적으로 처리하기 위해 스레드 풀을 활용**한다.
  * 스레드 풀은 사용자의 요청이 발생할 때마다 스레드를 처음부터 생성하는 대신, 미리 일정 개수의 스레드를 생성하여 사용자의 요청에 대응한다.
  * 그러나 너무 많은 요청이 발생할 경우, 스레드 풀에 가용한 유휴 스레드가 존재하지 않는다면 이로 인해 응답 지연이 발생하기 쉽다.
  * 이 때, 임의의 **사용자 요청을 처리한 후 스레드 풀에 반환된 스레드는 다시 가용해지기까지의 지연 시간을 추가로 소모**한다.

## 2023-10-28 Sat
### 논블로킹 IO란?
```
> 논블로킹 IO는 블로킹 IO와는 달리 스레드를 차단하지 않는다.
```
* **논블로킹 IO를 사용하는 경우, 실제로 작업을 수행하는 스레드의 종료 여부와 관계 없이 작업을 요청한 스레드가 차단되지 않는다**.
  * 또한, **스레드가 차단되지 않는 특징으로 인해 단일 스레드로도 많은 수의 요청을 처리**할 수 있게 된다.
* **논블로킹 IO 방식은 블로킹 IO 방식과 멀티스레딩 기법에 비해 적은 스레드를 사용하므로 상술한 문제에서 자유로우며, CPU 사용량 측면에서도 효율적**이다.
* 반면, 논블로킹 IO 방식은 다음과 같은 상황에서는 좋지 않은 효율을 보여주는 특징을 갖는다.
  1. 스래드 내적으로 **CPU 바운드 작업을 포함하는 경우, 성능에 악영향**을 준다.
  2. 작업 요청에서 완료까지의 전체 흐름 중에 **블로킹 IO 방식이 포함된 경우, 논블로킹 IO 방식만의 장점을 발휘할 수 없다**.

## 2023-10-29 Sun
### Spring MVC와 Spring WebFlux
* **웹 애플리케이션 개발 시 자주 사용되는 `Spring MVC`는 매우 대중적이나, 내부적으로는 블로킹 IO 방식을 사용한다는 특징**을 갖는다.
  * 이로 인해 스프링 MVC 기반의 애플리케이션으로는 많은 수의 요청을 처리하기 버거울 수 있다.
* **`Spring WebFlux`는 스프링 MVC의 대안으로 제안되었으며, 내부적으로 논블로킹 IO 벙식을 사용한다는 특징**을 갖는다.
* 스프링 MVC의 경우, 서블릿 컨테이너를 기반으로 동작하므로 요청 당 하나의 스레드를 사용하므로 멀티스레딩 기법의 단점을 그대로 답습한다.
* 반면 **스프링 WebFlux의 경우, 비동기 논블로킹 IO 기반의 서버 엔진인 `Netty`를 사용하므로 적은 수의 스레드로 더 많은 요청을 처리**할 수 있다.
  * 이 과정에서 CPU나 메모리 자원을 더 효율적으로 사용하므로, 결과적으로는 같은 조건에서 더 좋은 성능의 애플리케이션을 운영할 수 있도록 지원한다.

## 2023-10-30 Mon
### RestTemplate과 WebClient, 그리고 Mono
* **스프링 MVC의 경우, `RestTemplate`을 활용하여 요청을 전송하고 결과를 `ResponseEntity<T>` 클래스를 활용하여 반환**한다.
* 반면 **스프링 WebFlux의 경우, `WebClient`를 활용하여 요청을 전송하고 결과를 `Mono<T>` 클래스를 활용하여 반환**한다.
  * 이 때, **스프링 WebFlux에서는 `WebClient`가 논블로킹 IO 방식으로 리액티브 타입을 전송하고 수신하는 역할을 전담**한다.
* 또한 **`Mono<T>`는 Reactor 라이브러리에서 지원하는 `Publisher` 타입 중 하나로, 단 하나의 데이터를 `emit`하는 타입에 해당**한다.
  * 일반적인 **HTTP 통신은 응답으로 JSON 형태를 활용하며, JSON은 하나의 문자열로 구성된 단일 데이터이기 때문에 `Mono<T>`를 활용하기에 적합**하다.
  * 중요한 것은 **`Mono<T>` 역시 `Publisher` 타입이므로, `Mono<T>`로 반환된 응답은 `subscribe()` 메소드로 실제 데이터를 처리**해주어야 한다.
  * 즉, **어떠한 `Publisher` 구현테던 간에 수신한 데이터를 처리하기 위해서는 반드시 `subscribe()` 메소드를 호출하는 과정이 전제**되어야 한다.

## 2023-10-31 Tue
### 블로킹 IO와 논블로킹 IO의 비교
* 블로킹 IO의 경우, 실제로 작업을 수행하는 스레드의 작업이 종료될 때까지 작업을 요청한 스레드는 차단된다.
  * 반면 논블로킹 IO의 경우, 작업 스레드의 종료 여부와 관계 없이 요청 스레드는 차단되지 않는다.
* 블로킹 IO의 경우, 스레드 차단 문제를 보완하기 위해 멀티스레딩 기법을 도입할 수 있으나 이는 여러 문제를 유발할 수 있다.
  * 반면 논블로킹 IO의 경우, 적은 수의 스레드를 사용하므로 스레드 전환 비용이 작아 CPU를 효율적으로 처리할 수 있다.
  * 그러나 논블로킹 IO 작업이 CPU 바운드 작업인 경우에는 성능에 악영향을 줄 가능성이 더 크다.

## 2023-11-01 Wed
### 스프링 WebFlux 적용을 고려하기
* 일반적으로 논블로킹 IO 방식을 활용하는 스프링 WebFlux가 스프링 MVC에 비해 더 좋은 성능을 보여주나, 이외에도 다음과 같은 사항을 고려할 필요가 있다.
  1. 스프링에 익숙한 개발자는 금방 사용할 수 있는 스프링 MVC에 비해, 스프링 WebFlux는 리액티브 스트림즈 자체에 대한 러닝 커브가 존재한다.
  2. 스프링 MVC에 비해 스프링 WebFlux에 익숙한 개발자를 찾기 어려우므로 프로젝트 자체의 위험부담이 높아질 수 있다.
* 반면, 스프링 WebFlux는 일반적으로 다음과 같은 유형의 애플리케이션에 적합하므로 우선적으로 고려해볼 수 있다.
  1. 대량의 요청 트래픽이 발생할 수 있는 시스템: 바꿔 말해, **트래픽을 충분히 감당할 수 있는 수준이라면 서블릿 기반의 블로킹 IO 방식으로도 충분**하다.
  2. 마이크로 서비스 기반의 시스템: **마이크로 서비스 기반의 시스템은 그 특성상 서비스들 간에 수많은 IO가 지속적으로 발생**한다.
  3. 끊임없이 전달되는 데이터 스트림을 처리하는 시스템: **스프링 WebFlux는 무한한 데이터 스트림을 처리하기 위한 스트리밍 시스템을 구축하기에 적합**하다.