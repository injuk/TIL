# Kotlin
## 2022-02-07 Mon

### Kotlin의 컬렉션 생성
* Kotlin은 Java의 컬렉션과 맵을 그대로 사용한다.
  * Kotlin은 자체 컬렉션을 제공하지 않으므로 Java와의 상호운용이 쉽다.
```
val set = hashSetOf(1, 2, 3)
val list = arrayListOf(1, 2, 3)
// Java 컬렉션은 자체적인 toString을 지원한다.
println(list)
val map = hashMapOf(1 to 1, 2 to 2, 3 to 3)
```
* Kotlin의 컬렉션은 Java의 것을 사용함으로써 상호운용성을 보장하면서도 더 확장된 기능을 제공한다.
```
val set = hashSetOf(1, 2, 3, 8)
println(set.last())
val list = arrayListOf(1, 20, 3)
println(list.maxOrNull())
```

### 파라미터에 이름 붙이기
* 함수의 파라미터 목록이 복잡한 경우, 혼동을 피하기 위해 파라미터에 이름을 붙여줄 수 있다.
  * 이 때, 함수 매개 변수 각각에 이름을 붙이거나 붙이지 않을 수 있다.
  * 아래의 예시는 모두 호출이 가능하다.
```
fun main(args: Array<String>) {
    // 함수에 전달되는 파라미터에 이름을 붙여 호출할 수 있다.
    complicatedFunction(num = 4, name = "EFG", false)
    complicatedFunction(3, name = "ABC", false)
    complicatedFunction(name = "Hey", bool = true, num = 4)
}

fun complicatedFunction(num:Int, name: String, bool: Boolean) {
    println("$num $name $bool")
}
```

### 디폴트 파라미터 값 설정
* Java에서는 일부 클래스의 메소드가 많이 오버로딩 되는 경우가 있다.
  * 여러 이유에서 오버로딩된 메소드가 추가되지만, 단적으로 이는 코드의 중복이다.
* **Kotlin에서는 함수의 선언에서 파라미터의 디폴트 값을 설정하는 것으로 많은 오버로딩을 피할 수 있다**.
  * 디폴트 값은 함수의 시그니쳐에 작성하며, 함수 호출시에는 순서를 고려해야 한다.
  * 순서를 고려하지 않는 경우, 파라미터에 이름을 붙여 호출해야 한다.
  * **디폴트 값을 지정하지 않은 매개 변수는 필수로 입력받아야 한다**.
```
fun main(args: Array<String>) {
    // bool 매개 변수는 디폴트 값으로 호출된다.
    complicatedFunction(0, "name")
    complicatedFunction(name = "name2")
    
    // 이름 붙인 파라미터가 아닌 아래와 같은 형식으로 호출할 수는 없다.
    // complicatedFunction("name")
    
    // 디폴트 값이 없는 매개변수는 반드시 작성해야 한다.
    // complicatedFunction()
}
// 매개 변수에 디폴트 값을 설정한 예시이다.
fun complicatedFunction(num:Int = 0, name: String, bool: Boolean = false) {
    println("$num $name $bool")
}
```
* **함수의 디폴트 파라미터 값은 함수를 호출하는 쪽이 아닌 함수 선언 쪽에서 지정**된다.
  * 따라서 이 **함수 정의 부분의 디폴트 값을 변경한 후 재컴파일하면, 함수를 호출하는 다른 코드들은 자동으로 바뀐 값을 적용**받는다.
* Java에는 디폴트 파라미터 값이라는 개념이 없으므로, Kotlin 함수를 Java에서 호출하는 경우 모든 인자를 명시해야 한다.
  * Java에서 Kotlin 함수를 자주 호출하는 경우, 이를 조금 더 편하게 하기 위해 @JvmOverloads 어노테이션을 활용할 수 있다.
  * @JvmOverloads 어노테이션이 붙은 함수는 Kotlin 컴파일러가 자동으로 마지막 파라미터부터 하나씩 생략한 Java 메소드를 오버로딩한다.
  * 이 때, 생략된 매개 변수에는 Kotlin 함수의 디폴트 값을 적용한다.

