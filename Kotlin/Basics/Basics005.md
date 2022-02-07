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

## 2022-02-08 Tue
### Kotlin 문자열
* Kotlin의 문자열은 Java의 문자열과 같으며, 동일한 기능을 한다.
* Kotlin은 다양한 확장 함수를 제공하여 표준 Java 문자열을 더욱 쉽게 다룰 수 있도록 한다.
  * 특히 Java에서 혼동을 야기할 수 있는 일부 메소드에 대해서 더 정확한 확장 함수를 제공한다.
* Java의 split은 인자로 문자열 형태의 정규표현식을 받지만, 메소드 명에서 이러한 사실이 드러나지 않아 혼동을 준다.
```
// 아래의 결과는 빈 배열인 []이다.
public class Person {
    public static void main(String[] args) {
        System.out.println(
                Arrays.toString("145.23-6.sd".split("."))
        );
    }
}
```
* Kotlin의 split은 확장 함수로서 문자열을 받으면 구분자로 활용하거나, 명시적으로 Regex 타입의 값을 받을 수 있다.
  * 문자열을 파라미터로 사용한 경우, 문자열을 기준으로 split한다.
  * 문자열을 Regex 형태로 파라미터에 넘긴 경우, 문자열의 내용을 정규표현식으로 취급하여 split한다.
  * Kotlin의 정규표현식 문법은 Java와 동일하며, 문자열 이스케이프에는 \ 기호를 사용한다.
```
fun main(args: Array<String>) {
    val test: String = "145.23-6.sd"
    // 아래의 실행문은 온점을 기준으로 split하며, 개발자의 의도대로 동작한다.
    println(test.split("."))
    // 또는 Regex 형의 매개 변수를 명시적으로 넘겨줄 수도 있다.
    println(test.split("\\.".toRegex()))
}
```
* 상술한 예시와 같이 **정규표현식은 toRegex 확장 함수를 사용하여 문자열을 정규식으로 변환할 수 있도록 편의성을 제공**한다.
* Kotlin의 split은 또한 여러 개의 구분자를 지원한다.
  * 하나의 구분자만 인자로 받는 Java split에 비해 사용이 용이하다.
```
fun main(args: Array<String>) {
    val test: String = "145.23-6.sd"
    // 아래의 파라미터 목록은 모두 문자열 구분자로 취급된다.
    println(test.split(".", "-", "s"))
}
```
* Kotlin에서는 정규식을 사용하지 않고도 문자열을 쉽게 파싱할 수 있도록 여러 메소드를 제공한다.
  * 정규식은 강력하지만, 가독성 측면에서 어려움이 있을 수 있다.
  * 아래는 Kotlin의 라이브러리를 통해 주어진 경로로부터 파일명과 확장자를 구하는 코드이다.
```
fun main(args: Array<String>) {
    val path = "/Users/ingnoh/Downloads/study/TIL/README.md"
    val fullPath = path.substringBeforeLast("/")
    val fileFullName = path.substringAfterLast("/")

    val fileName = fileFullName.substringBeforeLast(".")
    val extension = fileFullName.substringAfterLast(".")

    println("$fullPath $fileFullName $fileName $extension")
}
```

### 3중 따옴표 문자열
* 3중 따옴표 문자열 내부에서는 역슬래쉬를 포함한 어떠한 문자도 이스케이프할 필요가 없다.
* 3중 따옴표 문자열은 줄 바꿈이 포함된 프로그램 텍스트를 쉽게 문자열로 만들기 위해 사용할 수 있다.
* 3중 따옴표 문자열은 크게 다음과 같은 용도로 사용할 수 있다.
  1. 정규식에서 가독성을 떨어트리는 이스케이프 문자의 사용을 피하기 위해
  2. 줄 바꿈을 표현하는 모든 문자열을 그대로 사용하기 위해
* 3중 따옴표 문자열 내부에서도 문자열 템플릿을 사용할 수 있다.
```
fun main(args: Array<String>) {
    // trimIndet를 사용하지 않으면 문자열 공백과 첫 줄, 마지막 줄의 개행이 모두 적용된다.
    val banner = """
        ||  ||
        ||--||
        ||--||
    """.trimIndent()

    println(banner)
}
```

### 확장 함수의 존재 의의
* 이렇듯 확장 함수는 기존 Java 라이브러리에 포함된 API를 확장하고, 기존 라이브러리를 새로운 언어에 맞추어 사용할 수 있게 돕는다.
* 기존 라이브러리를 새 언어에서 활용하는 패턴은 Pimp My Library 패턴이라고 부른다.
* 실제로도 Kotlin 표준 라이브러리 중 상당 부분은 표준 Java 라이브러리의 확장으로 구성된다.

### 로컬 함수와 확장
* DRY 원칙: 좋은 코드의 특징 중 하나는 중복이 없는 것이다.
* Java의 경우 메소드 추출 패턴을 통해 긴 메소드를 짧은 여러 개의 메소드로 리팩토링 하는 기법이 흔히 적용된다.
  * 그러나 **이러한 방법은 클래스 내부에 작은 메소드가 많아지고, 메소드 간의 관계를 파악하기 어려워 가독성이 떨어질 수도 있다**.
