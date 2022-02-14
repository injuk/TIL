# Kotlin
## 2022-02-14 Mon

## Kotlin의 타입 시스템
* Kotlin의 경우, Java보다 가독성을 향상시키기 위한 몇 가지 특성이 제공된다.
  * 예를 들어 nullable type과 읽기 전용 컬렉션이 있다.
* 또한 Kotlin은 Java의 타입 시스템에서 불필요하거나 문제가 될 수 있는 부분을 제거하였다.

### 널 가능성
* **널 가능성(nullability)는 NPE를 피하기 위한 Kotlin 타입 시스템의 특성**이다.
* Kotlin과 같은 최신 언어에서, null과 NPE에 대한 접근 방법은 다음과 같다.
  1. 가능하다면 NPE의 발생 시점을 런타임이 아닌 컴파일 시점으로 옮긴다.
  2. null이 될 수 있는지에 대한 여부를 타입 시스템에 추가하고, 컴파일러가 이를 컴파일 시점에 감지할 수 있도록 한다.
* **이러한 접근 방법의 목적은 컴파일 시점에 오류를 감지하고, 런타임에 발생할 수 있는 예외의 가능성을 줄이는 데에 있다**.

### 널이 될 수 있는 타입
* 어떤 변수가 null이 될 수 있다면, 해당 변수에 대해 메소드를 호출했을 때 NPE가 발생할 수 있으므로 안전하지 않다.
* **Kotlin의 경우, 타입 시스템 차원에서 널이 될 수 있는 타입 개념을 명시적으로 지원**한다.
  * Kotlin은 이러한 특성으로 인해 널이 될 수 있는 타입의 메소드 호출을 금지하여 많은 오류를 방지한다.
* **Kotlin에서 함수를 작성하려면, 함수가 널을 인자로 받을 수 있는지를 우선적으로 결정**해야 한다.
* 함수에서 null을 인자로 받을 수 있게 하려면, 함수의 인자 타입 뒤에 물음표를 명시한다.
  * 아래의 예시에서, nullSafe와 달리 notSafe(num: Int?)는 null을 인자로 받을 수 있다.
  * **nullSafe의 경우 항상 Int 타입의 인자를 받아야 하므로, 런타임에 NPE가 발생하지 않으리라는 사실을 장담**할 수 있다.
```
fun main(args: Array<String>) {
     // 아래와 같은 함수의 호출은 컴파일할 수 없다.
     // nullSafe(null)
     notSafe(null)
}
fun nullSafe(num: Int) = println(num)
fun notSafe(num: Int?) = println(num)
```
* 상술한 예시와 같이, **어떤 타입도 물음표를 붙이는 것으로 타입의 변수나 프로퍼티에 null 참조를 저장할 수 있게 된다**.
  * 반대로, 물음표가 없는 타입은 해당 변수가 null 참조를 저장할 수 없다는 의미가 된다.
  * 즉, **모든 타입은 기본적으로 null이 될 수 없다**.
* **널이 될 수 있는 타입의 변수는 다음과 같이 수행할 수 있는 연산이 제한**된다.
1. 널이 될 수 있는 타입의 변수는 메소드나 프로퍼티를 직접 호출할 수 없다.
```
// 아래와 같이 직접 프로퍼티를 호출할 수 없다.
fun nullSafe(string: String?) = string.length
// 직접 프로퍼티를 호출하려면 ?. 연산자를 사용해야 한다.
// fun nullSafe(string: String?) = string?.length
```
2. 널이 될 수 있는 값을 널이 될 수 없는 타입의 변수에 저장할 수 없다.
```
fun set(string: String?) {
     // 아래의 주석을 해제할 경우 컴파일이 불가능하다.
     // val temp: String = string
}
```
3. 널이 될 수 있는 타입의 값은 널이 될 수 없는 타입의 인자를 받는 함수에 전달할 수 없다.
   * 아래의 예시는 초기화를 마친 변수를 넘기지만 상술한 규칙에 의해 컴파일이 불가능하다.
