# Kotlin
## 2022-02-18 Fri

### 구조 분해 선언
* Kotlin에서의 구조 분해 선언 역시 관례를 사용하는 예시이다.
* 구조 분해 선언의 경우, 각 변수를 초기화하기 위해 componentN 함수를 호출한다.
  * 이 때, N은 구조 분해 선언의 변수 위치에 따라 1부터 붙는 번호이다.
* 아래의 예시에서, component1과 2는 각각 Point의 프로퍼티인 x와 y에 대응된다.
  * **Kotlin은 data 클래스의 주 생성자에 포함된 프로퍼티에 대해 자동으로 component 함수 모음을 생성**한다.
```
fun main(args: Array<String>) {
     var point1: Point = Point(10, 22)
     // 구조 분해 할당의 예시이다.
     val( a, b ) = point1;
     println(a);
     println(b);
}

data class Point(val x: Int, val y: Int)
```
* data 클래스가 아닌 경우, 아래와 같이 operator 키워드와 확장 함수 형태로 직접 구현할 수 있다.
  * 이 경우에도 구조 분해 할당의 사용법은 같다.
```
fun main(args: Array<String>) {
     val bird = Bird("bird", 10);
     val (a, b) = bird
     println(a)
     println(b)
}

operator fun Bird.component1() = name;
operator fun Bird.component2() = age;
class Bird(val name: String, val age: Int)
```
* **구조 분해 할당은 함수에서 여러 값을 반환해야 하는 경우에 유용**하다.
  * 함수에서 여러 값을 반환해야 하는 경우, 모든 값을 포함하는 data 클래스를 정의하고 반환한다.
* 배열이나 컬렉션에도 component 함수가 포함된다.
  * Kotlin의 경우, 표준 라이브러리로부터 맨 앞의 다섯 요소에 대한 component 함수를 제공한다.
```
fun main(args: Array<String>) {
     val list = listOf(1, 2, 3, 4, 5);
     val (a, b, c, d, e) = list;
     println(a)
     println(b)
     println(c)
     println(d)
     println(e)
}
```
* 컬렉션의 크기가 다섯개 미만인 경우에도 componet5를 사용할 수 있으나, 이 경우 런타임 에러가 발생한다.
  * 반면 컬렉션의 크기가 여섯 개 이상인 경우에 component6을 정의 없이 사용하는 경우, 컴파일 에러가 발생한다.

### 구조 분해 할당을 통한 Map 활용
* 반복문 내부에서도 구조 분해 할당을 활용할 수 있으며, 특히 Map의 원소를 출력할 때 유용하다.
```
fun main(args: Array<String>) {
     val map = mapOf(1 to "A", 2 to "B", 3 to "C")
     for((k, v) in map) {
          println("$k : $v")
     }
}
```
* 위 예시는 Kotlin 관례 중 iterator와 component를 활용한다.
  * Map에 대한 in 키워드는 iterator를 호출한다.
  * Map에 대한 구조 분해 할당은 component 함수를 호출한다.

## 2022-02-19 Sat
### 위임 프로퍼티와 접근자
* 디자인 패턴으로서의 위임은 인스턴스가 작업을 직접 수행하지 않고, 다른 인스턴스에게 작업을 처리하도록 맡긴다.
  * **이 때, 실제로 작업을 처리하는 인스턴스는 위임 객체**라고 한다.
* 게터, 세터 등의 접근자를 활용하여 프로퍼티에 접근하는 로직을 위임 객체에게 위임할 수 있다.
  * 위임 객체는 미리 이름이 정해진 메소드인 getValue와 setValue를 operator 키워드와 함께 작성해야 한다.
  * 위임 객체는 반드시 getValue와 setValue 메소드를 멤버 또는 확장 함수의 형태로 제공해야 한다.
* 프로퍼티의 위임은 다음과 같이 작성한다.
  * 아래의 예시에서, name에 접근하는 동작은 doNothing.name의 형태로 작성되지만 실제로는 위임 객체에서 관례를 따르는 메소드를 호출한다.
```
class DoNothing {
     var name: String by DelegateClass()
}
```

### 프로퍼티 지연 초기화
* 인스턴스의 초기화 과정에서 많은 자원을 사용하거나, 인스턴스 사용시 반드시 초기화하지 않아도 되는 프로퍼티에 대해 초기화를 지연할 수 있다.
  * 때문에 **인스턴스의 일부는 초기화되지 않고, 실제로 그 값을 사용하는 시점에 초기화**하게 된다.
* **지연 초기화를 위해 위임 프로퍼티를 사용할 수 있으며, 이러한 방식은 뒷받침 필드와 게터를 함께 캡슐화**한다.
* 지연 초기화에 필요한 위임 객체를 반환하는 표준 라이브러리 함수인 lazy를 호출할 수 있다. 
  * **lazy는 getValue 메소드가 포함된 객체를 반환하므로, by 키워드와 함께 조합하여 위임 프로퍼티를 만들 수 있다**.
  * lazy 함수에 전달할 파라미터로는 값을 초기화하기 위해 사용할 람다식을 작성한다.
  * **lazy 함수는 기본적으로 스레드 안전하며, 필요한 경우에는 동기화를 적용할 수 있다**.
