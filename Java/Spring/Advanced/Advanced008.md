# Spring Advanced
## 2025-01-01 Wed
### this와 target의 차이
* `this`와 `target` 포인트컷 지시자는 각각 다음과 같은 형태로 사용이 가능하다.
```java
class Temp {
    @Before("allMember() && this(obj)")
    public void thisArgs(JoinPoint joinPoint, MyService obj) {
        System.out.println("[this] obj=" + obj.getClass());
    }
    @Before("allMember() && target(obj)")
    public void targetArgs(JoinPoint joinPoint, MyService obj) {
      System.out.println("[target] obj=" + obj.getClass());
    }    
}
```
* 이 때, 로그의 결과를 토대로 확인했을 경우 각각의 포인트컷 지시자는 다음과 같은 의미를 갖는 것을 이해할 수 있다.
  * `this`: 스프링 컨테이너에 등록된 실제 인스턴스로, 일반적으로 프록시 객체가 된다.
  * `target`: 어드바이스 적용 대상 객체로, 이 경우 프록시가 내부적으로 실제로 호출하게 될 실제 대상 객체인 `MyService` 클래스 자체가 된다.

## 2025-01-02 Thu
### this와 target 주의사항
* `this`와 `target`은 모두 `*`과 같은 패턴을 사용할 수 없는 대신 적용 대상의 타입을 명확히 지정해야 하며, 부모 타입을 허용하는 특징이 있다.
* 또한, 상술한 바와 같이 두 지시자는 각각 스프링 컨테이너에 등록된 스프링 AOP 프록시 객체 또는 실제 대상 객체를 대상으로 동작하는 조인 포인트에 해당한다.
  * **프록시가 등록되는 경우 프록시 인스턴스가 빈으로 등록되는 반면, 프록시 대상 객체는 엄밀히 말해 빈 객체가 아니라는 점에서 차이**가 있다.
  * 다시 말해 `this`는 스프링 빈에 등록된 프록시 객체를 대상으로 포인트컷을 매칭하는 반면, `target`은 프록시가 참조하는 실제 객체를 대상으로 동작한다.
* 이 때, `this` 표현식의 경우 구체 클래스 기반 프록시를 대상으로 했다면 CGLIB과 JDK 동적 프록시 중 무엇을 사용하느냐에 따라 다른 결과가 나올 수 있다.
  * 반면, `this`와 `target` 지시자는 그 동작 방식을 이해하기 다소 어렵지만 실무적인 관점에서 자주 사용되지는 않는다는 특징을 갖는다.
  * 이러한 특징으로 인해 두 지시자는 각각 단독으로 사용되기보다는 파라미터 바인딩에 활용되는 경우가 많다.

## 2025-01-03 Fri
### AOP 실무 예시 I
* 예를 들어 임의의 메소드에 특정한 어노테이션이 명시된 경우, 해당 메소드의 인자를 모두 로그로 출력하는 부가 기능은 다음과 같이 AOP로 구현할 수 있다.
```kotlin
/* 아래와 같은 어노테이션을 별도로 정의했다고 가정했을 때,
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class Trace
*/

// 로그를 남기는 부가 기능은 아래와 같은 Aspect로 정의할 수 있다.
@Aspect
class TraceAspect {
  companion object {
    private val logger = LoggerFactory.getLogger(TraceAspect::class.java)
  }

  @Before("@annotation(ga.injuk.aop.exam.annotation.Trace)")
  fun log(joinPoint: JoinPoint) {
    val args = joinPoint.args
    logger.info("[trace] {} args = {}", joinPoint.signature, args)
  }
}
```

