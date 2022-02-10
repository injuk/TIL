# Kotlin
## 2022-02-10 Thu

### Kotlin의 equals, hashCode, toString
* Java에서는 생성자, 접근자 뿐만 아니라 equals, hashCode 등 클래스가 구현해야하는 메소드가 많다.
  * IDE에 의해 자동 생성된다고 해도 코드베이스가 지저분해지는 일을 피할 수는 없다.
* **Kotlin 컴파일러는 이러한 기계적인 작업을 보이지 않는 곳에서 자동으로 생성해준다**.
  * 앞선 생성자와 접근자의 예시와 같이 Kotlin은 자동으로 코드를 생성하며 코드베이스를 깔끔하게 유지할 수 있도록 한다.
* Java와 마찬가지로 Kotlin 역시 Object로부터 상속받는 toString, equals, hashCode 등을 오버라이드할 수 있다.
* toString: 기본적인 인스턴스의 문자열 표현은 [클래스명]@[값]의 형식이다.
* equals: **객체의 동등성을 검사하며, Kotlin의 == 연산자는 참조 동일성이 아닌 객체 동등성을 검사하기 위해 equals를 호출**한다.
  * Java에서는 == 연산자의 동작 방식이 원시 타입과 참조 타입에 따라 다음과 같다.
    * 원시 타입: 값의 동등성을 비교한다.
    * 참조 타입: 참조 비교이며, 각 참조 변수가 참조하는 객체의 주소가 같은지를 검사한다.
    * **때문에 객체 간의 == 연산은 원하는대로 동작하지 않으며, 비교를 위해서는 equals를 호출해야 한다**.
  * **Kotlin에서는 == 연산자가 두 객체를 비교하는 기본적인 방법**이다.
    * == 연산자는 내부적으로 equal 메소드를 호출하여 객체를 비교하도록 동작한다.
    * 따라서 클래스가 equals 메소드를 오버라이드하면, == 연산자를 통해 호출되는 equals 메소드로 두 객체를 안전하게 비교할 수 있다.
* equals 메소드를 오버라이드 하는 예시는 다음과 같다.
  * 아래의 예시에서, === 연산자는 Java의 참조 비교와 같은 동작을 수행한다. 
```
fun main(args: Array<String>) {
     // Dog 클래스는 equals가 오버라이드되지 않았으므로 false가 반환된다.
     val dog1: Dog = Dog("dog", 1)
     val dog2: Dog = Dog("dog", 1)
     println(dog1 == dog2)

     // Bird 클래스는 equals가 오버라이드되었으므로 == 연산 결과에 의해 true가 반환된다. 
     val bird1: Bird = Bird("bird", 1)
     val bird2: Bird = Bird("bird", 1)
     println(bird1 == bird2)
     // 위 연산과 같은 동작이지만, 위 방식이 더 권장된다.
     // println(bird1.equals(bird2))
     
     // 반면 === 연산자는 Java의 참조 비교를 수행하므로 false가 반환된다.
     println(bird1 === bird2)
}
class Dog(
     val name: String,
     val age: Int
)
class Bird(
     val name: String,
     val age: Int,
) {
     // Any는 Java의 Object에 대응되며, ?가 붙으므로 null일 수 있다. 
     // 즉, equals(null)과 같은 형식일 수 있다.
     override fun equals(other: Any?): Boolean {
          println("Bird equals called")
          // is 연산은 Java의 instanceof와 같다.
          if(other == null || other !is Bird)
               return false

          return name == other.name && age == other.age
     }
}
```
* **JVM 기반 언어의 경우, equals를 재정의하는 경우에는 반드시 hashCode도 함께 재정의되어야** 한다.
  * 정확히는 **JVM 언어에서는 equals가 true인 경우 hashCode에 의해 반환되는 값이 같아야 한다는 제약**이 있다.
  * 따라서 위 예시는 hashCode의 오버라이드가 누락되었으므로, 복잡한 연산에서는 정상 동작하지 않는다.