```
fun main(args: Array<String>) {
     val string: String? = "test"
     // nullSafe(string)
}

fun nullSafe(string: String) = println(string)
```
* 널이 될 수 있는 타입을 null과 비교하는 코드를 작성하여 사용할 수 있다. 
  * **컴파일러는 이 결과를 기억하고 null이 아님이 확실한 코드에서는 해당 값을 널이 될 수 없는 값처럼 취급**한다.
```
fun main(args: Array<String>) {
     val string: String? = "test"
     // 아래 if 문 내부의 영역은 null이 아님이 확실하므로 컴파일이 가능해진다.
     if(string != null) 
          nullSafe(string)
}

fun nullSafe(string: String) = println(string)
```

### 타입이란?
* 타입은 분류를 위한 개념이며, 다음의 두 항목을 결정한다.
  1. 어떤 값들이 가능한지 결정한다.
  2. 타입에 대해 어떤 연산을 수행할 수 있는지 결정한다.
* 타입의 존재로 인해, 임의의 타입의 변수가 있고 변수에 대한 연산이 컴파일되었다면 연산이 성공적으로 실행되리라 확신할 수 있다.
* Java의 타입 시스템은 null 값을 제대로 다루지 못하며, 철저한 처리가 없는 경우 언제나 NPE가 발생할 가능성이 존재한다.
* **Kotlin이 제동하는 널이 될 수 있는 타입 개념은 null 처리에 대한 해법을 제공한다**.
  * **널이 될 수 있는 타입을 구분하여 각 값에 대해 어떤 연산이 가능한지 명확히 이해하고, 런타임에 예외가 발생할 수 있는 연산을 판단하고 금지**시킨다.
  * **널이 될 수 있는 타입은 래퍼 타입이 아니므로, 모든 검사를 컴파일 시점에 수행하며 런타임 시점의 비용은 들지 않는다**.

### 안전한 호출 연산자
* **Kotlin이 제공하는 기능 중 안전한 호출 연산자인 ?.는 특히 유용**하다.
* 예를 들어, null 여부를 검사한 후 메소드를 호출하는 다음 코드는 안전한 호출 연산자인 ?.로 대체될 수 있다.
  * 아래의 예시에서, 정의된 두 최상위 함수의 의미는 완전히 같다.
```
fun complicatedDoSomethingWithString(string: String?) =
     if(string != null)
          string.length
     else
          null

// ?. 연산자에 의해 훨씬 쉽게 표현된다.
fun doSomethingWithString(string: String?) = string?.length
```
* **?. 연산자는 null이 아닌 값에 대해서만 메소드를 호출하므로, 수신 객체의 null 여부에 따라 다음과 같이 동작**한다.
  1. **null인 경우, 수신 객체에 대한 메소드 호출은 무시되고 null을 반환**한다.
  2. **null이 아닌 경우, 수신 객체에 대한 일반 메소드 호출처럼 동작**한다.
* **?. 연산자의 반환형도 널이 될 수 있는 타입**이다.
  * 예를 들어, string.length의 반환형은 String이지만 string?.length의 반환형은 String?이다.
* **?. 연산자는 변수에 대한 메소드 호출 뿐만 아니라 클래스의 프로퍼티에 접근할 때에도 사용**할 수 있다.
* **객체 그래프에서 널이 될 수 있는 중간 객체가 여럿 있는 경우, 한 식 안에서 ?. 연산자를 연쇄해서 사용할 수 있다**.
  * 아래의 예시와 같이 연쇄적인 ?.연산자를 사용함으로써 별다른 추가 검사 로직 없이 객체 그래프를 탐색할 수 있다.
  * Java에서는 복잡하게 작성되었을 널 검사 로직이 Kotlin에서는 ?. 연산자로 매우 간소화되는 것을 알 수 있다.
```
val name: String = animal?.parent?.name
```

