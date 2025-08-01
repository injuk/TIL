# Spring Boot
## 2025-06-28 Sat
### 모니터링의 중요성
* 앞서 다루었듯, **서비스를 운영하는 개발자 입장에서 장애는 언제든지 발생할 수 있다는 점을 인정하고 이에 대비하여 모니터링을 철저히하는 것이 바람직**하다.
  * 이 경우, 애플리케이션이 동작하는 서버의 CPU와 메모리 및 커넥션과 고객 요청 수 등의 수 많은 지표를 확인하는 것이 중요하다.
  * 이를 토대로 어디에 문제가 발생하는지 빠르게 찾아낼 수 있으며, 원인을 파악하여 대처하는 것이 가능하다.
* 이러한 모니터링을 돕기 위한 수 많은 도구가 있으며, 대부분의 경우 시스템의 정보를 이러한 모니터링 도구에 전달하는 방식으로 활용하게 된다.
  * 바꿔말해, 이러한 모니터링 도구를 활용하기 위해서는 그들이 원하는 형태의 지표 데이터를 전달해줄 필요가 있다.
  * 다행히도, 대부분의 경우 이러한 데이터 매핑 과정은 라이브러리에 의해 자동화되곤 한다.

## 2025-06-29 Sun
### 마이크로미터란?
* 상술한 바와 같이, 세상의 수많은 모니터링 도구들은 모니터링 기능을 위해 저마다 서로 다른 데이터 형태를 요구한다.
* 때문에 모니터링 도구 A를 사용할 경우 A가 요구하는 데이터 매핑을 처리하게 되지만, 이를 도구 B로 변경할 경우 많은 범위의 코드 수정이 필요할 수 있다.
  * 즉, 개발자 입장에서는 단순한 도구 변경에 지나지 않아야 할 변경 사항이 운영 코드에 영향을 미칠 가능성을 무시할 수 없게 된다.
* **마이크로미터는 이러한 문제를 해결하기 위한 라이브러리이며, 표준 측정 방식을 추상화된 계층을 통해 제공한다 점에서 큰 의의**를 갖는다.
* 모니터링에 필요한 여러 메트릭 정보는 마이크로미터가 제공하는 표준 측정 방식에 맞추어 전달되며, 개발자는 자신의 모니터링 도구에 맞는 구현체를 사용하게 된다.
  * 물론, **유명한 모니터링 도구들의 경우 각 모니터링 도구에 맞는 구현체가 이미 개발되어 있으므로 개발자는 그저 이를 가져다 사용하기만** 할 수 있다.

## 2025-06-30 Mon
### 마이크로미터의 특징
```
> 개발자는 마이크로미터가 정한 표준에 맞는 메트릭을 전달하는 것만으로도 유명한 모니터링 도구들을 편리하게 사용할 수 있다.
```
* 마이크로미터는 애플리케이션 메트릭 파사드라는 용어로도 지칭되며, 애플리케이션의 메트릭 정보를 마이크로미터가 정한 표준 방식으로 모아 제공한다.
  * 다시 말해, 마이크로미터는 추상화 계층을 도입하여 모니터링 구현체를 쉽게 변경할 수 있도록 다형성을 제공한다.
  * 이는 마치 `SLF4J`가 로그 계층을 추상화하는 것과 유사하다.
* 스프링 진영의 경우, 많은 상황에서 스프링이 직접 추상화를 진행하지만 마이크로미터는 이미 잘 만들어진 라이브러리이기에 스프링은 이를 활용한다.
  * 앞서 다루었던 스프링 부트의 액추에이터 역시 이러한 마이크로미터를 기본으로 내장하여 필요한 기능들을 제공한다.

## 2025-07-01 Tue
### 액추에이터와 metrics 엔드포인트
* CPU 사용량이나 JVM 정보, 커넥션 정보 등의 지표를 수집하기 위해서는 개발자가 이를 각각 수집하고 마이크로미터에 등록해야할 것처럼 보인다.
  * 그러나 마이크로미터는 다양한 지표 수집 기능을 이미 제공하며, 액추에이터는 이러한 지표 수집 기능을 `@AutoConfiguration`을 통해 자동으로 등록한다.
  * 다시 말해, 액추에이터만 사용하더라도 수 많은 지표 정보를 손쉽게 사용하는 것이 가능하다.
* 스프링 부트 액추에이터는 기본으로 제공되는 지표를 확인할 수 있도록 `metrics` 엔드포인트를 제공하며, 이를 조회할 경우 다음과 같은 정보를 확인할 수 있다.
  1. `system.cpu.count`
  2. `system.cpu.usage`
  3. `jvm.memory.max`
  4. `jvm.memory.used`
  5. `hikaricp.connections.active`
  6. `hikaricp.connections.idle`
  7. `disk.free`
  8. `disk.total`
