# Kotlin
## 2022-02-09 Wed

### 생성자
* Java는 생성자를 하나 이상 선언할 수 있으며, 이는 Kotlin 역시 마찬가지이다.
* Kotlin은 Java와 달리 생성자를 다음의 두 종류로 구분한다.
  1. 주 생성자: 클래스를 초기화하는 간략한 생성자이며, 클래스 본문 밖에서 정의한다.
  2. 부 생성자: 클래스 본문 안에서 정의한다.
* 생성자 외에도 초기화 블록을 통해 클래스 초기화 로직을 추가할 수 있다.

### 주 생성자와 초기화블록
* 클래스의 선언은 일반적으로 다음과 같은 구조를 갖는다.
  * class 클래스이름(프로퍼티) { 클래스 본문 }
```
class Animal(val name: String) {}
```
* 이 때, **클래스 이름 뒤에 오는 괄호로 둘러쌓인 코드가 주 생성자이다**.
  * 상술한 예시에서는 (val name: String)이 된다.
* 주 생성자는 다음의 두가지 목적을 갖는다.
  1. 주 생성자는 생성자에 전달될 파라미터를 정의한다.
  2. **주 생성자는 전달된 파라미터에 의해 초기화되는 프로퍼티를 정의**한다.
* 위 코드 블록을 자세히 풀어쓰면 다음과 같다.
```
fun main(args: Array<String>) {
    val animal: Animal = Animal("ingnoh")
    println(animal.name)
}
class Animal constructor(name: String) {
    val name: String
    init {
        this.name = name
    }
}
```
* constructor 키워드는 주 생성자 또는 부 생성자의 정의에 사용된다.
* init 키워드는 초기화 블록을 명시한다.
  * 초기화 블록은 클래스가 인스턴스화될 때 실행될 초기화 코드가 정의된다.
  * **주 생성자는 코드의 작성이 제한적이므로, 초기화 블록을 함께 사용하여 별도의 코드를 작성**할 수 있다.
  * **초기화 블록은 필요시 여럿을 정의할 수 있다**.
* 상술한 코드는 다음과 같은 원칙에 의해 수정될 수 있다.
  1. 프로퍼티를 초기화하는 코드는 프로퍼티 선언에 포함시킬 수 있으므로, 굳이 초기화 블록을 작성할 필요가 없다.
  2. 주 생성자에 특정한 어노테이션이나 가시성 변경자가 없는 경우, constructor 키워드를 생략할 수 있다.
  * 이 때, **주 생성자에 전달된 파라미터는 프로퍼티를 초기화하는 식, 또는 초기화 블록 내부에서만 사용이 가능하다**.
```
class Animal(name: String) {
    val animalName: String = name
    // 아래의 코드는 실행 불가하다. 
    // 주 생성자에 전달된 파라미터는 윗 줄과 같은 프로퍼티 초기화 식 또는 초기화 블록 내부에서만 사용이 가능하기 때문이다.
    // fun sayHello() = println(name)
}
```
* 상술한 코드는 아래의 원칙에 의해 다시 한 번 간소화될 수 있다.
  1. 주 생성자에 전달된 파라미터로 프로퍼티를 초기화하는 식은 주 생성자 파라미터에 val 키워드를 추가하는 것으로 간소화할 수 있다.
* 아래의 코드를 포함하여, 앞선 모든 클래스 선언은 같은 내용을 정의한다.
  * 그러나 하기의 내용이 가장 간결하다.
```
fun main(args: Array<String>) {
    val animal: Animal = Animal("ingnoh")
    println(animal.name)
    // 아래의 코드는 주 생성자에 필요한 파라미터가 전달되지 않으므로 실행이 불가하다.
    // val anonymous: Animal = Animal()
    // println(anonymous.name)
}

class Animal(val name: String) {
     // 해당 식이 가리키는 name은 주 생성자에 전달된 파라미터가 아닌 Animal 클래스의 프로퍼티인 name이므로, 이제 접근이 가능하다.
     fun sayHello() = println(name)
}
```

