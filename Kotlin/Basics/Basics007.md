# Kotlin
## 2022-02-09 Wed

### 가시성 변경자
* 가시성 변경자는 소스 코드에 정의된 선언에 대한 클래스 외부로부터의 접근을 제어한다.
  * 특정한 클래스의 구현에 대한 접근을 제한하는 것으로, 클래스에 의존하는 외부 코드를 깨지 않고도 클래스 내부의 구현을 변경할 수 있다.
* Kotlin의 가시성 변경자는 전체적으로 Java와 비슷하지만, 몇 가지 차이점이 있다.
  1. Kotlin의 기본 가시성 변경자는 public이다.
  2. **Kotlin은 default 가시성 변경자를 제공하지 않고, 대신 internal이라는 새로운 개념을 제공**한다.
      * Java의 defaul 가시성 변경자는 같은 패키지 내에 위치한 클래스들끼리만 접근할 수 있는 설정으로 제공된다.
      * **Kotlin은 패키지를 네임스페이스 관리 용도로만 사용하며, 따라서 패키지와 관련된 가시성 변경자를 제공하지 않는다**.
* Kotlin의 새로운 가시성 변경자인 internal은 Java의 default를 대체하기 위해 도입되었다.
  * internal 가시성 변경자는 선언을 모듈 내부에서만 공개하기 위해 사용한다.
  * **모듈은 한 번에 컴파일 되는 Kotlin 파일 단위이며, gradle의 프로젝트가 모듈이 될 수 있다**.
* 모듈 내부 가시성인 internal은 모듈 구현에 대한 진정한 캡슐화를 제공한다.
* Kotlin은 최상위에 선언된 클래스, 함수, 프로퍼티에 대한 private 가시성 변경자를 허용한다.
  * **private으로 정의된 최상위 선언은 해당 코드가 선언된 파일 내부에서만 사용할 수 있다**.
  * 따라서 하위 시스템의 자세한 구현을 외부에 감추고자 하는 경우에 유용하다.

### 가시성 변경자 정리
1. public
   * 기본 적용되는 가시성 변경자이다.
   * 클래스 멤버와 최상위 선언 모두 모든 곳에서 접근이 가능하다.
2. internal
   * 클래스 멤버 및 최상위 선언 모두 같은 모듈 내부에서만 접근이 가능하다.
3. protected
   * protected로 정의된 클래스 멤버는 하위 클래스 내부에서만 접근이 가능하다.
   * **해당 가시성 변경자는 최상위 선언에는 적용할 수 없다**.
4. private
   * 클래스 멤버 및 최상위 선언 모두 같은 소스 코드 파일 내부에서만 접근이 가능하다.
```
fun main(args: Array<String>) {
    val animal: Animal = Animal()
    // private, protected 메소드는 외부 호출이 불가능하다.
//    animal.move()
//    animal.stop()

    val bird: Bird = Bird()
    // public 메소드는 외부에서 호출이 가능하다.
    bird.greeting()
    // 부모의 확장 클래스는 호출이 가능하다.
    bird.run()

    // protected 메소드는 외부에서 호출이 불가능하다.
//    bird.stop()
    // 호출이 필요한 경우, 이를 대리 호출하는 public 메소드를 정의해야 한다.
    bird.temp()
}

private fun sayHello() = println("hello")

internal open class Animal() {
    private fun move() = println("moving...")
    protected fun stop() = println("stop...")

    // private 가시성 변경자로 최상위 선언된 메소드는 같은 소스 코드 내부에서는 접근이 가능하다.
    fun greeting() {
        sayHello()
    }
}
private fun Animal.run() {
    println("running!!!")
    // 확장 메소드는 private, protected 메소드에 접근할 수 없다.
//    move()
//    stop()
}
internal class Bird(): Animal() {
    fun temp() {
        // 부모의 private 메소드에는 접근이 불가능하다.
//        move()
        stop()
    }
}
```
* **더 제한적인 가시성 변경자로 정의된 클래스를 확장하는 함수는 더 허용적일 수 없다**.
  * 예를 들어, internal로 정의된 클래스에 public한 확장 함수를 정의할 수 없다.
  * internal로 정의된 클래스는 같은 모듈에서만 접근이 가능하며, 외부 모듈에서는 보여질 수 없다.
  * 그러나 public한 확장 함수를 정의할 경우, 수신 객체 타입을 외부에 공개시키는 상황이 발생할 수 있다.
    * 때문에 Kotlin 컴파일러는 이러한 상황을 경고하고, 컴파일을 수행하지 않는다.

