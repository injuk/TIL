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