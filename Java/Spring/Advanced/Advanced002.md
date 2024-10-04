# Spring Advanced
## 2024-09-24 Tue
### 동적 프록시 기초와 리플렉션
* 상술했듯 프록시 패턴을 적용할 경우, 프록시 대상 서버 객체의 수만큼 프록시 클래스가 늘어나는 단점이 수반된다.
  * 이는 Java가 제공하는 동적 프록시 또는 CGLIB과 같은 동적 프록시 기술을 활용하여 해결할 수 있으며, 이를 통해 프록시 객체를 동적으로 생성할 수 있다.
  * 즉, 프록시용 코드를 하나만 작성하고도 동적 프록시 기술을 기반으로 많은 프록시 객체를 생성할 수 있게 된다.
* 이 중 Java가 제공하는 JDK 동적 프록시를 이해하기 위해서는 리플렉션에 대한 이해가 선행될 필요가 있다.
* 리플렉션은 쉽게 말해 클래스나 메소드가 갖는 메타데이터를 동적으로 획득하고, 이를 기반으로 코드를 동적으로 호출할 수 있도록 지원하는 기능을 의미한다.

## 2024-09-25 Wed
### 리플렉션을 활용한 메소드 호출
* 예를 들어 아래와 같은 로직에서, 로직 1이 여기 저기에 중복되는 반면 중간에 위치한 `callX()` 메소드만 변경되는 경우가 발생할 수 있다.
  * 즉, 템플릿 메소드 패턴의 예시와 유사하게 중간 과정만 변경되는 경우를 의미한다.
```kotlin
class ReflectionTest {
    companion object {
        private val logger = LoggerFactory.getLogger(ReflectionTest::class.java)
    }

    @Test
    fun reflection0() {
        val hello = Hello()

        // 로직 1 시작
        logger.info("start")
        val result1 = hello.callA() // 여기만 다르다!
        logger.info("result = {}", result1)
        // 로직 1 종료

        // 로직 2 시작
        logger.info("start")
        val result2 = hello.callB() // 여기만 다르다!
        logger.info("result = {}", result2)
        // 로직 2 종료
    }

    class Hello {
        companion object {
            private val logger = LoggerFactory.getLogger(Hello::class.java)
        }

        fun callA(): String {
            logger.info("call A!")
            return "A"
        }

        fun callB(): String {
            logger.info("call B~")
            return "B"
        }
    }
}
```
* 이러한 코드는 언뜻 공통화하기 쉬워 보이지만, 중간 단계만이 달라지기 때문에 이러한 동적인 변경 사항을 공통화하기 어려운 경우가 많다.
* 리플렉션은 이러한 경우에 적합하며, 클래스나 메소드의 메타데이터를 활용할 수 있도록 지원하므로 호출 대상 메소드에 대한 동적인 변경을 적용하기 쉽다.
  * 반면, 실무에서는 리플렉션 대신 람다식을 사용하는 경우도 많지만 이 경우에는 리플렉션에만 집중하기로 한다.
```kotlin
class ReflectionTest {
    // ...생략

    @Test
    fun reflection2() {
        val hello = Hello()

        // 패키지명을 포함한 클래스 이름을 기반으로 클래스 메타데이터를 획득한다. 
        // 이 때, 임의의 클래스에 포함된 inner class의 경우에는 $ 표시를 명시한다.
        val clazz = Class.forName("ga.injuk.spring.study.advanced.proxy.jdkdynamic.ReflectionTest\$Hello")

        // 아래 코드를 별도의 메소드로 추출하여 메소드 메타데이터와 인자만을 전달하도록 구현하는 것도 가능하다.
        listOf("callA", "callB").forEach { methodName ->
            // 호출 대상 메소드의 이름을 기반으로 메소드 메타데이터를 획득한다. 
            val method = clazz.getMethod(methodName)

            // 공통 로직 시작
            logger.info("start")
            // 획득한 메소드 메타데이터를 기반으로 인자를 전달하여 호출한다.
            val result = method.invoke(hello)
            // val result = method(hello) // kotlin은 infix function인 invoke를 활용하므로 이렇게도 가능하다!
            logger.info("result = {}", result)
            // 공통 로직 종료
        }
    }

    // ...후략
}
```