### static 유틸리티 클래스 없애기
* 지금까지 Kotlin 함수의 선언에는 주위 환경을 신경쓰지 않았다.
  * 이 함수들은 실제로는 클래스 내부에 선언되어야 하지 않을까?
* **그러나 Kotlin의 함수는 클래스 내부에 선언될 필요가 없다**. 
* Java에서는 모든 함수 코드를 클래스의 메소드로 작성해야 한다.
  * 그러나 **실전에서는 어떤 클래스에 포함시키기 어려운 코드들이 생기기 마련**이다.
* **결과, static 메소드만 갖고 어떤 상태나 인스턴스 메소드를 갖지 않는 클래스가 생기게 된다**.
  * 이러한 클래스는 일반적으로 Util 클래스라고 불리운다.
* **Kotlin에서는 이렇듯 무의미한 유틸리티 클래스를 작성할 필요가 없다**.
  * 대신 함수를 최상위 수준에 위치시키며, 클래스의 밖에 위치시킨다.
  * 해당 함수를 다른 패키지에서 사용하려면 패키지에 명시된 경로를 import하며, 유틸리티 클래스를 import할 필요는 없다.
* 그러나 **JVM은 클래스 내부의 코드만 실행시킬 수 있으므로, Kotlin 컴파일러는 최상위 함수를 정적 메소드로 갖는 새로운 클래스를 생성**한다.
  * **생성되는 클래스의 이름은 최상위 함수가 포함된 Kotlin 소스 파일의 이름 + Kt**와 같다.
  * 예를 들어, ComplicatedFunction.kt에 작성한 최상위 함수들은 public class ComplicatedFunctionKt의 클래스 메소드가 된다.
  * Java 코드에서 이를 호출하려면 패키지 명에 더해 유틸리티 클래스의 이름인 ComplicatedFunctionKt를 import해주어야 한다.
* 최상위 함수와 마찬가지로 프로퍼티 역시 최상위 프로퍼티로 작성할 수 있다.
* 이 때, 가능한 작성 방법은 다음과 같다.
  1. val 또는 var로 작성하기
     * 둘 모두 static 필드로 컴파일된다.
     * 나아가 변경 가능성에 따라 public final static getter와 setter가 추가로 컴파일된다.
     * 그러나 **용도로 미루어보았을 때 getter와 setter를 추가하는 것은 어색**할 수 있다.
  2. const val로 작성하기
     * public static final 필드로 컴파일되며, 더 자연스럽다.
     * 그러나 **원시 타입과 String 타입의 프로퍼티만 const로 지정**할 수 있다.
```
var num = 3
const val temp = 10
```

### 확장 함수와 확장 프로퍼티
* Java 코드와 Kotlin 코드를 자연스럽게 통합하는 것은 Kotlin의 핵심 목표 중 하나이다.
  * 이에 따라, 기존 Java 프로젝트에 Kotlin을 통합하는 경우에는 Kotlin으로 변환할 수 없거나 미처 변환하지 못한 Java 코드를 처리할 수 있어야 한다.
  * **기존 Java API를 재작성하는 일 없이 Kotlin이 제공하는 편리한 기능을 사용할 수 있도록 확장 함수 개념이 지원된다**.
* **확장 함수란, 어떤 클래스의 멤버 메소드인것처럼 호출할 수 있지만 실제로는 클래스 밖에 선언된 함수**이다.
* 확장 함수의 작성 방식은 일반적인 메소드와 같지만, 메소드 이름 앞에 함수가 확장할 클래스의 이름을 작성한다.
```
// Person.kt
data class Person(
    val name: String,
    val age: Int = 0,
)
// MyFirstKotlin.kt
fun main(args: Array<String>) {
    val person = Person(name = "Hello", age = 11)
    println(person.isAgeOdd())
}
// 확장 함수는 확장하려는 클래스.메소드이름 형태로 작성한다.
fun Person.isAgeOdd(): Boolean = this.age % 2 === 0
```
* 확장 대상이 되는 클래스는 수신 객체 타입, 확장 함수가 호출하는 대상이 되는 값(=객체)은 수신 객체라고 한다.
  * 상술한 예시에서, 수신 객체 타입은 Person이며 수신 객체는 person이 참조하는 인스턴스이다.
