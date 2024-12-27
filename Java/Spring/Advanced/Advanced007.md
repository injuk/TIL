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

## 2024-12-19 Thu
### 패키지 및 메소드 매칭 조건 제어하기
* `"execution(public String ga.injuk.aop.member.MemberServiceImpl.hello(String))"` 조건에서, 패키지는 다음과 같은 의미를 갖는다.
  * `ga.injuk.aop.member` 패키지에 위치한 `MemberServiceImpl` 클래스의 `hello` 메소드
* 즉, 패키지 부분을 다시 항목별로 분석하면 `ga.injuk.aop.member.(대상클래스).(대상메소드)`가 된다.
* 이를 응용할 경우, `ga.injuk.aop.member.*.*(..)`와 같이 입력하는 것으로 해당 패키지의 모든 클래스와 모든 메소드에 대해 매칭할 수 있다.
* 반면, **임의의 클래스의 직계 자손 뿐만 아니라 그 하위에 위치한 모든 클래스를 매칭하고자 하는 경우 `.`이 아닌 `..`을 활용**할 수 있다.
  * 예를 들어, `ga.injuk.aop..*.*(..)`는 `ga.injuk.aop` 패키지의 모든 하위 클래스가 갖는 모든 메소드에 대해 매칭된다.

## 2024-12-20 Fri
### execution 기반 표현식 예시 - 하나 또는 전체
* 가장 기본적인 `execution` 표현식에는 정확히 해당하는 클래스의 특정 메소드를 명시하는 것과, 모든 클래스에 대해 동작하는 종류가 있다.
  * 아래의 예시에서도 알 수 있듯, `*` 및 `**` 기호와 `..` 기호가 혼용되어 원하는 효과를 얻을 수 있다.
```kotlin
class ExecutionTest {
    val pointcut: AspectJExpressionPointcut = AspectJExpressionPointcut()
    var helloMethod: Method? = null

    @BeforeEach
    fun init() {
        helloMethod = MemberServiceImpl::class.java.getMethod("hello", String::class.java)
    }
  
    @Test
    fun exactMatch() {
        pointcut.expression = "execution(public String ga.injuk.aop.member.MemberServiceImpl.hello(String))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }

    @Test
    fun allMatch() {
        pointcut.expression = "execution(* *(..))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }
}
```

## 2024-12-21 Sat
### execution 기반 표현식 예시 - 이름 기반 매칭
* 대상 클래스 또는 메소드의 이름을 기반으로 매칭할 수도 있으며, 이 경우 `*`기호를 활용하여 이름 일부만 일치하는 경우를 표현할 수도 있다. 
```kotlin
class ExecutionTest {
    val pointcut: AspectJExpressionPointcut = AspectJExpressionPointcut()
    var helloMethod: Method? = null

    @BeforeEach
    fun init() {
        helloMethod = MemberServiceImpl::class.java.getMethod("hello", String::class.java)
    }

    @Test
    fun nameMatch() {
        pointcut.expression = "execution(* hello(..))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }

    @Test
    fun nameMatchWithStar1() {
        pointcut.expression = "execution(* hel*(..))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }

    @Test
    fun nameMatchWithStar2() {
        pointcut.expression = "execution(* *el*(..))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }

    @Test
    fun nameMatchFailed() {
        pointcut.expression = "execution(* nono(..))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isFalse()
    }
}
```

## 2024-12-22 Sun
### execution 기반 표현식 예시 - 패키지 기반 매칭
* 대상 클래스가 포함되는 패키지 경로에 대응되는 표현식도 사용 가능하며, 전체적인 활용 방식은 상술한 경우와 동일하다.
```kotlin
class ExecutionTest {
  val pointcut: AspectJExpressionPointcut = AspectJExpressionPointcut()
  var helloMethod: Method? = null

  @BeforeEach
  fun init() {
    helloMethod = MemberServiceImpl::class.java.getMethod("hello", String::class.java)
  }

  @Test
  fun packageExactMatch1() {
    pointcut.expression = "execution(* ga.injuk.aop.member.MemberServiceImpl.hello(..))"

    Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
  }

  @Test
  fun packageExactMatch2() {
    pointcut.expression = "execution(* ga.injuk.aop.member.*.*(..))"

    Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
  }

  @Test
  fun packageMatchFailed() {
    pointcut.expression = "execution(* ga.injuk.aop.*.*(..))"

    Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isFalse()
  }

  @Test
  fun subpackageMatch1() {
    pointcut.expression = "execution(* ga.injuk.aop.member..*.*(..))"

    Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
  }

  @Test
  fun subpackageMatch2() {
    pointcut.expression = "execution(* ga.injuk.aop..*.*(..))"

    Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
  }
}
```

## 2024-12-23 Mon
### execution 기반 표현식 예시 - 타입 기반 매칭
* 타입 기반의 매칭 역시 가능하며, 이 경우 **인터페이스 등 상위 타입을 명시한 경우 그 자식 타입 모두가 매칭 대상**이 된다.
  * 이 때, **상위 타입을 표현하는 표현식에 메소드를 함께 명시하게 되는 경우 반드시 상위 타입의 메소드만이 매칭 가능한 것에 주의**해야 한다.
