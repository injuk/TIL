# Kotlin
## 2022-02-06 Sun

### enum
* Kotlin의 enum은 다음과 같이 작성한다.
  * enum은 소프트 키워드이며, class 키워드 앞에서만 의미를 갖는다.
  * 소프트 키워드는 변수 이름 등에 사용할 수 있다.
```
enum class MyEnum {
    ONE, TWO, THREE
}
```
* Java와 마찬가지로 enum은 단순히 값만 열거하는 존재가 아니며, 프로퍼티와 메소드를 포함할 수 있다.
  1. enum 내부에 메소드를 정의하려면, 마지막 상수와 메소드 사이에 반드시 세미콜론을 표기한다.
     * 이는 **유일하게 Kotlin에서 세미콜론이 필수인 문법**이다.
  2. enum 클래스에 프로퍼티를 정의하려면 일반적인 class 표기법과 같은 형태로 정의한다.
     * 정의된 프로퍼티는 var일 수 있으며, 변경 가능성에 따라 getter와 setter가 함께 정의된다.
  3. enum 클래스에 프로퍼티를 정의했다면, 각 상수 값이 모든 프로퍼티 값을 가질 수 있도록 괄호로 표기한다.
```
enum class MyEnum( // enum의 프로퍼티 정의
    val x: Int,
    val y: Int,
    val z: Int,
) {
    ONE(1, 1, 1), // 모든 상수 목록은 대응되는 프로퍼티 값을 가져야 한다.
    TWO(2, 2, 2),
    THREE(3, 3, 3); // 상수 목록과 메소드 사이에 세미콜론을 작성하여 구분

    fun printXyz() = println("$x $y $z")
}

fun main(args:Array<String>) {
    MyEnum.ONE.printXyz()
}
```

### when
* Kotlin의 when은 Java의 switch와 대응된다.
* **if문과 마찬가지로 when 역시 값을 반환하는 식**이다.
* when은 인자를 작성하거나 작성하지 않을 수 있다.
  * 아래의 예시는 인자를 작성한 예시이며, when(...){ ... }의 형태가 된다.
* Java의 switch와 달리, when은 break를 명시하지 않아도 된다.
* **when 내부 분기에서 여러 경우를 하나의 결과로 대응시키고 싶다면 콤마를 사용**한다.
```
enum class Numbers {
    ONE, FIRST, TWO, SECOND, THREE, FOUR, FIVE,
}

// when은 값을 반환하는 식이므로, 식이 본문인 함수 형태로 사용할 수 있다.
// 블록이 본문인 함수 형태로 사용하려면 return when(nunbers) { //... } 형태가 된다.
fun getIntFromNumbers(numbers: Numbers) =
    when(numbers) {
        // 여러 분기를 하나의 결과로 대응시키기 위해 콤마를 사용한다.
        Numbers.ONE, Numbers.FIRST -> 1
        // 명시적으로 break 문을 작성하지 않아도 좋다.
        Numbers.TWO, Numbers.SECOND -> 2
        Numbers.THREE -> 3
        Numbers.FOUR -> 4
        Numbers.FIVE -> 5
    }

fun main(args:Array<String>) {
    println(getIntFromNumbers(Numbers.SECOND))
}
```
* 다른 패키지의 enum을 import하는 경우, 상수 값 전체를 import하는 것으로 아래와 같이 축약할 수 있다.
```
// enum 클래스 자체와 enum 상수를 각각 import해야 한다.
import kotlinStudy.diff.test.deep.depth.Numbers
import kotlinStudy.diff.test.deep.depth.Numbers.*

fun getIntFromNumbers(numbers: Numbers): Int {
    return when (numbers) {
        ONE, FIRST -> 1
        TWO, SECOND -> 2
        THREE -> 3
        FOUR -> 4
        FIVE -> 5
    }
}
```
* **숫자 리터럴과 enum 상수만을 사용할 수 있는 Java와 달리, Kotlin의 when은 객체를 허용**한다. 
  * when 내부에서 대응되는 결과가 없는 경우, else 문을 사용한다.
    * **else 문은 함수의 반환형과 같은 값을 반환할 수 있지만, Exception을 던질 수도 있다**.
  * 임의의 객체를 매개변수로 받는 when은 반드시 else 문을 작성한다.