* 예를 들어, String 클래스에 확장 함수를 추가하는 경우를 가정하면 다음과 같은 큰 의미를 확인할 수 있다.
  1. String은 우리가 작성한 코드가 아님에도 메소드를 추가한 것처럼 동작시킬 수 있다.
  2. String은 우리가 소유한 클래스가 아님에도 메소드를 추가한 것처럼 동작시킬 수 있다.
* 확장 함수는 일반적인 인스턴스 메소드에서 this를 사용할 수 있는 것처럼 확장 함수에서도 this를 사용할 수 있다.
  * 또한 인스턴스 메소드에서 this를 생략할 수 있는 것처럼 확장 함수에서도 this를 생략할 수 있다.
```
fun main(args: Array<String>) {
    val person = Person(name = "Hello", age = 12)
    println(person.isAgeOdd())
    println(person.isAge(12))
}

// 마치 클래스 내부에 정의된 메소드인 것처럼 this를 생략할 수 있다.
fun Person.isAgeOdd(): Boolean = age % 2 === 0
// 마치 클래스 내부에 정의된 메소드인 것처럼 this를 활용할 수 있다.
fun Person.isAge(age: Int): Boolean = this.age == age
```
* 이렇듯 확장 함수 내부에서는 일반적인 인스턴스 메소드와 마찬가지로 수신 객체의 메소드와 프로퍼티를 사용할 수 있다.
  * 그러나 **클래스 내부에서 정의된 메소드와 달리 private, protected 멤버는 사용할 수 없다**.
  * **즉, 확장 함수는 캡슐화를 깨트리지 않는다**.
* 일반적으로 함수를 호출하는 측에서는 호출 대상이 확장 함수인지 멤버 메소드인지 구분할 수 없다.
  * 사실 이를 구분해야할 필요가 있는 경우도 많지 않다.

### import와 확장 함수
* 확장 함수는 정의된 동시에 전역에서 접근할 수 있지는 않으며, 필요시 import 문을 통해 가져와야 한다.
  * 이런 방식을 취하지 않으면 전역에서 동일한 확장 함수가 정의될 때마다 충돌할 수 있을 것이다.
* import는 일반적인 최상위 함수를 import하는 방법과 동일하며, 스타 임포트도 가능하다.
* as 키워드를 사용하면 다른 이름으로 가져와 활용할 수도 있다.
  * 하나의 파일 안에서 여러 다른 패키지의 동명의 메소드를 사용하는 경우, as 키워드를 통해 이름 충돌을 방지할 수 있다.
  * Kotlin 문법 상 확장 함수는 짧은 이름을 사용하므로, as를 사용하여 이름을 바꾸는 것이 확장 함수의 이름 충돌을 방지할 수 있는 유일한 방법이다.
```
// Funcs.kt
package kotlinStudy.diff.test.deep.depth

fun String.isHelloWorld(): Boolean = this === "HelloWorld"

// MyFirstKotlin.kt
// 다른 패키지에 있는 함수를 호출하려면 반드시 import해야 한다.
import kotlinStudy.diff.test.deep.depth.isHelloWorld

// 위 문장을 제거하고 아래 문장을 주석 해제하면, isHelloWorld 메소드를 test라는 이름으로 사용할 수 있다.
// import kotlinStudy.diff.test.deep.depth.isHelloWorld as test

fun main(args: Array<String>) {
    println("HelloWorld".isHelloWorld())
    println("ByeWorld".isHelloWorld())
}
```
* 내부적으로 확장 함수는 수신 객체를 첫 번째 인자로 받는 정적 메소드로 변환된다.
  * Java에서 확장 함수를 사용하려면 정적 메소드를 호출하며 수신 객체를 인자로 넘겨주기만 하면 된다.
  * 즉, **확장 함수는 정적 메소드 호출에 대한 문법적인 편의 기능에 지나지 않는다**.
* 클래스 이름은 최상위 함수와 마찬가지로 소스 코드 파일 명에 따라 결정되며, 상술한 isHelloWorld의 경우 다음과 같이 변환된다.
```
FuncsKt.isHelloWorld("HelloWorld");
FuncsKt.isHelloWorld("ByeWorld");
```

