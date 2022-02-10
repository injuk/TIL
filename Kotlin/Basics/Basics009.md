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