* 다음의 예제에서 사용하는 setOf 메소드는 Kotlin의 표준 라이브러리에 포함된 래퍼로, 인자로 전달받은 객체들을 포함하는 Set으로 만들어준다.
```
fun getSum(num1: Numbers, num2: Numbers): Int {
    return when (setOf(num1, num2)) {
        setOf(ONE, TWO) -> 3
        setOf(ONE, THREE) -> 4
        setOf(ONE, FOUR), setOf(TWO, THREE) -> 5
        setOf(TWO, FOUR) -> 6
        setOf(THREE, FOUR) -> 7
        else -> -1 // 대응되는 결과가 없는 경우 else 분기가 실행된다.
    }
}

fun main(args:Array<String>) {
    println(getSum(THREE, FOUR))
    println(getSum(FIVE, ONE))
}
```
* 인자를 작성하지 않는 when의 경우, when { ... } 형태로 작성한다.
  * 아래의 예시는 가독성을 희생하는 대신 불필요한 Set 객체를 양산하지 않는다.
  * **when에 아무런 인자를 작성하지 않으려면 분기의 조건은 반드시 Boolean 식이어야 한다**.
```
fun getSum(num1: Numbers, num2: Numbers): Int {
    // when의 인자에 아무것도 작성하지 않는 대신, 분기 조건에 가능한 모든 식을 작성한다.
    return when {
        (num1 == ONE && num2 == TWO) || (num1 == TWO && num2 == ONE) -> 3
        (num1 == ONE && num2 == THREE) || (num1 == THREE && num2 == ONE)  -> 4
        (num1 == ONE && num2 == FOUR) || (num1 == FOUR && num2 == ONE) ||
        (num1 == TWO && num2 == THREE) || (num1 == THREE && num2 == TWO) -> 5
        (num1 == TWO && num2 == FOUR) || (num1 == FOUR && num2 == TWO) -> 6
        (num1 == THREE && num2 == FOUR) || (num1 == FOUR && num2 == THREE) -> 7
        else -> -1
    }
}

fun main(args:Array<String>) {
    println(getSum(THREE, FOUR))
    println(getSum(FIVE, ONE))
}
```

### 스마트 캐스트
* Kotlin의 스마트 캐스트는 타입 검사와 타입 캐스트를 조합한 기능이다.
* Java 방식의 경우, if와 instanceof 및 명시적 형변환을 조합하여 작업하는 경우가 있다.
* Kotlin의 경우, is 키워드는 instanceof와 같은 변수 타입 검사를 실행한다.
  * **일단 한 번 검사를 통과하고 나면 타입 캐스트가 자동으로 수행되어 마치 처음부터 해당 타입이었던 것처럼 사용할 수 있다**.
  * 실제로는 Kotlin 컴파일러가 캐스팅을 수행해준다.
* Kotlin에서 명시적 형변환이 필요한 경우, 아래와 같이 작성한다.
  * val cast = before as After, before를 After 타입으로 캐스팅하여 cast로 참조한다.
```
// 인터페이스 Num은 아무런 메소드를 가지지 않고, 공통 타입으로서의 역할만을 수행한다.
interface Num
class Leaf(val number: Int): Num
class Sum(val left: Num, val right: Num): Num

fun eval(node: Num): Int {
    if(node is Leaf) {
        // Java 스타일로 작성되었으며, 불필요한 타입 캐스팅을 수행한다.
        val number = node as Leaf
        return number.number
    }
    if(node is Sum)
        // Kotlin의 스마트 캐스트를 활용하였으며, 타입 검사 이후 마치 원래 Sum 타입이었던 것처럼 동작한다.
        return eval(node.left) + eval(node.right)

    else
        throw IllegalArgumentException()
}
```
* 위 함수는 Kotlin에서의 if문이 값을 반환하는 식이라는 점을 활용하여 다음과 같이 수정될 수 있다.
  * Kotlin의 if 문은 자체적으로 값을 반환할 수 있으므로, Java와 달리 3항 연산자가 존재하지 않는다.