### 생성자 파라미터의 디폴트 값 및 이름 붙은 파라미터
* 함수 파라미터와 마찬가지로 생성자 파라미터에도 디폴트 값을 정의할 수 있다.
* 프로퍼티가 여럿인 경우, 파라미터에 이름을 붙여 구분할 수 있다.
```
fun main(args: Array<String>) {
    // 파라미터에 이름을 붙여 초기화할 수 있다.
    val animal: Animal = Animal(age = 0, name = "ingnoh", isBird = true)
    println(animal.name)
    // 디폴트 값이 지정된 name은 생략할 수 있다.
    val anonymous: Animal = Animal(false, 1)
    println(anonymous.name)
}
class Animal(val isBird: Boolean, val age: Int, val name: String = "anonymous") {
     fun sayHello() = println(name)
}
```
* **모든 생성자 파라미터에 디폴트 값을 지정하면 컴파일러는 자동으로 파라미터가 없는 생성자를 만들어낸다**.
  * 파라미터 없는 생성자가 생성자 본문에서 디폴트 값을 활용하여 클래스를 초기화한다.
  * 이는 Kotlin 코드가 반드시 파라미터가 없는 생성자를 활용해야만 하는 일부 Java DI(의존성 주입) 라이브러리와 잘 통합되도록 한다.

### 부모 클래스, 인터페이스와 자식 클래스 / 구현 클래스의 선언
* 부모 클래스가 있는 **자식 클래스라면 주 생성자에서 우선 부모 클래스의 생성자를 호출하기 위해 다음의 형식으로 작성**한다.
```
open class Animal(val name)
// 자식 클래스는 파라미터로 받은 인자를 부모 클래스의 생성자에 넘겨 부모 클래스 생성자를 우선 호출한다.
class Bird(name): Animal(name)
```
* **Java와 마찬가지로, 생성자를 정의하지 않은 클래스는 컴파일러에 의해 기본 생성자가 생성된다**.
  * 자식 클래스의 경우, 부모 클래스에 생성자를 정의하지 않았더라도 기본 생성자를 호출하는 형식으로 작성해야 한다.
  * 때문에 **자식 클래스의 생성자 뒤에 명시되는 부모 클래스는 최소한 빈 괄호가 반드시 작성된다**.
```
// Java의 예를 들어, 아래 클래스는 생성자를 정의하지 않았으므로 Temp(){} 형식의 기본 생성자가 자동 생성될 것이다.
open class Temp
// 자식 클래스 역시 생성자가 없어도 되지만, 반드시 부모 클래스의 생성자를 호출하도록 작성한다.
class TempChild : Temp()
```
* 반면 인터페이스는 생성자가 없으므로, 구현 클래스에 괄호를 붙이지 않아도 무방하다.
  * 이러한 규칙으로 미루어 **여러 인터페이스와 부모 클래스를 상속 및 구현하는 클래스는 목록에서 괄호를 찾는 것으로 인터페이스와 부모를 구분할 수 있다**.
```
interface One
interface Two
open class Three
interface Four
// 정의만 보고도 Three가 부모 클래스이고, 나머지는 인터페이스임을 알 수 있다.
class ChildImpl: One, Two, Three(), Four
```

### 비공개 생성자
* 클래스를 클래스 외부에서 인스턴스화하지 못하도록 하고 싶은 경우, 모든 생성자를 private으로 정의한다.
  * **Java의 경우, 다음과 같은 상황에 인스턴스화를 막기 위해 private 생성자를 정의하는 경우가 있다**.
    1. 정적 유틸리티 함수만을 담아두는 클래스
       * 이러한 클래스는 함수를 모으는 역할만 수행하므로, 인스턴스화할 필요가 없다.
    2. 싱글톤이 필요한 클래스
       * 일반적으로 생성자가 아닌 별도의 팩토리 메소드와 같은 방법을 사용한다.
  * **Kotlin의 경우, 상술한 요구사항을 언어 차원에서 지원**한다.
    1. 정적 유틸리티 함수는 최상위 함수로 구현할 수 있다.
    2. 싱글톤이 필요한 클래스는 object 키워드를 활용하여 객체화할 수 있다.
```
fun main(args: Array<String>) {
    // 주 생성자가 private이므로 주 생성자를 통해 생성할 수 없다.
    // val restricted: Restricted = Restricted("ingnoh")
}
// 주 생성자를 private으로 정의하려면 constructor를 생략하지 않은 형식을 따르도록 한다.
class Restricted private constructor(val name: String)
```
* **일반적으로 클래스의 생성자는 단순한 형태를 갖기에 Kotlin은 주 생성자 문법을 제공**한다.
  * 대부분의 경우는 주 생성자 구문을 통해 대처할 수 있다.
  * 주 생성자만으로 대처하기 어려운 경우, 아래의 부 생성자 문법을 사용한다.

