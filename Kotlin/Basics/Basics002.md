# Kotlin
## 2022-02-05 Sat

### Hello world
* Kotlin Hello world는 다음과 같이 작성한다. 
  1. 함수 키워드는 fun이다.
  2. class 없이 함수를 최상위 위치에 작성할 수 있다.
  3. 파라미터 이름 뒤에 파라미터의 타입을 작성한다.
  4. 배열도 일반적인 클래스처럼 처리하며, Java와 달리 Kotlin에는 배열 처리를 위한 문법이 없다.
  5. System.out.println를 println과 같이 작성한다.
     * Kotlin에는 표준 Java 라이브러리를 이토록 쉽게 사용할 수 있도록 래퍼가 제공된다.
```
package kotlinStudy

fun main(args:Array<String>) {
    println("Hello world")
}
```

### Kotlin의 함수
* Kotlin의 함수 선언 형태는 다음과 같다.
```
fun [함수이름]([매개 변수 목록]): [반환형] {
  [함수 본문]
}
```
* 함수의 **반환형이 void인 경우 반환형을 생략**하며, void가 아닌 경우 매개 변수 목록 뒤에 작성한다.
```
fun main(args:Array<String>) {
    printStr(max(3, 4).toString())
}
// 반환형이 Int인 함수
fun max(a: Int, b: Int): Int {
    return if(a>b) a else b // Kotlin에서 if는 값을 만드는 '식'이다. 
}
// 반환형이 void인 함수
fun printStr(str: String) {
    println(str)
}
```
* Kotlin의 함수는 다시 다음의 두 가지로 나뉠 수 있다.
  1. 블록이 본문인 함수
     * 상술한 함수와 같이 중괄호로 둘러 쌓인 함수이다.
     * void형이 아닌 경우, 반드시 반환 타입과 return 문이 사용되어야 한다.
  2. 식이 본문인 함수
     * 상술한 함수는 다음과 같이 간략히 작성할 수 있다.
```
// 반환형과 return문, 함수를 둘러싼 중괄호가 제거된다.
fun max(a: Int, b: Int) = if(a > b) a else b
// void형 함수도 가능하다.
fun printStr(str: String) = println(str);
```
* **식이 본문인 함수를 작성할 수 있는 이유는 Kotlin 컴파일러가 타입 추론을 지원하기 때문**이다.
  * 즉, 컴파일러가 함수 본문 식을 분석하여 함수 반환 타입을 정해주기 때문이다.
  * **타입 추론이란, 컴파일러가 타입을 분석하여 개발자 대신 프로그램 구성 요소의 타입을 지정하는 기능*이다.
* 블록이 본문인 함수는 왜 반환 정보를 강제할까?
  * 함수가 길어지는 경우 return 문은 여럿이 존재할 수 있다.
  * 이 경우, 반환 타입과 return 문을 강제함으로써 **함수가 어느 시점에 무엇을 반환하는지 쉽게 알 수 있어야 하기 때문**이다.

### Kotlin의 변수
* 변수 앞에 타입을 선언하는 Java와 달리, **Kotlin에서는 타입 지정을 생략할 수 있다**.
  * 타입으로 변수 선언을 시작하는 경우, 식과 변수의 선언을 구분할 수 없었기 때문이다.
  * 식이 본문인 함수와 마찬가지로, 타입 지정이 누락된 경우 Kotlin 컴파일러가 타입 추론을 통해 변수의 타입을 지정한다.
* 변수 선언시 다음과 같이 val [변수명]: [변수형] = [값]로 선언한다.
  * 변수 선언 후 **초기화를 생략한 경우, 타입 추론이 불가하므로 반드시 변수형을 명시**한다.
```
// 타입은 생략할 수 있다.
val maybeIAmString = "Hello world!"
val maybeIAmInteger = 3
// 타입을 생략하지 않는 경우, : 타입 형태로 작성한다.
val iAmDefinitelyInteger: Int = 15
// 변수의 선언과 초기화를 함께하지 않는 경우에는 반드시 타입을 지정한다. 
val whoAmI: Boolean
whoAmI = false
```
* 변수 선언시 사용 가능한 키워드는 다음과 같다.
  1. val: 변경 불가능한 참조를 저장한다.
     * 초기화 한 후에는 값을 변경할 수 없다.
     * Java의 final 변수에 해당한다.
     * final 변수와 달리, **초기화와 대입을 반드시 함께할 필요는 없다**.
  2. var: 변경 가능한 참조를 저장한다.
     * Java의 일반 변수에 해당한다.
