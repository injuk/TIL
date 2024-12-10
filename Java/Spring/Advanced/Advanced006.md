# Spring Advanced
## 2024-11-27 Wed
### 스프링 부트 프로젝트에 AOP 기본 설정하기
* 스프링 부트의 경우, AOP를 활용하기 위해서는 아래와 같은 의존성을을 `build.gradle`에 작성해주어야 한다.
  * `spring-data-jpa` 등의 일반적인 의존성을 사용할 경우 아래 의존성 역시 함께 포함되지만, 그렇지 않은 경우에는 직접 명시해야 한다.
```kotlin
dependencies {
  implementation("org.springframework.boot:spring-boot-starter-aop")
}
```
* 또한, `@Aspect`와 같은 어노테이션을 사용하기 위해서는 `@EnableAspectJAutoProxy`와 같은 스프링 설정이 전제되어야 한다.
  * 그러나, 스프링 부트를 사용할 경우 이러한 설정은 자동으로 추가된다.

## 2024-11-28 Thu
### 간단한 AOP 구현하기
* 스프링 AOP를 구현하는 가장 일반적인 방법은 `@Aspect`를 사용하는 것이며, 예를 들어 다음과 같은 코드를 작성해볼 수 있다.
```kotlin
@Aspect
class AspectV1 {
  @Around("execution(* ga.injuk.aop.order..*(..))")
  fun doLog(joinPoint: ProceedingJoinPoint): Any {
    println("${joinPoint.signature}") // join point 시그니쳐 확인

    return joinPoint.proceed()
  }
}
```
* `@Around` 어노테이션의 경우, 인자로 전달된 값에 대한 포인트컷 역할을 수행한다.
  * 이 때, `execution...`으로 시작하는 인자는 `ga.injuk.aop.order` 패키지 및 그 하위 모든 패키지를 지칭하는 `AspectJ` 포인트컷 표현식이다.
  * 또한, **해당 어노테이션이 명시된 메소드인 `doLog`는 일종의 어드바이스로서 기능**하게 된다.
* 이를 통해 `ga.injuk.aop.order` 및 그 하위 패키지에 위치한 모든 클래스가 AOP 적용 대상이 될 수 있다.
  * 또한, **프록시 방식의 AOP를 사용하는 스프링 특성상 프록시를 통하는 메소드만 AOP의 적용 대상**이 된다.

## 2024-11-29 Fri
### 스프링 부트와 AspectJ의 관계
* `@Aspect`와 같이 `org.aspectj` 패키지에 포함되는 기능들은 모두 `aspectjweaver.jar` 라이브러리로부터 제공된다.
* 반면, **스프링의 경우 `AspectJ`의 어노테이션이나 그와 관련된 인터페이스만 사용하는 것으로, 실제 `AspectJ`가 제공하는 위버를 사용하지는 않는다**.
  * 상술했듯, 스프링은 프록시 방식의 AOP를 사용한다.

## 2024-11-30 Sat
### @Aspect 대상을 빈으로 등록하기
* 또한, **`@Aspect`는 어디까지나 어노테이션일 뿐 컴포넌트 스캔의 대상인 것은 아니다**.
  * 이로 인해 `@Aspect` 어노테이션이 할당된 클래스를 AOP로 활용하고자 하는 경우에는 반드시 스프링 빈으로 등록할 필요가 있다.
  * 스프링 빈 등록 방법은 크게 `@Bean` 또는 컴포넌트 스캔 및 임의의 설정 파일을 명시적으로 등록하기 위한 `@Import` 등이 있다.

## 2024-12-01 Sun
### 포인트컷 분리하기
* `@Aspect`의 포인트컷은 아래와 같은 형태로 별도의 메소드로 분리하는 것이 가능하다.
```kotlin
@Aspect
class AspectV2 {

  @Pointcut("execution(* ga.injuk.aop.order..*(..))")
  private fun allOrder() {} // pointcut signature

  @Around("allOrder()")
  fun doLog(joinPoint: ProceedingJoinPoint): Any {
    println("${joinPoint.signature}") // join point 시그니쳐 확인

    val result =  joinPoint.proceed()

    return result
  }
}
```
* 상술한 바와 같이 **`@Pointcut`의 인자에 포인트컷 표현식을 명시하며, 이렇듯 메소드 명과 인자를 합쳐 포인트컷 시그니쳐라고 지칭**한다.
  * 이 때, **중요한 것은 메소드의 반환 타입이 `void`여야 하며 메소드 본문은 비어 있어야 한다는 점**이다.
  * 또한, 메소드의 접근 제어자는 일반 메소드와 마찬가지로 메소드 사용 범위에 따라 `private` 또는 `public`을 명시한다.