```
// 재귀 함수에서는 타입 추론이 동작하지 않을 수 있으므로, 함수의 반환형을 작성해주는 것이 좋다.
fun eval(node: Num): Int =
    if(node is Leaf)
        node.number // 마지막 식인 node.number의 값을 if문이 그대로 반환한다.
    else if(node is Sum)
        eval(node.left) + eval(node.right)
    else
        throw IllegalArgumentException()
```
* when을 사용하여 더 깔끔하게 수정할 수도 있다.
```
fun evalWithWhen(node: Num): Int =
    when(node) {
        // node is Leaf가 아닌 is Leaf로 곧바로 작성한다.
        is Leaf -> node.number
        is Sum -> evalWithWhen(node.left) + evalWithWhen(node.right)
        else -> throw IllegalArgumentException()
    }

fun main(args: Array<String>) {
    val result = evalWithWhen(Sum(Sum(Leaf(1), Leaf(2)), Leaf(4)))
    println(result)
}
```

### if와 when 분기별 블록의 사용
* if나 when 모두 분기 내부의 로직이 복잡해지는 경우에 블록을 사용할 수 있다.
  * **이 경우, 블록의 마지막 문장이 블록 전체의 결과가 된다**.
* 블록의 마지막 식이 블록의 결과라는 규칙은 값을 만드는 블록 모두에 대해 성립한다.
* 반면, 해당 규칙한 함수에는 성립하지 않는다.
  1. 식이 본문인 함수는 블록을 본문으로 가질 수 없다.
  2. **블록이 본문인 함수는 반드시 return 문을 명시해야 한다**.

### while 문
* Kotlin의 while, do while문은 Java와 다른 것이 없으며, Kotlin에서 추가된 새로운 기능도 없다.
  * 즉, 조건이 참인 동안 본문을 반복한다.

### Kotlin의 범위와 for문을 활용한 수열
* Kotlin은 Java의 for(int i=0;i<10;i++) 형태의 for문이 존재하지 않으며, **대신 범위 개념을 사용**한다.
* **범위는 기본적으로 두 값으로 이루어진 폐구간이며, 정수..정수 형태로 작성**된다.
  * 폐구간이므로 끝 값 또한 범위에 포함된다.
```
fun main(args: Array<String>) {
    // 1부터 10까지의 정수 범위를 선언하며, 10 역시 범위에 포함된다.
    val range = 1..10
    for(num in range)
        println(num)
    // 범위 리터럴로 사용할 수도 있다.
    for(num in 1..2)
        println(num)
}
```
* 상술한 예시는 범위에 속한 값을 순서대로 순회하는 수열의 예시이다.
* **수열을 위한 범위는 다음과 같은 요소를 포함**할 수 있다.
  1. .. : 정수 순방향 범위를 나타낸다.
     * 순방향 수열의 기본 증가값은 step 1이다.
     * 'A'..'Z'와 같이 문자 타입에도 적용할 수 있다.
  2. downTo: 정수 역방향 범위를 나타낸다.
     * 역방향 수열의 기본 증가값은 step -1이다.
  3. until: 끝 값을 포함하지 않는 범위를 정의한다.
  4. step: 각 순회에서 증가할 절대값을 나타낸다.
     * **step의 값은 절대값이므로 증감 방향에 영향을 주지 않는다**.