## 2022-02-10 Thu
### Kotlin의 부 생성자
* 일반적으로 Kotlin은 생성자가 여럿 필요한 경우가 적다.
  * **Java에서 생성자 오버로딩이 필요한 상황은 Kotlin의 디폴트 파라미터 값과 이름 붙인 인자 문법을 통해 해결이 가능**하다. 
  * **즉, 단순히 인자에 대한 디폴트 값을 제공하기 위한 부 생성자 정의는 지양해야 한다**.
* 그럼에도 생성자 오버로딩이 필요한 경우가 있을 수 있으며, 그 형식은 다음과 같다.
  1. **부 생성자가 필요한 주된 이유는 Java 애플리케이션과의 상호운용성 보장**이다.
  2. 또는 인스턴스화 과정에서 파라미터 목록이 다른 생성 방법이 반드시 여럿 존재해야 하는 경우에도 부 생성자를 활용할 수 있다.
  * 상술한 내용과 같은 상황의 경우, **부 생성자는 필요에 따라 얼마든지 정의할 수 있다**.
* **아래의 예시는 클래스 이름 뒤에 괄호가 없으므로, 주 생성자 없이 부 생성자만 둘 정의한 형태**이다.
```
fun main(args: Array<String>) {
    val overload1: Overload = Overload()
    val overload2: Overload = Overload("World")
}

class Overload {
     // 명시적으로 기본 생성자를 정의하나, 이는 부 생성자이다.
     constructor() {
          val name: String = "Hello"
          println(name)
     }
     constructor(name: String) {
          val name: String = name
          println(name)
     }
}
```
* 주 생성자도 있고 부 생성자도 있는 클래스의 경우, 반드시 this()를 호출하도록 구현되어야 한다.
  * **주 생성자와 같은 형태의 부 생성자는 정의할 수 없다**.
```
class Temp() {
     // 기본 생성자와 같은 형태의 생성자는 오버로딩할 수 없다.
     // constructor() : this() {}
     
     // 클래스의 주 생성자를 반드시 호출하도록 작성되어야 한다.
     constructor(name: String) : this() {}
     constructor(name: String, age: Int) : this() {}
}
```
* 부 생성자가 정의된 클래스를 상속 받는 자식 클래스의 예시는 다음과 같다.
  * **적절한 부모 클래스의 생성자를 반드시 호출하는 것을 확인할 수 있다**.
```
open class Overload {
     constructor(name: String) {}
     constructor(name: String, age: Int) {}
}
class Child: Overload {
     constructor(name: String): super(name) {}
     constructor(name: String, age: Int): super(name, age) {}
}
```

### 생성자의 위임
* 상술한 예시에서, 자식 클래스에 정의된 부 생성자의 형태는 다음과 같다.
```
constructor(name: String): super(name) {}
```
* 이는 **자식 클래스의 해당 생성자가 실제 객체 생성을 부모 클래스의 super(name)에 대응되는 생성자에게 위임한다는 의미를 갖는다**.
* Java와 마찬가지로, **클래스의 부 생성자에서 this() 키워드를 통해 클래스 자신의 다른 생성자에게 객체 생성을 위임할 수도 있다**.
  * 아래의 예시는 각 클래스에 정의된 부 생성자가 다른 생성자에게 객체 생성을 위임하는 코드이다.
```
class Temp {
     // 첫 번째 부 생성자는 두 번째 부 생성자에게 객체 생성을 위임한다.
     constructor(name: String) : this(name, 10) {}
     constructor(name: String, age: Int) {}
}
class Temp2(val name: String) {
     // 부 생성자는 주 생성자에게 객체 생성을 위임한다.
     constructor(): this("Hello")
}
```

### 인터페이스에 선언된 추상 프로퍼티
* Kotlin에서는 인터페이스에 추상 프로퍼티 선언을 다음과 같은 형식으로 정의할 수 있다.
```
fun main(args: Array<String>) {
     val animal: Animal = Bird()
     println(animal.name)
}
interface Animal {
     val name: String
}
// val name은 Animal 인터페이스로부터 구현하므로 override 키워드를 필수적으로 입력한다.
class Bird(override val name: String): Animal {
     constructor(): this("Hello")
}
```
* 추상 프로퍼티가 정의된 인터페이스는 다음을 의미한다.
  1. 추상 프로퍼티가 포함된 인터페이스를 구현하는 클래스는,
  2. **반드시 추상 프로퍼티의 값을 얻을 수 있는 방법을 제공해야만 한다**.

