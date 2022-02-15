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

## 2022-02-16 Wed
### Any, Any?
* **Java의 경우, Object 클래스는 원시 타입을 제외한 참조 타입의 상속 계층도에서 최고 조상을 차지**한다.
  * Java에서 원시 타입을 Object 인자에 넘길 경우 대응되는 래퍼 클래스로 박싱하는 과정이 필요하다.
* **Kotlin의 경우, Any 타입이 Kotlin 원시 타입을 포함한 모든 널이 될 수 없는 모든 타입의 조상**이다.
  * Kotlin 역시 Any에 원시 타입의 값을 대입하면 오토 박싱이 발생한다.
* Kotlin Any는 물음표가 붙지 않은 널이 될 수 없는 타입이므로, null을 참조할 수 없다.
  * null을 포함하는 모든 값을 대입해야 하는 경우, Any?를 사용한다.
* **Kotlin의 모든 클래스에 포함되는 toString, equals, hashCode는 Any 클래스의 메소드를 상속 받은 것**이다.
  * 반면 Java Object 클래스의 wait, notify 메소드 등은 Any 클래스에 포함되어 있지 않다.
  * 이러한 메소드를 활용해야 하는 경우, java.lang.Object 타입으로 캐스트한 후에 사용해야 한다.

### Unit
* **Kotlin의 Unit 타입은 Java의 void와 같은 기능을 수행**한다.
  * 따라서 내용을 반환하지 않는 함수의 반환 타입으로 Unit을 작성할 수 있다.
  * 반환 타입이 없는 함수는 Unit을 명시해도 좋고, 명시하지 않아도 무방하다.
* Kotlin **Unit 타입의 메소드는 묵시적으로 Unit을 반환**한다.
```
fun main(args: Array<String>) {
     val withoutUnit = doSomething()
     println(withoutUnit is Unit) // true
     val withUnit = doSomethingWithUnit()
     println(withUnit is Unit) // true
}
// 반환 타입을 명시해도 좋고, 하지 않아도 좋다.
fun doSomething() = println("Hello World")
fun doSomethingWithUnit(): Unit = println("Hello World")
```
* **Java void와 달리 Kotlin의 Unit 타입은 일반적인 타입으로, 타입 인자로 사용이 가능**하다.
* 이러한 특성으로 인해 제네릭 파라미터를 반환하는 함수의 오버라이드에 유용하게 사용이 가능하다.
  * 아래의 예시에서, MustReturnT 인터페이스의 메소드는 반드시 T를 반환하도록 한다.
  * 그러나 **이를 구현한 NoReturnsT의 메소드는 Unit 타입으로 구현되어 메소드가 아무 값도 반환하지 않도록 오버라이드**한다. 
  * Unit 타입의 메소드이므로, 실제로는 컴파일러에 의해 return Unit 문장이 자동으로 추가되어 Unit을 반환한다.
```
fun main(args: Array<String>) {
     val noReturn = NoReturnsT()
     // 실제로는 Unit 타입의 값이 묵시적으로 반환된다.
     val unit = noReturn.returnT()
     println(unit is Unit)
}

interface MustReturnT<T> {
     // 반드시 T를 반환하도록 구현되어야 한다.
     fun returnT(): T
}
// T를 Unit 타입으로 구현하여 메소드가 아무 것도 반환하지 않도록 오버라이드한다.
class NoReturnsT: MustReturnT<Unit> {
     override fun returnT() = println("no returns!")
}
```

### Nothing
* Kotlin에는 절대로 성공적으로 값을 돌려주는 일이 없고, 항상 실패하므로 반환 값의 의미가 무색한 메소드가 존재한다.
  * 이러한 경우를 표현하기 위해 Nothing 타입을 활용할 수 있다.
* **Nothing은 말 그대로 아무 값도 포함하지 않으므로, 함수의 반환형이나 반환형의 타입 파라미터로만 사용**할 수 있다.
* Nothing 타입은 아무런 값을 저장할 수 없으므로 변수 타입으로 사용할 수 없다.
* Nothing 타입은 컴파일러의 문맥 추론에 유용하게 사용된다.
  * 아래의 코드에서, 엘비스 연산자에 의해 호출되는 우항의 메소드는 반환 타입이 Nothing이다.
  * 엘비스 연산자에 의해, 좌항이 null인 경우 반드시 실패하는 코드이다.
  * 따라서 엘비스 연산자를 사용한 아랫줄부터는 변수가 null일 수 없음을 추론할 수 있다.
