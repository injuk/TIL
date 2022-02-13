# Kotlin
## 2022-02-08 Tue

### Kotlin의 클래스와 인터페이스
* Kotlin의 클래스와 인터페이스는 Java의 개념과는 다른 부분이 있다.
  1. 인터페이스에 프로퍼티 선언을 포함할 수 있다.
  2. **Kotlin의 선언은 기본적으로 public final**이다.
  3. Kotlin의 중첩 클래스 구조는 기본적으로 inner class가 아니다.
  4. **Kotlin 컴파일러는 data 키워드를 통해 일부 표준 메소드를 자동으로 생성**해준다. 
  5. Kotlin 언어 자체적으로 지원되는 위임 기능을 통해 위임을 처리하기 위한 메소드를 직접 작성할 필요가 없다.

### 인터페이스
* Kotlin의 인터페이스에는 추상 메소드와 구현된 메소드가 포함될 수 있다.
  * 구현된 메소드는 Java 8의 디폴트 메소드와 유사하다.
* 인터페이스에는 상태를 나타내는 필드 값은 포함될 수 없다.
* Kotlin의 인터페이스 정의에는 interface라는 키워드를 사용한다.
* **Kotlin의 인터페이스를 구현하는 클래스는 클래스 이름 뒤에 콜론 기호를 붙인 후 인터페이스를 명시**한다.
  * **implements와 extends로 나뉘는 Java의 방식과 달리, Kotlin은 상속과 구현을 모두 콜론으로 명시**한다.
```
fun main(args: Array<String>) {
    val movable: Movable = MovableImpl()
    movable.move()
}

interface Movable {
    fun move()
}
class MovableImpl(): Movable {
    override fun move() = println("moving...!")
}
```
* Java와 마찬가지로, 인터페이스는 다중 상속을 지원하지만 부모 클래스는 하나만을 허용한다.
* 각 추상 메소드는 구현 클래스 내부에서 override 키워드를 사용하여 구현할 수 있다.
  * **Java의 @Override 어노테이션과 달리 override 키워드는 생략할 수 없다**.
  * 이러한 강제성으로 인해 부모 클래스의 메소드를 자식 클래스에서 실수로 재정의하는 것을 방지한다.
```
open class Parent() {
    fun sayHello() = println("Helloooo")
}
class Child(): Parent() {
    // 컴파일 불가
    // fun sayHello() = println("Child helloooo")
}
```
* 상술한 예시에서, 자식 클래스를 작성하던 개발자가 실수로 부모 클래스의 메소드와 같은 이름의 메소드를 작성할 수도 있다.
* 그러나 부모 클래스로부터 메소드를 재정의하기 위해서는 다음과 같은 전제 조건이 필요하다.
  1. 부모 클래스의 메소드가 open 상태여야 한다.
  2. 자식 클래스의 메소드에 override가 정의되어 있어야 한다.
* 두 전제 조건 중 하나라도 충족이 되지 않는 경우 컴파일이 불가하며, 이로 인해 실수로 인한 메소드 오버라이딩을 방지할 수 있다.
* 인터페이스에는 Java 8의 디폴트 메소드와 같이 구현된 메소드를 작성할 수 있다.
  * 그러나 Kotlin의 경우, default 키워드를 작성할 필요가 없다.
  * **인터페이스를 구현하는 클래스는 인터페이스에 구현된 메소드를 그대로 사용해도 좋고, 재정의하여 사용할 수도 있다**.
    * 이 경우, 인터페이스에는 open을 작성할 필요가 없으나 **구현 클래스에서는 override 키워드를 반드시 작성**해야 한다.