```
// val로 선언된 변수는 참조를 변경할 수 없다.
val final = 10
// final = 20

// var로 선언된 변수는 참조의 변경이 가능하다.
var notFinal = 10
notFinal = 20
```
* **val로 선언된 변수는 블록을 실행할 때 정확히 한 번만 초기화되어야 한다**.
  * 반면 if 문과 같이 **오직 하나의 초기화 문장만 실행될 수 있는 경우, 조건에 따라 여러 값으로 초기화시킬 수 있다**.
```
val pleaseInitMe: String
// 선언과 초기화가 다른 문장에서 동작할 수 있다.
if(true)
    pleaseInitMe = "I am true!"
else
    pleaseInitMe = "I am false!"
```
* **val로 선언된 변수의 참조는 불변하지만, 참조된 객체의 내부는 변경될 수 있다**.
* **var로 선언된 변수의 값은 변경될 수 있지만, 타입은 고정되어 변경되지 않는다**.
  * **변수의 타입은 선언 시점의 초기화 식으로부터 컴파일러에 의해 추론**된다.
  * 이후 **변수에 다른 값을 대입할 경우, 컴파일러는 추론한 값을 기준으로 대입문의 타입을 검사**한다.
```
var iAmInteger = 10
iAmInteger = 20
// type mismatch!
// iAmInteger = "30"
```
* **기본적으로 모든 변수를 val로 선언하여 불변 변수로 선언하고, 꼭 필요할 때에만 var로 선언**한다.
  * **변경 불가능한 참조와 변경 불가능한 객체를 순수 함수와 함께 사용하여 함수형 코드에 가까워질 수 있기 때문**이다.

### 문자열 템플릿
* 여러 스크립트 언어의 기능과 유사하게, Kotlin에서도 문자열 내부에서 변수를 사용할 수 있다.
* 이러한 문자열 템플릿 기능은 문자열 내부에서 $변수 형식으로 사용한다.
  * 만약 진짜 $기호를 활용하고 싶은 경우라면 \ 기호를 통해 이스케이프 시킨다.
* **컴파일러는 모든 식을 정적으로 컴파일러 시점에 검사**하므로, 존재하지 않는 변수를 문자열 템플릿으로 사용할 수는 없다.
```
val name = "ingnoh"
print("$name has 1\$")
// 컴파일 오류: 선언되지 않은 변수는 사용할 수 없다.
// print("$name2 has 1\$")
```
* 복잡한 식을 문자열 템플릿도 사용하려는 경우, ${} 내부에 작성한다.
```
val one = 1
val two = 2
val three = 3

println("${one + two * three}") // 7
println("\${one + two * three}") // ${one + two * three}
```
* ${}로 표현된 문자열 템플릿 내부에서는 String 리터럴인 쌍따옴표를 사용할 수 있다.
```
println("${if(one > 0) one + two * three else "one is zero!"}")
```
* 기본적으로 가독성 측면에서 $만 사용하는 것보다 ${}로 작성하는 습관을 들이는 것이 바람직하다.

## 2022-02-06 Sun
### Kotlin의 클래스
* **값 객체란, 코드 없이 데이터만 저장하기 위한 클래스**이다.
* Kotlin에서는 이러한 값 객체를 간결하게 표현할 수 있다.
```
class Person(val name: String)
```
* Kotlin의 기본 접근제어자는 public으로 적용되므로, public을 생략할 수 있다.
* 클래스의 목적은 데이터를 캡슐화하고, 캡슐화된 데이터를 다루는 코드를 하나의 주체 아래에 두는 것이다.
  * Java에서, 데이터는 필드에 저장된다.
  * Java에서, 데이터에 접근하거나 변경할 수 있도록 getter나 setter와 같은 접근자 메소드를 생성할 수 있다.
  * **Java에서, 필드와 접근자 메소드를 합쳐 프로퍼티라고 부른다**. 
