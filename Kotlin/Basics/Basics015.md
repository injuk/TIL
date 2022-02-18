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