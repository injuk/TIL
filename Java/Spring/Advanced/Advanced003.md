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

## 2024-10-09 Wed
### 프록시 팩토리 활용하기
* 상술한 `Advice`를 기반으로 프록시 팩토리를 활용하여 프록시를 생성하는 경우, 아래와 같은 코드를 작성할 수 있다.
```kotlin
class ProxyFactoryTest {
    companion object {
        private val logger = LoggerFactory.getLogger(ProxyFactory::class.java)
    }

    @Test
    fun `인터페이스가_있으면_JDK_동적프록시를_사용`() {
        // given
        val target: ServiceInterface = ServiceImpl()
        val factory = ProxyFactory(target) // target 정보를 ProxyFactory에 넘겨주므로 MethodInterceptor에서 target 정보를 명시하지 않는다.
        // factory.isProxyTargetClass = true // 해당 코드가 작성된 경우, 인터페이스를 확장하는 클래스에 대해서도 CGLIB를 적용한다.
        factory.addAdvice(TimeAdvice())

        // when
        val proxy = factory.proxy as ServiceInterface
        logger.info("targetClass={}", target.javaClass) // targetClass=class ga.injuk.spring.study.advanced.proxy.common.service.ServiceImpl
        logger.info("proxyClass={}", proxy.javaClass) // proxyClass=class jdk.proxy3.$Proxy16, target이 인터페이스 기반인 경우 JDK 동적 프록시가 적용된다.
        proxy.save()

        // then
        Assertions.assertThat(AopUtils.isAopProxy(proxy)).isTrue() // 프록시 팩토리를 활용하여 생성한 proxy는 AopUtils로 검증할 수 있다.
        Assertions.assertThat(AopUtils.isJdkDynamicProxy(proxy)).isTrue() // 또한 JDK 동적 프록시를 기반으로 생성되었는지까지 확인 가능하다.
        Assertions.assertThat(AopUtils.isCglibProxy(proxy)).isFalse() // 당연스레 CGLIB 기반의 프록시 객체인지도 확인이 가능하다.
    }
}
```
* 상술한 코드와 같이 `ProxyFactory(target)` 형태로 프록시 대상을 함께 전달하며, 프록시 팩토리는 해당 인스턴스 정보를 기반으로 프록시를 생성한다.
  * 예를 들어 해당 인스턴스가 인터페이스를 확장한 클래스의 객체인 경우 JDK 동적 프록시를 적용하며, 그렇지 않다면 CGLIB를 활용한다.
* **프록시 팩토리가 제공하는 `addAdvice()` 메소드를 활용하는 것으로 프록시가 제공할 부가 기능을 명시**할 수 있다.
* 코틀린에서는 `proxy` 접근자로 호출되는 `getProxy()` 메소드는 프록시 객체를 동적으로 생성하기 위해 호출할 수 있다.
* 또한, 프록시 팩토리로 생성된 프록시 객체들은 `AopUtils`를 활용한 여러 검증이 가능하다.
  * 이 때, **`AopUtils`를 활용한 프록시 객체의 검증은 프록시 팩토리를 기반으로 생성된 프록시 객체에 대해서만 검증이 가능한 점에 주의**를 기울여야 한다.

## 2024-10-10 Thu
### 프록시 팩토리의 기반 기술 선택 기준
* 프록시 팩토리는 프록시 대상이 되는 서버 클래스에 대해 다음과 같은 기준을 통해 어떤 프록시 기술을 적용할지 결정한다.
  1. 대상이 인터페이스를 확장한 경우, JDK 동적 프록시를 활용하여 인터페이스 기반 프록시를 적용한다.
  2. 대상이 인터페이스를 확장하지 않는 경우, CGLIB를 활용하여 구체 클래스 기반의 프록시를 적용한다.
  3. **`proxyTargetClass`가 `true`로 설정된 경우, 인터페이스 확장 여부와 관계 없이 CGLIB를 활용한 구체 클래스 기반의 프록시를 적용**한다.