```
fun main(args: Array<String>) {
     println(LazyClass().name)
}

class LazyClass {
     val name: String by lazy { myNameIs() }
}
fun myNameIs() = "hello"
```

### 결론
* Kotlin에서, 정해진 이름의 메소드를 operator 키워드와 함께 오버로딩하는 것으로 연산자를 재정의할 수 있다.
  * 그러나 새로운 연산자를 만들어낼 수는 없다.
* Kotlin에서, 비교 연산자는 equals와 compareTo 메소드로 변환된다.
* Kotlin에서, 위임 프로퍼티를 통해 프로퍼티의 값을 관리하기 위한 로직을 재사용할 수 있다.
* Kotlin에서, 표준 라이브러리 함수인 lazy를 통해 지연 초기화 프로퍼티를 손쉽게 구현할 수 있다.

## 고차 함수
* 고차 함수는 람다를 인자로 받거나 반환하는 함수를 가리킨다.
  * 이러한 고차 함수는 코드의 중복성을 줄이고 추상화를 돕는다.
* Kotlin의 경우, 람다나 함수의 참조를 활용하여 함수를 값으로 표현할 수 있다.
  * 즉, 고차 함수는 람다나 함수 참조를 인자로 넘기거나 반환하는 함수이다.
  * 예를 들어, 앞서 다룬 filter는 함수를 인자로 받으므로 고차 함수라고 볼 수 있다.

## 2022-02-20 Sun
### 함수 타입
* Kotlin에서는 람다를 변수에 저장할 수 있었다.
* 이 경우 Kotlin 컴파일러에 의해 타입이 추론되어 적용되며, 실제로는 다음과 같이 작성할 수 있다.
```
fun main(args: Array<String>) {
     val lambda1 = { x: Int, y: Int -> x * y }
     val lambda2: (Int, Int) -> Int = { x, y -> x * y }
     val lambda3: (Int, Int?) -> Unit = { x, y -> println("$x $y") }
     val lambda4: ((Int, Int) -> Int)? = null
     val lambda5: (x: Int, y: Int) -> Int = { x, y -> x - y }
}
```
* 상술한 예시에서, 각 람다를 통해 다음을 알 수 있다.
  1. lambda1: 람다를 변수에 저장할 수 있으며, 이 경우 타입은 컴파일러에 의해 추론된다.
  2. lambda2: 람다의 타입을 직접 명시할 수도 있으며, 이는 함수 타입이 된다.
     * 함수 타입이 작성된 경우, 람다식 내부의 인자 타입을 명시하지 않을 수 있다.
  3. lambda3: 반환 값이 없는 경우, Unit을 작성한다. 또한, 인자 타입 역시 널이 될 수 있는 타입일 수 있다.
  4. lambda4: 함수 타입 자체가 널이 될 수 있는 타입일 수 있으며, 이 경우 람다 본문에서 사용되는 변수들의 널 가능성과는 관계가 없다.
     * lambda4의 내용이 의도된 경우라면 함수 타입 전체를 감싸는 괄호를 반드시 작성한다.
     * 괄호가 누락될 경우, 람다의 반환값을 널이 될 수 있는 타입으로 취급한다.
  5. lambda5: 함수 타입의 인자에 이름을 붙일 수 있다.
     * lambda5를 사용하는 다른 사용자가 이 이름을 써도 좋고, 자신이 원하는 이름을 사용할 수도 있다.
* 람다를 인자로 받는 고차 함수의 구현은 함수 타입을 활용하여 다음과 같이 정의할 수 있다.
  * high(lambda)와 같이 일반적인 함수 호출처럼 사용할 수 있다.
  * high { x, y -> x * y * 2 }와 같이 마지막 인자인 람다를 외부로 빼내어 호출할 수 있다.
  * 고차 함수 내부에서, 인자로 받은 함수는 파라미터명으로 호출한다.
```
fun main(args: Array<String>) {
     val lambda: (Int, Int) -> Int = { x, y -> x * y }

     high(lambda)
     high { x, y -> x * y * 2 }
}

fun high(callback: (Int, Int) -> Int) {
     println(callback(1, 2))
}
```
* 컴파일된 코드에서 함수 타입은 invoke 메소드를 갖는 인터페이스로 변환된다.
  * 즉, 함수 타입의 변수는 인터페이스를 구현하는 인스턴스 형태로 전달된다.