```
fun main(args: Array<String>) {
    // 정수 순방향으로 1부터 시작하여 3씩 증가한다.
    val range = 1..10 step 3
    for(num in range)
        println(num)

    // 범위 리터럴로 작성할 수 있으며, 정수 역방향으로 10부터 시작하여 2씩 감소한다.
    for(num in 10 downTo 1 step 2)
        println(num)
        
    // 0부터 시작하여 step 1로 10까지 진행하되, 10은 범위에 포함하지 않는다.
    for(num in 0 until 10)
        println(num)
}
```

### for문을 활용한 Map 순회
* Kotlin for 반복문은 Map을 순회하는데에도 사용할 수 있다.
* 아래의 예시에서, 다음과 같은 Kotlin의 특징을 확인할 수 있다.
  1. 문자형 범위도 사용이 가능하다.
  2. hashMap.put(값)대신 hashMap[key] = value 형태를 사용할 수 있다.
     * 물론 값을 가져올 때에도 hashMap[key] 형태를 사용할 수 있다.
  3. Map 순회시 for 문의 범위 대신 hashMap 자체를 작성할 수 있다.
  4. Map 개별 요소 순회시 hashMap.get(key) 대신 (변수1, 변수2)의 형태로 바로 값을 받아올 수 있다.
     * 즉, **Kotlin은 Map에 대해 구조 분해 할당을 지원**한다.
```
fun main(args: Array<String>) {
    val hashMap = HashMap<Char, String>()

    for(char in 'A'..'Z')
        hashMap[char] = Integer.toHexString(char.code)

    for((key, value) in hashMap)
        println("$key : $value")
}
```
* **list에서도 withIndex 메소드를 활용한 구조 분해 할당을 활용할 수 있다**.
  * 이 경우, 인덱스와 요소를 각각의 변수로 할당할 수 있다.
  * 따라서 인덱스의 사용이 필요한 경우 인덱스용 변수를 별도로 선언할 필요가 없다.
```
fun main(args: Array<String>) {
    val list = arrayListOf<Int>()
    for(num in 1..10)
        list.add(num)

    for((index, element) in list.withIndex())
        println("list[$index]: $element")
}
```

### in 연산자를 활용한 요소 검사
* **in 연산자를 활용하여 어떤 값이 범위에 속하는지 검사할 수 있다**.
  * !in을 활용하면 속하지 않는지 검사할 수 있다.
* 또한 컬렉션에 값이 포함되는지 검사하기 위해서도 사용할 수 있다.
```
fun main(args: Array<String>) {
    // 각각 true, false, false가 반환된다.
    println(10 in 1..10)
    println(10 in 1 until 10)
    println(1 !in 1..10)
    
    // 컬렉션 내부에 값이 포함되는지 검증할 수 있다.
    println(1 in arrayListOf(1, 2, 3))
    println(4 in arrayListOf(1, 2, 3))
}
```
* in 연산자는 when과 함께 사용할 수도 있다.
```
fun main(args: Array<String>) {
    println(check('1'))
    println(check('D'))
}

fun check(target: Char) = when(target) {
    in '0'..'9' -> "Is digit"
    in 'a'..'z', in 'A'..'Z' -> "Is char"
    else -> "???"
}
```
* **이러한 범위 검사는 Comparable 인터페이스를 구현한 클래스라면 모두 적용이 가능**하다.
```
fun main(args: Array<String>) {
    // 문자열은 알파벳 순서에 따른 비교가 가능하므로, 다음과 같은 응용이 가능하다.
    // 결과는 Maybe가 H 다음, W 이전에 위치하므로 true가 반환된다.
    println("Maybe" in "Hello".."World")
}
```

### Kotlin과 예외 처리
* Kotlin의 예외 처리는 Java와 유사하며, 오류 발생시 예외를 던지도록 할 수 있다.
  * Java와 마찬가지로 try, catch, finally 구문을 사용한다.
  * 함수를 호출하는 쪽에서는 예외를 잡아 처리할 수 있다.
  * 처리되지 않은 예외는 호출 스택을 거슬러 올라가며 구현된 예외 처리기를 찾을 때까지 예외를 다시 던진다.
