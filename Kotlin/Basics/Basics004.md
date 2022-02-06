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