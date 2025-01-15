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

## 2025-01-12 Sun
### JDK 동적 프록시의 한계점
```
> JDK 동적 프록시는 대상 객체인 구체 클래스로 타입 캐스팅이 불가능하다.
> 반면, CGLIB 프록시는 대상 객체인 구체 클래스로 타입 캐스팅이 가능하다.
```
* **JDK 동적 프록시의 경우 인터페이스를 기반으로 프록시를 생성하므로, 구체 클래스로 타입캐스팅을 할 수 없는 명확한 한계**를 갖는다.
* 예를 들어, `MemberService` 인터페이스를 구현하는 `MemberServiceImpl`에 대해 프록시를 생성하는 경우, 해당 현상을 아래와 같이 확인할 수 있다.
```kotlin
class ProxyCastingTest {
  @Test
  fun jdkProxy() {
    val target: MemberServiceImpl = MemberServiceImpl()

    val proxyFactory: ProxyFactory = ProxyFactory(target)
      .apply { isProxyTargetClass = false } // JDK 동적 프록시 사용을 강제. 생략하더라도 기본 값은 false!

    // 프록시를 인터페이스로 캐스팅할 수는 있다.
    val memberServiceProxy = proxyFactory.proxy as MemberService
    println(memberServiceProxy) // ga.injuk.aop.member.MemberServiceImpl@68df9280

    // JDK 동적 프록시를 구현 클래스로 캐스팅할 수는 없다.
    assertThrows(ClassCastException::class.java) {
      proxyFactory.proxy as MemberServiceImpl // java.lang.ClassCastException: class jdk.proxy3.$Proxy16 cannot be cast to class ga.injuk.aop.member.MemberServiceImpl
    }
  }
}
```
* 이는 **JDK 동적 프록시가 내부적으로 `MemberService` 인터페이스를 구현한 프록시를 사용하기 때문**이다.
  * 즉, 또 다른 구현체인 `MemberServiceImpl`과는 직접적인 관계가 없기에 캐스팅이 불가능하다.
* 반면, 아래와 같이 **`CGLIB` 프록시 사용을 강제할 경우 `MemberServiceImpl` 자체에 대한 프록시가 생성되므로 해당 문제는 발생하지 않는다**.
  * 즉, 이 경우 `MemberService` <- `MemberServiceImpl` <- `CGLIB Proxy` 와 같은 일종의 상속 계층도가 구성되는 것으로 이해할 수 있다.
```kotlin
class ProxyCastingTest {
    @Test
    fun cglibProxy() {
        val target: MemberServiceImpl = MemberServiceImpl()

        val proxyFactory: ProxyFactory = ProxyFactory(target)
            .apply { isProxyTargetClass = true } // CGLIB 프록시 사용을 강제

        // 프록시를 인터페이스로 캐스팅할 수 있다.
        val memberServiceProxy = proxyFactory.proxy as MemberService
        println(memberServiceProxy)

        // CGLIB 프록시를 구현 클래스로 캐스팅하더라도 성공한다.
        val castingMemberServiceImpl = proxyFactory.proxy as MemberServiceImpl
        println(castingMemberServiceImpl)
    }
}
```
* 실제로는 **프록시를 캐스팅할 일이 없을 것처럼 느껴질 수 있으나, 상술한 모든 내용은 하술할 의존 관계 주입과 관련된 문제를 유발**할 수 있다.

## 2025-01-13 Mon
### AOP 실무 주의 사항 - 프록시와 의존 관계 주입
```
> JDK 동적 프록시를 사용하는 경우, 항상 인터페이스 기반의 의존성 주입만을 사용해야 하며 구체 클래스 기반의 주입은 실패할 수 있다.
```
* **스프링 부트 애플리케이션이 JDK 동적 프록시를 기본적으로 사용하고 있다고 가정했을 때, 구체 클래스를 기반으로 한 의존성 주입에 실패할 수 있다**.
  * 예를 들어, **`MemberService`를 구현하는 `MemberServiceImpl`에 AOP를 적용했을 때 `MemberServiceImpl` 타입의 의존성 주입은 실패**한다.
  * 이는 상술한 내용과 같이 **JDK 동적 프록시가 인터페이스를 구현하는 별도의 구현체를 생성하기에 각 구현체 간의 타입 캐스팅이 불가능한 문제에 해당**한다.
  * 물론, 의존성 주입 자체를 `MemberService` 타입을 사용하는 경우에는 JDK 동적 프록시를 사용하더라도 상술한 문제가 발생하지 않는다.