### 확장 함수와 오버라이드
* **확장 함수는 정적 메소드와 같은 특징을 지니므로, 하위 클래스에서 오버라이드할 수 없다**.
* 일반적으로, 부모 클래스의 메소드는 자식 클래스에서 오버라이드할 수 있다.
  * 또한 부모 클래스의 타입 변수에 자식 클래스의 변수를 대입할 수 있다.
  * 이 경우, 메소드를 호출하면 자식 클래스의 메소드가 호출된다.
* **이는 실행 시점에 객체의 실제 타입에 따라 동적으로 호출될 대상 메소드를 결정하는 방식이므로, dynamic dispatch에 해당**한다.
  * 반면 컴파일 시점에 정해진 메소드를 호출하는 방식은 static dispatch이다.
  * **프로그래밍 언어에서, dynamic은 런타임을 / static은 컴파일 시점을 의미**한다.
* 확장 함수는 동적으로 동작하지 않으며, 애초에 클래스의 일부가 아니다.
  * **확장 함수는 클래스 밖에 선언된다**.
* 따라서 메소드 시그니쳐가 같은 함수를 자식 클래스에서 정의하더라도 실제로는 수신 객체로 지정한 변수의 정적 타입에 의해 호출될 함수가 결정된다.
  * **val animal: Animal = Bird()의 경우, Animal이 정적 타입이며 이 변수에 저장된 실제 객체인 Bird는 동적 타입이 된다**.
  * 즉, 확장 함수는 항상 정적 타입을 따르며 dynamic dispatch가 아닌 static dispatch로 동작한다.
    * **Kotlin은 호출될 확장 함수를 정적으로 결정한다**.
```
fun main(args: Array<String>) {
    val animal: Animal = Bird()
    // 런타임 시점에 변수가 참조하는 대상의 실제 타입인 Bird의 오버라이드된 메소드로 결정된다.
    animal.move()
    
    // 호출될 확장 함수는 수신 객체의 정적 타입인 Animal에 따라 컴파일 시점에 결정된다.
    // 때문에 이 경우 Bird.stop이 아닌 Animal.stop이 호출된다.
    animal.stop()
}

open class Animal() {
    open fun move() = println("Animal moving")
}

fun Animal.stop() = println("Animal stopping")

class Bird: Animal() {
    override fun move() = println("Bird flying")
}

fun Bird.stop() = println("Bird stopping")
```
* 확장 함수가 실제로 컴파일되는 방식을 알고 있다면 이해하기 쉽다.
  * animal.stop은 [파일명]Kt.stop(animal) 형식의 정적 메소드가 된다.
  * 이 경우, animal이 참조하는 정적 타입은 알 수 있지만 런타임에 결정되는 실제 타입은 알 수 없다.
  * Java 역시 호출될 static 메소드는 static dispatch 방식으로 결정된다.
* **동명의 확장 함수와 멤버 함수가 존재할 수 있으며, 호출시 멤버 함수가 우선적으로 호출**된다.
```
fun main(args: Array<String>) {
    val animal = Animal()
    animal.move() // dynamic move
}

fun Animal.move() = println("static move")

class Animal {
    fun move() = println("dynamic move")
}
```

### 확장 프로퍼티
* 확장 프로퍼티는 상태를 저장할 방법이 없으므로, 실제로는 어떠한 상태도 가질 수 없다.
* 확장 프로퍼티 역시 일반 프로퍼티와 동일하게 작성하지만, 수신 객체 타일을 추가한다.
```
fun main(args: Array<String>) {
    val animal = Animal();
    // 확장 프로퍼티인 greeting의 getter
    println(animal.greeting)
    // 확장 프로퍼티인 greeting의 setter
    animal.greeting = "not animal"
    println(animal.greeting)
}

// val인 경우 setter를 설정할 수 없으므로, 학습을 위해 var로 선언
var Animal.greeting: String
get() = "Hello, $name!"
set(name: String) {
    this.name = name
}

class Animal (var name: String = "animal")
```
* Java에서 확장 프로퍼티를 사용하는 경우, [소스 코드 파일명]Kt.getGreeting(animal)과 같은 형식이 될 것이다.