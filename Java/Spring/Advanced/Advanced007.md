# Spring Advanced
## 2024-12-15 Sun
### 포인트컷 지시자
* AspectJ는 포인트컷을 편리하게 표현하기 위한 지시자를 제공하며, 이를 포인트컷 표현식이라는 용어로 지칭한다.
* AspectJ 진영에서 제공하는 이러한 포인트컷 표현식은 포인트컷 지시자로 시작하며, 다른 표현으로는 PCD라는 약어를 갖는 `Pointcut Designator`가 있다.
* 이 때, 제공되는 포인트컷 지시자는 크게 다음과 같이 정리해볼 수 있다.
  1. `execution`: 메소드 실행 조인 포인트를 매칭하며, 스프링 AOP에서 가장 많이 사용되면서도 복잡한 축에 속한다.
  2. `within`: 특정한 타입 내부의 조인 포인트를 매칭한다.
  3. `args`: 인자가 주어진 타입의 인스턴스에 해당하는 조인 포인트를 매칭한다.
  4. `this`: 스프링의 빈 객체, 즉 스프링 AOP 프록시를 대상으로 하는 조인 포인트를 매칭한다.
  5. `target`: 스프링 AOP 프록시가 가리키는 실제 대상 객체에 대한 조인 포인트를 매칭한다.
  6. `@target`: 실행 객체 클래스에 주어진 타입의 어노테이션이 포함된 경우에 대한 조인 포인트에 해당한다.
  7. `@within`: 주어진 어노테이션이 포함된 타입 내의 조인 포인트에 해당한다.
  8. `@annotation`: 메소드가 주어진 어노테이션을 갖고 있는 경우의 조인 포인트에 매칭된다.
  9. `@args`: 전달된 실제 인수의 런타임 타입이 주어진 어노테이션을 갖는 경우에 매칭되는 조인 포인트에 해당한다.
  10. `bean`: 스프링의 전용 포인트컷 지시자로, 빈의 이름을 기반으로 포인트컷을 지정한다.
* 이 중 대부분의 경우에 `execution`이 사용되며, 그 이외의 경우는 필요할 때에 이해를 해도 무방하다.

## 2024-12-16 Mon
### execution의 매칭 대상
* 예를 들어 아래와 같은 코드를 작성할 경우, 클래스의 메소드 정보를 출력할 수 있다.
```kotlin
class ExecutionTest {
    var helloMethod: Method? = null

    @BeforeEach
    fun init() {
        helloMethod = MemberServiceImpl::class.java.getMethod("hello", String::class.java)
        // helloMethod public java.lang.String ga.injuk.aop.member.MemberServiceImpl.hello(java.lang.String) 출력됨
    }

    @Test
    fun printMethod() {
        println("helloMethod ${helloMethod}")
    }
}
```
* 이 때, **`execution` 포인트컷 지시자는 출력된 정보와 유사한 메소드 정보를 토대로 매칭하여 포인트컷 대상을 결정**한다.