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