## 2024-09-26 Thu
### 리플렉션 주의사항
```
> 리플렉션은 필연적으로 런타임 예외를 유발하기에 위험하다! 
```
* 리플렉션은 클래스와 메소드의 메타데이터를 동적으로 획득하여 활용할 수 있게하므로 애플리케이션을 매우 유연하게 만들어줄 수 있다.
* 그러나 이러한 **리플렉션은 런타임에서 동작하므로, 컴파일 시점에서는 어떠한 오류도 잡아낼 수 없다는 치명적인 단점을 수반**한다.
  * 예를 들어, `clazz.getMethod(methodName)`의 인자에 존재하지 않는 메소드명을 전달하더라도 컴파일은 이상 없이 처리되며 런타임에서 예외가 발생한다.
  * 익히 알려졌듯 **가장 좋은 예외는 개발자가 사전에 대처할 수 없는 컴파일 타임 예외인 반면, 가장 무서운 오류는 사용자로부터 발생한 런타임 예외**이다.
  * 프로그래밍이라는 분야는 타입을 기반으로 컴파일 시점에 최대한 많은 예외를 잡아내는 방식으로 발전해온 반면, 리플렉션은 이러한 방향성에 거스른다.
* 상술한 이유에서 **리플렉션은 일반적인 상황에서는 사용을 지양해야 하며, 프레임워크 개발이나 매우 일반적인 공통화된 처리에 대해서만 사용을 고려**해야 한다.

## 2024-09-27 Fri
### 동적 프록시란?
* 동적 프록시는 말 그대로 여러 프록시 객체를 런타임에 동적으로 생성하며, 이로 인해 개발자는 단지 프록시에 원하는 로직만을 구현할 수 있게 된다.
  * 이 때, **동적 프록시의 일종인 JDK 동적 프록시는 인터페이스를 기반으로 동작하므로 프록시 대상에 대한 인터페이스 정의가 필수적**이다.
* 추가적으로 **JDK 동적 프록시의 경우 공통으로 사용될 로직에 대해 `InvocationHandler` 인터페이스를 활용하는 핸들러를 작성**해야 한다.
* JDK 동적 프록시를 위해 활용할 수 있는 `InvocationHandler` 인터페이스는 다음과 같은 형태를 갖는다.
  * 첫 번째 인자인 proxy는 프록시 자신을 의미한다.
  * 두 번째 인자인 method는 호출한 메소드를 의미한다.
  * 세 번째 인자인 args는 메소드 호출시 전달된 인수를 의미한다.
```java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}
```

## 2024-09-28 Sat
### 동적 프록시를 활용하는 예시
* 예를 들어 어떠한 메소드를 호출하여 실행 시간을 측정하는 간단한 `InvocationHandler`는 다음과 같이 정의해볼 수 있다.
  * 아래의 `TimeInvocationHandler`는 `InvocationHandler`를 구현하며, 이러한 방법을 통해서만 JDK 동적 프록시를 활용할 수 있다.