## 2025-01-04 Sat
### AOP 실무 예시 II
* 예를 들어 임의의 메소드에 특정한 어노테이션이 명시된 경우, 해당 메소드에서 예외가 발생한 경우 재시도하는 부가 기능 역시 다음과 같이 AOP로 구현할 수 있다.
```kotlin
/* 아래와 같이 value에 기본 값을 갖는 어노테이션을 별도로 정의했다고 가정했을 때,
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class Retry(val value: Int = 3)
*/

// 재시도 부가 기능은 아래와 같은 Aspect로 정의할 수 있다.
@Aspect
class RetryAspect {
  companion object {
    private val logger = LoggerFactory.getLogger(RetryAspect::class.java)
  }

  @Around("@annotation(retry)") // retry 메소드의 두 번째 인자로 전달된 어노테이션 인자를 @Around 어노테이션에 활용한다.
  fun retry(joinPoint: ProceedingJoinPoint, retry: Retry): Any? {
    logger.info("[retry] {} retryAnnotation = {}", joinPoint.signature, retry)

    val maxRetry = retry.value

    lateinit var exHolder: Exception // 마지막에 예외를 던져주기 위해 예외를 일시적으로 저장하는 exHolder를 사용한다.
    for (i in 1..maxRetry) {
      try {
        logger.info("[retry] try count={}/{}", i, maxRetry)
        return joinPoint.proceed()
      } catch (e: Exception) {
        exHolder = e
      }
    }

    throw exHolder // 재시도 횟수 이내에서 성공하지 못한 경우 일시적으로 저장해둔 예외를 던진다.
  }
}
```

## 2025-01-05 Sun
### AOP 실무 주의 사항 - 프록시와 내부 호출
```
> 프록시 대상 객체 내부에서 자신에 대한 메소드 호출이 발생한 경우, 프록시를 거치지 않고 대상 객체가 직접 호출되므로 어드바이스를 적용할 수 없다.
```
* **스프링은 AOP에 프록시를 활용하므로, AOP를 적용하는 과정에서 언제나 프록시를 경유한 대상 객체의 호출이 발생**한다.
  * 이를 통해 프록시가 우선 어드바이스를 호출하고, 그 이후에 대상 객체를 호출하여 부가 기능을 적용할 수 있다.
  * 바꿔 말해, 프록시 없이 대상 객체를 호출할 경우 AOP는 적용되지 않으므로 어드바이스 역시 호출되지 않게 된다.
* **AOP를 적용할 경우, 스프링은 의존성 주입 과정에서 대상 객체 대신 프록시 객체를 스프링 빈으로 등록하고 주입**하게 된다.
  * 즉, **프록시 객체가 주입되었기에 대상 객체를 직접 호출하는 문제는 일반적인 경우에 발생하지 않아야 한다**.
* 그러나 **대상 객체 내부에서 메소드 호출이 발생한 경우, 프록시를 거치지 않고 대상 객체를 직접 호출하는 문제가 발생할 수 있다**.
  * 이러한 문제는 실무에서 반드시 한 번은 마주치는 문제이기에 이를 반드시 기억해둘 필요가 있다.

## 2025-01-06 Mon
### 프록시 방식을 활용하지 않는 AspectJ
* `AspectJ` 프레임워크를 그대로 사용하는 경우, 부가 기능과 관련된 로직은 바이트 코드 차원에서 코드에 직접 삽입되므로 상술한 문제는 발생하지 않는다.
  * 즉, 프록시를 활용하지 않기에 내부 호출과 무관하게 AOP를 적용할 수 있다.
* 그러나 **`AspectJ`는 설정이 복잡하거나 별도의 JVM 옵션을 설정해야하는 등 번거로움이 있으며, 상술한 문제 역시 `AspectJ` 없이 대응이 가능**하다.
  * 이러한 이유에서 `AspectJ` 프레임워크의 경우 실무에서는 잘 사용되지 않는다.

## 2025-01-07 Tue
### 프록시와 내부 호출 해결하기 - 자기 자신을 주입하기
* 이러한 **내부 호출 문제를 해결하는 가장 간단한 방법은 자기 자신에 대한 의존성을 `setter`를 통해 주입받는 것**이다.
  * 이 때, **생성자 주입을 통해 자기 자신에 대한 의존성을 주입받을 경우 순환 의존성 문제가 발생함에 주의**해야 한다.
  * 반면, **`setter`의 경우 스프링은 빈 객체의 인스턴스화를 마치는 단계와 `setter`를 활용한 주입 단계가 분리되어 있기에 문제가 발생하지 않는다**.
  * 다시 말해, `setter`에서 요청하는 의존성이 자기 자신과 동일한 객체일지라도 스프링에 의해 생성된 프록시 객체가 주입된다.
* 또한, **자기 자신에 대한 내부 호출시 명시적 또는 암시적 `this` 참조 대신 주입 받은 의존성에 대한 호출로 변경하는 것으로 해당 문제를 해결**할 수 있다.