```kotlin
class ExecutionTest {
    val pointcut: AspectJExpressionPointcut = AspectJExpressionPointcut()
    var helloMethod: Method? = null

    @BeforeEach
    fun init() {
        helloMethod = MemberServiceImpl::class.java.getMethod("hello", String::class.java)
    }

    @Test
    fun typeExactMatch() {
        pointcut.expression = "execution(* ga.injuk.aop.member.MemberServiceImpl.*(..))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }

    @Test
    fun superTypeMatch() {
        pointcut.expression = "execution(* ga.injuk.aop.member.MemberService.*(..))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }

    @Test
    fun superTypeMatchFailed() {
        pointcut.expression = "execution(* ga.injuk.aop.member.MemberService.*(..))"
        val internalMethod: Method = MemberServiceImpl::class.java.getMethod("internal", String::class.java)

        Assertions.assertThat(pointcut.matches(internalMethod, MemberServiceImpl::class.java)).isFalse()
    }
}
```

## 2024-12-24 Tue
### execution 기반 표현식 예시 - 인자 기반 매칭
* 임의의 메소드에 포함된 인자를 기반으로 매칭하는 것 역시 가능하며, 이 경우 인자의 개수 및 타입에 주의해야 한다.
* 이 때, 인자를 기반으로 매칭하는 각각의 경우는 다음과 같이 요약할 수 있다.
  1. `(String)`: 정확하게 문자열 유형의 인자를 하나만 전달 받는 메소드에 매칭한다.
  2. `()`: 아무 인자도 전달 받지 않는 메소드에 매칭한다.
  3. `(*)`: 정확히 하나의 인자를 전달 받는 메소드에 매칭되되, 인자의 유형에 무관하게 매칭한다.
  4. `(*, *)`: 정확히 두 개의 인자를 전달 받는 메소드에 매칭되되, 인자의 유형에 무관하게 매칭한다.
  5. `..`: 0을 포함하여 인자의 개수와 무관하고, 메소드에 전달되는 인자의 유형에 무관하게 매칭한다.
  6. `(String, ..)`: 첫 번째 인자는 반드시 문자열로 시작하되, 그 이후로는 인자의 유형에 관계 없이 0개 이상의 인자를 전달 받는 메소드에 대해 매칭한다.
```kotlin
class ExecutionTest {
    val pointcut: AspectJExpressionPointcut = AspectJExpressionPointcut()
    var helloMethod: Method? = null

    @BeforeEach
    fun init() {
        helloMethod = MemberServiceImpl::class.java.getMethod("hello", String::class.java)
    }
  
    @Test
    fun allStringTypeArgsMatching() {
        pointcut.expression = "execution(* *(String))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }

    @Test
    fun onlyNoArgsMatching() {
        pointcut.expression = "execution(* *())"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isFalse()
    }

    @Test
    fun onlyOneArgsButAcceptEveryTypeMatching() {
        pointcut.expression = "execution(* *(*))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }

    @Test
    fun allArgsMatching() {
        pointcut.expression = "execution(* *(..))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }

    @Test
    fun allArgsButOnlyStartsWithStringMatching() {
        pointcut.expression = "execution(* *(String, ..))"

        Assertions.assertThat(pointcut.matches(helloMethod!!, MemberServiceImpl::class.java)).isTrue()
    }
}
```

## 2024-12-25 Wed
### within 기반 표현식
* `within` 지시자는 임의의 타입에 대해 매칭될 경우 해당 타입의 모든 메소드를 조인 포인트로 취급하여 매칭한다.
  * 예를 들어 `within(ga.injuk.aop.member.MemberServiceImpl)` 형태는 `MemberServiceImpl` 클래스의 모든 메소드에 대해 매칭한다.
  * 다시 말해, 이는 `execution` 지시자에서 타입 기반 매칭만을 사용하는 것으로 이해할 수 있다.
* `within` 지시자 역시 `*` 기호를 기반으로 클래스 이름 일부에 대한 일치 여부로 매칭하거나, `..` 기호로 서브 패키지를 표현할 수 있다.
* 반면, **부모 타입을 명시할 경우 자식 타입도 함께 매칭 가능하던 `execution`과 달리 `within` 지시자는 정확히 명시된 타입만을 매칭**한다.

## 2024-12-26 Thu
### args 기반 표현식
* `args` 지시자는 주어진 타입의 인스턴스를 인자로 받는 조인 포인트에 대해 매칭되며, 기본적인 문법은 `execution`의 인자 기반 매칭과 동일하다.
* 반면, 인자의 타입이 정확히 매칭되어야 하는 `execution`과 달리 `args`는 부모 타입을 활용한 자식 타입과의 매칭을 허용한다.
  * 즉, `args` 지시자는 실제로 전달된 인스턴스를 기반으로 매칭 여부를 결정한다.
* 실무의 경우, `args` 지시자는 단독으로 사용되기보다는 파라미터 바인딩과 함께 사용되는 경우가 많다.

## 2024-12-27 Fri
### @target과 @within 기반 표현식
* **`@target`과 `@within`은 모두 `@표현식(어노테이션)` 형태로 사용되어 어노테이션을 기반으로 AOP 적용 여부를 판단**한다.
* 이 중, `@target` 지시자는 실행 객체의 클래스에 주어진 어노테이션이 명시된 경우에 대해 매칭한다.
  * 반면, `@within` 지시자는 주어진 어노테이션이 명시된 타입에 대해서만 매칭된다.
* 즉, `@target`은 부모 클래스를 포함한 모든 메소드에 대해 어드바이스를 적용하는 반면 `@within`은 클래스 자신에 정의된 메소드에 대해서만 적용한다.
  * 쉽게 말해, 임의의 부모 클래스 A의 자식 클래스 B에 `@target` 지시자가 명시된 경우 부모 클래스 A의 메소드 호출시에도 AOP를 적용한다.
  * 반면, `@within` 지시자가 명시된 경우 부모 클래스 A의 메소드 호출시에는 AOP가 적용되지 않고 오로지 B의 메소드 호출시에만 적용된다.