```
fun main(args: Array<String>) {
    val movable: Movable = MovableImpl()
    movable.move()
    movable.isMovable()
}

interface Movable {
    fun move()
    fun isMovable() = println("this is movable class")
}
class MovableImpl(): Movable {
    override fun move() = println("moving...!")
    // 아래와 같이 작성한 경우, override가 누락되었으므로 컴파일할 수 없다.
//    fun isMovable() = println("asdasd")
    // 아래의 내용처럼 작성할 경우, main의 호출은 MovableImpl 클래스의 내용을 따른다.
    // 재정의하지 않은 경우, main의 호출은 Movable 인터페이스의 내용을 따른다.
//    override fun isMovable() = println("asdasd")
}
```
* 하나의 구현 클래스가 둘 이상의 인터페이스를 구현하고, 각각의 인터페이스에 동명의 구현 메소드가 있는 경우는 다음과 같이 동작한다.
  1. 구현 클래스에서 메소드를 재정의한 경우, 구현 클래스의 내용을 따른다.
  2. **구현 클래스에서 메소드를 재정의하지 않은 경우, 컴파일이 불가능하다**.
  * 즉, Kotlin 컴파일러는 두 메소드를 아우르는 기능을 구현 클래스에서 정의하도록 강제한다.
* 둘 이상의 인터페이스 구현은 Java와 마찬가지로 쉼표를 사용한다.
```
fun main(args: Array<String>) {
    val movable: Movable = MovableImpl()
    movable.isMovable()
    movable.move()
}

interface Animal {
    // 같은 메소드 시그니쳐를 갖는 추상 메소드는 어차피 구현 클래스에서 구현되어야 하므로 인터페이스 다중 상속이 가능하다. 
    fun move()
    fun isMovable() = println("Animal can moving")
}
interface Movable {
    // 같은 메소드 시그니쳐이나 정상 동작한다.
    fun move()
    fun isMovable() = println("this is movable class")
}
// 다중 인터페이스 구현은 쉼표로 구분한다.
class MovableImpl(): Movable, Animal {
    override fun move() = println("moving...!")
    // 아래 내용이 없는 경우 컴파일이 불가능하다. 
    override fun isMovable() = println("I am movable animal")
}
```
* 만약 구현 클래스의 메소드에서 인터페이스의 구현 메소드를 호출하고자 하는 경우, 다음과 같이 작성한다.
  * Java의 경우, 인터페이스명.super.메소드명 형태로 호출했다.
  * **Kotlin의 경우, super<인터페이스명>.메소드명 형태로 작성**한다.
```
class MovableImpl(): Movable, Animal {
    override fun move() = println("moving...!")
    override fun isMovable() {
        super<Movable>.isMovable()
        super<Animal>.isMovable()
    }
}
```
* 내부적으로 Kotlin은 Java 6과 호환되도록 설계되었다.
* Java 6에는 인터페이스의 디폴트 메소드 개념이 없으므로, Kotlin은 이러한 인터페이스를 인터페이스와 정적 메소드가 포함된 클래스를 조합하여 표현한다.
  * 따라서 Java 코드는 Kotlin의 디폴트 메소드가 구현된 인터페이스에 의존할 수 없다.

### open, final, abstract 키워드
* Java의 클래스는 final 키워드를 명시적으로 작성하지 않는 이상 다른 클래스에 자유롭게 상속할 수 있다.
  * 이러한 구조는 편리한 대신 상속에서 오는 잠재적인 문제점을 가질 수 있다.
  1. 자식 클래스가 부모 클래스를 상속 받으면서 가졌던 가정(또는 믿음)이 부모 클래스의 변경에 의해 깨어질 수 있다. 
  2. 부모 클래스가 자신을 상속받으려면 어떠한 규칙을 따라야하는지 명시하지 않는다면, 자식 클래스는 부모의 멤버를 잘못된 방향으로 오버라이드할 수 있다. 
  3. 부모 클래스가 자식 클래스를 모두 파악하는 것은 사실상 불가능하며, 부모 클래스를 변경하는 시점에서 자식 클래스의 동작이 예기치 못하게 바뀔 수 있다.
* 상술한 점에서, Java의 자유로운 상속은 취약한 기반 클래스 문제에서 자유로울 수 없다.
```
> 상속을 위한 설계 및 문서를 갖출 수 없다면 상속을 금지하라
                        - Effective Java
```
* **이러한 조언은 자식이 오버라이드하도록 의도된 클래스와 메소드가 아니라면 모두 final로 작성하라는 의미**를 갖는다.
* **Kotlin은 이러한 조언을 기꺼이 따르기 위해 모든 클래스와 메소드를 기본적으로 final로 정의**한다.
  * 기본적으로 상속에 대해 열려 있는 Java와는 다른 점이 있다.