* 이는 기본적으로 제공되는 다양한 지표 중 일부에 불과하며, 이렇듯 **아무런 설정 없이도 마이크로미터는 수 많은 지표 정보를 기본적으로 수집**한다.

## 2025-07-02 Wed
### metrics 엔드포인트를 활용한 지표 정보 조회
* 마이크로미터에 의해 수집된 지표 정보는 `/actuator/metrics/[지표명]` 형태의 엔드포인트를 조회하는 것으로 확인할 수 있다.
  * 상술한 지표 이름 목록을 예로 들었을 때, JVM 메모리 사용량은 `/actuator/metrics/jvm.memory.used` 형태의 엔드포인트로 조회할 수 있다.
```json
{
  "name": "jvm.memory.used",
  "description": "The amount of used memory",
  "baseUnit": "bytes",
  "measurements": [
    {
      "statistic": "VALUE",
      "value": 42
    }
  ],
  "availableTags": [
    {
      "tag": "area",
      "values": [
        "heap",
        "noheap"
      ]
    },
    {
      "tag": "id",
      "values": [
        "G1 Survivor Space",
        "Compressed Class Space",
        "Metaspace",
        "CodeCache",
        "G1 Old Gen",
        "G1 Eden Space"
      ]
    }
  ]
}
```

## 2025-07-03 Thu
### metrics 엔드포인트와 태그 필터
* 마이크로미터가 기본으로 수집하는 여러 지표는 해당 지표에 대해 보다 세부적인 정보를 제공할 수 있도록 태그 필터를 제공한다.
  * 이 경우, 임의의 지표 정보에 적용 가능한 태그 필터 목록은 `availableTags` 프로퍼티에 표현된다.
* 반환 데이터 중 **`availableTags`의 `tag`는 임의의 지표 정보를 필터하기 위한 조건으로, `tag=[키]:[값]` 형태의 쿼리 파라미터로 사용**할 수 있다.
  * 예를 들어 `/actuator/metrics?tag=area:heap`를 입력한 경우, 전체 메모리 사용량 중 힙 메모리 사용량만을 반환한다.
* 반면, **필터를 여러 개 적용하고자 하는 경우 `?tag=[키]:[값]&tag=[키]:[값]`과 같이 `tag` 쿼리 파라미터를 여러번 명시**할 수 있다.

## 2025-07-04 Fri
### 마이크로미터와 액추에이터의 다양한 지표
* 마이크로미터와 액추에이터는 기본적으로 다음과 같은 다양한 지표 정보를 제공한다.
  1. JVM 메트릭
  2. 시스템 메트릭
  3. 애플리케이션 시작 메트릭
  4. 스프링 MVC 메트릭
  5. 톰캣 메트릭
  6. 데이터 소스 메트릭
  7. 로그 메트릭
* 뿐만 아니라, **개발자가 원한다면 자신만의 사용자 정의 지표 정보를 추가적으로 정의하는 것 역시 가능**하다.

## 2025-07-05 Sat
### JVM 메트릭이란?
* JVM 메트릭은 `jvm` 접두사를 가지며, 다음과 같은 다양한 지표 정보를 제공한다.
  1. 메모리 및 버퍼 풀의 세부 정보
  2. 쓰레기 수집 관련 통계 정보
  3. 스레드 활용과 관련된 정보
  4. 로드 및 언로드 된 클래스 수
  5. JVM의 버전 정보
  6. JIT 컴파일 시간과 관련된 정보

## 2025-07-06 Sun
### 시스템 메트릭이란?
* 시스템 메트릭은 `system`이나 `process` 또는 `disk` 형태의 접두사를 가지며, 다음과 같은 다양한 지표 정보를 제공한다.
  1. CPU 지표 정보
  2. 파일 디스크립터 지표 정보
  3. 가동 시간 지표 정보
  4. 사용 가능한 디스크 공간

## 2025-07-07 Mon
### 애플리케이션 시작 메트릭이란?
* 애플리케이션 시작 메트릭은 애플리케이션 시작 시간과 관련된 다음과 같은 지표 정보를 제공한다.
  1. 애플리케이션을 시작하는데까지 걸리는 시간
  2. 애플리케이션이 요청을 처리할 준비가 완료되는데까지 걸리는 시간