```kotlin
class TimeInvocationHandler(
    private val target: Any,
) : InvocationHandler {
    companion object {
        private val logger = LoggerFactory.getLogger(TimeInvocationHandler::class.java)
    }

    override fun invoke(proxy: Any?, method: Method?, args: Array<out Any>?): Any {
        logger.info("TimeInvocationHandler invoked!")

        val startTime = System.currentTimeMillis()

        val result = method!!.invoke(target, *args ?: emptyArray())

        val endTime = System.currentTimeMillis()

        logger.info("TimeInvocationHandler ended, time: {}ms", endTime - startTime)

        return result
    }
}
```
* 이렇게 정의한 `InvocationHandler`는 다음과 같이 `Proxy.newInstance()` 메소드를 활용하여 사용해볼 수 있다.
```kotlin
class JdkDynamicProxyTest {
    companion object {
        private val logger = LoggerFactory.getLogger(JdkDynamicProxyTest::class.java)
    }

    @Test
    fun dynamicA() {
        val target: AInterface = AImpl()

        val handler = TimeInvocationHandler(target)

        // JDK 동적 프록시에 의해 AInterface 타입의 프록시 객체가 동적으로 생성된다.
        // 또한, proxy는 내부적인 동작 과정에서 세 번째 인자인 handler에 있는 로직을 호출하게 된다.
        val proxy: AInterface = Proxy.newProxyInstance(
            AInterface::class.java.classLoader,
            arrayOf(AInterface::class.java),
            handler,
        ) as AInterface

        proxy.call()

        logger.info("target class {}", target.javaClass) // target class class ga.injuk.spring.study.advanced.proxy.jdkdynamic.code.AImpl
        logger.info("proxy class {}", proxy.javaClass) // proxy class class jdk.proxy3.$Proxy16
    }
}
```
* 이 때, 상술한 `dynamicA()` 테스트 코드는 다음과 같은 절차에 따라 실행된다.
  1. `Proxy.newProxyInstance()` 메소드에 의해 `AInterface` 타입의 프록시 객체가 동적으로 생성된다.
  2. `proxy.call()` 메소드가 호출되었을 경우, 로직에 해당하는 핸들러의 `TimeInvocationHandler.invoke()` 메소드가 호출된다.
  3. `TimeInvocationHandler.invoke()` 메소드에는 1.에서 호출된 `proxy.call()` 메소드와 그 메소드에 전달된 인자 정보가 모두 전달된다.
  4. `TimeInvocationHandler`는 `invoke()` 메소드에 정의된 로직을 모두 실행한 후, `method.invoke()`를 호출한다.
  5. `method.invoke()` 과정에서 해당 메소드를 실제로 호출하는 것은 `target` 멤버 변수가 참조하는 `AImpl` 구현체에 해당한다.
  6. `AImpl` 구현체가 동작을 마친 후, `TimeInvocationHandler`는 남은 로직을 모두 실행한 후 결과를 반환한다.

## 2024-09-29 Sun
### Proxy.newProxyInstance 메소드 활용하기
* `Proxy.newProxyInstance()` 메소드를 활용할 경우, 인자로 전달한 클래스 로더와 인터페이스 및 핸들러 로직을 기반으로 프록시 객체가 동적 생성된다.
  * 이 떼, **프록시 객체는 결과적으로 두 번째 인자로 전달된 인터페이스를 기반으로 생성되어 반환**된다.
  * 이렇게 생성된 **프록시 객체의 동작 방식은 단지 핸들러로 전달된 로직을 호출하면서 어떤 메소드에 어떤 인자가 전달되었는지 포워딩하는 것에 불과**하다.
* 이러한 방식을 활용할 경우 프록시 대상마다 프록시 클래스를 정의할 필요가 없으며, 핸들러를 한 번만 정의한 후 필요한 만큼 프록시 객체를 동적 생성할 수 있다.
  * 결과적으로는 **프록시 클래스 폭발 문제가 해결됨과 동시에 부가 기능 역시 하나의 클래스에 응집되므로 SRP를 준수**할 수 있게 된다.

## 2024-09-30 Mon
### JDK 동적 프록시의 한계와 CGLIB
* **JDK 동적 프록시를 사용하는 경우, 인터페이스를 반드시 필요로 하기에 구체 클래스에 대해서는 동적 프록시를 적용할 수 없다**.
  * 그러나 실무에서는 인터페이스 없이 클래스만 있는 경우도 존재하므로 이러한 경우에 동적 프록시를 적용할 수 없는 것은 명확한 한계로 이해할 수 있다.
* 구체 클래스에 대해 동적 프록시를 적용하고자 하는 경우, JDK 동적 프록시가 아닌 바이트코드를 직접 조작하는 CGLIB 라이브러리를 고려할 필요가 있다.

## 2024-10-01 Tue
### CGLIB이란?
* CGLIB는 바이트코드를 조작하여 동적으로 클래스를 생성하는 기능을 제공하는 라이브러리이며, 이를 활용하면 구체 클래스만으로도 동적 프록시를 생성할 수 있다.
* **CGLIB는 기본적으로 서드 파티 라이브러리에 해당하지만, 스프링 프레임워크는 해당 라이브러리를 프레임워크 내에 포함**한다.
  * 때문에 스프링을 사용하는 상황에서는 별도의 라이브러리 추가 없이 CGLIB를 사용할 수 있다.
* 반면, **일반적인 경우에서 CGLIB를 사용해야할 상황은 거의 발생하지 않으며 대부분의 경우 후술할 스프링의 `ProxyFactory`를 사용**하게 된다.
  * 이 때, `ProxyFactory`는 CGLIB를 편리하게 사용할 수 있도록 지원하는 기능으로 이해할 수 있다.