### 엘비스 연산자
* 어떤 값의 null 여부를 검사하고, null인 경우 디폴트 값을 할당하고 싶은 경우가 있다.
```
fun main(args: Array<String>) {
     println(returnDefaultValueWhenParameterIsNull("this is string value!"))
     println(returnDefaultValueWhenParameterIsNull(null))
}

fun returnDefaultValueWhenParameterIsNull(string: String?): String {
     if(string == null)
          return "NULL!"
     else
          return string
}
```
* 이러한 경우, **불필요한 null 검사 및 반환값 결정 코드를 엘비스 연산자 ?:로 축약**할 수 있다.
  * **?: 연산자는 좌항이 null인 경우 우항을 반환하고, null이 아닌 경우 좌항의 값을 그대로 사용**한다.
```
// 엘비스 연산자는 좌항이 null인 경우 우항을 반환한다.
fun returnDefaultValueWhenParameterIsNull(string: String?) = string ?: "NULL!"
```
* **?: 연산자는 객체가 null인 경우 null을 반환하는 ?. 연산자와 조합하여 null인 경우를 대비할 수 있도록 코드를 작성할 수 있다**.
* **Kotlin에서는 return, throw 문 역시 식이므로 ?: 연산자의 우항에 작성할 수 있다**.
  * 이 경우, 검증 대상이 null이면 어떤 값을 반환하거나 예외를 던지는 식으로 동작한다.
  * **이러한 코드 작성 방식은 함수의 전제 조건을 검증하는 데에 유용**하다.

### 안전한 타입 캐스트
* Kotlin은 Java와 마찬가지로 as로 지정한 타입으로 변환할 수 없는 경우에는 ClassCastException이 발생한다.
  * 아래의 코드는 실행이 가능하지만 ClassCastException이 발생한다.
```
fun main(args: Array<String>) {
     val string: String = "string"
     println(string as Long)
}
```
* 상술한 경우를 방지하기 위해 as를 사용하는 모든 코드의 앞 부분에 is로 변환 가능성을 검사하는 코드를 추가할 수도 있다.
  * 그러나 Kotlin은 더 좋은 방법인 안전한 캐스트를 제안한다.
* **안전한 캐스트 연산자 as?는 어떤 값을 지정된 타입으로 캐스트하며, 이 과정에서 다음과 같이 동작**한다.
  * 변환이 가능한 경우, 변환한다.
  * **변환 가능한 타입이 아닌 경우, null을 반환**한다.
* 이제 아래의 코드는 ClassCastException을 던지는 대신 null을 반환한다.
```
fun main(args: Array<String>) {
     val string: String = "string"
     println(string as? Long)
}
```
* 안전한 캐스트 연산자 as?는 주로 캐스트 ?: 연산자와 조합하여 사용된다.
  * 예를 들어, 이러한 조합은 equals 메소드를 오버라이드할 때 유용하다.

### 널 아님 단언 연산자 - not null assertion 연산자
* **?., as?, ?: 연산자는 유용하지만, Kotlin 컴파일러에게 어떤 값이 null이 아니라는 사실을 직접 알려주고 싶은 경우가 생길 수 있다**.
* 널 아님 단언 연산자 **!!는 Kotlin에서 널이 될 수 있는 타입을 다룰 때 사용할 수 있는 도구 중 가장 단순 무식한 도구**이다.
* **!! 연산자를 사용하면 어떤 값도 널이 될 수 없는 타입으로 강제로 변환한다**.
* !! 연산자는 값의 null 여부에 따라 다음과 같이 동작한다.
  1. 값이 null이 아닌 경우, 그 값을 그대로 적용한다.
  2. 값이 실제로 null인 경우, NPE를 던진다.
```
fun main(args: Array<String>) {
     maybeNotNull(Bird("bird"))
     // 아래 줄을 실행하는 과정에서 NPE가 발생한다.
     maybeNotNull(null)
}
data class Bird(val name: String?)

fun maybeNotNull(bird: Bird?) {
     println("${bird!!.name} is not null!!!")
}
```
* **!! 연산자는 근본적으로 컴파일러에게 이 값이 null이 될 수 없을 것이며, null인 경우에 발생하는 NPE는 감내하겠다는 의미를 갖는다**.
* **!! 연산자는 최선의 방법이 아니며, 가능하다면 개발자는 다른 더 좋은 방법을 찾아내야 한다**.
  * 그러나 !! 연산자를 절대 사용하지 말아야 하는 것은 아니다.
  * 로직상 null이 될 수 없음에도 null 검사가 강제되는 경우에는 !! 연산자를 아래와 같이 활용할 수 있다.