* **포인트컷을 분리할 경우 `AspectV1`과 동일한 기능을 수행하지만, 여러 어드바이스로부터 하나의 포인트컷 표현식을 공유할 수 있다는 이점**을 얻는다.

## 2024-12-02 Mon
### 복합적인 포인트컷 적용하기
* `@Around` 어노테이션의 경우, 인자에 다음과 같이 여러 포인트컷을 참조하는 것으로 복합적인 포인트컷의 적용이 가능하다.
```kotlin
@Aspect
class AspectV3 {

  @Pointcut("execution(* ga.injuk.aop.order..*(..))")
  fun allOrder() {}

  @Pointcut("execution(* *..*Service.*(..))")
  fun allService() {}

  @Around("allOrder()")
  fun doLog(joinPoint: ProceedingJoinPoint): Any {
    println("${joinPoint.signature}") // join point 시그니쳐 확인

    return joinPoint.proceed()
  }

  @Around("allOrder() && allService()") // allOrder와 allService가 갖는 각 포인트컷 조건을 모두 충족하는 경우에만 어드바이스를 적용한다.
  fun doTransaction(joinPoint: ProceedingJoinPoint): Any {
    try {
      println("트랜잭션 시작! ${joinPoint.signature}")
      val result = joinPoint.proceed()
      println("트랜잭션 커밋! ${joinPoint.signature}")

      return result
    } catch (e: Exception) {
      println("트랜잭션 롤백... ${joinPoint.signature}")
      throw e
    } finally {
      println("리소스 릴리즈... ${joinPoint.signature}")
    }
  }
}
```
* 상술한 예시의 경우, `doTransaction()` 메소드는 `allOrder()`와 `allService()` 포인트컷의 모든 조건을 만족하는 경우에만 어드바이스를 실행한다.
  * 이러한 **포인트컷의 조합은 크게 `&&`, `||`, `!` 세가지 연산자를 활용하여 표현**하게 된다.

## 2024-12-03 Tue
### 포인트컷 참조하기
* **애플리케이션의 여러 위치에서 공통으로 사용되는 포인트컷이 있는 경우, 아래와 같이 이러한 포인트컷을 모두 별도의 공용 외부 클래스로 추출해도 무방**하다.
  * 반면, 이 경우에는 포인트컷 메소드의 접근 제어자가 반드시 `public`이어야 한다.
```kotlin
class Pointcuts {
    @Pointcut("execution(* ga.injuk.aop.order..*(..))")
    fun allOrder() {}

    @Pointcut("execution(* *..*Service.*(..))")
    fun allService() {}

    @Pointcut("allOrder() && allService()")
    fun orderAndService() {}
}
```
* 이 경우, 이렇게 **추출된 포인트컷을 참조하는 클래스는 패키지를 포함하는 클래스의 메소드 호출 형태를 활용**해야 한다.
```kotlin
@Aspect
class AspectV4Pointcut {
    // 패키지를 포함한 클래스의 메소드 호출 형태로 명시한다.
    @Around("ga.injuk.aop.order.aop.Pointcuts.allOrder()")
    fun doLog(joinPoint: ProceedingJoinPoint): Any {
        println("[doLog] ${joinPoint.signature}")

        return joinPoint.proceed()
    }

    @Around("ga.injuk.aop.order.aop.Pointcuts.orderAndService()")
    fun doTransaction(joinPoint: ProceedingJoinPoint): Any {
        try {
            println("[doTransaction] 트랜잭션 시작! ${joinPoint.signature}")
            val result = joinPoint.proceed()
            println("[doTransaction] 트랜잭션 커밋! ${joinPoint.signature}")

            return result
        } catch (e: Exception) {
            println("[doTransaction] 트랜잭션 롤백... ${joinPoint.signature}")
            throw e
        } finally {
            println("[doTransaction] 리소스 릴리즈... ${joinPoint.signature}")
        }
    }
}
```

## 2024-12-04 Wed
### 어드바이스의 적용 순서 조절하기
* 어드바이스는 기본적으로 순서를 보장하지 않으며, 순서를 명시하고자 하는 경우에는 `@Aspect` 단위마다 `@Order` 어노테이션을 사용해야 한다.
  * 즉, **`@Aspect`라는 애스팩트 클래스 단위마다 하나씩 적용되므로 단일 애스팩트에 여러 어드바이스가 적용된 경우에는 순서를 명시할 수 없다**.