```
fun main(args: Array<String>) {
     val nullBird: Bird? = null
     // throwException 메소드의 반환형은 Nothing이므로, nullBird가 null인 경우 이 코드는 반드시 실패한다.
     val bird = nullBird ?: throwException()
     // 따라서 여기까지 진행된 코드의 경우, bird의 값은 절대 null일 수 없으므로 ?., !! 연산자 등을 사용할 필요가 없다.
     println(bird.name)
}

class Bird(val name: String?)
fun throwException(): Nothing = throw IllegalArgumentException("error!!!")
```

### 널이 될 수 있는 컬렉션, 널이 될 수 있는 요소
* List<Int>?와 List<Int?>는 다르다.
  1. List<Int>?: 컬렉션 자체가 null이 될 수 있으며, 컬렉션에 포함되는 모든 요소는 널이 될 수 없는 타입이다.
  2. List<Int?>: 컬렉션 자체는 null이 아니며, 컬렉션에 포함되는 모든 요소는 원시 타입이거나 null이다.
  * 만약 컬렉션 자체도 null이 될 수 있을 뿐더러 요소 역시 널이 될 수 있는 경우, List<Int?>? 형식으로 표현한다.
* 즉, 변수 타입의 널 가능성과 타입 파라미터의 널 가능성은 다르다.
  * 이 경우, 변수 타입은 List이며 타입 파라미터는 Int가 된다.
* 컬렉션에 null이 포함될 수 있는 경우, Kotlin 표준 라이브러리의 filterNotNull 등의 메소드를 통해 null을 걸러낼 수 있다.
  * 이 경우, filterNotNull의 결과는 널이 될 수 없는 타입의 컬렉션이다.
```
fun main(args: Array<String>) {
     val origin = listOf<Int?>(null, 1, null, 2)
     println(origin is List<Int?>)

     // null을 걸러내므로 결과는 List<Int>가 된다.
     val filtered = origin.filterNotNull()
     println(filtered is List<Int>)
     println(filtered)
}
```

### Collection, MutableCollection
* **Kotlin의 컬렉션 API는 Java와 달리 컬렉션 데이터에 접근하는 인터페이스와 컬렉션의 데이터를 변경하는 인터페이스가 분리**되어 있다.
  1. kotlin.collections.Collection: 컬렉션 내부의 요소를 순회하고, 컬렉션의 크기를 얻고, 값을 읽을 수 있도록 한다.
     * 예를 들어 size, iterator(), contains() 메소드 등을 지원한다.
  2. kotlin.collections.MutableCollection: 1.의 인터페이스를 확장하면서 컬렉션의 요소를 추가하거나 제거할 수 있도록 한다.
     * 예를 들어 Collection에 포함된 size, iterator(), contains() 메소드에 더해 add(), remove(), clear() 등을 지원한다.
* **가능하다면 항상 읽기 전용 인터페이스를 사용하는 것을 원칙으로 하되, 컬렉션의 변경이 필요한 경우에만 MutableCollection을 사용하도록 한다**.
* Collection과 MutableCollection의 구분은 인자의 타입만으로도 변경 가능성을 직관적으로 이해할 수 있도록 하기 위함이다.
* **MutableCollection이 Collection을 확장하므로, MutableCollection 파라미터에 읽기 전용 컬렉션을 전달할 수 없다**.
  * 같은 원리로, 읽기 전용 컬렉션의 실체가 항상 변경 불가능한 컬렉션인 것은 아니다.
* 아래의 예시에서, 같은 MutableCollection 인스턴스를 mutable과 collection 변수가 참조한다.
  * collection은 변경 불가능한 타입이지만 실제로는 MutableCollection 인스턴스이다.
  * **이러한 경우, collection 변수가 가리키는 인스턴스 실체와 mutable 변수의 존재로 인해 다중 스레드 애플리케이션에서 스레드 안전하지 않을 것**이다.
```
fun main(args: Array<String>) {
     val mutable: MutableCollection<Int> = mutableListOf(1, 2, 3)
     val collection: Collection<Int> = mutable

     println(collection) // [1, 2, 3]
     mutable.add(4)
     // collection 변수는 Collection 타입이므로, 자식 클래스의 메소드를 호출할 수 없다.
     // collection.add(4)
     println(collection) // [1, 2, 3, 4]
}
```