* 참고로, **스프링은 내부적으로 여러 초기화 단계를 갖고 있으며 각 단계 별로 애플리케이션 이벤트가 발행**된다.
  * 애플리케이션 시작 메트릭은 이를 활용하며, 예를 들어 애플리케이션 시작까지 걸리는 시간은 `ApplicationStartedEvent`로 측정한다.
  * 해당 이벤트의 경우, 스프링 컨테이너가 완전히 실행되었으며 직후에 커맨드 라인 러너를 호출하는 시점에 발행된다.
  * 반면, 요청 처리 준비 시간은 `ApplicationReadyEvent`로 측정하며 해당 이벤트는 커맨드 라인 러너까지 실행된 후에 발행된다.

## 2025-07-08 Tue
### 스프링 MVC 메트릭이란?
* 스프링 MVC 메트릭은 스프링 MVC 컨트롤러가 처리하는 모든 요청을 다루며, `http.server.requests`와 같은 형태의 이름을 갖는다.
* 해당 지표 정보의 경우, `TAG` 기능을 활용하여 다음과 같은 정보를 분류한 후에 확인하는 것이 가능하다.
  1. `uri`: 요청 URI를 기준으로 분류한다.
  2. `method`: HTTP 메소드를 기준으로 분류한다.
  3. `status`: HTTP 상태 코드를 기준으로 분류한다.
  4. `exception`: 예외를 기준으로 분류한다.
  5. `outcome`: 상태 코드를 `1xx:INFORMATIONAL`이나 `2xx:SUCCESS`와 같은 형태의 그룹으로 모아 분류한다.

## 2025-07-09 Wed
### 데이터소스 메트릭이란?
* 데이터소스 메트릭은 `DataSource` 자체나 커넥션 풀과 관련된 지표 정보를 제공하며, 일반적으로 `jdbc.connections`와 같은 접두사를 갖는다.
  * 이를 통해 최대 커넥션이나 최소 커넥션, 활성 커넥션과 대기 커넥션 수 등의 커넥션 관련 지표 정보를 확인할 수 있다.
* 반면, Hikari CP를 사용하는 경우 `hikaricp`와 같은 접두사를 갖는 지표 정보를 통해 Hikari CP와 관련된 지표 정보를 확인할 수 있다.

## 2025-07-10 Thu
### 로그 메트릭이란?
* 로그 메트릭은 `logback.events`라는 이름의 지표를 통해 logback 로그와 관련된 지표 정보를 제공한다.
  * 이를 통해 `trace`나 `debug`, `info`나 `error`와 같은 여러 로그 레벨을 기준으로 분류된 로그의 개수를 확인할 수 있다.
  * 예를 들어, 임의의 시간대에 `error` 로그 수가 급격히 높아지는 것을 지표 정보를 통해 확인할 수 있다면 문제 상황에 미연하게 대처할 수 있다.

## 2025-07-11 Fri
### 톰캣 메트릭이란?
* 톰캣 메트릭은 `tomcat` 형태의 접두사를 가지며, 모든 지표 정보를 제공 받기 위해서는 외부 설정 파일에 다음과 같은 정보를 입력할 필요가 있다.
  * 입력하지 않을 경우, 톰캣 메트릭은 기본적으로 `tomcat.session` 접두사를 갖는 지표 정보만을 노출한다.
```yaml
server:
  tomcat:
    mbeanregistry:
      enabled: true
```
* 상술한 설정을 적용할 경우, 톰캣의 최대 스레드나 사용 중인 스레드 수 등의 다양한 톰캣 관련 지표 정보를 제공받을 수 있다.
  * 실무적인 관점에서, 해당 메트릭들은 모두 매우 유용하기에 기본적으로 활성화하는 것이 권장된다.

## 2025-07-12 Sat
### 여러 메트릭과 사용자 정의 메트릭
* 이외에도 `RestTemplate`이나 `WebClient`와 같은 HTTP 클라이언트 메트릭은 물론, 다음과 같은 다양한 지표 정보 등이 제공된다.
  1. 캐시 메트릭
  2. 작업 실행 및 스케줄 메트릭
  3. 스프링 데이터 리포지토리 메트릭
  4. MongoDB 메트릭
  5. Redis 메트릭
* 또한, 사용자가 직접 자신이 원하는 지표 정보를 제공할 수도 있으며 이를 위해서는 마이크로미터의 사용법을 이해할 것이 전제된다.

## 2025-07-13 Sun
### 지표 정보 활용하기
```
> 지표 정보는 수집되는 것에서 그치지 않고, 저장되고 시각화될 때 비로소 새로운 통찰을 제공할 수 있다.
```
* 앞서 다루었듯, 액추에이터를 활용할 경우 수많은 지표 정보가 자동으로 제공되는 것을 알 수 있다.
* 그러나 이러한 지표 정보들은 모두 어딘가에 보관될 필요가 있으며, 이를 통해 과거의 지표 정보를 통해 유용한 정보를 얻어낼 수 있게 된다.
  * 즉, **지속적으로 수집되는 지표 정보들을 저장할 수 있을만한 일종의 데이터베이스가 필수적**이다.
  * 또한, **수집되어 저장된 지표 정보를 시각화할 수 있을만한 대시보드 역시 필수적**이다.