* 예를 들어, HashSet의 경우 원소 비교에 다음과 같은 방식으로 동작한다.
  1. 비용을 줄이기 위해, 우선 hashCode를 비교한다.
  2. hashCode가 같은 경우에만 equals를 호출하여 비교한다.
  * 이는 **해시 기반 컨테이너 클래스들이 key 값으로 hashCode를 사용하기 때문**이다.
* 때문에 위 예시에서 구현된 코드는 이러한 hash 컬렉션의 연산에 영향을 전혀 주지 못한다.
```
fun main(args: Array<String>) {
     val bird1: Bird = Bird("bird", 1)
     val bird2: Bird = Bird("bird", 1)

     // false가 반환되며, equals 메소드는 호출되지 않는다.
     val set = hashSetOf(bird1)
     println(set.contains(bird2))

     // equals 메소드가 호출되어 true가 반환된다.
     println(bird1 == bird2)
}
class Bird(
     val name: String,
     val age: Int,
) {
     override fun equals(other: Any?): Boolean {
          println("Bird equals called")
          if(other == null || other !is Bird)
               return false

          return name == other.name && age == other.age
     }
}
```
* 이러한 **문제의 원인은 equals를 오버라이드했지만 hashCode는 오버라이드하지 않았기 때문**이다.
  * 코드에 정의된 Bird 클래스 내부에서 hashCode를 다음과 같이 오버라이드할 경우 정상 동작하는 것을 확인할 수 있다.
```
class Bird(
     val name: String,
     val age: Int,
) {
     // equals 구현은 생략
     override fun hashCode(): Int {
          println("Bird hashCode called")
          return name.hashCode() + age
     }
}
```

### 데이터 클래스
* 상술한 equals, hashCode, toString의 오버라이드 작업은 기계적이다.
  * 때문에 **Kotlin은 이러한 메소드를 자동으로 생성할 수 있는 기능을 제공**한다.
  * **이 기능은 어떤 클래스가 데이터를 저장하는 역할만 수행할 때 적용할 수 있다**.
* **데이터를 관리하는 클래스의 앞에 data 키워드를 붙이면 Kotlin 컴파일러는 상술한 모든 메소드를 자동 생성**한다.
  * 이렇듯 data 키워드가 붙은 클래스는 데이터 클래스라는 명칭으로 부를 수 있다.
* data 키워드를 사용하는 것만으로 코드는 다음과 같이 간단해진다.
```
fun main(args: Array<String>) {
     val bird1: Bird = Bird("bird", 1)
     val bird2: Bird = Bird("bird", 1)

     val set = hashSetOf(bird1)
     println(set.contains(bird2))
     println(bird1 == bird2)
     println(bird1)
}
// 불필요한 오버라이드 코드가 모두 제거된다!
data class Bird( val name: String, val age: Int )
```

### 데이터 클래스의 메소드 자동 생성
* 데이터 클래스는 다음과 같은 메소드가 자동 생성되어 포함된다.
  1. 클래스의 인스턴스 비교를 위한 equals 메소드
  2. 해시 기반 컨테이너 클래스에서 키로 사용하기 위한 hashCode 메소드
  3. 클래스의 각 필드를 선언 순서대로 표시하는 toString 메소드
     * 위 예시에서, toString의 결과는 Bird(name=bird, age=1) 형태이다.
* 자동 생성되는 equals와 hashCode는 **주 생성자의 모든 프로퍼티 목록을 고려하여 오버라이드**된다.
  * 예를 들어, equals는 모든 프로퍼티의 동등성을 확인하고 hashCode는 모든 프로퍼티의 해시 값을 바탕으로 계산된다.
* 주 생성자 밖에 정의된 프로퍼티는 equals와 hashCode 메소드 계산에 고려되지 않는다.
* 나아가 데이터 클래스는 copy와 같은 몇가지 유용한 메소드를 추가로 자동 생성한다. 