* 아래의 코드는 널이 될 수 없는 타입을 메소드에 전달한 후 그대로 반환받지만, 컴파일러에 의해 null 검사가 강제된다.
  * 이러한 경우 !! 연산자를 활용하여 코드를 간소화할 수 있다.
```
fun maybeNotNull(bird: Bird) {
     // temp는 언제나 null이 아니지만, definitelyNotNull 함수의 반환형으로 인해 null 검사가 강제된다.
     val temp = definitelyNotNull(bird)
     // if(temp != null)
     //      println(temp.name)
     // 이러한 경우에는 !! 연산자를 사용하는 것이 이상적이다.
     println(temp!!.name)
}
fun definitelyNotNull(bird: Bird?) = bird
```
* !! 연산자에서 발생한 예외는 어떤 코드에서 예외가 발생했는지 명시하지만, 어떤 !! 구문에서 발생한 예외인지 명시하지는 않는다. 
  * 예를 들어, 아래의 코드를 실행한 경우 발생하는 stack trace에는 어떤 !! 연산자에서 발생한 예외인지 명시되지 않는다.
  * **때문에 !! 연산자를 연쇄하여 코드를 작성하는 것은 지양**해야 한다.
```
fun main(args: Array<String>) {
     complicatedNotNulAssertion(null)
}

// 예외가 발생하더라도 어떤 !! 연산자에서 문제가 발생하는지 알 방법이 없다.
fun complicatedNotNulAssertion(animal: Animal?) = println(animal!!.parents[3]!!.length)
```

## 2022-02-15 Tue
### let 함수
* let은 일반적으로 널이 아닌 값만 받을 수 있는 함수에 널이 될 수 있는 값을 넘기는 경우에 사용할 수 있다.
* **let 함수는 자신의 수신 객체를 인자로 전달 받은 람다에 넘기며, 수신 객체가 null이 아닌 경우에만 람다를 실행**하게 된다.
* 따라서 let 함수는 수신 객체의 null 여부에 따라 다음과 같이 실행될 수 있다.
  1. null인 경우, let 함수에 전달된 람다식은 실행되지 않는다.
  2. null이 아닌 경우, let 함수에 전달된 람다식이 실행되며 널이 될 수 없는 타입처럼 실행된다.
* 아래의 예시에서, definitelyNotNull 메소드는 널이 될 수 없는 타입을 인자로 받는다.
  * 때문에 null을 바로 넘겨줄 수 없으므로, doSomethingWithNullable에서 null이 될 수 있는 값을 받아 let 함수를 통해 null을 검사한다.
  * doSomethingWithNullable 메소드에 null이 넘겨진 경우, let 함수에 전달된 람다 본문은 실행되지 않는다.
  * **null이 아닌 경우, let 함수 내부의 it은 null이 아닌 것으로 취급**된다.
```
fun main(args: Array<String>) {
     doSomethingWithNullable("hello")
     doSomethingWithNullable(null)
}

fun doSomethingWithNullable(string: String?) = string?.let {
     println("lambda called!")
     definitelyNotNull(it)
}

fun definitelyNotNull(string: String) = println(string)
```
* **let 함수는 어떤 식의 결과가 null이 아닌 경우에만 실행해야 하는 경우에 유용**하다.
  * 아래의 예시에서, 번거로운 null 검사 로직을 ?.연산자와 let 함수를 조합하여 간단하게 작성할 수 있다. 
```
fun main(args: Array<String>) {
     // 아래의 식처럼 작성할 필요 없이 let 함수를 활용한다.
     /* val string: String? = getString()
        if(string != null) definitelyNotNull(string)
      */
     
     getString()?.let { definitelyNotNull(it) }
}
fun getString(): String? = null

fun definitelyNotNull(string: String) = println(string)
```
* 만약 여러 값의 null 여부를 검사해야 하는 경우 let 함수를 중첩시킬 수 있다.
  * 그러나 가독성 측면에서 이는 단점이 더 큰 방법으로, 이 경우에는 여러 개의 if 문을 통해 코드를 작성하는 것이 더 바람직하다.