## 2025-07-14 Mon
### 프로메테우스와 그라파나
```
> 프로메테우스는 지표 정보를 위한 데이터베이스로 활용되는 반면, 그라파나는 유연한 그래프 형태의 시각화를 담당한다.
```
* 발생한 지표 정보를 그 순간에만 활용하지 않고, 이를 저장하여 과거 이력을 확인할 수 있도록 하기 위해서는 지표 정보용 데이터베이스가 필요하다.
  * 프로메테우스는 이렇듯 지표 정보를 지속적으로 수집하고, 영속화하는 역할을 담당한다.
* 지표 정보가 영속화되었다고 할 때, 이를 그래프 등의 사용자 친화적인 형태로 시각화하는 도구 역시 필수적이다.
  * 그라파나는 데이터를 다양한 그래프 형태로 시각화하는 도구이며, 프로메테우스에 종속되지 않고 다양한 데이터소스를 지원한다.

## 2025-07-15 Tue
### 프로메테우스와 그라파나를 활용한 지표 수집 및 시각화 흐름
* 스프링 부트의 액추에이터와 마이크로미터를 사용할 경우 수많은 지표 정보가 자동 생성되며, 프로메테우스를 활용하면 다음과 같은 흐름이 적용된다.
  1. 마이크로미터의 프로메테우스 구현체가 지표 정보를 프로메테우스와 호환되는 형태로 생성한다.
  2. 프로메테우스는 이러한 지표 정보를 지속적으로 수집한다.
  3. 프로메테우스는 수집한 지표 정보를 자신의 내부 데이터베이스에 저장한다.
  4. 사용자가 임의의 정보를 조회하고자 하는 경우, 그라파나에 쿼리를 요청하는 것으로 필요한 데이터를 프로메테우스로부터 조회한다.
  5. 그라파나는 프로메테우스로부터 조회한 정보를 시각화한다.

## 2025-07-16 Wed
### 프로메테우스를 사용하기 위한 전제 조건
* 앞서 다루었듯, **프로메테우스는 메트릭을 수집하고 보관하는 일종의 데이터베이스 역할**을 담당한다.
* 예를 들어 프로메테우스가 임의의 스프링 애플리케이션으로부터 지표 정보를 수집할 수 있도록 하기 위해서는 다음과 같은 작업이 전제되어야 한다.
  1. 애플리케이션 설정하기: 프로메테우스가 애플리케이션의 지표 정보를 수집할 수 있도록 애플리케이션 차원에서 프로메테우스에 맞는 지표를 생성할 필요가 있다.
  2. 프로메테우스 설정하기: 프로메테우스가 임의의 스프링 애플리케이션의 지표를 주기적으로 수집하도록 설정할 필요가 있다.

## 2025-07-17 Thu
### 프로메테우스를 위한 애플리케이션 설정
* 프로메테우스가 임의의 스프링 애플리케이션으로부터 지표를 수집하기 위해서는 우선 해당 애플리케이션이 프로메테우스의 스펙에 맞는 지표를 생성해야 한다.
  * 예를 들어, 앞서 다루었던 액추에이터의 여러 지표 정보를 표기하는 JSON 표기법은 프로메테우스가 받아들이지 못한다.
* 반면, 마이크로미터는 이러한 작업을 위해 개발자가 직접 데이터 모델을 정의하고 파싱하는 로직을 작성할 필요 없도록 지원한다.
  * **마이크로미터를 사용할 경우, 필요한 각각의 지표 정보는 마이크로미터의 표준 방식으로 수집되므로 어떤 구현체를 사용할지 명시하기만 해도 무방**하다.
* 프로메테우스가 지표 정보를 원활히 수집할 수 있도록 하기 위해 마이크로미터를 활용하는 경우, 다음과 같은 설정 작업이 전제되어야 한다.
```groovy
implementation 'io.micrometer:micrometer-registry-prometheus' // build.gradle에 마이크로미터의 프로메테우스 구현체 의존성을 명시한다.
```
* 단지 **의존성을 추가하는 것만으로도 스프링 부트와 액추에이터는 자동으로 마이크로미터가 프로메테우스 구현체를 사용하도록 설정**한다.
  * 또한, **액추에이터가 제공하는 여러 지표 정보에 `/actuator/prometheus`와 같은 프로메테우스용 지표 정보 수집 엔드포인트가 등록**된다.
  * 해당 엔드포인트를 브라우저를 통해 조회할 경우, 익숙한 JSON 표기법이 아닌 프로메테우스 전용 표기법에 맞는 데이터를 확인할 수 있다.