### 데이터 클래스와 불변성
* **데이터 클래스의 프로퍼티가 반드시 val일 필요는 없으나, 읽기 전용 프로퍼티만을 포함하는 불변 객체로 만드는 것이 권장**된다.
* 불변 객체인 데이터 클래스는 다음과 같은 몇 가지 장점이 있다.
  1. 해시 기반 컨테이너 클래스의 동작이 잘못될 일이 없다.
    * 불변 객체가 아닌 경우, 컨테이너에 포함된 데이터 인스턴스의 프로퍼티가 변경되면 컨테이너의 상태가 잘못될 수 있다.
  2. 애플리케이션을 훨씬 더 쉽게 이해할 수 있다.
  3. **다중 스레드 애플리케이션에서 스레드들이 서로가 사용하는 객체의 상태를 변경할 수 없으므로 스레드 동기화 코드가 줄어든다**.

### 데이터 클래스와 copy 메소드
* 데이터 클래스의 인스턴스를 불변 객체로 쉽게 활용할 수 있도록 Kotlin 컴파일러는 copy 메소드를 제공한다.
  * copy 메소드는 객체를 복사하는 과정에서 일부 프로퍼티의 값을 변경할 수 있도록 해준다.
* **복사본은 원본과 다른 생명 주기를 가지며, 복사본에 가해진 변경 사항은 원본에 영향을 주지 못한다**.
* copy 메소드는 '[수신객체].copy(변경하고 싶은 프로퍼티 정의)' 와 같은 형태로 사용한다.
```
fun main(args: Array<String>) {
     val bird1: Bird = Bird("bird", 1)
     // 복사된 bird2는 원본과 같은 프로퍼티를 갖고, 동등하게 취급된다.
     val bird2: Bird = bird1.copy()
     // 복사된 bird3는 복사 과정에서 원본과 다른 프로퍼티를 갖게 되어 동등하지 않다.
     val bird3: Bird = bird1.copy(name = "bird3", age = 10)

     val set = hashSetOf(bird1)
     println(set.contains(bird2))
     println(bird1 == bird2)

     println(set.contains(bird3))
     println(bird1 == bird3)
}
data class Bird( val name: String, val age: Int )
```

### 클래스의 위임
* 구현과 상속은 종종 애플리케이션에 다음과 같은 문제를 일으킬 수 있다.
* 자식 클래스가 부모 클래스의 메소드를 오버라이드하는 경우, 부모 클래스의 세부 구현 사항에 의존하게 된다.
  * 만약 요구사항에 의해 부모 클래스의 구현이 변경되는 경우, 자식 클래스는 이에 영향을 받아 예상치 못한 문제가 발생할 수 있다.