### 추상 프로퍼티가 포함된 인터페이스의 구현
* 추상 프로퍼티가 포함된 인터페이스를 구현하는 방법은 크게 다음과 같이 나뉜다.
  1. 구현 클래스의 주 생성자에서 추상 프로퍼티를 초기화하는 방법.
     * 주 생성자에서 프로퍼티를 직접 선언하는, 가장 간결한 방식이다.
  2. 구현 클래스 내부에서 커스텀 게터를 정의하는 방법.
     * 해당 방식은 프로퍼티를 뒷받침하는 필드에 값을 저장하는 것이 아닌, 매번 값을 계산하는 방식이다.
  3. 구현 클래스 내부에서 추상 프로퍼티를 초기화하는 방법.
     * 객체를 초기화하기 위해 별도로 구현된 함수로부터 값을 가져와 프로퍼티를 초기화한다.
     * 만약 DB 접근 등의 함수를 호출하여 초기화해야 하는 경우, 비용이 큰 작업이므로 객체 초기화 단계에 한 번만 호출되도록 구현한다.
* **2.와 3.의 방식은 유사해보이지만, 뒷받침 필드의 존재 유무와 값을 매 번 계산하는지, 초기화 시점에 한 번만 계산하는지에 대해 결정적인 차이가 있다**.
```
fun main(args: Array<String>) {
     val bird: Animal = Bird("Bird!")
     val dog: Animal = Dog("Dog!")
     val cat: Animal = Cat("Cat!")

     println(bird.name)
     println(dog.name)
     println(cat.name)
}

interface Animal {
     val name: String
}
// 1. 구현 클래스의 주 생성자에서 추상 프로퍼티를 초기화하는 방법.
class Bird(override val name: String): Animal
// 2. 구현 클래스 내부에서 커스텀 게터를 정의하는 방법.
class Dog(private val maybeName: String): Animal {
     override val name: String
          get() = maybeName
}
// 3. 구현 클래스 내부에서 추상 프로퍼티를 초기화하는 방법.
class Cat(private val maybeName: String): Animal {
     override val name: String = getName()
     private fun getName() = maybeName
}
```

### 커스텀 게터를 포함하는 인터페이스
* 인터페이스에는 추상 프로퍼티 뿐만 아니라 커스텀 게터가 있는 프로퍼티도 선언할 수 있다.
  * 커스텀 게터가 포함된 프로퍼티는 구현 클래스에서 오버라이드할 필요가 없다.
    * **추상 프로퍼티와의 차이점으로, 추상 프로퍼티는 구현하는 클래스에서 값을 초기화할 수 있는 방법을 반드시 제공해야 한다**.
  * 이러한 게터는 뒷받침하는 필드를 참조할 수 없다.
  * **프로퍼티를 뒷받침하는 필드는 인스턴스 상태의 저장을 의미하는 반면, 인터페이스는 절대로 상태값을 가질 수 없다**.
```
fun main(args: Array<String>) {
     val bird: Animal = Bird()
     println(bird.name)
}
interface Animal {
     val name: String
          get() = "Animal"
}
// 커스텀 게터가 포함된 프로퍼티는 오버라이드할 필요가 없다.
class Bird: Animal
```

### 게터와 세터에서 뒷받침 필드에 접근
* 프로퍼티에는 다음과 같은 두 유형이 있다.
  1. 값을 저장하는 프로퍼티
  2. 값을 저장할 수 없으며, 매 번 값을 계산하여 반환하는 프로퍼티
* 위 두 유형을 조합하면 다음과 같은 프로퍼티를 구현할 수 있다.
  1. 값을 저장하되,
  2. 값을 변경하거나 읽을 때마다 정해진 로직을 수행하는 프로퍼티
  * **이를 위해서는 게터와 세터 내부에서 프로퍼티를 뒷받침하는 필드에 접근할 수 있어야 한다**.
1. 잘 못 작성된 예시
* 아래의 예시는 커스텀 게터가 선언된 프로퍼티에 값을 명시적으로 저장하고자 하지만, 커스텀 게터는 뒷받침 필드를 가질 수 없으므로 컴파일되지 않는다.
  * 아래의 예시는 다음과 같은 두 방법 중 하나로 수정할 수 있다.
  1. 게터를 지워 1.번 유형의 프로퍼티로 만들어준다.
  2. 또는 name의 초기화 문을 제거하고, 게터에서 문자열을 반환하도록 하여 2.번 유형의 프로퍼티로 만들어준다.
  3. **또는 1.번과 2.번 유형을 조합하여 값을 저장하면서도 게터를 가질 수 있도록 수정한다**.