### Kotlin - 가시성 변경자의 특징 II
* Java에서는 같은 패키지 안의 protected 멤버에 접근할 수 있지만, Kotlin은 오직 클래스 자신이나 상속받은 클래스에서만 보인다.
  * 즉, **Java와 Kotlin의 protected는 다른 개념**이다.
* Kotlin의 가시성 변경자는 정말 '가시성'을 나타내는 단위이다.
  * 때문에 protected 멤버는 클래스 본인이나, 상속 받은 클래스에서만 '보인다'.
  * private 멤버는 클래스 자신에게만 '보인다'.
* 또한 **클래스를 확장하는 확장 함수는 절대 클래스의 protected, private 멤버에 접근할 수 없다**.

### Java 바이트 코드로 컴파일된 경우의 Kotlin 가시성 변경자
* 기본적으로 public, private, protecte 변경자는 컴파일된 바이트 코드 내부에서도 유지된다.
* 예외적으로 Kotlin 코드 상에서 private로 정의된 클래스의 경우, default 가시성 변경자로 변경된다.
  * Java는 클래스를 private으로 정의할 수 없기 때문이다.
* **Kotlin 코드 상에서 internal로 정의된 가시성 변경자는 Java 바이트 코드에서는 public이 된다**. 
  * 때문에 Kotlin 코드에서는 접근 불가능한 경로로 Java 코드에서는 접근이 가능할 수 있다.
* Kotlin 컴파일러는 internal 가시성 변경자가 적용된 선언을 가독성이 떨어지는 이름으로 바꾼다.
  1. 이는 컴파일 후에 발생할 수 있는 우연한 메소드의 오버라이드를 막고,
  2. 실수로 internal 클래스를 외부 모듈에서 사용하는 것을 미연에 방지하기 위함이다.

### Java의 중첩 클래스
* Java에서는 클래스 내부에 다른 클래스를 선언할 수 있다.
  * 이러한 구조는 캡슐화에 유용하며, 특정한 클래스 코드를 해당 클래스를 사용하는 코드에 가까이 두고 싶을 떄 유용하다.
* Java의 경우, 다른 클래스 안에 정의한 클래스는 자동으로 내부 클래스가 된다.
  * 기본적으로, **Java의 내부 클래스는 외부 클래스에 대한 참조를 암시적으로 포함**한다.
  * 직렬화 등의 이유로 인해 암시적인 참조를 제거해야하는 경우, 내부 클래스를 static으로 정의하여 중첩 클래스로 만들어야 한다.
    * **static으로 선언된 중첩 클래스는 외부 클래스에 대한 암시적인 참조가 사라진다**.

### Kotlin의 중첩 클래스
* Kotlin의 중첩 클래스는 Java와 달리 기본적으로 **외부 클래스에 대한 접근이 불가능**하다.
* Kotlin에서 클래스 정의 내부에 포함된 클래스는 기본적으로 Java의 중첩 클래스와 대응된다.
* **내부에 포함된 클래스를 내부 클래스로 변경하여 외부 클래스에 대한 참조를 포함하도록 하고자 하는 경우, inner 키워드를 붙여주어야 한다**.
* Java와 딜리, **Kotlin은 내부 클래스가 외부 클래스에 접근하기 위해 this@[Outer_Class_Name] 키워드를 사용**해야 한다.
  * Inner 클래스에 동명의 프로퍼티가 없다면 this@키워드를 생략할 수 있다.
```
fun main(args: Array<String>) {
    val outer: Outer = Outer("ingnoh")
    val nested: Outer.Nested = outer.getNested()
    val inner: Outer.Inner = outer.getInner()

    nested.sayHello()
    inner.sayHello()
}

class Outer(val name: String) {

    fun getNested() = Nested("nested")
    fun getInner() = Inner("inner")

    class Nested(private val name: String) {
        fun sayHello() {
            // 중첩 클래스는 외부 프로퍼티에 접근할 수 없으므로, name과 this.name 모두 자신의 프로퍼티이다.
            println(name)
            println(this.name)
            // println(this@Outer.name)
        }
    }

    // 내부 클래스가 외부 클래스의 참조에 접근할 필요가 있다면 inner로 정의한다.
    inner class Inner(private val name: String) {
        fun sayHello() {
            // name, this.name은 Inner.name이다.
            println("hello, ${name}!")
            // Outer.name을 참조하려면 this@Outer.name 키워드를 사용한다.
            // 만약 Inner에 name이라는 프로퍼티가 없으면, this@Outer를 생략할 수 있다.
            println("hello, ${this.name}!")
            println("hello, ${this@Outer.name}!")
        }
    }
}
```