* 이렇듯 **프록시 팩토리를 사용할 경우, 충분한 추상화 덕분에 특정한 프록시 기술에 의존하지 않고 편리하게 동적 프록시 객체를 생성**할 수 있다.
  * 추가적으로 프록시에 작성하는 부가 기능 로직도 특정 기술에 종속되는 클래스가 아닌 `Advice`에 작성할 수 있게 된다.
  * 예를 들어, 프록시 팩토리가 선택한 기반 기술이 JDK 동적 프록시인 경우 `InvocationHandler`가 `Advice`를 호출한다.
  * 반면, CGLIB 기반의 프록시의 경우 `MethodInterceptor`가 `Advice`를 호출하는 형태로 동작한다.

## 2024-10-11 Fri
### 스프링 부트와 CGLIB
* 추가적으로 **스프링 부트는 AOP를 적용하는 과정에서 기본적으로 `proxyTargetClass` 설정에 항상 `true`로 설정**한다.
  * 때문에 **대상 서버 클래스에 인터페이스가 적용되어 있는지 유무와 관계 없이 항상 구체 클래스를 기반으로 프록시를 생성**한다.

### 포인트컷과 어드바이스, 어드바이저
* 스프링 AOP에서 자주 나오는 개념인 포인트컷과 어드바이스, 그리고 어드바이저는 크게 다음과 같은 의미를 갖는다.
  * `Pointcut`: **어디에 부가 기능을 적용하고 어디에 적용하지 않을지 결정하는 필터링 로직이며, 주로 클래스 또는 메소드 이름을 기반으로 필터링**한다.
  * `Advice`: **프록시가 호출하는 부가 기능으로, 프록시 로직으로 이해해도 무방**하다.
  * `Advisor`: **하나의 포인트컷과 하나의 어드바이스를 갖는, 단순한 묶음으로 이해**할 수 있다.
* 즉, **부가 기능을 어디에(=`Pointcut`) 어떤 로직(=`Advice`)으로 적용할지 결정해야 하며 이를 모두 알고 있는 것이 `Advisor`에 해당**한다.
  * **이러한 설계는 역할과 책임을 명확히 분리한 것에 해당하며, 포인트컷과 어드바이스는 각각 필터링 역할과 부가 기능 로직 역할을 담당**한다.

## 2024-10-12 Sat
### 프록시 팩토리에 어드바이저의 관계
* 어드바이저는 하나의 포인트컷과 어드바이스를 가지므로, 프록시 팩토리를 활용하는 과정에서 제공되는 어드바이저를 통해 어디에 어떤 기능을 적용할지 알 수 있다.
```kotlin
class AdvisorTest {
    @Test
    fun advisor() {
        // given
        val target: ServiceInterface = ServiceImpl()

        // when
        val proxy = ProxyFactory(target).run {
            // DefaultPointcutAdvisor는 가장 일반적인 Advisor 구현체로, 생성자를 통해 포인트컷과 어드바이스를 전달받는다.
            // 이 때, Pointcut.TRUE는 항상 true를 반환하는 포인트컷을 의미한다.
            val advisor = DefaultPointcutAdvisor(Pointcut.TRUE, TimeAdvice())
          
            // 기생성된 advisor를 프록시 팩토리에 명시하며, 이를 통해 추후 팩토리로부터 생성될 프록시 인스턴스가 어디에 어떤 부가 기능을 적용할지 이해할 수 있다.
            addAdvisor(advisor)

            proxy as ServiceInterface
        }

        // then
        proxy.save()
        proxy.find()
    }
}
```
* 이 경우 `factory.addAdvisor()` 메소드를 활용하지만 이러한 방식은 이전에 사용했던 `factory.addAdvice()` 메소드와 유사해보일 수 있다.
  * 이는 `factory.addAdvice()` 메소드가 편의 메소드이기 때문으로, 해당 메소드는 내부적으로 `addAdvisor()` 메소드를 호출하는 방식으로 동작한다.
  * 때문에 `factory.addAdvice()` 메소드를 호출하더라도 해당 메소드 내부적으로는 상술한 테스트 코드와 같은 어드바이저가 생성되어 호출된다.
* 결국 **구조상 프록시 팩토리는 어드바이저를 인식하고 있으며, 어드바이저가 포인트컷과 어드바이스 정보를 갖게 되므로 이를 활용할 수 있는 것에 해당**한다.