### 프로퍼티를 나중에 초기화하기
* Kotlin에서, 클래스 내부에서 널이 될 수 없는 프로퍼티를 생성자 내부가 아닌 메소드 안에서 초기화하는 방법은 없다.
  * Kotlin에서는 일반적으로 생성자 안에서 모든 프로퍼티를 초기화해야 한다.
* 만약 인스턴스를 우선 생성한 후에 프로퍼티를 초기화해야 하는 경우, 부득이하게 프로퍼티를 널이 가능한 타입으로 선언하고 다음과 같이 사용해야 한다.
  * 그러나 널이 될 수 있는 타입은 참조시 항상 널 검사를 하거나, !! 연산자를 사용해야 하는 문제가 있다.
  * 이는 널이 될 수 있는 프로퍼티를 여러 번 사용할수록 코드의 가독성을 떨어트린다.
```
class TargetService {
     fun sayHello() = "hello world!"
}
class LazyClass {
     // 인스턴스화 시점과 프로퍼티 값이 설정되는 시점이 다르다면 널이 될 수 있는 타입을 사용해야 한다.
     var target: TargetService? = null

     // 이 경우, 널이 될 수 있는 타입의 메소드를 호출하려면 항상 널을 검사하는 코드가 필요하다.
     fun sayIt() = println(target!!.sayHello())
}
```
* 이러한 문제점을 해결하기 위해 lateinit 키워드를 붙여 프로퍼티를 나중에 초기화할 수 있다.
  * 이 때, 나중에 초기화할 프로퍼티는 항상 var로 선언되어야 한다.
  * 반면 val 프로퍼티는 final 필드로 컴파일되며, 생성자 내부에서 초기화되어야 한다.
* **lateinit 키워드는 프로퍼티를 나중에 초기화할 수 있게 하며, 클래스 내부의 다른 코드에서 널을 검사하는 코드를 작성하지 않아도 되도록 한다**.
  * 만약 프로퍼티에 접근하는 시점까지 초기화되지 않은 경우, UninitializedPropertyAccessException이 발생한다.
  * 해당 예외를 통해 어느 지점이 잘 못 되었는지 확인할 수 있으므로 단순한 NPE 보다 디버깅이 쉬워진다. 
```
fun main(args: Array<String>) {
     val lazy:LazyClass = LazyClass()
     lazy.target = TargetService()
     lazy.sayIt()
}
class TargetService {
     fun sayHello() = "hello world!"
}
class LazyClass {
     // 인스턴스화 시점과 프로퍼티 접근 시점이 다른 경우 var로 선언된 lateinit 프로퍼티를 사용한다.
     lateinit var target: TargetService

     fun sayIt() = println(target.sayHello())
}
```
* 일반적으로 lateinit 프로퍼티는 DI 프레임워크와 함께 사용되는 경우가 많다.
  * Kotlin은 호환성을 위해 lateinit 프로퍼티와 같은 가시성을 갖는 필드를 생성한다.

### 널이 될 수 있는 타입에 확장 함수 정의하기
* 널이 될 수 있는 타입에 대해 확장 함수를 정의할 수 있으며, 이는 null을 처리하기 위해 활용될 수 있다.
* 이 경우, **수신 객체 역할을 하는 인스턴스의 null 검사를 직접 작성하는 대신 확장 함수에서 이를 처리하도록 할 수 있다**.
  * 이러한 처리는 확장 함수에서만 가능하다.
    * 일반적인 함수의 경우, 동적 디스패치되므로 인스턴스의 null 여부를 메소드에서 검사하지 않는다.
* **널이 될 수 있는 타입에 대해 확장 함수를 정의한 경우, 널이 될 수 있는 값에 대해 확장 함수를 호출할 수 있다**.
* 널이 될 수 있는 타입에 확장 함수를 정의하는 방법은 다음과 같다.
  * 널이 될 수 있는 타입에 대해 정의된 확장 함수는 ?. 없이 호출이 가능하다.
  * **이 때, 확장 함수 내부에서는 this가 null이 될 수 있다**.
    * 때문에 **확장 함수에서는 반드시 명시적으로 null 검사 로직이 포함되어야** 한다.