* 때문에 **Kotlin에서 클래스를 상속하려면 open 키워드를 사용하며 다음의 원칙을 따라야만 한다**.
  1. 상속을 허용하려는 클래스에 open 키워드를 정의한다.
  2. 상속을 허용하려는 클래스의 메소드, 프로퍼티 중 상속을 허용하려는 멤버 앞에 open 키워드를 정의한다.

### 열린 클래스와 열린 메소드
* 클래스와 메소드가 열려있다는 것을 나타내기 위해 open 키워드를 사용한다.
  1. open 클래스는 열려있으며, 다른 클래스가 상속이 가능하다.
  2. open 메소드는 열려있으며, 재정의가 가능하다.
     * 가능한 것이므로 재정의를 하지 않아도 무방하다.
  3. override 메소드는 열려있으며, 재정의가 가능하다.
     * 상속 계층을 따라 올라갔을 때, 최고 조상 클래스의 해당 메소드가 open 상태였을 것이기 때문이다.
     * 이를 자식 클래스가 재정의하지 못하게 하려면 final 키워드를 사용해야 한다.
  4. 2, 3에 해당하지 않는 메소드는 final이며, 오버라이드가 불가능하다.
     * 때문에 상속 받은 메소드를 그대로 사용한다.
```
open class GrandParent() {
    fun isGrandParent() = println("GrandParent")
    open fun sayHello() = println("Hello, GrandParent!")
}
open class Parent(): GrandParent() {
// 부모 클래스의 final 메소드는 재정의할 수 없으며, 부모 클래스로부터 상속받은 메소드를 그대로 사용한다.
//    override fun isGrandParent() = println("final cannot be overriden")
    
    fun isParent() = println("Parent")
    open fun isParentOfChild() = println("is parent of child?")
    override fun sayHello() = println("Hello, Parent!")
}
open class Child(): Parent() {
// 부모 클래스의 final 메소드는 재정의할 수 없으며, 부모 클래스로부터 상속받은 메소드를 그대로 사용한다.
//    override fun isGrandParent() = println("final cannot be overriden")
//    override fun isParent() = println("final cannot be overriden")
    
    // 부모 클래스의 open 메소드는 재정의할 수 있다.
    // 부모 클래스가 override한 메소드는 애초에 open이었을 것이므로 재정의할 수 있다.
    override fun isParentOfChild() = println("yes!")
    override fun sayHello() = println("Hello, Child!")
}
```
* 즉, **open 클래스는 상속 가능성을 의미하며 open 메소드는 재정의 가능성을 의미**한다.
  * open 메소드는 상속 가능성과 관계가 없다. 
  * 애초에 부모의 final 메소드도 자식이 재정의만 못할 뿐, 상속 받아 사용할 수 있다.
* 상술했듯 기본적으로 open 상태인 **override 메소드를 자식 클래스에서 재정의하지 못하도록 하려면 final 키워드를 사용**한다.
  * 따라서 final override의 final 키워드는 무의미한 중복 키워드가 아니다.
  * 이 경우, **자식 클래스가 상속 받은 메소드의 내용은 한 단계 위의 부모 메소드에 정의된 내용을 따른다**.
```
fun main(args: Array<String>) {
    val gp = GrandParent()
    val p = Parent()
    val c = Child()

    gp.sayHello() // Hello, GrandParent!
    p.sayHello() // Hello, Parent!
    c.sayHello() // Hello, Parent!
}
open class GrandParent() {
    open fun sayHello() = println("Hello, GrandParent!")
}
open class Parent(): GrandParent() {
// 자신은 부모로부터 메소드를 재정의하였으나, 자식 클래스에서 재정의할 수 없도록 final 키워드를 사용한다.
    final override fun sayHello() = println("Hello, Parent!")
}
open class Child(): Parent() {
// 부모 클래스의 final 메소드는 아래와 같은 형식으로 재정의할 수 없으며, 상속 받은 내용을 그대로 따라야 한다. 
//    override fun sayHello() = println("final method cannot be overriden")
}
```

