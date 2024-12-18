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

## 2024-12-17 Tue
### 상세하게 작성된 execution의 예시
* `execution` 포인트컷 지시자는 다음과 같은 형태를 갖는다.
```
execution(modifiers-pattern? ret-type-parttern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)
```
* 이를 한글로 풀어쓸 경우 `execution(접근제어자? 반환타입 선언타입?메소드명(파라미터) 예외?)` 형태가 된다.
  * 이 때, `?` 가 붙은 키워드는 생략할 수 있으며 `*`과 같은 패턴을 명시하는 것으로 메소드의 실행 조인 포인트를 자유로이 매칭할 수 있다.
* 이러한 조건에 맞추어 다음과 같이 원하는 지점에 매칭될 수 있도록 `execution` 포인트컷 지시자를 직접 작성해볼 수도 있다.
```kotlin
class ExecutionTest {
    val pointcut: AspectJExpressionPointcut = AspectJExpressionPointcut()
    var helloMethod: Method? = null

    @BeforeEach
    fun init() {
        helloMethod = MemberServiceImpl::class.java.getMethod("hello", String::class.java)
    }

    @Test
    fun printMethod() {
        println("helloMethod $helloMethod")
    }

    @Test
    fun exactMatch() {
        // MemberServiceImpl 클래스에 명시된 hello 메소드 중 접근 제어자가 public이고 인자와 반환형이 모두 String인 대상을 찾는다.
        pointcut.expression = "execution(public String ga.injuk.aop.member.MemberServiceImpl.hello(String))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }
}
```
* 상술한 예시에서, `pointcut.expression`에 초기화된 포인트컷 지시자는 각 부분 별로 다음과 같은 의미를 갖는다.
  * `접근제어자?`: `public`을 명시한다.
  * `반환타입`: 메소드의 반환형인 `String`을 명시한다.
  * `선업타입?`: 대상 메소드를 포함하는 클래스를 `ga.injuk.aop.member.MemberServiceImpl` 형태로 패키지 경로까지 포함하여 작성한다.
  * `메소드명`: `hello`와 같이 메소드 이름을 명시한다. 
  * `파라미터`: 해당 메소드의 인자 타입을 명시한다.
  * `예외?`: 해당 메소드는 던지는 예외가 없기에 예외를 작성할 필요가 없으며, `?`로 인해 생략 가능하므로 생략되었다.

## 2024-12-18 Wed
### 필요한 정보를 최소화하여 작성된 execution의 예시
* 반면, `execution` 포인트컷 지시자는 다음과 같이 많은 부분을 생략할 수 있도록 지원한다.
```kotlin
class ExecutionTest {
    // ...생략

    @Test
    fun allMatch() {
        pointcut.expression = "execution(* *(..))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }
}
```
* 이 때, `pointcut.expression`에 초기화된 포인트컷 지시자는 각 부분 별로 다음과 같은 의미를 갖는다.
  * `접근제어자?`: `?`로 인해 생략 가능하므로 생략되었다.
  * `반환타입`: `*`로 명시되었으므로 어떠한 반환타입과도 매칭될 수 있다.
  * `선업타입?`: `?`로 인해 생략 가능하므로 생략되었다.
  * `메소드명`: `*`로 명시되었으므로 어떠한 메소드 이름과도 매칭될 수 있다.
  * `파라미터`: `(..)`로 명시되었으므로 인자 개수와 인자 타입과 관계 없이 매칭될 수 있다.
  * `예외?`: `?`로 인해 생략 가능하므로 생략되었다.