* **Kotlin에서, 프로퍼티는 언어 자체적인 기능으로 제공**된다.
  * 클래스의 프로퍼티 선언은 읽기 전용 또는 변경 가능성에 따라 val과 var 키워드로 정의된다.
  1. val: **읽기 전용이며, Kotlin은 비공개 필드와 공개된 getter를 자동으로 생성**한다.
  2. var: **변경 가능한 프로퍼티이며, Kotlin은 비공개 필드와 공개된 getter와 setter를 자동으로 생성**한다.
* 이렇듯 Kotlin의 프로퍼티 선언은 프로퍼티와 관련이 있는 접근자 메소드를 선언하는 것이다. 
  * **이 과정에서 비공개 필드와 getter, setter와 같은 간단한 디폴트 접근자 구현이 제공**된다.
```
private class Animal(
    val name: String,
    var age: Int
)

fun main(args:Array<String>) {
    // new 키워드 없이 생성자를 바로 호출한다.
    val dog = Animal("dog", 1)
    // 읽기 전용 프로퍼티 name에 대한 setter는 생성되지 않는다.
    // dog.name = "temp" 
    dog.age = 10
    println(dog.name)
    println(dog.age)
}
```
* 상술한 Animal 클래스에는 비공개 필드가 포함되고, 생성자가 각 필드를 초기화하며, 각 필드에 접근하기 위한 getter와 setter가 제공된다.
* Kotlin에서 자동으로 제공되는 getter의 특징은 다음과 같다.
  1. getXXX의 형식을 사용하지 않고, 바로 프로퍼티에 접근한다.
     * 상술한 예시에서, dog.getName()이 아닌 dog.name을 호출하는 것을 확인할 수 있다.
     * 이렇듯 프로퍼티의 이름을 바로 사용해도 Kotlin은 자동으로 getter를 호출한다.
  2. 예외적으로 **프로퍼티 명이 is로 시작하는 경우, Java 코드에서는 getIsXXX가 아닌 프로퍼티 명을 그대로 사용**한다.
     * 예를 들어, 프로퍼티 명이 isChild라면 Java에서는 getIsChild가 아닌 isChild로 호출해야 한다.
* Kotlin에서 자동으로 제공되는 setter의 특징은 다음과 같다.
  1. var로 선언되어 변경 가능한 프로퍼티인 경우에만 setter가 자동으로 제공된다.
  2. setter 역시 getter와 같은 형식으로 사용하여, 프로퍼티에 바로 접근한다.
     * 상술한 예시에서, setAge의 형식이 아닌 dog.age로 바로 접근하는 것을 확인할 수 있다.

### 커스텀 접근자 정의하기
* 대부분의 프로퍼티에는 프로퍼티의 값을 저장하기 위한 필드가 있으며, 이를 backing field라고 부른다.
* 필요한 경우, 프로퍼티의 값을 그때그때 계산하는 접근자 메소드를 정의할 수 있다.
```
private class Animal(
    val name: String,
    var age: Int
) {
    // 이미 정의된 프로퍼티를 사용자 정의할 수는 없다.
    // val name: String
    
    // 커스텀 접근자
    val isLessThanTen: Boolean
    get() = age < 10 // get() { return age < 10 }과 같은 블록 형태로도 작성할 수 있다.
}

fun main(args:Array<String>) {
    val dog = Animal("dog", 1)
    println(dog.isLessThanTen) // true
    
    dog.age = 10
    // 커스텀 접근자는 val이지만 값을 호출한 시점에 계산하므로, 매 호출마다 결과가 바뀔 수 있다.
    println(dog.isLessThanTen) // false
}
```
* **사용자가 직접 정의한 isLessThanTen 프로퍼티는 값을 저장하는 비공개 필드를 따로 두지 않는다**.
  * 사용자가 자체적으로 구현한 getter만이 존재하며, 클라이언트가 프로퍼티에 접근할 때마다 getter가 그 값을 계산한다.
  * 프로퍼티와 필드는 같은 의미가 아닌 것을 명심하자. 상술한 isLessThanTen은 사용자 정의된 프로퍼티이지, 필드가 아니다!