* **여러 어드바이스에 대해 명확한 순서를 조절하고자 하는 경우, 각각의 어드바이스를 아래와 같이 별도의 애스팩트 클래스로 분리해줄 필요**가 있다.
  * 이 역시도 **`@Order` 어노테이션을 활용하는 방법에 해당하며, 해당 어노테이션의 인자로 전달된 수가 낮은 애스팩트가 더 높은 우선 순위**를 갖는다.
```kotlin
class AspectV5 {

    @Aspect
    @Order(2)
    class LogAspect {
        @Around("ga.injuk.aop.order.aop.Pointcuts.allOrder()")
        fun doLog(joinPoint: ProceedingJoinPoint): Any {
            println("[doLog] ${joinPoint.signature}")

            return joinPoint.proceed()
        }
    }

    @Aspect
    @Order(1)
    class TransactionAspect {
        @Around("ga.injuk.aop.order.aop.Pointcuts.orderAndService()")
        fun doTransaction(joinPoint: ProceedingJoinPoint): Any {
            try {
                println("[doTransaction] 트랜잭션 시작! ${joinPoint.signature}")
                val result = joinPoint.proceed()
                println("[doTransaction] 트랜잭션 커밋! ${joinPoint.signature}")

                return result
            } catch (e: Exception) {
                println("[doTransaction] 트랜잭션 롤백... ${joinPoint.signature}")
                throw e
            } finally {
                println("[doTransaction] 리소스 릴리즈... ${joinPoint.signature}")
            }
        }
    }
}
```

## 2024-12-05 Thu
### 어드바이스의 종류
* 상술한 `@Around`와 같은 어드바이스를 포함해서, 스프링 AOP는 다음과 같은 다양한 어드바이스를 제공한다.
  * `@Around`: 메소드 호출 전후에 수행된다.
  * `@Before`: 조인 포인트 실행 이전에 수행된다.
  * `@AfterReturning`: 조인 포인트가 정상적으로 처리된 후에 수행된다.
  * `@AfterThrowing`: 메소드가 예외를 던진 경우에 대해 실행된다.
  * `@After`: 조인 포인트가 정상적으로 실행되거나, 예외를 던진 것과 관계 없이 실행된다.
* 이 때, **`@Around` 어드바이스는 조인 포인트의 실행 여부를 선택하거나 반환 값을 변환하고, 예외를 번역할 수 있는 등 가장 강력**한 축에 속한다.
  * 극단적으로 말해서, 실무에서는 `@Around` 어드바이스만으로도 대부분의 상황을 처리할 수 있다.
* `@After` 어드바이스의 경우, 마치 `try-catch` 문에서 사용되는 `finally` 절처럼 동작하는 것으로 이해할 수 있다.

## 2024-12-06 Fri
### @Around와 그 외 어드바이스의 차이
* 예를 들어 아래와 같은 애스팩트 클래스의 경우, 실제로 작성된 어드바이스는 `@Around` 하나에 해당한다.
  * 반면, 용도에 따라서는 표시된 부분만을 담당하는 별도의 어드바이스를 사용할 수 있다.
```kotlin
@Aspect
class MyAspect {
    @Around("ga.injuk.aop.order.aop.Pointcuts.orderAndService()")
    fun doTransaction(joinPoint: ProceedingJoinPoint): Any {
        try {
            // @Before 어드바이스로 아래 한 줄을 처리할 수 있다.
            println("[doTransaction] 트랜잭션 시작! ${joinPoint.signature}")
            val result = joinPoint.proceed()
          
            // @AfterReturning 어드바이스로 아래 두 줄을 처리할 수 있다.
            println("[doTransaction] 트랜잭션 커밋! ${joinPoint.signature}")

            return result
        } catch (e: Exception) {
            // @AfterThrowing 어드바이스로 아래 두 줄을 처리할 수 있다.
            println("[doTransaction] 트랜잭션 롤백... ${joinPoint.signature}")
            throw e
        } finally {
            // @After 어드바이스로 아래 한 줄을 처리할 수 있다.
            println("[doTransaction] 리소스 릴리즈... ${joinPoint.signature}")
        }
    }
}
```
* 즉, **기본적으로 `@Around`로 모든 것을 처리할 수 있으며 나머지 어드바이스들은 `@Around`를 구성하는 기능 일부를 담당하는 조각**과도 같다.