## 2024-10-02 Wed
### CGLIB와 MethodInterceptor
* JDK 동적 프록시가 동적 프록시의 로직 실행을 위해 `InvocationHandler`를 제공하듯, CGLIB 측은 `MethodInterceptor`를 제공한다.
  * 이는 **동적 프록시 객체가 생성되는 것과는 별개로 각 동적 프록시 객체가 실행할 로직을 작성하기 위한 공간에 해당**한다.
* 이 때, `MethodInterceptor`는 다음과 같은 구조를 갖으며 각 인자의 의미는 `InvocationHandler`와 유사하다.
```java
public interface MethodInterceptor extends Callback {
    public Object intercept(Object obj, java.lang.reflect.Method method, Object[] args, MethodProxy proxy) throws Throwable;
}
```
* 이러한 인터페이스를 확장하는 `MethodInterceptor`의 예시는 다음과 같다.
```kotlin
class TimeMethodInterceptor(
    private val target: Any, // 해당 동적 프록시가 실제로 호출할 서버 객체에 해당한다.
) : MethodInterceptor {
    companion object {
        private val logger = LoggerFactory.getLogger(TimeMethodInterceptor::class.java)
    }

    override fun intercept(obj: Any?, method: Method?, args: Array<out Any>?, proxy: MethodProxy?): Any {
        logger.info("TimeInvocationHandler invoked!")

        val startTime = System.currentTimeMillis()

        val result = proxy!!.invoke(target, args) // 실제 대상 서버 객체를 동적으로 호출하며, 이 과정에서 method.invoke를 사용해도 되지만 CGLIB는 성능상 MethodProxy의 사용을 권장한다.

        val endTime = System.currentTimeMillis()

        logger.info("TimeInvocationHandler ended, time: {}ms", endTime - startTime)

        return result
    }
}
```

## 2024-10-03 Thu
### CGLIB을 활용한 동적 프록시 객체 생성
* 상술했듯, CGLIB은 동적 프록시 핸들러 작성을 위한 `MethodInterceptor` 클래스를 제공하며 이를 활용한 동적 프록시 객체 생성 코드는 다음과 같다.
```kotlin
class CglibTest {
    companion object {
        private val logger = LoggerFactory.getLogger(CglibTest::class.java)
    }

    @Test
    fun cglib() {
        val target = ConcreteService() // kotlin의 경우, 대상 구현체 클래스는 open이어야 한다는 점에 주의한다.
      
        val proxy: ConcreteService = Enhancer().run {
            setSuperclass(ConcreteService::class.java)
            setCallback(TimeMethodInterceptor(target))
            create() as ConcreteService
        }

        proxy.call() // call!!!
        // kotlin의 경우, 대상 구현체 클래스의 메소드는 open이어야 한다는 점에 주의한다.

        logger.info("targetClass = {}", target.javaClass) // targetClass = class ga.injuk.spring.study.advanced.proxy.common.service.ConcreteService
        logger.info("proxyClass = {}", proxy.javaClass) // proxyClass = class ga.injuk.spring.study.advanced.proxy.common.service.ConcreteService$$EnhancerByCGLIB$$4d438e28
    }
}
```
* 이 때, CGLIB은 `Enhancer`를 활용하여 부모 클래스와 `MethodInterceptor`를 할당한 후 동적 프록시 객체를 생성하는 방식으로 동작한다.
  * 이 경우, 인터페이스를 확장하여 동적 프록시 객체를 생성하던 JDK 동적 프록시와 달리 CGLIB는 구체 클래스를 상속하는 것으로 프록시 객체를 생성한다.
* 추가적으로 CGLIB에 의해 생성된 동적 프록시 객체의 이름은 `패키지.대상클래스$$EnhancerByCGLIB$$임의코드` 형태가 된다.

## 2024-10-04 Fri
### CGLIB의 한계
* CGLIB는 클래스 상속을 활용함에 이에 따른 제약 사항을 갖는다.
  1. CGLIB는 자식 클래스를 동적으로 생성하기에 부모 클래스의 기본 생성자를 필요로 한다.
  2. final 키워드가 명시된 클래스에 대해서는 상속이 불가능하므로 예외가 발생할 수 있다.
  3. final 키워드가 명시된 메소드에 대해서는 재정의가 불가능하므로 프록시에 작성된 로직이 동작하지 않을 수 있다.