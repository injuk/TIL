# DesignPattern
## 2022-02-10 Thu

## Decorator Pattern
### 기본 개념
* 데코레이터 패턴은 특정 객체를 꾸며주는 역할을 한다.
* 데코레이터 패턴은 기존 클래스의 기능에 더해 개발자가 원하는 기능을 덧붙일 때 유용하다.
* 만약 클래스 A의 기능에 데코레이터 패턴을 활용하여 기능을 추가한다면, 클래스 A와 공통의 인터페이스를 갖도록 한다.
  * A가 구현하는 인터페이스가 있다면 이를 구현하고, 없다면 공통 부분을 갖도록 인터페이스를 정의한다.
* 이렇게 공통의 인터페이스를 갖도록 정의된 데코레이터 클래스는 내부적으로 클래스 A 인스턴스를 멤버로 갖는다. 
* 만약 여러 다른 기능을 추가하는 경우, 데코레이터 클래스 하나에 모두 정의할 수도 있지만 다시 자식 클래스를 만드는 방법도 고려할 수 있다.
  * 즉, **클래스 A와 공통의 인터페이스를 갖는 데코레이터 클래스 B의 자식 클래스 C, D를 정의해볼 수 있다**.
  * 이렇게 정의된 자식 데코레이터 클래스들은 클래스 A와 공통의 인터페이스를 가지면서도 기능을 여러 방향으로 정의할 수 있다. 
* **데코레이터 패턴은 상속 없이 기존 오브젝트에 더 많은 기능을 추가하기 위해 꾸며주기 위해 사용하는 패턴**이다.

### 시나리오
* 예를 들어, 상속이 허용되지 않는 클래스에 새로운 동작을 추가하고 싶은 경우에 데코레이터 패턴을 고려할 수 있다.
* 데코레이터 패턴의 핵심은 다음과 같은 절차로 작성된다.
  1. 기존 클래스 대신에 새로운 기능을 정의할 클래스를 만들되,
  2. 기존 클래스와 같은 인터페이스를 새로운 클래스가 제공해야 하고,
  3. **기존 클래스는 새로운 클래스 내부에 멤버로서 유지**되어야 한다.
  4. 새로 정의해야 하는 기능을 새로운 클래스에 정의하되,
  5. **기존 기능이 그대로 필요한 경우에는 새로운 클래스의 메소드에서 기존 클래스의 메소드에 요청을 포워딩**한다.

### 코드 예시
* Kotlin을 배우고 있으므로 Kotlin으로 작성하였다.
* 데코레이터 클래스를 상속받는 클래스는 기존 클래스를 감싸 추가적인 동작을 구현한다.
```
fun main(args: Array<String>) {
     val bird: Animal = Bird()
     val cat: Animal = Cat()
     makeMove(bird)
     makeMove(cat)

     val questioningBird: Animal = QuestionMarkDecorator(bird)
     val questioningCat: Animal = QuestionMarkDecorator(cat)
     makeMove(questioningBird)
     makeMove(questioningCat)

     val questioningWithSmileBird: Animal = SmileDecorator(questioningBird)
     val questioningWithSmileCat: Animal = SmileDecorator(questioningCat)
     makeMove(questioningWithSmileBird)
     makeMove(questioningWithSmileCat)

     val complicatedBird: Animal = QuestionMarkDecorator(SmileDecorator(QuestionMarkDecorator(SmileDecorator(questioningWithSmileBird))))
     val complicatedCat: Animal = QuestionMarkDecorator(SmileDecorator(QuestionMarkDecorator(SmileDecorator(questioningWithSmileCat))))
     makeMove(complicatedBird)
     makeMove(complicatedCat)
}
fun makeMove(animal: Animal) = println(animal.move())
interface Animal {
     fun move(): String
}
class Bird: Animal {
     override fun move() = "bird flying"
}
class Cat: Animal {
     override fun move() = "cat moving"
}
// Kotlin의 by 키워드를 활용한 위임 기능을 통해 아래와 같이 작성할 수도 있다.
// open class Decorator(val animal: Animal): Animal by animal {}
open class Decorator(val animal: Animal): Animal {
     override fun move() = animal.move()
}
class QuestionMarkDecorator(animal: Animal): Decorator(animal) {
     override fun move() = super.animal.move() + " ?"
}
class SmileDecorator(animal: Animal): Decorator(animal) {
     override fun move() = super.animal.move() + " :D"
}
```

### 실행 결과
```
bird flying
cat moving
bird flying ?
cat moving ?
bird flying ? :D
cat moving ? :D
bird flying ? :D :D ? :D ?
cat moving ? :D :D ? :D ?
```