## 2024-12-07 Sat
### 어드바이스 타입 별 참고 정보 획득하기
* `@Around`를 제외한 모든 어드바이스는 메소드의 첫 번째 인자에 `JoinPoint`를 사용하는 것으로 추가 정보를 획득할 수 있다.
* 이 때, `JoinPoint` 클래스는 추가 정보 획득을 위해 다음과 같은 여러 유용한 메소드를 제공한다.
  * `getArgs()`: 어드바이스 대상 메소드에 전달된 인자를 반환한다.
  * `getThis()`: 어드바이스를 적용하기 위해 생성된 프록시 객체를 반환한다.
  * `getTarget()`: 어드바이스 대상 객체를 반환한다.
  * `getSignature()`: 어드바이스 대상 메소드에 대한 설명을 반환한다. 
  * `toString()`: 어드바이스 방법에 대한 유용한 설명을 의미하는 문자열을 반환한다.
* 반면, `@Around` 어드바이스의 경우 `JoinPoint`의 하위 클래스인 `ProceedingJoinPoint`를 사용해야 한다.
  * 이 때, `ProceedingJoinPoint` 클래스는 다음 어드바이스 또는 대상 객체를 호출하는 `proceed()` 메소드를 추가적으로 제공한다.

## 2024-12-08 Sun
### @Before 어드바이스란?
* `@Before` 어드바이스는 조인 포인트의 실행 전에 수행되며, `@Around` 어드바이스와 달리 작업 흐름을 변경할 수는 없다.
* 예를 들어, `@Around` 어드바이스는 `ProceedingJoinPoint.proceed()` 메소드를 통해 다음 대상을 호출한다.
  * 바꿔 말해 이를 호출하지 않을 경우, 다음 대상 자체가 호출되지 않아 의도한 동작을 처리할 수 없게 된다.
* 반면, **`@Before` 어드바이스는 `ProceedingJoinPoint.proceed()` 메소드 자체를 사용하지 않으며 메소드 종료시 자동으로 다음 대상을 호출**한다.
  * 물론, 처리 과정에서 예외가 발생할 경우 다음 코드는 호출되지 않는다.

## 2024-12-09 Mon
### @AfterReturning 어드바이스란?
* `@AfterReturning` 어드바이스는 메소드 실행이 정상적으로 처리되어 성공적으로 반환된 경우에만 수행된다.
* 이 때, 해당 어드바이스는 `@AfterReturning` 어노테이션에 다음과 같은 두 인자를 받아 동작한다.
  * `value`: 포인트컷 메소드 정보를 명시한다.
  * `returning`: 어드바이스 메소드의 매개변수 이름으로, 해당 인자에 명시된 매개변수에 해당하는 타입의 값을 반환하는 메소드를 대상으로만 실행한다.
* 다시 말해, `@AfterReturning` 어드바이스는 `returning`에 명시된 인자 이름인 `result`의 타입인 `Object`를 반환하는 메소드에 대해서만 수행된다.
  * 이 때, **`Object`와 같이 임의의 클래스에 대한 공통 부모 역할을 할 수 있는 타입인 경우 그 자식 타입을 반환하는 모든 메소드에 대해 적용**된다.
* 또한, `@Around` 어드바이스와 달리 반환되는 객체를 변경할 수는 없기에 반환되는 객체를 변경하고자 하는 경우에는 `@Around` 어드바이스를 사용해야 한다.
  * 반면, **반환되는 객체의 상태를 변경하는 등의 조작은 가능**하다.

## 2024-12-10 Tue
### @AfterThrowing 어드바이스란?
* `@AfterThrowing` 어드바이스는 메소드 실행 도중 예외가 발생하여 종료되는 경우에만 수행된다.
* 이 때, 해당 어드바이스는 `@AfterThrowing` 어노테이션에 다음과 같은 두 인자를 받아 동작한다.
  * `value`: 포인트컷 메소드 정보를 명시한다.
  * `throwing`: 어드바이스 메소드의 매개변수 이름으로, 해당 인자에 명시된 매개변수에 해당하는 타입의 예외에 부합하는 예외를 대상으로만 실행한다.
* **`throwing`의 경우, `@AfterReturning`의 경우와 마찬가지로 부모 타입의 예외를 명시한 경우 모든 자식 타입의 예외에 대해 적용**될 수 있다.