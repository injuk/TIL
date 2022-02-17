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
  * 이러한 방식은 += 연산으로 인해 객체의 참조를 변경하지 않고, 기존 객체의 상태를 변경하고 싶은 경우에 유용하다.
* plus와 plusAssign은 동시에 재정의하는 경우 오류가 발생하므로, 이러한 방식을 지양해야 한다.
  * 정확히는 둘을 동시에 재정의할 수 있지만, += 연산자를 사용하는 문장에서 오류가 발생한다.

## 2022-02-18 Wed
### 단항 연산자 오버로딩
* 단항 연산자 역시 +, * 등의 이항 연산자와 마찬가지로 미리 정해진 이름의 함수를 선언하여 operator 키워드를 정의한다.
  1. +a: unaryPlus
  2. -a: unaryMinus
  3. !a: not
  4. ++a / a++: inc
  5. --a / a--: dec
* 아래의 예시에서, -point를 원하는 방식으로 동작시키고자 하므로 unaryMinus를 사용한다.
```
fun main(args: Array<String>) {
     var point1: Point = Point(10, 22)
     println(-point1)
}
// -point 형태의 단항 연산자 역시 다른 산술 연산자와 동일한 방식으로 오버로딩한다.
operator fun Point.unaryMinus(): Point {
     return Point(-x, -y)
}
data class Point(val x: Int, val y: Int)
```

## 비교 연산자 오버로딩
* Kotlin에서는 산술 연산자와 마찬가지로 원시 타입 값 이외의 모든 객체에 대해 비교 연산을 수행할 수 있다.
* 비교 연산자인 ==, >, < 등은 클래스 내부에서 equals와 compareTo 메소드를 오버라이드하는 것으로 재정의할 수 있다.

### equals
* a == b는 내부적으로 a.equals(b)로 컴파일된다.
  * != 연산자의 결과값 역시 equals를 활용하며, !a.equals(b)의 형식과 같다.
  * 때문에 != 연산자를 재정의하는 방법은 없다.
* data 클래스는 Kotlin 컴파일러에 의해 equals 메소드가 자동으로 생성되지만, 필요시 직접 구현할 수도 있다.
* 식별자 비교 연산자 ===는 두 피연산자가 서로 같은 객체를 참조하는지 비교한다.
  * === 연산자는 오버로딩할 수 없다.
* equals는 Any에 오버로딩된 operator 메소드이며, 이를 상속 받는 클래스에서는 재정의를 위해 override 키워드를 사용한다.
  * 즉, Any의 자손은 operator fun equals가 아닌 override fun equals로 재정의해야 한다.
  * 한 편, equals는 Any에서 오버로드되었으므로 모든 Kotlin 객체는 동등성을 비교할 수 있다.
* Any로부터 상속받는 equals 메소드가 확장 함수보다 높은 우선 순위를 가지므로, equals 메소드는 확장 함수로 정의할 수 없다.

### compareTo
* Java의 경우, 값을 비교하는 알고리즘을 적용하는 클래스는 Comparable을 구현해야 한다.
  * 해당 인터페이스에는 크기를 비교하여 정수를 반환하는 compareTo 메소드가 정의되어 있다.
* 그러나 Java는 반드시 a.compareTo(b)의 형식으로 호출해야 하며, 이를 짧게 호출할 수 있는 방법을 지원하지 않는다.
  * <, >와 같은 연산은 Java의 원시 타입에만 적용이 가능하다.
* Kotlin 역시 Comparable 인터페이스를 지원하며, 나아가 compareTo를 호출하는 관례를 제공한다.
  * 때문에 Kotlin의 Comparable 구현체는 <, >와 같은 비교 연산자로 비교가 가능하다.
* 비교 연산자 역시 equals의 경우와 마찬가지로 확장 함수를 작성할 필요가 없다.

## 컬렉션과 관례
### 인덱스로 컬렉션 요소에 접근
* 맵의 요소에 접근하는 등의 용도로 사용하는 인덱스 연산자 [] 역시 Kotlin이 제공하는 관례를 따른 예시이다.
* 인덱스 연산자를 사용해 요소를 읽으려면 get 연산자를 오버로드하고, 요소를 쓰려면 set 연산자를 오버로드한다.
  * Map, MutableMap은 이미 get과 set 메소드가 정의되어 있다.
* get의 경우, 반드시 Int 인자를 사용할 필요는 없다.
  * Map의 예시와 유사하며, 필요하다면 다양한 인자에 대해 여러 get 메소드를 오버로드하여 다양한 키 타입을 지원할 수 있다.
* set의 경우, 여러 인자를 받을 수 있지만 할당되는 값은 항상 가장 오른쪽의 인자이다.
  * 예를 들어, point[4, 5] = 3은 point.set(4, 5, 3)으로 컴파일된다.

### in 연산자
* 컬렉션에 지원되는 또 다른 연산자이며, 객체가 컬렉션 내부에 들어있는지 검사하기 위해 사용한다.
* in 연산자에 대응되는 메소드는 contains이며, contains 함수를 operator 키워드와 함께 오버로드하는 것으로 in 연산자를 활용할 수 있다.
* 이 때, element in collection 은 collection.contains(element)로 컴파일된다.

### .. 연산자
* 단힌 범위를 만들기 위해 사용하는 .. 구문은 rangeTo 함수로 재정의할 수 있다.
  * 예를 들어, first..end는 first.rangeTo(end)로 컴파일된다.
* rangeTo는 어떠한 클래스에도 정의할 수 있으나, Comparable을 구현한 경우에는 필요가 없다.
  * Kotlin의 표준 라이브러리에는 모든 Comparable에 적용 가능한 rangeTo 메소드가 포함된다.

### for 반복문에서의 in 연산자
* for 반복문 역시 in 연산자를 사용할 수 있지만, 이 in은 컬렉션에서 살펴본 경우와는 의미가 다르다.
  * for(e in collection)을 예로 들면, collection.iterator()를 호출한 후 hasNext와 next를 반복하는 식으로 동작한다.
* 이 역시 Kotlin이 제공하는 관례에 해당하므로, iterator라는 이름의 메소드를 operator 키워드와 함께 확장 함수로 정의할 수 있다.
* 예를 들어, Kotlin의 표준 라이브러리에는 String의 상위 클래스에 대해 iterator 확장 함수를 제공하므로 문자열 순회가 가능하다.