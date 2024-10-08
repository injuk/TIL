# Spring Advanced
## 2024-10-05 Sat
### JDK 동적 프록시와 CGLIB을 함께 활용하기 위한 고려사항
* 상술한 내용에 따라 인터페이스에 대해서는 자동으로 JDK 동적 프록시를 적용하고, 구체 클래스에 대해서는 CGLIB를 자동으로 적용하고 싶을 수 있다.
  * 또한, 같은 부가 기능을 위해 상술한 두 기술 각각에 대해 `InvocationHandler`와 `MethodInterceptor`를 중복으로 관리하고 싶지 않을 수 있다.
* 나아가 조건에 부합하는 경우에만 프록시 객체의 공통 로직을 적용하는 기능까지도 제공된다면 이상적이며, `ProxyFactory`는 이를 위해 제공되는 기능이다.

## 2024-10-06 Sun
### 프록시 팩토리란?
```
> JDK 동적 프록시와 CGLIB과 같이, 스프링은 유사한 기술들이 있을 때 이를 통합하여 일관성 있는 접근이 가능하도록 추상화된 기술을 제공한다.
```
* 프록시 팩토리는 동적 프록시 기술을 통합하여 쉽게 사용할 수 있도록 지원하는 기능으로, 대상 서버 클래스의 유형에 따라 서로 다른 프록시 기술을 적용한다.
  * 예를 들어 인터페이스 대상으로는 JDK 동적 프록시를 적용하고, 구체 클래스 대상으로는 CGLIB를 적용하는 형태로 동작하며 이러한 설정을 변경할 수도 있다.
* **개발자는 단지 프록시 팩토리에 프록시 객체 생성을 요청하면 되며, 프록시 팩토리는 대상 서버 클래스의 유형에 따라 적절한 프록시 기술을 선택**한다.
* 추가적으로 스프링은 부가 기능의 중복 작성을 방지하기 위해 `InvocationHandler`와 `MethodInterceptor`를 통합하는 `Advice` 개념을 도입한다.
  * 프록시 팩토리를 사용할 경우, 내부적으로 `Advice`를 호출하는 전용 핸들러와 인터셉터 개념을 활용하게 된다.
  * 예를 들어 `adviceMethodInterceptor`와 `adviceInvocationHandler`는 각각 개발자가 작성한 `Advice`를 호출하는 역할만을 담당한다.
* 스프링은 호출 대상 메소드의 이름이 특정한 조건에 부합할 경우에만 부가 기능을 적용할 수 있도록 `Pointcut` 개념까지 제공한다.

## 2024-10-07 Mon
### Advice 작성을 위한 MethodInterceptor 인터페이스
* `Advice`는 다음과 같은 `org.aopalliance.intercept` 패키지의 `MethodInterceptor`를 확장하여 작성할 수 있다.
```java
package org.aopalliance.intercept;

public interface MethodInterceptor extends Interceptor {
    Object invoke(MethodInvocation invocation) throws Throwable;
}
```
* 이 때, `invoke` 메소드의 인자인 `MethodInvocation`은 메소드를 호출하는 방법은 물론이고 현재 프록시 객체 인스턴스나 메소드 메타데이터가 포함된다.
  * 즉, 기존에는 파라미터로 전달하던 모든 데이터를 포함하는 요청 객체에 해당한다.
* 해당 인터페이스는 CGLIB의 `MethodInterceptor`와 동일한 이름을 가져 혼동되기 쉽지만, 스프링이 제공하는 AOP 모듈에 포함된다는 점에서 차이가 있다.
  * 또한, 상술한 `MethodInterceptor`는 `Advice`로부터 시작되는 상속 계층도에 위치한다.

## 2024-10-08 Tue
### MethodInterceptor 확장하기
* 상술한 `MethodInterceptor` 인터페이스는 다음과 같은 구체 클래스 형태로 작성이 가능하다.
```kotlin
class TimeAdvice : MethodInterceptor {
    companion object {
        private val logger = LoggerFactory.getLogger(TimeAdvice::class.java)
    }
    
    override fun invoke(invocation: MethodInvocation): Any? {
        logger.info("TimeAdvice invoked!")

        val startTime = System.currentTimeMillis()

        val result: Any? = invocation.proceed() // 알아서 target을 찾아 인자를 넘겨 실행한다.

        val endTime = System.currentTimeMillis()

        logger.info("TimeAdvice ended, time: {}ms", endTime - startTime)

        return result
    }
}
```
* 상술한 코드 중 `invocation.proceed()` 메소드 호출 과정에서 `target` 클래스가 호출되어 원본 결과가 반환된다.
  * 반면, **기존 예시들과는 달리 `target`은 `invocation` 내부에 포함되므로 그 처리 과정이 캡슐화**된다.
  * 이는 **프록시 팩토리를 생성하는 과정에서 `target` 정보가 팩토리의 인자로 전달되기 때문**이다.