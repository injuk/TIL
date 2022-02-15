# Kotlin
## 2022-02-15 Tue

## Kotlin의 원시 타입
* Int, Boolean, Any 등은 Kotlin의 원시 타입이다.
* Java는 원시 타입과 참조 타입을 다음과 같이 구분한다.
  1. 원시 타입: int 등의 변수에는 값이 직접 할당된다.
  2. 참조 타입: 변수에 메모리 상의 객체 주소가 할당된다.
* 원시 타입의 경우 값을 효율적으로 저장할 수 있지만, 메소드 호출 및 컬렉션 활용이 불가능하다.
  * 따라서 Java는 참조 타입의 활용이 필요한 경우 각 원시 타입에 대응되는 래퍼 타입으로 감싸는 방법을 택한다.
* **Kotlin은 Java와 달리 원시 타입과 래퍼 타입을 구분하지 않으므로 항상 같은 타입을 사용**한다.
  * 따라서 Kotlin은 원시 타입인 Int 변수에 대해 메소드를 호출할 수 있다.
* 숫자 등의 타입은 런타임에서 가장 효율적인 방식으로 표현된다.
  * 원시 타입과 참조 타입을 구분하지 않는다고 해서 Kotlin이 항상 래퍼 클래스를 활용하는 것은 아니다.
    * 예를 들어, Kotlin Int는 대부분의 경우 Java int로 컴파일된다.
    * 컬렉션, 제네릭과 같이 원시 타입의 활용이 불가능한 경우에는 Java Integer로 컴파일한다.

### 널이 될 수 없는 타입, 널이 될 수 있는 타입의 처리
* Kotlin Int 타입은 널이 될 수 없는 타입이므로, 대응되는 Java 원시 타입인 int로 컴파일할 수 있다.
  * 반대로 Java 원시 타입을 Kotlin에서 사용하는 경우, 원시 타입은 null을 참조할 수 없으므로 널이 될 수 없는 타입으로 취급할 수 있다.
  * 이 경우, 플랫폼 타입을 사용하지 않는다.
* Kotlin Int?와 같은 널이 될 수 있는 타입의 경우, 대응되는 Java 래퍼 클래스로 컴파일된다. 
  * Java 원시 타입은 null을 참조할 수 없기 때문이다.

### 제네릭과 래퍼 클래스
* 제네릭 클래스의 경우, 항상 대응되는 Java 래퍼 클래스로 컴파일된다.
  * 이 경우, Kotlin 원시 타입의 값을 넘기더라도 Java 래퍼 클래스를 활용하게 된다.
* 이는 JVM에서의 제네릭 구현 방법 때문이다.
  * JVM은 타입 파라미터로 원시 타입을 허용하지 않는다.
  * 때문에 Java와 Kotlin 모두 제네릭 클래스에 대해 반드시 박싱된 클래스를 사용해야 한다.
* 효율성을 위헤 원시 타입으로 구성된 컬렉션을 사용하고자 하는 경우, 서드 파티 라이브러리 또는 배열을 사용해야 한다.

### 원시 타입 간의 숫자 변환
* Kotlin과 Java의 큰 차이점 중 하나는 원시 타입 간의 숫자 변환이다.
* Kotlin은 한 숫자 타입의 변수를 다른 숫자 타입으로 자동 변환하지 않는다.
  * Java의 경우, 더 큰 타입에 대해 자동 형변환되지만 Kotlin은 이를 지원하지 않는다.
  * 때문에 필요한 경우에는 항상 명시적인 타입 변환 메소드를 호출해주어야 한다.
```
// 아래의 Kotlin 코드는 컴파일이 불가능하다.
fun main(args: Array<String>) {
     val intNum: Int = 1
     val longNum: Long = intNum
     // 컴파일이 가능하려면 명시적으로 변환해주어야 한다.
     // val longNum: Long = intNum.toLong()
     println(longNum)
}
// 아래의 Java 코드는 컴파일이 가능하다.
public class TempClass {
    public static void main(String[] args) {
        int intNum = 1;
        long longNum = intNum;
        System.out.println(longNum);
    }
}
```
* **Kotlin은 모든 원시 타입 간의 변환 메소드를 양방향으로 제공**한다.
  * 범위가 큰 타입으로 손실 없이 변환할 수도 있고, 범위가 더 작은 타입으로 변환하며 손실을 감내할 수도 있다.
* 타입 변환을 강제하는 것은 개발자의 혼돈을 피하기 위해서이다.
  * 예를 들어, Java의 래퍼 클래스 간 equals 메소드는 값이 아닌 객체를 비교한다.
* **Kotlin에서는 반드시 타입을 명시적으로 변환하여 같은 타입의 값을 비교하는 습관이 들어야 한다**.
  * 아래의 코드는 그 예시를 보여준다. intNum은 Int 타입이므로, 반드시 Long 타입으로 형변환한 후에 비교해야 한다.
```
fun main(args: Array<String>) {
     val intNum: Int = 1
     val longCollection = listOf(1L, 2L, 3L)
     // 아래의 문장은 명시적인 형변환이 누락되었으므로 컴파일이 불가하다. 
     // println(intNum in longCollection)
     println(intNum.toLong() in longCollection)
}
```
* 숫자 리터럴을 사용하는 경우 명시적인 형변환 메소드의 호출이 필요 없다.
  * 이 경우, 다른 숫자 타입을 사용하는 변수 또는 함수에 전달하더라도 문제 없이 실행된다.
  * Kotlin 컴파일러는 숫자 리터럴을 사용하는 경우에 한해 형변환 메소드를 자동으로 추가한다.
  * 산술 연산자는 여러 숫자 타입의 조합을 받아들일 수 있도록 오버로드되어 있다.
```
fun main(args: Array<String>) {
     // Long 을 받아들이는 함수에 Int 리터럴을 넘길 수 있다.
     runWithInt(1)
     // Int 리터럴과 Long 리터럴을 더하는 경우 자동으로 Long 타입으로 추론된다.
     // 산술 연산자는 여러 숫자 타입의 조합을 적용할 수 있도록 오버로드되어 있다.
     val result = 1 + 1L
     println(result is Long)
}

fun runWithInt(num: Long) = println(num)
```
* Kotlin은 Java와 마찬가지로 산술 연산에 의한 overflow가 발생할 수 있다.