* 다른 클래스와 마찬가지로, 예외를 던질 때에는 new 키워드를 사용하지 않는다.
* **Java와 달리, Kotlin의 throw 문은 식이므로 다른 식에 포함될 수 있다**.
* **Java와 달리, Kotlin의 try 문은 식이다**.
  * 때문에 **블록을 사용하는 경우, 블록 마지막 문장이 블록이 반환하는 값이 된다**.
* **Java와 달리, Kotlin 메소드는 던질 수 있는 예외를 throws로 작성하지 않는다**. 
```
fun main(args: Array<String>) {
    println(divide(4, 2))
    println(divide(0, 0))
}

// try 문을 식처럼 처리한다.
// 메소드에서 발생할 수 있는 예외를 명시하지 않는다.
fun divide(num1: Int, num2: Int): Int? =
    try {
        // 마지막 식이 블록이 반환하는 값이 된다.
        num1 / num2
      // 예외 타입은 콜론 다음에 작성한다.
    } catch(e: Exception) {
        null
    } finally {
        println("done.")
    }
```
* Java의 경우, 메소드에서 checked exception이 발생하는 경우 이를 throws를 통해 반드시 명시해야 한다.
  * 그러나 **Kotlin의 경우, checked / unchecked 예외를 구별하지 않는다**.
* Kotlin에서는 함수가 던질 수 있는 예외를 지정하지도 않고, 이를 잡아내도 되고 잡지 않아도 된다.
  * **Java의 경우 checked 예외 처리를 강제하지만, 개발자들이 무의미하게 예외를 되던지거나 처리를 무시하는 경우가 많다**.
  * 때문에 예외 처리 규칙이 오류 발생을 방지하지 못하는 경우가 많기에 Kotlin은 이러한 설계 방식을 따르게 되었다.
* IOException을 예로 들어, 실제 스트림을 닫다가 실패하더라도 클라이언트 코드에서 유의미한 대응을 수행할 수 없다.
  * 즉, **대부분의 경우 IOException을 잡아내는 코드는 불필요**하다.

### 식으로서의 try 문
* **Kotlin에서 try문 역시 값을 반환하는 식**이다.
  * 즉, try의 결과를 변수에 대입할 수 있다.
  * try 블록 역시 블록의 마지막 문장이 반환값이다.
* **if문과 달리 try문은 반드시 중괄호를 사용해야 한다**.
* **catch 블록에서 return만을 명시할 수 있지만, 예외 발생시에도 다음의 코드를 실행하고 싶다면 catch 문 역시 값을 반환해야 한다**.
```
fun main(args: Array<String>) {
    val num1 = 10
    val num2 = 0
    try {
        num1 / num2
    } catch(e: Exception) {
        // return을 사용하는 경우, catch 문에서 진행이 종료되므로 Done은 출력되지 않는다.
        // return을 제거하고 null의 주석을 해제하면 코드의 진행은 계속되어 Done이 출력된다.
        return
        // null
    }
    println("Done!")
}
```
* try 블록의 실행이 정상적으로 종료되면, 블록 마지막 식의 값이 결과이다.
  * 예외가 발생하여 catch 블록의 실행이 정상적으로 수행되면, 해당 catch 블록의 마지막 식의 값이 결과이다.

### 결론
* Kotlin에서, 값 객체 클래스는 아주 간결하게 표현할 수 있다.
* Kotlin에서, if / when / try 문은 값을 반환하는 식이다.
* Kotlin에서, 변수의 타입을 검사하고 나면 그 변수를 굳이 명시적으로 형변환할 필요가 없다.
* Kotlin에서, for 문은 Java의 for loop보다 강력하다. 특히 맵의 순회나 컬렉션의 요소 및 인덱스를 활용하는 경우에 적절하다.
* Kotlin에서, 예외 처리는 Java와 비슷하지만 메소드가 던질 수 있는 예외를 명시하지 않아도 된다.