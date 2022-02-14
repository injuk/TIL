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