## 2025-07-18 Fri
### 프로메테우스 전용 표기법이란?
* 임의의 지표 정보의 키를 예로 들어, 프로메테우스의 표기법은 기존의 JSON 표기법과는 달리 `_` 기호를 자주 활용한다.
  * 예를 들어, `jvm.info` 지표는 프로메테우스 표기법 상 `jvm_info`로 표기된다.
* 지속적으로 숫자가 증가해가는 지표 정보의 경우 `카운터`라는 용어로 지칭하며, 프로메테우스 표기법 상 카운터는 관례적으로 `_total`이라는 접미사를 명시한다.
  * 예를 들어, `logback.events`는 점점 그 숫자가 늘어나는 지표 정보에 해당하므로 `logback_events_total`이라는 이름을 갖게 된다.
  * 반면, CPU 사용량과 같이 값이 누적되지 않고 증감을 반복하는 지표 정보의 경우 `게이지`라는 용어로 지칭한다.
* `http.server.requests` 지표 정보는 내부적으로 요청 수와 시간 합, 최대 시간 정보를 갖기에 프로메테우스 상에서는 이를 다음과 같이 분리한다.
  1. `http_server_requests_seconds_count`: 요청 수를 의미한다.
  2. `http_server_requests_seconds_sum`: 요청 수의 시간 합을 의미한다.
  3. `http_server_requests_seconds_max`: 최근 발생한 요청들 중 가장 오래 걸린 요청 수의 최대 시간을 의미한다.

## 2025-07-19 Sat
### 프로메테우스의 수집 설정
```
> 임의의 스프링 애플리케이션이 마이크로미터의 프로메테우스 구현체를 등록하여 지표 정보를 수집하고 있을 경우, 프로메테우스 역시 적절히 설정될 필요가 있다.
```
* 프로메테우스는 앞서 다루었던 `/actuator/prometheus` 엔드포인트를 주기적으로 호출하여 지표 정보를 수집하고, 저장한다.
  * 때문에 이를 위해 프로메테우스 역시 설정될 필요가 있으며, 프로메테우스의 설정은 `prometheus.yml` 파일을 활용한다.
* 프로메테우스의 설정 파일인 `prometheus.yml`에 다음과 같은 내용을 추가하는 것으로 스프링 애플리케이션으로부터 지표 정보를 수집할 수 있게 된다.
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
alerting:
  alertmanagers:
    - static_configs:
      - targets:
        # - alertmanager:9093

rule_files:
scrape_configs:
  # 해당 내용은 기본적으로 명시되어 있으며, 프로메테우스 서버가 자기 자신의 지표 정보를 수집하기 위한 설정에 해당한다.
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  # 아래의 내용을 추가한다.
  - job_name: "spring-actuator" # 사용자가 알아보기 쉬운 임의의 이름을 명시한다.
    metric_path: "/actuator/prometheus" # 액추에이터가 제공하는 지표 정보 수집용 엔드포인트를 명시한다.
    scrape_interval: 1s # 수집 주기를 명시하며, 일반적으로는 10초 ~ 1분 정도의 값을 명시한다.
    static_configs:
      - targets: ["localhost:8080"] # 지표 정보를 수집하기 위한 서버의 주소와 포트 정보를 명시한다.
