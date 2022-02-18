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