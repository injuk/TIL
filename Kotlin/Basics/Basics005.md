# Kotlin
## 2022-02-07 Mon

### Kotlin의 컬렉션 처리
* 앞서 확인했듯 Kotlin은 last나 max와 같은 확장된 API를 제공한다.
  * 사실 last와 max는 확장 함수이다!
* Kotlin의 표준 라이브러리는 사용자의 편의성을 지원하기 위한 수많은 확장 함수를 포함한다.
* 이러한 표준 라이브러리를 모두 외울 필요는 없으며, 그 때 그 때 IDE의 추천을 받아 원하는 함수를 사용하는 것이 좋다.

### Kotlin의 가변 인자 함수
* Kotlin의 가변 인자는 함수의 파라미터 앞에 vararg 키워드를 붙여 표현할 수 있다.
* 내부적으로는 Java와 마찬가지로 매개 변수 목록을 배열에 넣어 처리한다.
* Java의 경우 배열을 그냥 넘겨 처리하지만, Kotlin의 경우 내부적으로 배열의 요소를 하나 하나 인자로 전달되도록 해야 한다.
  * 이러한 작업은 수작업보다는 스프레드 연산자인 *을 사용할 수 있다.
  * *배열의 형태로 작성하여 스프레드 연산자가 배열의 내용을 펼쳐 전달하도록 할 수 있다.
```
fun main(args: Array<String>) {
    val list = makeList(1, 2, 3, 4, 5)
    println(list)
}

fun <T> makeList(vararg elements: T): List<T> = listOf(*elements)
```

### 중위 호출
* hashMap에 값을 put하는 대신 to 키워드를 사용할 수 있다.
```
fun main(args: Array<String>) {
    val map = hashMapOf(1 to "injuk", 2 to "ingnoh")
    println(map)    
}
```
* 이 떄, **to는 사실 키워드가 아닌 중위 호출을 지원하는 메소드에 해당**한다.
  * 즉, 위 예시는 사실 다음과 같은 형식으로 호출되는 것과 동일하다.
  * to 역시 Any 수신 객체 타입의 확장 함수인 infix fun Any.to로 정의되었다.
```
fun main(args: Array<String>) {
    val map = hashMapOf(1.to("injuk"), 2.to("ingnoh"))
    println(map)
}
```
* 메소드가 중위 호출이 가능하도록 만드려면 fun 키워드 앞에 infix 키워드를 작성한다.
  * 중위 호출은 메소드의 파라미터가 하나 뿐일 경우에만 가능하다.
  * 중위 호출시 수신 객체와 유일한 파라미터 사이에 메소드의 이름을 작성한다.
  * 중위 호출 변경자 **infix는 일반적인 인스턴스 메소드와 확장 함수 모두에 적용 가능**하다.
```
fun main(args: Array<String>) {
    val animal = Animal()
    animal.greeting("injuk")
    animal greeting "ingnoh"

    animal farewell "ingnoh"
}

infix fun Animal.farewell(target: String) = println("Bye, $target!")

class Animal(val name: String =  "Aminal") {
    infix fun greeting(target: String) = println("Hello, $target! My name is $name!")
}
```

### 구조 분해 할당
* Pair는 Kotlin의 표준 라이브러리 클래스이며, 두 원소로 이루어진 순서쌍을 표현한다.
  * to 메소드는 Pair 인스턴스를 반환한다.
* 구조 분해 할당은 이러한 순서쌍의 내용을 토대로 변수를 초기화하기 위해 사용할 수 있다.
```
fun main(args: Array<String>) {
    // val pair = 1 to "ingnoh"
    val pair = Pair(1, "ingnoh")
    println(pair)

    val (key, value) = pair
    println("key is $key and value is $value")
}
```
* 이러한 구조 분해 할당은 Pair 뿐만 아니라 다른 객체에도 적용할 수 있다.
```
fun main(args: Array<String>) {
    val list = listOf(1, 2, 3, 4)
    for((index, element) in list.withIndex())
        println("[$index]: $element")
}
```
* listOf, mapOf 등은 Kotlin을 모르는 사람이 보기에는 Kotlin만의 특별한 기능이나 문법처럼 보일 수 있다.
  * 그러나 사실은 **일반적인 Java 메소드를 더 간결한 구문으로 호출할 수 있도록 하는 것에 지나지 않는다**.