* Java의 경우, 추출된 메소드를 inner class로 모아 코드를 깔끔하게 구성할 수 있지만 이를 위한 불필요한 코드들이 발생한다.
* **Kotlin에서는 메소드 추출 과정에서 추출된 함수를 원래의 함수 내부에 포함시킬 수 있다**.
  * 아래의 예시에서, 각 필드의 유효성을 검증하는 코드는 중복되는 코드에 해당한다.
```
fun main(args: Array<String>) {
//    val person1 = Person("", "")
//    val person2 = Person("John", "")
    val person3 = Person("Jane", "Doe")

//    isValidPerson(person1)
//    isValidPerson(person2)
    isValidPerson(person3)
}

fun isValidPerson(person: Person) {
    // 필드별 유효성 검증 로직은 중복된다.
    if(person.name.isEmpty()) {
        println("there is no name!!!")
        throw IllegalArgumentException()
    }
    if(person.family.isEmpty()) {
        println("there is no family name!!!")
        throw IllegalArgumentException()
    }
    println("valid user")
}
```
* 위 메소드를 **메소드 내부에 작성되는 메소드인 로컬 함수를 통해 다음과 같이 수정**할 수 있다.
  * **이 때, 로컬 함수는 자신이 속한 외부 함수의 모든 변수와 파라미터에 접근할 수 있다**.
```
fun main(args: Array<String>) {
//    val person1 = Person("", "")
//    val person2 = Person("John", "")
    val person3 = Person("Jane", "Doe")

//    isValidPerson(person1)
//    isValidPerson(person2)
    isValidPerson(person3)
}

fun isValidPerson(person: Person) {
    fun validate(value: String, field: String) {
        // 로컬 함수는 외부 함수의 값에 접근할 수 있다.
        println("validating person... ${person.name}")
        if(value.isEmpty())
            throw IllegalArgumentException("there is no $field!!!")
    }
    validate(person.name, "name")
    validate(person.family, "family")
    println("valid user")
}
```
* **위와 같이 개선된 메소드는 다시 확장 함수의 형식으로 개선하여 더 직관적인 사용이 가능하며, 이러한 방식이 권장된다**.
```
fun main(args: Array<String>) {
//    val person1 = Person("", "")
//    val person2 = Person("John", "")
    val person3 = Person("Jane", "Doe")

//    person1.isValidPerson()
//    person2.isValidPerson()
    person3.isValidPerson()
}

fun Person.isValidPerson() {
    fun validate(value: String, field: String) {
        println("validating person... ${this.name}")
        if(value.isEmpty())
            throw IllegalArgumentException("there is no $field!!!")
    }
    validate(this.name, "name")
    validate(this.family, "family")
    println("valid user")
}
```

### 확장 함수의 유용성
* **코드를 확장 함수로 추출하는 기법은 매우 유용하며, 권장되는 기법**이다.
  * **특히 비공개 멤버에 접근할 필요 없이 공개된 프로퍼티나 메소드만 활용하는 경우에 유용하다**.
* 상술한 예시에서, Person은 라이브러리 코드가 아닌 자신의 코드이지만 검증 로직을 다른 패키지에서는 사용하지 않는 기능으로 가정한다.
  * 이 경우, 확장 함수로 사용하는 것으로 Person에 검증 코드를 포함시키지 않을 수 있다.
  * 다른 곳에서 Person을 사용하는 개발자들은 깔끔하게 유지되는 Person 코드를 통해 더 쉽게 코드를 이해할 수 있다.
* 확장 함수를 로컬 함수로 정의할 수도 있으나, 중첩 구조가 깊어질수록 코드의 가독성이 떨어지고 이해하기 어렵다.
  * 따라서 **로컬 함수는 한 단계만 중첩시키는 것이 권장된다**.

### 결론
* Kotlin에서, 자체 클래스를 정의하지 않으면서도 Java 클래스를 확장하는 기법을 통해 풍부한 API가 제공된다.
* Kotlin에서, 함수 파라미터의 디폴트 값을 지정하는 것으로 불필요한 오버로딩을 줄일 수 있다.
* Kotlin에서, 이름 붙인 인자를 사용하는 것으로 인자가 많은 함수의 가독성을 높일 수 있다.
* Kotlin에서, 최상위 함수와 최상위 프로퍼티를 활용하는 것으로 더 유연한 코드 구조를 활용할 수 있다.
* Kotlin에서, **확장 함수와 확장 프로퍼티를 통해 외부 라이브러리의 소스 코드를 추가할 필요 없이 기능을 확장할 수 있다**.
  * **확장 함수는 실행 시점에 부가 비용도 들지 않는다**!
* Kotlin에서, 중위 호출을 통해 인자가 하나 뿐인 메소드 / 확장 함수를 더 가독성 있게 호출할 수 있다.
* Kotlin에서, 일반 문자열을 처리하기 위한 다양한 문자열 처리 메소드가 제공된다.
* Kotlin에서, **로컬 함수를 활용하는 것으로 중복을 제거하여 코드를 더 깔끔하게 유지**할 수 있다.