* 반면, **JDK 동적 프록시 대신 CGLIB를 사용할 경우 구현체인 `MemberServiceImpl`을 상속하는 프록시가 생성되므로 상술한 문제는 발생하지 않는다**.
* 하지만 인터페이스를 기반으로 한 클래스를 의존성 주입 받는 경우에는 애초에 인터페이스 기반의 DI를 활용하게 되므로, 상술한 문제는 자주 발생하지 않는다.
  * 그럼에도 여러 이유에서 AOP 프록시가 적용된 구체 클래스를 주입받아야 하는 경우가 발생할 수 있으며, 이 경우에는 CGLIB을 활용하는 것이 바람직하다.

## 2025-01-14 Tue
### CGLIB의 한계점
```
> 인터페이스가 아닌 대상 클래스의 구체 타입을 주입할 때 문제가 발생하는 JDK 동적 프록시와 달리, CGLIB은 생성자와 관련된 문제가 발생한다. 
```
* CGLIB는 구체 클래스를 상속받는 AOP 프록시를 생성하므로, 다음과 같은 문제를 수반할 수 있다.
  1. 대상 클래스에 기본 생성자를 강제하는 문제
  2. 생성자가 2번 호출되는 문제
  3. `final` 클래스 및 메소드에 대해 적용 불가능한 문제
* 우선, **Java의 특성 상 상속의 경우 자식 클래스 생성자 호출시 해당 생성자 내부에서 부모 클래스의 생성자 호출이 전제**되어야 한다.
  * 이는 피할 수 없는 Java의 문법으로, 자식 클래스의 생성자에 부모 클래스의 생성자가 생략된 경우 첫 줄에 `super();` 호출문이 자동으로 삽입된다.
* 또한, 같은 이유에서 **CGLIB에 의해 생성되는 프록시의 생성자는 부모 클래스의 기본 생성자를 호출하게 되므로 기본 생성자의 정의 역시 필수**적이다.
  * 이는 프록시 클래스의 직계 부모 클래스가 대상 클래스가 되기 때문으로, 결국 자식 클래스의 생성자에서 부모 클래스의 생성자를 호출하는 것이 원인이다.
  * 무엇보다 프록시 클래스는 개발자가 직접 정의하지 않으므로 생성자를 정의할 수도 없기에 이러한 문제가 발생할 수 밖에 없다.
* **CGLIB에 의해 생성된 프록시는 생성자 호출시 부모인 대상 클래스의 생성자를 호출되므로, 결국 대상 클래스의 생성자는 2번 호출**된다.
  * **2번이란, 대상 클래스가 실제로 생성될 때와 프록시 인스턴스가 생성되는 과정에서 부모 클래스의 생성자를 호출하는 때 각각을 의미**한다.
* 마지막으로, `final` 키워드가 할당된 경우 애초에 상속 또는 재정의가 불가능하므로 상속을 전제로 하는 CGLIB가 정상 동작하지 않게 된다.
  * 예를 들어, 프록시 인스턴스가 생성되지 않거나 정상적으로 동작하지 않게 된다.
  * 그러나 실무의 관점에서 봤을 때, 일반적인 웹 애플리케이션 개발 과정에서는 `final` 키워드를 잘 사용하지 않기 때문에 이는 큰 문제가 되지 않는다.

## 2025-01-15 Wed
### 스프링의 해법
* 스프링은 AOP 프록시 생성을 보다 편리하게 처리할 방법을 오랜 기간 고민하고 적용해왔으며, 이러한 고민의 흔적은 크게 다음과 같이 정리할 수 있다.
  1. `스프링 3.2`: CGLIB 라이브러리를 스프링 내부에 패키징하여 별도의 라이브러리 추가 없이 프록시를 사용할 수 있도록 지원하였다.
  2. `스프링 4.0`: 생성자 호출 없이 객체를 생성할 수 있도록 지원하는 `objenesis`라는 이름의 라이브러리를 통해 CGLIB의 생성자 관련 문제를 해결하였다.
  3. `스프링 부트 2.0`: CGLIB을 기본값으로 설정하는 것으로 JDK 동적 프록시의 구체 클래스 기반 의존 관계 주입 문제를 해결하고자 하였다.
* 이러한 과정 속에 개선된 **스프링 부트는 AOP에 대해 기본적으로 `proxyTargetClass=true` 설정을 적용하며, 항상 CGLIB를 사용**한다.
  * 물론 해당 설정을 `false`로 변경하는 것으로 JDK 동적 프록시를 사용할 수 있는 가능성을 열어두기도 했다.
* 반면, **CGLIB를 사용함에 따라 `final` 키워드와 관련된 문제는 여전히 발생할 수 있으나 상술한 바와 같이 일반적인 경우 이는 큰 문제가 되지 않는다**. 