```
* 이 때, 테스트를 위해 `scrape_interval`을 1초로 설정했으나 기본 값은 1분으로 설정된다는 점에 주의를 기울여야 한다.
  * **지표 정보 수집 주기가 너무 짧을 경우, 오히려 애플리케이션의 성능에 악영향을 줄 수 있으므로 운영 환경에서는 최소한 10초 정도의 설정이 권장**된다.
* 상술한 설정을 추가한 후 프로메테우스 서버를 재시작하면 설정이 적용되며, 신규 설정의 적용 여부는 프로메테우스 콘솔 상단의 `Status` 탭에서 확인 가능하다.
  * 정확히는 `Status` 탭의 하위 메뉴 중 `Configuration`과 `Targets` 메뉴에서 설정 내용이 적용된 것을 상세히 확인할 수 있다.

## 2025-07-20 Sun
### 프로메테우스의 테이블과 그래프
* 프로메테우스로 임의의 지표 정보를 조회할 경우, 기본적으로 다음과 같은 형태의 데이터를 확인할 수 있다.
```
http_server_requests_seconds_count{error="none",instance="localhost:8080",job="spring-actuator",method="GET",outcome="SUCCESS",status="200",uri="/actuator/prometheus"}
```
* 각 데이터의 키 값을 담당하는 `error`와 `instance`, `job` 등은 프로메테우스 상에서 레이블이라는 용어로 지칭한다.
  * 반면, 마이크로미터의 경우 이러한 정보를 태그라는 용어로 지칭한다.
  * 또한, 각 데이터의 우측에는 해당 지표의 값을 의미하는 임의의 자연수가 노출된다.
* 임의의 지표 정보에 대한 검색 결과는 `Table`에 노출되며, 각 테이블의 정보는 `Evaluation time` 정보를 수정하여 임의의 시간대를 조회할 수도 있다.
  * 나아가 `Graph` 기능을 활용할 경우, 임의의 지표 정보를 그래프 형태로 조회하는 것 역시 가능하다.

## 2025-07-21 Mon
### 프로메테우스의 필터 기능
* 레이블을 기준으로 원하는 결과만을 걸러낼 수 있으며, 이는 중괄호 표기법에 더해 다음과 같은 문법을 활용하는 필터 기능에 해당한다.
  1. `=`: 제공된 문자열과 정확히 동일한 레이블만을 필터링한다.
  2. `!=`: 제공된 문자열과 일치하지 않는 레이블만을 필터링한다.
  3. `=~`: 제공된 문자열과 정규표현식에 의해 일치하는 레이블만을 필터링한다.
  4. `!~`: 제공된 문자열과 정규표현식에 의해 일치하지 않는 레이블만을 필터링한다.
* 예를 들어, `http_server_requests_seconds_count{uri="/log", method="GET"}`과 같은 필터링이 가능하다.
  * 해당 표현식의 경우, `/log` 경로에 `GET`으로 요청된 지표 정보만을 필터링한다.

## 2025-07-22 Tue
### 프로메테우스의 연산자 쿼리
* 일반적인 프로그래밍 언어와 유사하게, 프로메테우스는 다음과 같은 연산자를 지원한다.
  1. `+`: 덧셈에 해당한다.
  2. `-`: 뺄셈에 해당한다.
  3. `*`: 곱셈에 해당한다.
  4. `/`: 나눗셈에 해당한다.
  5. `%`: 모듈러 연산에 해당한다.
  6. `^`: 지수에 해당한다.

## 2025-07-23 Wed
### 프로메테우스의 빌트인 함수
* SQL과 유사하게, 프로메테우스는 다음과 같은 유용한 빌트인 함수를 제공한다.
  1. `sum`: 조회된 결과의 전체 합을 반환한다.
  2. `sum by`: `sum by(method, status)(http_server_requests_seconds_count)`와 같은 형태로 그룹화한 합을 반환한다.
  3. `count`: 지표 정보의 갯수를 반환한다.
  4. `topk`: `topk(3, http_server_requests_seconds_count)`와 같은 형태로 임의의 지표 정보의 상위 `k` 개를 반환한다.
  5. `offset`: `http_server_requests_seconds_count offset 10m`과 같은 형태로, 현재를 기준으로 임의의 시간 전의 지표 정보를 반환한다.
* 이 밖에도 `http_server_requests_seconds_count[1m]`과 같은 형태로 입력하는 `범위 벡터 선택기` 기능 역시 제공된다.
  * 해당 기능은 현재를 기준으로 지난 단위 시간 동안의 모든 기록값을 선택하여 반환한다.
  * 반면, 해당 기능은 데이터로만 확인이 가능하며 이를 별도의 가공 과정을 거치지 않고서는 바로 시각화하는 것이 불가능하다.

## 2025-07-24 Thu
### 프로메테우스의 게이지와 카운터
* 지표 정보는 다음과 같이 크게 게이지와 카운터라는 두 가지 용어로 분류할 수 있다.
  1. 게이지: 임의로 오르내릴 수 있는 값으로, CPU 사용량이나 사용 중인 커넥션 수 등이 해당한다.
  2. 카운터: 단순하게 누적되어 증가하기만 하는 값으로, HTTP 요청 수 등이 해당된다.
* 상술한 특징에 의해, 지표 정보를 시각화하는 과정에서 게이지는 현재 상태를 그대로 출력하는 것으로 충분하다.
  * 이를 통해 임의의 과거 시점으로부터 현재에 이르는 지표 정보의 변화량을 한 눈에 확인할 수 있으며, 이렇듯 게이지는 가장 단순한 지표 정보에 해당한다.
* 반면, **카운터의 경우 꺾은선 그래프로 시각화했을 때 계속해서 우상향하기에 점차 한 눈에 확인하기 어려워진다**.
  * 구체적인 예를 들어, 특정 시간에 얼마나 많은 요청이 있었는지 확인하기 어렵다.
  * 이는 그래프에 표현되는 값이 계속해서 증가하기만 하기 때문으로, 이를 해결하기 위해 `increase()`나 `rate()` 같은 함수가 지원된다.

## 2025-07-25 Fri
### increase 함수와 rate 함수란?
* 상술한 카운터 지표 시각화 문제를 해결하기 위해 **`increase()` 함수가 제공되며, 이를 호출하는 것으로 지정 시간 단위 별로 증가를 확인**할 수 있다.
  * 해당 함수는 `increase(http_server_requests_seconds_count{uri="/log"}[1m])` 형태로 반드시 `[시간]` 형태의 범위 벡터를 명시해야 한다.
  * 이를 적절히 활용할 경우 단위 시간 마다 얼마나 카운터가 증가했는지 한 눈에 파악할 수 있다.
* 숫자를 직접 카운팅하는 `increase()` 함수와 달리, `rate()` 함수는 범위 벡터 상 초당 평균 증가율을 계산한다.
  * 즉, `increase()`가 1분을 기준으로 실행된 경우 단위 시간은 60초가 되므로 `rate()`를 호출하면 이에 60을 나눈 수가 반환된다.
  * 반면, `increase()`가 2분을 기준으로 실행된 경우 단위 시간은 120초가 되므로 `rate()`를 호출하면 이에 120을 나눈 수가 반환된다.
* 두 함수 간의 차이를 복잡하게 생각하거나 외울 것은 없으며, 전체적으로 유사한 형태의 그래프를 출력하지만 연산된 값이 달라지는 것에 지나지 않는다.
* 나아가 `irate()` 함수는 범위 벡터에서 초당 순간 증가율을 계산하며, 주로 급격한 증가 내용을 확인하는 데에 유용하다.

## 2025-07-26 Sat
### 게이지와 카운터 활용하기
* 이렇듯 수집된 지표 정보의 특징에 따라 게이지와 카운터로 분류할 수 있으며, 두 개념은 각각 다음과 같이 활용될 수 있다.
  1. 게이지: 값이 계속해서 오르내리는 게이지는 그저 현재 값을 그대로 그래프로 시각화해도 무방하다.
  2. 카운터: 값이 단조롭게 증가하는 카운터는 `increase()`나 `rate()`, `irate()` 등의 함수를 활용하여 표현해야 한다.
* 반면, **프로메테우스의 단점은 수집된 지표를 이해하기 쉽게 시각화하는 대시보드 기능이 부실하다는 점**에 있다.
  * **실무의 경우, 이러한 단점을 해소하기 위해 그라파나라는 시각화 도구의 도입을 고려**해볼 수 있다.

## 2025-07-27 Sun
### 그라파나 설치하기
* 그라파나 공식 홈페이지에서 파일을 다운로드한 후 압축을 풀 경우, `bin` 폴더 내에 `grafana-server` 파일을 실행해준다.
  * macOS의 경우, 해당 경로에서 `./grafana-server` 명령어를 입력한다.
  * 이 때, 프로메테우스와의 상호작용을 위해 프로메테우스를 종료하지 않도록 한다.
* 그라파나의 기본 포트는 3000번이며, 접속 후 기본 사용자 정보인 `admin` - `admin`을 입력하여 접속한다.
  * 이렇듯 그라파나는 기본적인 콘솔을 지원하며, 프로메테우스와 연동할 경우 지표 정보를 쉽게 시각화할 수 있도록 지원한다.

## 2025-07-28 Mon
### 그라파나와 프로메테우스를 연동하기
* 그라파나는 프로메테우스를 통해 데이터를 조회하고 시각화하는 역할을 수행하며, 이는 그라파나가 일종의 래퍼 역할을 담당하는 것으로 이해할 수 있다.
* 임의의 애플리케이션은 마이크로미터 표준 측정 방식을 통해 지표 정보를 생성하고, 프로메테우스는 프로메테우스 구현체를 통해 이를 지속적으로 수집한다.
  * 이렇게 수집된 지표 정보는 프로메테우스에 저장될 것이므로, 그라파나는 이를 조회하여 시각화하는 역할을 담당한다.
* 그라파나 콘솔에 접속한 후, 설정의 `Add data source` 버튼을 클릭하여 프로메테우스 서버 정보를 입력한다.
  * 입력을 마친 후 하단의 `Save & test` 버튼을 클릭할 경우, 큰 문제가 없다면 연동된 것을 확인할 수 있게 된다.
* 프로메테우스 연결 정보를 입력한 후, 다시 설정 페이지에 접속할 경우 `Data sources`에 프로메테우스가 등록된 것을 확인할 수 있다.

## 2025-07-29 Tue
### 대시보드와 패널
* 그라파나 대시보드를 실행하기 위해서는 모니터링 대상 애플리케이션과 프로메테우스, 그라파나가 반드시 모두 실행된 상태여야 한다.
* 좌측의 메뉴로부터 대시보드 메뉴를 찾아 진입한 후, `New` 버튼을 클릭하여 새로운 대시보드를 생성한다.
  * 버튼 클릭시 대시보드는 바로 생성되며, 우측 상단의 `Save dashboard` 버튼을 클릭하여 이름을 지정할 수 있다.
* 우측 상단의 `Add panel` 버튼을 활용하여 지표 정보를 시각화할 수 있다. 
  * 이렇듯 **그라파나의 대시보드는 시각화된 결과를 의미하지 않으며, 시각화된 결과는 패널이라는 이름의 구성 요소로 관리**된다.
  * 반면, **대시보드는 여러 패널을 포함하는 하나의 큰 단위로 이해**할 수 있다.

## 2025-07-30 Wed
### 그라파나와 공유 대시보드
* 상술한 그라파나의 대시보드와 패널을 통해, 적절한 지표 정보를 시각화된 그래프 형태로 확인할 수 있다.
  * 그러나 최초로 대시보드를 설정하는 과정은 예상 이상으로 번거로우며, 수작업으로 진행되기에 휴먼 에러 발생 가능성이 높다.
* 그라파나는 대시보드 설정을 공유할 수 있도록 하는 공유 대시보드 기능을 지원하며, 예를 들어 스프링에 특화된 공유 대시보드들을 가져다 사용할 수 있다.
* 예를 들어, `https://grafana.com/grafana/dashboards` 링크에서 여러 사용자들에 의해 생성된 공유 대시보드들을 확인할 수 있다.