### 추상 클래스
* Kotlin에서도 abstract 키워드를 활용하여 추상 클래스를 정의할 수 있다.
  * 일반적으로 **추상 클래스는 추상 멤버가 자식 클래스에서 구현되어야 하므로 open을 명시하지 않아도 open 클래스**이다.
* Java와 마찬가지로 추상 클래스는 인스턴스화할 수 없다.
* Kotlin의 추상 클래스 내부에 구현된 구현 메소드는 기본적으로 final이며, 필요시 open 키워드를 붙여 재정의가 가능하도록 할 수 있다.

```
fun main(args: Array<String>) {
    val instance: AbstractClass = Concrete()
    instance.sayHello()
    instance.sayBye()
    instance.greeting()
}

abstract class AbstractClass {
    fun sayHello() = println("hellooo")
    open fun sayBye() = println("byeeee")
    abstract fun greeting()
}
class Concrete(): AbstractClass() {
    //    override fun sayHello() = println("hello!")
    override fun sayBye() = println("bye!")
    override fun greeting() = println("greeting!")
}
```
* abstract 메소드는 기본적으로 반드시 오버라이드되어야 하나, 추상 클래스가 추상 클래스를 상속 받는 경우에는 구현되지 않을 수 있다.
  * **추상 클래스가 추상 클래스를 상속받는 경우, 자식 추상 클래스에는 abstract override로 정의된 메소드가 존재할 수 있다**.
  * 그러나 굳이 부모 추상 클래스의 메소드는 abstract override로 명시할 필요는 없다.
```
fun main(args: Array<String>) {
    val concrete: AbstTwo = Concrete()
    concrete.method()
    concrete.method2()
}

abstract class Abst {
    abstract fun method()
}
abstract class AbstTwo: Abst() {
// 이렇게 작성할 수도 있다.
    abstract override fun method()
    abstract fun method2()
}
class Concrete(): AbstTwo() {
    override fun method() { println("1") }
    override fun method2() { println("2") }
}
```

### 인터페이스와 Kotlin 변경자
* **인터페이스 멤버는 final, open, abstract를 사용하지 않는다**.
  1. 인터페이스의 추상 메소드는 반드시 재정의되어야 하므로 final을 사용하지 않는다.
  2. 인터페이스의 추상 메소드는 반드시 재정의되어야 하므로 open을 사용하지 않는다.
  3. 인터페이스에 정의된 메소드는 메소드 바디가 없는 경우 자동으로 추상 메소드로 취급되므로 abstract를 사용하지 않는다.
* 반면 **인터페이스가 인터페이스를 상속하는 경우, 하위 인터페이스가 상위 인터페이스의 메소드를 구현할 수 있으므로 override는 사용이 가능**하다.
```
fun main(args: Array<String>) {
    val owl: Animal = Owl()
    owl.move()
    owl.stop()
}

interface Animal {
    fun move()
    fun stop()
}
interface Bird: Animal {
// 하위 인터페이스는 상위 인터페이스인 Animal의 move를 재정의할 수 있다.
    override fun move() = println("I am coming!")
}
class Owl(): Bird {
    override fun stop() = println("stopping!")
}
```

### final, open, abstract, override 정리
1. final
   * 오버라이드할 수 없다.
   * 클래스에 기본으로 적용되는 키워드이다.
2. open
   * 오버라이드할 수 있다.
   * 반드시 open을 명시적으로 정의해야 오버라이드가 가능하다.
3. abstract
   * 반드시 오버라이드되어야 한다.
   * 추상 메소드는 추상 클래스에서 반드시 abstract를 명시한다.
   * 인터페이스의 메소드는 메소드 바디가 없는 경우 추상 메소드로 취급되므로, abstract를 반드시 명시할 필요는 없다.
4. override
   * 상위 클래스 또는 인터페이스의 멤버를 오버라이드하는 경우에 반드시 명시한다.
   * override된 멤버는 기본적으로 open이며, 재정의를 금지하려는 경우에는 final override로 명시한다.