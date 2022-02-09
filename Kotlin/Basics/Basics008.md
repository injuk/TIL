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