## 2025-01-08 Wed
### 프록시와 내부 호출 해결하기 - 지연 조회 활용하기
* 상술한 `setter` 주입 방식은 생성자 주입 방식을 권장하는 최근 스프링 애플리케이션과 맞지 않는 부분이 있으므로, 대안으로 지연 조회를 고려할 수 있다.
  * 이는 **말 그대로 스프링 빈을 지연 조회하는 방식에 대항하며, `ObjectProvider` 또는 `ApplicationContext`를 활용**하게 된다.
* 예를 들어 생성자로 `ApplicationContext`를 주입 받고, 부가 기능 적용 대상은 `ac.getBean([대상클래스명].class);` 형태로 지연 조회할 수 있다.
  * 그러나 `ApplicationContext`는 필요 이상으로 거대한 의존성에 해당하므로, 하술할 `ObjectProvider`를 활용하는 것이 권장된다.
* 또는 생성자로 `ObjectProvider<[대상클래스]>`를 주입 받고, `provider.getObject();` 형태로 지연 조회할 수도 있다.
  * **이 방식의 경우, 스프링 컨테이너에 등록된 싱글톤 객체를 그대로 조회하는 방식으로 정확히 빈 하나를 조회하는 의도에 특화된 기능으로 이해**할 수 있다.
* 상술한 두 방식 모두 자기 자신에 대한 의존성을 주입 받는 것은 아니므로 순환 의존성 문제는 발생하지 않는다.

## 2025-01-09 Thu
### 프록시와 내부 호출 해결하기 - 구조 자체를 변경하기
* **상술한 두 방식은 모두 자기 자신을 직접 주입하거나, `Provider`를 활용하여 의존성을 조회하는 다소 어색한 형태의 객체를 강요**하게 된다.
* 이로 미루어보았을 때, **가장 좋은 방식은 애초에 이러한 내부 호출이 발생하지 않도록 구조 자체를 변경하는 것이며 실제로도 해당 방법이 가장 바람직**하다.
  * 예를 들어, 내부 호출을 유발하는 로직을 별도의 헬퍼 클래스로 추출하고 이를 원본 클래스에서 주입 받는 방식을 고려해볼 수 있다.

## 2025-01-10 Fri
### 프록시와 내부 호출 - 참고
```
> AOP가 의도한대로 잘 적용되지 않는다면 꼭 내부 호출 문제를 의심해보자.
```
* AOP는 주로 트랜잭션이나 주요 컴포넌트의 로깅 등 굵직한 기능에 사용되며, 다시 말해 주로 `public`으로 공게된 메소드에 대해서만 적용하는 경우가 많다.
  * 즉, 가독성 및 유지보수성 향상을 위해 추출된 `private` 메소드처럼 작은 단위에까지 적용을 의도하지 않는다.
  * 따라서 단지 AOP만을 적용하기 위해 `private` 메소드의 가시성을 변경하는 일 자체가 자주 발생하지 않는다.
* 그러나 **재사용성을 위해 임의의 객체에 포함된 `public` 메소드가 자신의 또 다른 `public` 메소드를 호출하는 경우에는 내부 호출 문제가 발생하기 쉽다**.
  * 이러한 **내부 호출 문제는 모든 스프링 개발자들이 실무에서 AOP를 사용할 때 꼭 한 번 쯤은 겪는 현상이기에 특히 주의를 기울일 필요**가 있다.

## 2025-01-11 Sat
### ProxyFactory와 proxyTargetClass 옵션
* AOP 프록시를 위해 사용되는 두 기술인 JDK 동적 프록시 방식과 CGLIB 방식은 각각 장단점을 갖는다.
  * 예를 들어, JDK 동적 프록시는 인터페이스가 필수적인 반면 CGLIB는 구체 클래스를 기반으로 프록시를 생성한다.
* 반면, **구체 클래스에 대해서는 CGLIB만을 사용할 수 있으나 인터페이스에 대해서는 두 기술을 모두 사용할 수 있다는 특징**이 있다.
* 때문에 스프링이 프록시를 위해 제공하는 `ProxyFactory`에 `proxyTargetClass` 옵션을 적절히 사용할 경우 두 기술 중 하나를 선택할 수 있다.
  * 예를 들어, 해당 옵션에 `true`를 할당할 경우 CGLIB을 사용하여 구체 클래스 기반의 프록시를 생성한다.
  * 반면, **해당 옵션과 무관하게 인터페이스가 없는 구체 클래스에 대해서는 JDK 동적 프록시를 적용조차 할 수 없으므로 CGLIB만을 사용**한다.