* Java에서 Kotlin의 고차 함수를 호출할 수도 있으며, 이 경우 Java의 람다를 전달한다.
  1. Java 8 이후 버전의 경우, 람다식을 작성하여 전달한다.
  2. Java 8 이전 버전의 경우, 함수 타입 인터페이스를 구현하는 무명 객체를 전달한다.
* 반환형이 Unit인 Kotlin 고차 함수를 Java에서 호출하는 경우, return Unit.INSTANCE를 반드시 작성한다.
  * **Unit에 대응되는 Java의 타입은 void이지만, Kotlin의 Unit은 값이 존재하므로 void 형태의 Java 람다를 전달할 수는 없다**.
  
### 함수 타입
* 함수의 동작을 더 유연하게 바꾸는 방법 중 하나는 인자로 람다를 받아 함수 본문에서 활용하는 방식이다.
```
fun main(args: Array<String>) {
     println(doSomething(1, 3) { a, b -> a + b })
}

fun doSomething(a: Int, b: Int, lambda: (Int, Int) -> Int): Int {
     return lambda(a, b)
}
```
* **람다의 기본적인 동작을 지정하여 매번 람다를 전달하지 않아도 되게 하고 싶은 경우, 함수의 인자로 전달된 람다에 디폴트 값을 지정**할 수 있다. 
  * 함수의 파라미터에 전달 될 람다의 기본값은 일반적인 파라미터의의 기본값 설정 방법과 같다. 
* **널이 될 수 있는 함수 타입을 사용해도 좋지만, 이 경우 NPE를 방지하기 위해 반드시 함수 본문에서 null을 체크하는 로직이 작성되어야만 한다**.
  * 이러한 로직은 순수하게 null 여부를 if문으로 체크할 수도 있지만, 안전한 참조 또는 엘비스 연산자를 활용할 수도 있다.
```
fun main(args: Array<String>) {
     // 람다식을 전달하지 않은 경우, 두 값을 더하는 디폴트 람다가 실행된다.
     println(doSomething(1, 3))
     // 람다식을 전달한 경우, 디폴트 람다 대신 전달한 람다가 실행된다.
     println(doSomething(1, 3) { a, b -> a * b })
}

fun doSomething(a: Int, b: Int, lambda: (Int, Int) -> Int = { a, b -> a + b }): Int {
     return lambda(a, b)
}
```

### 고차 함수에서의 함수 반환
* 애플리케이션의 상태나 다른 조건에 따라 로직이 달라질 수 있는 경우, 함수를 반환하는 고차 함수를 유용하게 사용할 수 있다.
  * 아래의 예시에서, getSomething은 인자로 전달 받은 enum의 값에 따라 서로 다른 람다를 반환한다.
  * 반환된 람다는 변수에 저장할 수 있으며, 변수() 형태로 메소드처럼 호출할 수 있다.
* **고차 함수에서 함수를 반환하고자 하는 경우, 고차 함수의 반환형은 함수 타입**이 되어야 한다.
  * **단순히 메소드의 반환형에 함수 타입을 작성하는 것만으로, 함수를 반환하는 고차 함수를 쉽게 정의**할 수 있다.
```
fun main(args: Array<String>) {
     val sumLambda = getSomething(LAMBDA.SUM)
     val multiLambda = getSomething(LAMBDA.MULTI)
     println(sumLambda(1, 2))
     println(multiLambda(4, 3))
}

enum class LAMBDA {
     SUM,
     MULTI
}
fun getSomething(type: LAMBDA): (Int, Int) -> Int = when(type) {
     LAMBDA.SUM -> { a: Int, b: Int -> a + b }
     LAMBDA.MULTI -> { a: Int, b: Int -> a * b }
}
```
* **상술한 방식들을 활용하여 작성한 고차 함수는 코드 구조를 개선하고 중복을 제거하기 위해 유용하게 사용**할 수 있다.

## 2022-02-21 Mon
### 람다를 활용한 중복 제거
* 상술한 함수 타입과 람다는 코드의 재사용성을 높이는 유용한 도구이며, 코드의 중복을 줄이는 데에도 직접적인 영향을 준다.
  * 이는 개발자가 코드의 중복을 제거하기 위해 변수, 프로퍼티, 파라미터를 사용하는 것과 같은 원리이다..
* 이를 응용하여 코드의 일부를 복사 / 붙여넣기 하는 일이 잦은 코드를 람다로 추출하는 것을 고려할 수 있다.
  * 예를 들어, 어떤 목록에 대해 filter 함수를 자주 사용한다면 filter에 들어갈 법한 predicate 함수를 람다 인자로 전달받는 메소드를 정의해볼 수 있다.
* 또한, 람다는 잘 알려진 디자인 패턴을 단순화하는 데에도 사용할 수 있다.
  * 예를 들어, 전략 패턴은 일반적으로 인터페이스와 구현 클래스로 표현된다.
  * 그러나 람다를 적용한다면 전략에 따라 다른 람다를 넘김으로써 전략 패턴을 쉽게 구현할 수 있다.