* 일반적으로, 상술한 기능의 구현을 위해서 파라미터가 없는 함수와 커스텀 접근자 메소드 중 하나를 선택할 수 있다.
  * 이 경우, **구현하려는 기능이 클래스의 프로퍼티와 관련이 있다면 시멘틱상 커스텀 접근자가 권장**된다.
  * 두 방식의 구현 및 성능 상의 차이는 없다.

### Kotlin의 소스 코드 구조
* Java의 경우 모든 클래스를 패키지 단위로 관리한다.
* Kotlin에도 패키지 개념이 존재하며, 모든 Kotlin 파일 맨 앞에 package 문을 작성할 수 있다.
  1. **해당 파일의 모든 클래스, 함수, 프로퍼티 선언은 해당 패키지에 속하게 된다**.
  2. **같은 패키지에 속한 클래스, 함수, 프로퍼티는 다른 파일에서 정의되었더라도 직접 사용**할 수 있다.
* 다른 패키지에서 정의된 클래스, 함수, 프로퍼티는 import 키워드를 통해 불러와야 한다.
  * Java와 마찬가지로 import 문은 파일의 최상단에 위치해야 한다.
* 최상위 함수의 경우, 클래스 임포트와 마찬가지 방식으로 import 키워드를 통해 가져올 수 있다.
  * 최상위 함수의 경우 함수의 이름을 써서 import 할 수 있다.
* 패키지 이름 뒤에 *을 추가하는 경우, 패키지 내부에 선언된 모든 내용을 임포트할 수 있다.
  * **이 경우, 패키지 내부의 클래스 및 함수와 프로퍼티 모두를 가져온다**.
```
// kotlinStudy/diff/Funcs.kt
package kotlinStudy.diff

fun hello(str: String) = println(str)
fun getMax(num1: Int, num2: Int) = if(num1 > num2) num1 else num2

// kotlinStudy/MyFirstKotlin.kt
import kotlinStudy.diff.getMax // 같은 파일의 함수도 각각 import한다.
import kotlinStudy.diff.hello  // 같은 파일의 함수도 각각 import한다.
// import kotlinStudy.diff.* // star import를 통해 한 번에 import할 수 있다.

private class Animal(
    val name: String,
    var age: Int
)

fun main(args:Array<String>) {
    val dog = Animal("dog", 1)

    hello(dog.name)
    println(getMax(1, dog.age))
}
```

### Java와 Kotlin의 패키지 구조 차이
* Java에서는 패키지 구조와 디렉토리 계층 구조를 일치화시켜야 한다.
  * Kotlin에서는 패키지 구조와 관계 없이 어느 디렉토리든 소스 코드를 위치시킬 수 있다.
* Java에서는 소스 코드의 이름과 작성된 클래스의 이름이 같아야 한다.
  * Kotlin에서는 소스 코드의 이름과 클래스의 이름이 일치하지 않아도 무방하다.
  * Kotlin에서는 하나의 소스 코드 내부에 여러 클래스가 포함될 수 있다.
* **대부분의 경우, Kotlin의 패키지 구조 역시 상호운용성을 보장하도록 Java의 관례를 따르는 것이 권장**된다.
* 그러나 **여러 작은 클래스를 하나의 파일에 작성하는 것은 주저하지 않아야 한다**.
```
// 실제 디렉토리는 kotlinStudy/diff/Funcs.kt이다.
// 다른 소스 코드에서 import시 아래에 명시된 패키지 구조만 사용하면 정의된 선언을 사용할 수 있다.
package kotlinStudy.diff.test.deep.depth

fun hello(str: String) = println(str)

fun getMax(num1: Int, num2: Int) = if(num1 > num2) num1 else num2

// 여러 public 클래스를 하나의 파일에 작성할 수 있으며, 파일의 이름과 클래스의 이름을 일치시킬 필요가 없다.

public class Test(val name: String)
public class Temp(val age: String)
public class A(val a: String)
```