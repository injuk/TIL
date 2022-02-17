# Kotlin
## 2022-02-17 Thu

## 연산자 오버로딩과 관례
* Kotlin에서, 어떤 언어 기능이 특정한 사용자 작성 함수와 연결되는 경우가 있가.
  * 예를 들어, 어떤 클래스 안에 정의된 plus라는 이름의 메소드는 해당 클래스에 대해 + 연산자를 적용할 수 있도록 한다.
* 이렇듯 **언어의 어떠한 기능과 미리 정해진 이름의 함수를 연결해주는 기법을 Kotlin에서는 관례라고 표현**한다. 

### 산술 연산자 오버로딩
* Kotlin에서 관례를 적용하는 대표적인 예시이다.
* Java의 경우, 원시 타입과 String에서만 + 연산자를 사용할 수 있다.
  * 그러나 BigInteger나 컬렉션을 구현한 클래스에서도 + 연산자를 사용할 수 있다면 편리할 수 있을 것이다.
  * Kotlin에서는 산술 연산자 오버로딩을 통해 이를 구현할 수 있다.
* Kotlin의 산술 연산자 오버로딩은 산술 연산자를 활용하고 싶은 클래스 내부에서 plus라는 이름의 메소드를 구현하는 것으로 가능하다.
  * 아래의 예시에서, operator fun plus 메소드를 통해 + 연산자를 오버로딩하고 있다.
  * 오버로딩된 + 연산자는 같은 Point 클래스끼리의 연산에 활용할 수 있다.
* **연산자를 오버로딩하는 함수 앞에 operator를 붙여줌으로써 해당 함수가 관례를 따르는 함수임을 명시**한다.
```
fun main(args: Array<String>) {
     val origin: Point = Point(10, 20)
     val x10y20: Point = Point(10, 20)
     // 오버로딩된 + 연산자를 활용할 수 있다.
     println(origin + x10y20) // Point(x=20, y=40)
}

data class Point(val x: Int, val y: Int) {
     // + 연산자의 오버로딩은 plus 메소드를 활용하는 것으로 가능하다.
     operator fun plus(other: Point): Point {
          return Point(x + other.x, y + other.y)
     }
}
```
* 산술 연산자의 오버로딩은 확장 함수 형식으로도 가능하다.
  * 즉, 반드시 클래스 내부에 작성할 필요가 없다.
  * **외부 함수의 클래스에 대해 연산자를 정의하는 경우, 관례를 따르는 확장 함수를 사용하는 것이 바람직**하다.
```
fun main(args: Array<String>) {
     val origin: Point = Point(10, 22)
     val x10y20: Point = Point(10, 20)
     println(origin + x10y20)
}

operator fun Point.plus(other: Point): Point {
     return Point(x + other.x, y + other.y)
}
data class Point(val x: Int, val y: Int)
```
* Kotlin의 경우, **연산자를 개발자가 직접 정의할 수 없으며 언어 차원에서 미리 지정된 연산자만 오버로딩이 가능**하다.
  * 정의할 수 있는 연산자와 대응되는 메소드의 이름은 다음과 같다.
  1. times: *
  2. div: /
  3. rem: %
  4. plus: +
  5. minus: -
* **오버로딩된 연산자의 우선 순위는 기본적인 연산자 우선 순위와 같다**.
* Kotlin의 연산자 오버로딩에서, 두 피연산자가 같은 타입일 필요는 없다.
* Kotlin의 연산자 오버로딩에서, operator 함수는 여러 형태로 오버로딩이 가능하다.
  * **즉, plus 함수를 여러 인자 타입에 대해 오버로딩하여 서로 다르게 동작하는 연산자 함수를 여럿 만들 수 있다**.
* Kotlin의 연산자는 교환 법칙을 자동으로 지원하지 않는다.
  * 아래의 예시에서, 2 * origin은 자동으로 지원되지 않는다.
  * 별도로 operator fun Double.times(origin: Point): Point 를 정의해야 해당 연산을 수행할 수 있다.
```
fun main(args: Array<String>) {
     val origin: Point = Point(10, 22)
     println(origin * 2)
     // 아래와 같이 교환 법칙을 적용할 수는 없다.
     // println(2 * origin)
}

operator fun Point.times(time: Int): Point {
     return Point(x * time, y * time)
}
```
* Kotlin 연산자 오버로딩에서, 반환 타입이 피연산자의 타입과 일치할 필요는 없다.
  * 아래의 예시에서, 반환형은 Point가 아닌 String으로 오버로딩되었다.
```
fun main(args: Array<String>) {
     val point1: Point = Point(10, 22)
     val point2: Point = Point(5, 2)
     println(point1 + point2)
}

operator fun Point.plus(other: Point): String {
     return (x + other.x).toString() + (y + other.y).toString()
}
```

### += 연산자 오버로딩
* plus와 같은 + 연산자를 오버로딩하면 Kotlin은 += 연산자도 자동으로 지원한다.
* 아래의 예제에서, += 연산자 적용시 기존의 참조가 반환되는 새로운 객체의 참조로 변경된다.
```
fun main(args: Array<String>) {
     var point1: Point = Point(10, 22)
     val point2: Point = Point(5, 2)

     val temp: Point = point1;
     // 참조를 비교한 경우 true가 반환된다.
     println(temp === point1);
     point1 += point2;
     // 새로운 객체가 반환되었으므로, false가 반환된다.
     println(temp === point1);
}

operator fun Point.plus(other: Point): Point {
     return Point(x + other.x, y + other.y)
}
```
* += 연산자를 재정의하고 싶은 경우 plusAssign 메소드를 재정의할 수 있다.
  * plus와 plusAssign은 동시에 재정의하는 경우 오류가 발생하므로, 이러한 방식을 지양해야 한다.
    * 정확히는 둘을 동시에 재정의할 수 있지만, += 연산자를 사용하는 문장에서 오류가 발생한다.