## 2024-10-13 Sun
### 어드바이저를 활용하는 호출 흐름
* 클라이언트가 프록시를 기반으로 서버 인스턴스가 제공하는 임의의 메소드를 호출할 경우, 다음과 같은 흐름에 따라 부가 기능의 적용 여부가 결정된다.
  1. 클라이언트는 프록시에게 서버의 메소드 호출을 요청한다.
  2. **프록시는 자신이 인식하고 있는 어드바이저가 갖고 있는 포인트컷에게 어드바이스 적용 여부를 확인**한다.
    1. `true`가 반환된 경우, 어드바이스를 적용한다.
    2. `false`가 반환된 경우, 어드바이스를 적용하지 않는다.
  3. **어드바이스의 적용 여부와 관계 없이 서버 인스턴스의 메소드를 호출하여 결과를 반환**한다.

## 2024-10-14 Mon
### Pointcut 인터페이스 분석하기
* 스프링 차원에서 제공되는 `Pointcut` 인터페이스는 다음과 같은 구조를 갖는다.
```java
public interface Pointcut {
    Pointcut TRUE = TruePointcut.INSTANCE;

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();
}
```
* `getClassFilter()` 메소드는 대상이 임의의 클래스가 맞는지 여부를 확인하기 위해 사용된다.
  * 반면, `getMethodMatcher()` 메소드는 대상이 임의의 메소드가 맞는지 여부를 확인하기 위해 사용된다.
* 이 때, 포인트컷은 두 메소드가 모두 `true`를 반환한 경우에 대해서만 어드바이스를 적용할 수 있도록 하는 정책을 갖는다.
* **일반적으로는 스프링이 이미 작성해둔 구현체를 활용하는 것으로 무방하지만, 자신이 원하는 포인트컷 구현체를 직접 정의해볼 수도** 있다.

## 2024-10-15 Tue
### 포인트컷 정의하기
* 예를 들어 서버 인스턴스가 제공하는 임의의 메소드에 대해서만 어드바이스를 적용하지 않고자 하는 경우, 다음과 같은 포인트컷을 정의해볼 수 있다.
  * 물론 **이전의 방식과 유사하게 메소드의 이름을 기반으로 분기를 작성해도 무방하지만, 포인트컷은 이러한 용도에 특화되므로 보다 유용하게 사용**할 수 있다.
```kotlin
class AdvisorTest {
  @Test
  fun `직접_만든_커스텀_포인트컷`() {
    // given
    val target: ServiceInterface = ServiceImpl()

    // when
    val proxy = ProxyFactory(target).run {
      // DefaultPointcutAdvisor의 인자에 자신이 개발한 포인트컷 구현체를 전달할 수 있다.
      val advisor = DefaultPointcutAdvisor(MyCustomPointcut(), TimeAdvice())
      addAdvisor(advisor)

      proxy as ServiceInterface
    }

    // then
    proxy.save() // 부가 기능 적용됨
    proxy.find() // 부가 기능 적용되지 않음
  }

  // 직접 주현한 MyCustomPointcut 구현체는 모든 클래스에 대해 true를 반환하는 반면, 메소드에 대해서는 직접 구현한 MethodMatcher 구현체를 활용한다.
  class MyCustomPointcut: Pointcut {
    override fun getClassFilter(): ClassFilter
            = ClassFilter.TRUE

    override fun getMethodMatcher(): MethodMatcher
            = MyCustomMethodMatcher()
  }

  class MyCustomMethodMatcher: MethodMatcher {
    companion object {
      private val WHITE_LIST = setOf("save")
      private val logger = LoggerFactory.getLogger(MyCustomMethodMatcher::class.java)
    }

    // MethodMatcher 구현체의 matches 메소드는 화이트리스트에 허용된 메소드 이름만을 허용하며, 이외의 경우 false를 반환하여 어드바이스가 적용될 수 없도록 동작한다.
    override fun matches(method: Method, targetClass: Class<*>): Boolean
        = WHITE_LIST.contains(method.name)
            .also { isMatched -> logger.info("is method(${method.name}) matched? :$isMatched") }

    override fun matches(method: Method, targetClass: Class<*>, vararg args: Any?): Boolean {
      return false
    }

    override fun isRuntime(): Boolean {
      return false
    }
  }
}
```