* Kotlin은 이러한 문제점에 대비하기 위해 기본적으로 final로 취급하되, 필요한 경우 open을 통해 상속을 신중히 수행하도록 설계되었다.
* 그럼에도 때로는 상속을 허용하지 않는 클래스에 기능을 추가해야하는 경우가 있으며, 이 때 일반적으로 데코레이터 패턴을 적용한다.
  * [Decorator 패턴](https://github.com/injuk/TIL/blob/master/Code/DesignPattern/DesignPattern009.md)
  * **그러나 데코레이터 패턴은 기존 클래스의 기능을 데코레이터 클래스에 위임하기 위한 준비 코드가 많이 작성되어야 하는 단점**이 있다.
* **Kotlin은 이러한 위임 기능을 언어가 제공하는 일급 시민 기능으로 지원**한다.
  * 아래의 Decorator 클래스는 내부 필드로 Animal 인스턴스를 갖고, 각 메소드 호출시 animal에 포워딩하는 코드가 반복적으로 작성된다.
  * **반면 DecoratorWithBy 클래스는 파라미터로 전달 받은 animal을 통해 메소드 자동 생성 과정을 컴파일러에게 맡긴다**.
```
interface Animal {
     fun move(): String
     fun say(): String
     fun isCat(): Boolean
     fun getAge(): Int
}
class Decorator(private val animal: Animal): Animal {
     override fun move() = animal.move()
     override fun say() = animal.say()
     override fun isCat() = animal.isCat()
     override fun getAge() = animal.getAge()
}
// Animal by animal을 통해 Animal 인터페이스의 메소드 구현 작업은 animal 인스턴스에게 위임한다
class DecoratorWithBy(private val animal: Animal): Animal by animal
```
* by 키워드를 활용한 클래스 위임 기능은 단순한 코드 작성 작업을 줄여준다.
* 상술한 예시에서, by 키워드가 활용된 Animal by animal의 의미는 다음과 같다.
  * **Animal 인터페이스의 메소드 구현 작업은 animal 인스턴스에게 위임한다**.
* 만약 위임되는 메소드 중 일부의 동작을 변경하고 싶은 경우, 클래스 본문에서 오버라이드하는 것으로 동작을 수정할 수 있다.
  * **이 경우 컴파일러가 생성한 메소드 대신 개발자가 명시적으로 오버라이드한 메소드를 사용**한다.
  * 당연히 기존 클래스로부터 위임되는 기본 동작으로 충분하다면, 명시적으로 오버라이드할 필요는 없다.
```
class DecoratorWithBy(private val animal: Animal): Animal by animal {
     // 오버라이드가 필요한 메소드만 재정의한다.
     override fun getAge(): Int = animal.getAge() + 10
}
```

### 클래스 위임의 의의
* 위의 예시에서, **DecoratorWithBy 클래스는 Animal 클래스의 세부 구현 사항에 더 이상 의존되지 않는다**.
* DecoratorWithBy 클래스를 호출할 때 발생하는 일은 개발자가 제어할 수 있지만, 이 과정에서 실제 동작은 Animal 클래스의 메소드를 활용한다.
* **따라서 Animal의 문서화된 API가 변경되지 않는 이상 DecoratorWithBy 클래스의 코드는 계속해서 잘 동작할 것임을 확신할 수 있다**.

## 2022-02-11 Fri
### object 키워드
* Kotlin에서 object 키워드는 다양한 상황에서 사용되며, 모두 공통적으로 **클래스 정의와 동시에 인스턴스를 생성**한다는 공통점이 있다.
  * 아래의 예시에서, Animal 클래스는 별도로 인스턴스화 과정이 없다.
```
fun main(args: Array<String>) {
     // 인스턴스화 없이 호출이 가능하다.
     Animal.move()
}
object Animal {
     fun move() = println("animal moving")
}
```
* object 키워드는 다음과 같은 요구사항에서 사용을 고려할 수 있다.
  1. 객체에 싱글톤 패턴을 적용해야 하는 경우
  2. 인스턴스의 메소드는 아니지만, 관련 있는 메소드나 팩토리 메소드를 담기 위한 동반 객체(companion object)를 사용하려는 경우
  3. Java의 익명 내부 클래스를 대신하여 사용하려는 경우

### object 키워드와 싱글톤
* 객체지향 프로그램에서는 인스턴스가 단 하나만 필요한 경우가 생기기 마련이다.
  1. Java에서는 일반적으로 생성자를 private으로 제한하고, 클래스의 정적 멤버에 유일한 객체를 저장하는 방식으로 구현한다.
  2. **Kotlin에서는 object 키워드가 제공하는 기능을 통해 싱글톤 패턴을 언어 차원에서 지원**한다.
* Kotlin에서 object 키워드를 사용한 방식인 '객체 선언'은 클래스의 선언과 해당 클래스의 단일 인스턴스 선언을 합친 선언이다.
* Kotlin에서의 객체 선언은 다음의 동작을 단 한 줄로 처리한다.
  1. 객체 선언은 클래스를 정의하고,
  2. 그 클래스의 인스턴스를 생성하여 변수에 저장하는 모든 작업을 수행한다.
* **클래스와 마찬가지로 객체 선언 내부에도 프로퍼티, 메소드, 초기화 블록을 사용할 수 있지만 생성자는 사용할 수 없다**.
  * 주 생성자 / 부 생성자 모두 사용할 수 없다.
  * **이는 객체 선언으로 생성되는 싱글톤 인스턴스가 객체 선언문이 있는 위치에서 생성자 호출 없이 즉시 생성되기 때문**이다.
* 객체 선언 내부의 프로퍼티나 메소드에는 다른 클래스와 마찬가지로 참조 기호인 온점(.)을 이용하여 접근한다.
* **객체 선언 역시 클래스나 인터페이스를 상속할 수 있다**.
  * 특히 인터페이스 구현 내부에 별다른 상태값이 필요하지 않은 경우에 유용하다.
  * **예를 들어 Comparator 인터페이스는 구현 과정에서 상태가 필요 없고, 일반적으로 클래스마다 하나의 인스턴스만 존재해도 된다**. 
    * 이러한 상황이 객체 선언을 사용하기에는 최적이다!
```
fun main(args: Array<String>) {
     val bird1: Bird = Bird("bird")
     val bird2: Bird = Bird("aird")
     println(BirdComparator.compare(bird1, bird2)) // 1
     println(BirdComparator.compare(bird2, bird1)) // -1
     val bird3: Bird = Bird("aird")
     println(BirdComparator.compare(bird2, bird3)) // 0
}
data class Bird(val name: String = "")
object BirdComparator: Comparator<Bird> {
     override fun compare(o1: Bird?, o2: Bird?): Int {
          if(o1 == null || o2 == null)
               throw NullPointerException()
          return o1.name.compareTo(o2.name)
     }
}
```
* 일반적인 인스턴스를 사용할 수 있는 곳이라면 객체 선언에 의해 생성된 싱글톤 인스턴스도 넘길 수 있다.
  * 상술한 예시에서, Compartor를 파라미터로 받는 메소드에 BirdComparator를 넘길 수 있을 것이다.

### 중첩 객체와 객체 선언
* 클래스 내부에도 객체 선언 키워드인 object를 사용할 수 있다.
  * 이 역시 **외부 클래스의 인스턴스 개수에 영향을 받지 않고 객체 선언된 중첩 클래스는 단 하나만이 존재**한다.
* 때문에 **상술한 예시에서, Comparator는 자신과 밀접한 관련이 있는 클래스 내부에 정의되는 것이 더 바람직하다**.
  * 중첩 객체로 정의된 Comparator 클래스에 객체 선언을 정의하면 외부 클래스의 인스턴스 수와 관계 없이 싱글톤이 적용되기 때문이다. 
```
fun main(args: Array<String>) {
     val bird1: Bird = Bird("bird")
     val bird2: Bird = Bird("aird")
     println(Bird.BirdComparator.compare(bird1, bird2)) // 1
     println(Bird.BirdComparator.compare(bird2, bird1)) // -1
     val bird3: Bird = Bird("aird")
     println(Bird.BirdComparator.compare(bird2, bird3)) // 0
}
data class Bird(val name: String = "") {
     object BirdComparator: Comparator<Bird> {
          override fun compare(o1: Bird?, o2: Bird?): Int {
               if(o1 == null || o2 == null)
                    throw NullPointerException()
               return o1.name.compareTo(o2.name)
          }
     }
}
```
* Kotlin의 객체 선언은 내부적으로 유일한 인스턴스에 대한 정적 멤버가 있는 Java 클래스로 컴파일된다.
  * 이 때, 정적 인스턴스 멤버의 이름은 항상 INSTANCE이다.
  * 따라서, 상술한 중첩 객체를 적용하지 않았던 예시의 BirdComparator는 Java 코드에서 다음과 같은 형식으로 호출해야 한다.
  * BirdComparator.INSTANCE.compare(bird1, bird2)

### 동반 객체(companion object)
* **동반 객체는 정적 멤버와 팩토리 메소드를 정의하기 위해 사용**한다.
* Kotlin 클래스에는 정적인 멤버가 존재하지 않으며, 대신 다음의 대안을 사용할 수 있다.
  1. Java의 정적 메소드는 패키지 수준의 최상위 함수로 대체한다.
  2. Java의 정적 필드, 또는 정적 메소드 역할 중 1.이 대체할 수 없는 용도에 object 키워드를 활용한 객체 선언을 사용한다.
  * **두 방식 중에서는 대부분의 경우 최상위 함수가 권장**된다.
* 그러나 **최상위 함수는 다른 클래스의 private 멤버에 접근할 수 없다**.
* 만약 **클래스의 인스턴스와 관계 없이 호출이 필요하지만, 클래스의 내부 정보에 접근해야 하는 경우에 동반 객체를 활용**할 수 있다.
  * Kotlin에서는 클래스에 중첩된 객체 선언의 멤버 메소드로 정의하는 것으로 구현할 수 있다.
  * **대표적인 예시로 팩토리 메소드**가 있다.
* 동반 객체는 클래스 내부에 정의된 객체에 companion 키워드를 붙이는 것으로 정의할 수 있다.
  * **동반 객체의 프로퍼티나 메소드에 접근하려면, 동반 객체가 포함되어 있는 외부 클래스의 이름을 사용**한다.
  * 이는 마치 Java의 정적 메소드 호출 또는 정적 필드 사용 구문처럼 인스턴스의 이름을 지정하지 않는 것과 유사하다.

### 동반 객체와 private 생성자
* **private 생성자를 호출하기 가장 좋은 위치는 동반 객체**이다.
  * **동반 객체는 자신을 둘러싼 클래스의 모든 private 멤버에 접근할 수 있으므로**, private 생성자도 호출이 가능하다.
* **동반 객체의 내부는 팩토리 패턴을 구현하기에 가장 적합한 위치**이다.
  * 예를 들어, **부 생성자를 여럿 포함하는 클래스는 동반 객체에 부 생성자를 정의하는 것이 더 바람직**하다.
```
fun main(args: Array<String>) {
     // 생성자가 private이므로 이런 형식으로 인스턴스화할 수 없다.
     // val animal1: Animal = Animal("bird")
     val animal1: Animal = Animal.getInstance("bird")
     val animal2: Animal = Animal.getInstance("cird")
     val animal3: Animal = Animal.getInstance("dird")

     animal1.sayHello()
     animal2.sayHello()
     animal3.sayHello()
}

class Animal private constructor(private val name: String) {
     companion object {
          fun getInstance(name: String) = Animal(name)
     }
     fun sayHello() = println("Hello! My name is $name")
}
```
* 이러한 팩토리 메소드는 팩토리 메소드가 선언된 클래스의 하위 클래스가 있는 경우, 하위 클래스 인스턴스를 반환할 수도 있으므로 매우 유용하다.
  * 또한 팩토리 메소드를 활용할 경우, 생성할 필요가 없는 인스턴스는 생성하지 않고 기존 인스턴스를 반환하는 등의 활용이 가능하다.
* 그러나 **클래스가 확장될 가능성이 있는 경우, 동반 객체에 정의된 멤버는 오버라이드가 불가능하므로 여러 부 생성자를 사용하는 것이 바람직**하다.

### 동반 객체의 명명
* **동반 객체는 클래스 내부에 정의된 일반 객체이다**.
* 때문에 동반 객체 역시 인터페이스를 상속하거나, 이름을 붙이거나, 확장 함수와 프로퍼티의 정의가 가능하다.
  * 이름 붙인 동반 객체는 기존 호출 방식과 더불어 클래스.동반객체명.동반객체멤버 형식으로 사용할 수도 있다.
  * 대부분의 경우 명명에 고민하지 않도록 이름을 붙이지 않는 것이 권장되지만, 필요시 동반 객체에도 명명할 수 있다.
  * 만약 동반 객체에 명명하지 않은 경우, 컴파일러는 자동으로 Companion이라는 이름으로 정의한다.
```
fun main(args: Array<String>) {
     val animal1: Animal = Animal.Factory.getInstance("bird")
     val animal2: Animal = Animal.getInstance("cird")

     animal1.sayHello()
     animal2.sayHello()
}

class Animal private constructor(private val name: String) {
     companion object Factory {
          fun getInstance(name: String) = Animal(name)
     }
     fun sayHello() = println("Hello! My name is $name")
}
```

### 동반 객체의 인터페이스 구현
* **다른 객체 선언과 마찬가지로 동반 객체 역시 인터페이스의 구현이 가능**하다.
* 예를 들어, 많은 유형의 Animal 인스턴스가 존재할 수 있고, 이는 모두 팩토리 메소드를 활용하여 생성된다고 가정하면 다음과 같은 코드를 작성할 수 있다.
  * 이 때, Animal 클래스 내부에 정의된 동반 객체를 참조하려면 Animal 클래스의 이름을 그대로 사용할 수 있다.
    * 아래의 예시에서, fromFactory 메소드는 AnimalFactory를 구현한 클래스의 인스턴스를 전달받아야 한다.
    * Animal 클래스의 동반 객체는 해당 클래스를 구현하지만, Animal 클래스 자체는 AnimalFactory와 관계가 없다.
    * 그러나 **fromFactory에 동반 객체를 넘기기 위해 Animal 클래스를 전달**한다.
```
fun main(args: Array<String>) {
     // AnimalFactory와 Animal은 관계가 없음에도 Animal의 동반 객체를 넘기기 위해 Animal 클래스를 사용한다.
     val animal: Animal = fromFactory(Animal)
     println(animal.name)
}

interface AnimalFactory {
     fun getAnimal(): Animal
}
class Animal(val name: String) {
     companion object: AnimalFactory {
          override fun getAnimal() = Animal("animal!")
     }
}
fun fromFactory(animalFactory: AnimalFactory) = animalFactory.getAnimal()
```

### 동반 객체의 확장
* 비즈니스 모듈과 데이터 타입을 분리하는 등, 동반 객체 내부의 메소드를 외부에 물리적으로 분리시키고 싶은 경우에 확장 함수를 사용할 수 있다.
* 동반 객체의 이름을 지정하지 않아 기본 값인 Companion이 정의되는 경우를 예로 들어 확장 함수를 활용한다면 코드는 다음과 같아진다.
  * 이 경우, 확장 함수는 [외부클래스명].[동반객체명].[확장함수명]의 형태로 정의 되지만 호출은 여전히 [외부클래스명].[확장함수명]의 형태이다.
```
fun main(args: Array<String>) {
     // 동반 객체를 사용하지만, 호출은 확장 함수의 형태와 같다.
     val animal: Animal = Animal.getAnimal()
     println(animal.name)
}

class Animal(val name: String) {
     // 빈 동반 객체를 정의한다.
     companion object
}
// 동반 객체 내부의 코드는 확장 함수로 옮긴다.
fun Animal.Companion.getAnimal() = Animal("animal!")
```
* 이러한 방식은 마치 동반 객체 내부에 정의된 함수를 호출하는 것처럼 메소드를 호출할 수 있도록 한다.
  * 그러나 **실제로는 클래스 밖에서 동반 객체에 대해 정의한 확장 함수이므로, 클래스의 private한 멤버에 접근할 수 없어진다**.
  * 멤버 함수처럼 호출하지만 실제로는 확장 함수이므로, 멤버 함수가 아니다. 
* **상술한 코드와 같이 동반 객체에 대한 확장 함수의 정의는 반드시 원본 클래스에 동반 객체가 정의되어 있어야 한다**.
  * 빈 동반 객체를 정의하더라도 무방하다.

### 객체 식
* 무영 객체를 정의할 때에도 object 키워드를 활용할 수 있다.
* **무명 객체는 Java의 무명 내부 클래스를 대체하는 개념**이다.
  * Java의 경우, 무명 내부 클래스는 하나의 인터페이스 또는 하나의 클래스만 구현 / 확장이 가능하다.
  * Kotlin의 경우, 무명 객체는 여러 인터페이스를 구현하거나 클래스를 확장과 동시에 인터페이스의 구현이 가능하다.
* 객체 식은 객체 선언에서 인스턴스의 이름이 빠진 형태로 작성한다.
* **객체 식은 클래스를 정의하고 해당 클래스의 인스턴스를 생성하지만, 클래스나 인스턴스에 이름을 붙이지는 않는다**.
  * 만약 객체에 이름을 붙여야 하는 경우라면, 변수에 무명 객체를 대입한다.
```
fun main(args: Array<String>) {
     // 특정 타입의 인스턴스가 필요한 메소드의 호출에 무명 객체를 활용하여 인스턴스화할 수 있다.
     println(isAnimal( object : Animal(){} ))
     val animal = Bird()
     println(isAnimal(animal.anonymous))
}

open class Animal
class Bird : Animal() {
     // 무명 객체에 이름을 붙이고 싶다면 변수에 대입한다.
     val anonymous = object: Animal() {
          fun sayHello() = println("Hello")
     }
}
fun isAnimal(animal: Any) = animal is Animal
```
* 객체 선언과 달리 무명 객체는 싱글톤 패턴이 적용되지 않는다.
  * 즉, **객체 식을 사용할 때마다 생성되는 무명 객체는 모두 별개의 새로운 인스턴스**이다.
* Java의 무명 내부 클래스와 마찬가지로, 객체 식 내부의 코드는 그 식이 포함된 함수의 변수에 접근이 가능하다.
  * Java와 달리, final이 아닌 변수도 객체 식 안에서 사용할 수 있다.
```
fun main(args: Array<String>) {
     count()
}

fun count() {
     var count = 10
     // 무명 객체는 어떠한 인터페이스나 클래스도 확장하지 않을 수 있다.
     val anonymous = object {
          fun printCount() {
               // 무명 객체는 자신이 포함된 함수의 지역 변수에 접근할 수 있다.
               count++
               count++
               println(count)
          }
     }
     anonymous.printCount()
}
```
* **객체 식은 무명 객체 내부에서 여러 메소드를 오버라이드 하는 경우에 유용**하다.
  * Runnable과 같이 **하나의 메소드만을 오버라이드하는 경우, Kotlin이 지원하는 SAM 변환을 활용하는 것이 바람직**하다.
  * SAM 변환의 경우, 무명 객체 대신 람다와 같은 함수 리터럴을 사용한다.
* SAM이란 Single Abstract Method의 약자이며, 이름 그대로 추상 메소드가 하나만 있는 인터페이스이다.
  * **Java의 경우 상당수의 인터페이스가 SAM에 해당하며, 함수형 인터페이스라고도 부른다**.

### 결론
* Kotlin에서, 인터페이스는 Java 인터페이스와 유사하지만 디폴트 구현과 프로퍼티를 포함할 수 있다.
  * 디폴트 구현은 Java의 default 메소드에 대응한다.
* Kotlin에서, 모든 선언은 기본적으로 public final이다.
* Kotlin에서, 중첩 클래스는 기본적으로 내부 클래스가 아니므로 외부 참조를 포함하지 않는다.
  * 외부 참조를 포함해야 하는 내부 클래스의 정의에는 inner 키워드를 사용한다.
* Kotlin에서, sealed 클래스를 상속하는 클래스를 정의하려면 다음의 방식 중 하나를 선택한다.
  1. 부모 클래스 내부에 자식 클래스를 중첩(또는 내부)시키거나,
  2. 부모 클래스와 자식 클래스를 같은 파일 안에 정의한다.
* Kotlin에서, data 키워드를 붙이는 것으로 Kotlin 컴파일러가 equals, hashCode, toString, copy 등의 유용한 메소드를 자동 생성하도록 할 수 있다.
* Kotlin에서, 클래스 위임을 사용하는 것으로 위임 패턴에 필요한 보일러 플레이트 코드를 줄일 수 있다.
* Kotlin에서, 객체 선언을 활용하는 것으로 Kotlin다운 싱글톤 클래스를 정의할 수 있다.
* Kotlin에서, 동반 객체는 최상위 함수와 프로퍼티와 더불어 Java의 정적 메소드와 필드 정의를 대체한다.
* Kotlin에서, 동반 객체 역시 다른 객체와 마찬가지로 인터페이스를 구현할 수 있다.
* Kotlin에서, 객체 식은 Java의 무명 내부 클래스를 대신하며 더 많은 기능을 제공한다.
  * 예를 들어, 여러 인터페이스 / 부모 클래스로부터 확장하거나 객체가 포함된 영역의 변수에 접근할 수 있다.