```
fun main(args: Array<String>) {
     // null에 대해 확장 함수를 호출하더라도 문제가 발생하지 않는다.
     doSomething(null)
     doSomething("hello")
}

fun doSomething(string: String?) {
     // ?. 연산자 없이 호출이 가능하다.
     if(string.testString())
          println("true!")
     else
          println(string)
}
// String이 아닌 String?에 확장 함수를 정의한다.
// this == null인 경우에 대한 방어 로직을 작성해야 한다.
fun String?.testString(): Boolean = this == null || this.isBlank()
```
* 상술한 경우와 같이, **Kotlin에서는 null이 될 수 있는 타입의 확장 함수 내부에서는 this 키워드가 null이 될 수 있다**.
  * Java의 경우, 메소드가 정상 실행 되었다는 사실 자체가 메소드 내부에서 this가 null이 될 수 없음을 의미한다.
  * 때문에 Java에서는 메소드 내부에서의 this는 항상 null이 될 수 없다.
* 확장 함수를 정의할 필요가 있는 경우, 해당 함수를 널이 될 수 있는 타입에 대해 정의할지 신중히 고민한다.
  * 우선 널이 될 수 없는 타입에 대해 정의한 후, 널이 될 수 있는 타입에도 확장 함수의 적용이 필요한 경우에 전환하는 방법이 권장된다.
  * 이 경우, 반드시 확장 함수 내부에서 null을 처리하도록 작성해야 안전한 전환이 가능하다.
* **중요한 것은 string.testString() 형식의 함수 호출이 가능하다고 해서 string이 항상 널이 될 수 없는 타입이라는 것은 아니라는 점**이다.

### 동적 디스패치, 직접 디스패치
* 동적 디스패치: 객체지향에서, 객체의 동적 타입에 따라 적절한 메소드를 호출하는 방식이다.
  * 동적 디스패치를 처리하는 경우, 객체 별로 자신의 메소드에 대한 테이블을 저장하는 방식을 많이 사용한다.
  * 일반적으로 같은 클래스의 객체는 같은 메소드 테이블을 사용하므로, 메소드 테이블은 클래스마다 하나씩만 생성된다.
  * 각각의 객체는 자신의 클래스에 대한 참조를 통해 메소드 테이블을 찾아 실행한다.
* 직접 디스패치: 컴파일러가 컴파일 시점에 어떤 메소드가 호출될지 결정하여 코드를 생성하는 방식이다.

### 타입 파라미터와 null 가능성
* **클래스나 메소드 안에서 사용되는 타입 파라미터 T는 물음표가 붙지 않더라도 널이 될 수 있는 타입**이다.
  * **이는 Kotlin에서 타입 파라미터는 널이 될 수 있는 타입을 표시하기 위해 물음표를 붙여야 한다는 규칙의 유일한 예외**에 해당한다.
  * <T>는 null을 포함한 모든 타입을 의미한다.
```
fun main(args: Array<String>) {
     // T?가 아님에도 null을 받을 수 있다.
     doSomething(null)
}
fun <T> doSomething(input: T) {
     println(input.toString())
}
```
* **타입 파라미터가 null이 아닌 것을 확실히 하려면 널이 될 수 없는 타입 상한을 다음과 같이 지정**해야 한다.
  * **<T: Any>는 타입 파라미터 T가 널이 될 수 없는 타입 Any를 상속받는 타입이라는 점을 의미**한다.
  * 반면 <T: Any?>라면 다시 null을 파라미터로 전달할 수 있게 된다.
```
fun main(args: Array<String>) {
     // doSomething의 타입 상한이 지정되었으므로, 이제 null로 호출할 수는 없다.
     // doSomething(null)
     doSomething("null")
}

fun <T: Any> doSomething(input: T) {
     println(input.toString())
}
```