### 중첩 클래스와 내부 클래스
1. 중첩 클래스(Nested class): 외부 클래스에 대한 암시적인 참조를 저장히지 않는다.
   * Java의 경우, 클래스 내부에 정의된 static class이다.
   * Kotlin의 경우, 클래스 내부에 정의된 class이다.
2. 내부 클래스(Inner class): 외부 클래스에 대한 암시적인 참조를 저장한다.
   * Java의 경우, 클래스 내부에 정의된 class이다.
   * Kotlin의 경우, 클래스 내부에 정의된 inner class이다.

### 봉인된 클래스
* **when 식에서 부모 클래스를 자식 클래스로 스마트 캐스트하는 과정은 반드시 else 분기가 강제된다**.
  * 일반적으로 상술한 용도에서는 else 분기는 유효하지 않은 처리를 하므로 예외를 던지도록 한다.
  * 만약 부모 클래스에 새로운 자식 클래스가 추가되었는데, 이를 when 분기에 추가하는 것일 잊었다면 시스템의 에러로 이어질 수 있다.
```
fun main(args: Array<String>) {
    checkChild(FirstBone())
    checkChild(SecondBone())
    checkChild(ThirdBone())
}

open class Parent()
class FirstBone(): Parent()
class SecondBone(): Parent()
class ThirdBone(): Parent()

fun checkChild(parent: Parent) =
    when(parent) {
        is FirstBone -> println("First!")
        is SecondBone -> println("Second!")
        else -> {
            println("...") 
            // 만약 아래처럼 구현되었다면 이 시스템은 컴파일은 되지만 예외가 발생하여 종료될 것이다.
            // throw IllegalArgumentException("...")
        }
    }
```
* 즉, 상술한 방식은 다음의 두 가지 문제가 있다.
  1. else 분기의 처리가 강제된다.
  2. 자식 클래스가 늘어날 때 시스템이 에러가 발생하지 않으리라고 보장할 수 없다.
* Kotlin의 sealed 클래스는 이에 대한 해법을 제공한다.
  * **sealed 키워드가 붙은 부모 클래스는 상속될 자식 클래스를 제한할 수 있다**.
  * sealed 클래스의 자식 클래스는 반드시 sealed 클래스 내부에 정의되어야 한다.
    * 자식 클래스를 같은 소스 코드의 외부에 정의할 수는 있으나, 이는 sealed 클래스의 용도에 맞지 않으므로 지양한다.
  * sealed 클래스는 기본적으로 open 되어 있으므로, open 키워드를 생략할 수 있다.
* **sealed 클래스를 사용하는 when식이 자식 클래스의 처리를 잊었다면 Kotlin 컴파일러는 컴파일하지 않는다**.
  * 다음과 같은 오류 메시지를 통해 해당 자식을 처리하는 분기 또는 else와 같은 추가 분기를 요구한다.
  * 따라서 자식 클래스가 추가되더라도 처리를 잊어 버그가 발생하는 상황은 미연에 방지된다.
* sealed 클래스의 생성자는 외부에서 호출이 불가하며, 클래스 내부에서만 호출이 가능하다.
```
Kotlin: 'when' expression must be exhaustive, add necessary 'is ThirdBone' branch or 'else' branch instead
```
* sealed 클래스는 다음과 같이 작성한다.
```
fun main(args: Array<String>) {
    checkChild(Parent.FirstBone())
    checkChild(Parent.SecondBone())
    checkChild(Parent.ThirdBone())
    checkChild(Who())
    // sealed 클래스의 생성자는 기본적으로 외부에서 직접 호출될 수 없다.
    // checkChild(Parent())
}

sealed class Parent {
    class FirstBone(): Parent()
    class SecondBone(): Parent()
    class ThirdBone(): Parent()
}
// sealed 클래스 외부에 자식을 둘 수는 있다.
class Who(): Parent()

fun checkChild(parent: Parent) =
    when(parent) {
        is Parent.FirstBone -> println("First!")
        is Parent.SecondBone -> println("Second!")
        is Parent.ThirdBone -> println("Third!")
        // sealed 외부에 정의된 자식 클래스도 분기 처리가 강제된다.
        is Who -> println("who")
        // 더 이상 else 분기가 필요하지 않다.
    }
```
* **sealed 키워드는 인터페이스에도 적용할 수 있다**.
```
sealed interface Hello {
    fun hello()
    class HelloImpl(): Hello {
        // 인터페이스의 메소드는 반드시 구현되어야 한다.
        override fun hello() {
            TODO("Not yet implemented")
        }

    }
}
```