## 2025-07-31 Thu
### 공유 대시보드의 장점
* 다른 사용자들이 이미 만들어둔 공유 대시보드를 활용할 경우, 직접 작업하지 않으므로 편리하면서도 유지보수성이 좋다.
  * 덕분에 양질의 모니터링 환경을 편리하게 구성할 수 있다.
* 반면, 각 대시보드에 포함된 패널의 지표 설정을 분석하는 것으로 추후 원하는 그라파나 대시보드를 구성해야할 때에 대비할 수 있다.
  * 그라파나 대시보드 구성은 러닝 커브가 다소 있는 편이지만, 다른 사용자들이 제작해둔 양질의 공유 대시보드는 좋은 교재가 되어준다.
  * 실무적인 관점에서, 그라파나를 깊이 있게 학습하는 것도 좋지만 마치 도구를 사용하듯 바로 사용 가능한 정도로 실용성에 초점을 두는 것을 고려할 수 있다. 

## 2025-08-01 Fri
### 그라파나와 지표 정보를 활용한 모니터링 대상
* 애플리케이션에서 실제로 문제가 발생한 경우, 그라파나를 통해 이를 조기에 모니터링한 후 대응할 수 있어야 한다.
* 이 때, **실무에서는 일반적으로 다음과 같은 상황이 자주 발생하므로 이들 지표 정보를 그라파나를 통해 각각 어떻게 모니터링하는지 알아두는 것은 필수적**이다.
  1. CPU 사용량 초과
  2. JVM 메모리 사용량 초과
  3. 커넥션 풀 고갈
  4. 에러 로그 급증

## 2025-08-02 Sat
### 그라파나를 활용한 CPU 지표 확인
* 에를 들어 애플리케이션에 CPU 관련한 문제가 발생한 상황을 가정하기 위해서는 다음과 같은 CPU 부하를 주는 코드를 작성해볼 수 있다.
  * Postman 등의 도구를 통해 해당 API를 호출할 경우, for 문 처리를 위해 즉각적인 응답이 반환되지 않고 다소 시간이 걸리는 것을 확인할 수 있다.
```kotlin
@Slf4j
@RestController
class TestController {

    @GetMapping("cpu")
    fun cpu(): String {
        log.info("cpu!")
        var value: Long = 0
        for (i in 1..1000000000000L) {
            value++
        }
      
        return "result: $value"
    }
}
```
* 이를 통해 그라파나 대시보드로부터 CPU 사용량 증가 지표 정보를 확인할 수 있으나, 프로메테우스의 지표 수집은 동기적이지 않으므로 약간의 시간이 필요하다.