```
class Animal() {
    // 아래는 프로퍼티의 두 유형을 혼합하려고 시도하며, 컴파일 되지 않는다.
     val name: String = "animal"
     get() {
          println("name getter called")
          return name
     }
}
```
* 새로운 유형의 프로퍼티로 수정하면 다음과 같다.
  * name 프로퍼티는 이제 값을 저장하기 위한 뒷받침 필드가 존재하고, 커스텀 게터를 가진다.
  * **커스텀 게터 내부에서 사용된 키워드인 field는 프로퍼티의 값을 의미**한다.
```
class Animal() {
     val name: String = "animal"
     get() {
          println("name getter called")
          // 아래에서 리턴하는 키워드인 field는 name 프로퍼티의 값을 반환한다.
          return field
     }
     val age: Int = 1
     get() {
          println("age getter called")
          // 아래에서 리턴하는 키워드인 field는 age 프로퍼티의 값을 반환한다.
          return field
     }
}
```
* **field 키워드는 게터, 또는 세터 접근자의 본문에서 사용하는 식별자로 프로퍼티를 뒷받침하는 필드에 접근할 수 있도록 해준다**.
  * 게터는 field의 값을 읽을 수만 있고, 세터는 field의 값을 읽고 쓸 수 있다.
* 특별한 이유가 아니라면 게터 또는 세터 중 하나만 직접 정의해도 된다.
  * 대부분의 경우 게터와 세터는 특이한 작업을 수행하지 않기 때문에, 기본적으로 제공되는 게터와 세터를 사용하는 것이 바람직하다.
* var로 정의된 프로퍼티에 대해 게터, 세터를 모두 구현한 예제는 다음과 같다.
```
fun main(args: Array<String>) {
     val bird: Animal = Animal()
     println(bird.name)
     bird.name = "bird"
     println(bird.name)
}
class Animal() {
     var name: String = "animal"
     get() {
          println("name getter called")
          // 게터는 field 식별자의 값을 읽을 수만 있다.
          return field
     }
     set(value) {
          println("name has changed from $field to $value")
          // 세터는 field 식별자의 값을 읽고 쓸 수 있다.
          field = value
     }
}
```

### 뒷받침하는 필드가 있는 프로퍼티와 없는 프로퍼티의 차이
* **클래스의 프로퍼티를 사용하는 클라이언트 측에서, 프로퍼티를 뒷받침하는 필드의 존재 유무는 의미가 없다**.
* Kotlin 컴파일러는 디폴트 접근자를 사용하는 경우나, 커스텀 게터 / 세터를 정의한 경우 모두 field 식별자를 사용하는 프로퍼티에 뒷받침 필드를 생성한다. 
  * **예외적으로, 커스텀 게터 / 세터에서 field 식별자를 사용하지 않는다면 뒷받침 필드는 생성되지 않는다**.
  * 때문에 아래의 코드는 커스텀 게터에서 field를 사용하지 않지만 뒷받침하는 필드를 사용하려 시도하므로 컴파일할 수 없다.
```
class Animal() {
     // 게터에서 field를 사용하지 않으므로, Kotlin 컴파일러는 뒷받침하는 필드를 생성하지 않는다.
     // 따라서 프로퍼티에 값을 저장할 수 없으므로, 아래의 코드는 컴파일할 수 없다.
     val name: String = "animal"
     get() {
          println("name getter called")
          return "temp"
     }
}
```

### 게터 / 세터 접근자의 가시성 변경
* **두 접근자의 가시성은 기본적으로 프로퍼티의 가시성과 같다**.
* 예를 들어, 다음의 코드는 클래스 외부에서 프로퍼티의 값을 변경할 수 없도록 한다.
  * 또한 Java의 고전적인 세터를 작성하여 클래스 내부의 메소드는 private 세터에 접근할 수 있음을 보여준다.
  * 반면 name 프로퍼티 자체는 public이므로, **컴파일러가 생성하는 디폴트 게터 역시 public으로 정의되어 외부에서 값을 읽어들일 수 있다**.
```
fun main(args: Array<String>) {
     val bird: Animal = Animal()
     println(bird.name)
     // 세터가 private이므로 외부에서 값을 직접 변경할 수는 없다.
     // bird.name = "bird"
     bird.setName("bird")
     println(bird.name)
}
class Animal() {
     var name: String = "animal"
         private set
     // 고전적인 세터를 직접 정의하는 경우, 클래스 내부에서 private 세터에 접근할 수 있다.    
     fun setName(value: String) {
          name = value
     }
}
```