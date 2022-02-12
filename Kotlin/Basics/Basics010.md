# Kotlin
## 2022-02-11 Fri

## 람다식
* 람다식, 람다는 다른 함수에 넘길 수 있는 작은 코드 조각을 뜻한다.
* 람다를 적절히 사용하면 공통 코드 구조를 쉽게 라이브러리 함수로 빼낼 수 있으며, Kotlin 역시 표준 라이브러리에서 람다식을 많이 사용한다.
* 데이터 구조의 모든 원소에 대해 임의의 연산을 적용할 수 있도록, 일련의 동작을 변수에 저장하거나 다른 메소드에 넘겨야하는 경우가 있다.
  * 예전의 Java에서는 이를 무명 내부 클래스로 구현하였으나, 이는 번거로운 방식이다.
* 함수형 프로그래밍에서는 함수를 값처럼 다루는 접근 방법을 택함으로써 이를 구현한다.
  * 클래스를 선언하고 인스턴스를 메소드에 넘기는 방식보다 람다식을 활용하는 것이 코드가 더 간결해진다.
  * **람다 식을 활용하면 메소드를 별도로 선언할 필요도 없으며, 코드 블록을 직접 함수의 인자로 전달**할 수 있다.

### 람다와 컬렉션
* 코드에서 중복을 제거하는 것은 좋은 코드를 위한 중요한 방법이다.
* 이러한 전제와 더불어, **컬렉션에 수행하는 대부분의 작업은 몇 가지의 패턴으로 나눌 수 있다**.
  * 따라서, 이러한 패턴은 라이브러리화해야 한다.
* 람다식이 없다면 이렇듯 컬렉션을 편리하게 처리할 수 있을만한 좋은 라이브러리를 제공하기가 어렵다.
  * 실제로 Java 8 이전에는 편한 컬렉션 라이브러리가 적었고, 개발자들은 필요한 기능을 직접 구현해야 했다.
  * Kotlin에서는 이러지 말아야 한다!
* 예를 들어, 나이를 프로퍼티로 갖는 동물 목록에서 가장 나이가 많은 동물을 찾는 코드를 직접 구현하면 다음과 같다.
```
fun main(args: Array<String>) {
     val oldest = getOldest(listOf(
          Animal("Animal", 5),
          Animal("Bnimal", 15),
          Animal("Cnimal", 3),
          Animal("Dnimal", 20),
          Animal("Enimal", 12),
          Animal("Fnimal", 7),
          Animal("Gnimal", 9),
     ))
     println(oldest)
}
data class Animal(val name: String, val age: Int)
fun getOldest(animals: List<Animal>): Animal? {
     var age: Int = 0
     var oldest: Animal? = null
     for(animal in animals) {
          if(animal.age > age) {
               oldest = animal
               age = animal.age
          }
     }
     return oldest
}
```
* 간단한 로직을 구현하면서도 코드가 길어지므로, 휴먼 에러가 발생할 가능성이 높다.
* Kotlin에서는 람다식을 활용하여 다음과 같이 수정할 수 있다.
  * **메소드 정의조차 필요 없어지며, 코드 길이의 대부분은 목록화에 사용된다**.
```
fun main(args: Array<String>) {
     val animals = listOf(
          Animal("Animal", 5),
          Animal("Bnimal", 15),
          Animal("Cnimal", 3),
          Animal("Dnimal", 20),
          Animal("Enimal", 12),
          Animal("Fnimal", 7),
          Animal("Gnimal", 9),
     )
     // 메소드 정의도 필요 없다!
     println(animals.maxByOrNull { it.age })
}
data class Animal(val name: String, val age: Int)
```
* 상술한 예시의 maxBy 함수는 모든 컬렉션에 대해 호출할 수 있다.
* maxBy는 가장 큰 원소를 찾기 위해 **비교에 사용할 값을 돌려주는 함수를 인자로** 받는다.
  * 이 때, **중괄호로 둘러쌓인 { it.age }가 비교에 사용할 값을 돌려주는 함수 조각**이다.
  * **해당 코드는 컬렉션의 원소를 it이라는 인자로 받아, 비교에 사용할 값을 반환**한다.
* 상술한 예시와 같이 함수나 프로퍼티를 반환하는 역할만을 수행하는 람다는 멤버 참조로 대체할 수 있다.
```
// 멤버 참조를 활용한 예시이다.
println(animals.maxByOrNull(Animal::age))
```
* **컬렉션에 대해 수행하던 대부분의 작업은 람다 또는 멤버 참조를 인자로 취하는 라이브러리 함수로 개선할 수 있다**.

### 람다식 문법
* 상술했듯, **람다는 마치 하나의 값처럼 여기저기에 전달할 수 있는 일련의 동작**이다.
  * 때문에 람다는 별도로 선언하여 변수로 관리할 수도 있다.
  * 하지만 **대부분의 경우 함수에 인자로 넘기는 시점에 람다를 정의**한다.
* 람다식 선언의 일반적인 문법은 다음과 같다.
  * Kotlin 람다식은 중괄호로 감싼다.
  * **Kotlin 람다식은 파라미터 목록 주변에 괄호를 작성하지 않는다**.
  * -> 기호는 파라미터 목록과 람다식 본문을 구분한다.
```
fun main(args: Array<String>) {
     // { 파라미터 목록 -> 람다식 본문 }
     val lambda = { num1: Int, num2: Int -> num1 + num2 }
     // 변수에 저장한 람다식을 메소드처럼 바로 호출할 수 있다.
     println(lambda(32, 1))
}
```
* 람다식은 즉시 실행 함수 표현식을 지원한다.
  * 형태는 람다식에 뒤이어 괄호를 통해 파라미터 목록을 전달한다.
  * { 파라미터 목록 -> 람다식 본문 }(전달 파라미터 목록)
```
fun main(args: Array<String>) {
     println(
          { num1: Int, num2: Int -> num1 * num2}(3, 4)
     )
}
```
* 그러나 **이러한 방식은 가독성이 떨어지고, 쓸모도 없다**.
  * 복잡하게 람다를 호출하느니, 람다식 본문을 바로 사용하는 것이 낫다.
  * 실제로 IDE 자체가 이렇듯 불필요한 람다식의 사용을 바로잡아주려고 한다.
* 부득이하게 코드 블록을 람다식으로 호출할 필요가 있다면, 인자로 받은 람다식을 실행해주는 run 메소드를 사용한다.
```
fun main(args: Array<String>) {
     run { println("lambda!") }
}
```
* **런타임에서 Kotlin의 람다 호출은 아무런 부가 비용이 들지 않고, 프로그램의 기본 구성 요소와 비슷한 성능을 갖는다**.

### 람다식 문법 II - maxByOrNull 예제
* 앞선 maxBy 예제에서, maxByOrNull { it.age }는 실제로는 다음과 같이 작성해야 한다.
```
animals.maxByOrNull({ animal: Animal -> animal.age })
```
* 위의 식은 다음과 같은 원칙에 의해 가독성을 향상시킨다.
1. Kotlin 컴파일러가 문맥으로부터 인자의 타입을 유추할 수 있다면, 인자 타입을 생략할 수 있다.
   * **로컬 변수가 그랬듯, Kotlin 컴파일러는 람다식의 인자 타입도 추론할 수 있다**.
     * 예를 들어, maxByOrNull의 경우 인자 타입은 항상 컬렉션의 원소 타입과 같다.
   * Kotlin 컴파일러는 Animal 객체에 대해 maxByOrNull을 호출하므로, 인자 타입 역시 Animal일 것이라고 추론할 수 있다.
   * 일단 인자 타입을 생략해보고, 컴파일러가 추론하지 못하는 경우에 인자 타입을 붙이는 식으로 개발하도록 한다.
   * **람다식의 인자가 여럿인 경우, 인자 타입의 일부만 명시하고 추론 가능한 인자 타입은 생략해도 좋다**.
     * 언제나 가독성이 좋은 방향을 따르도록 한다.
```
animals.maxByOrNull({ animal -> animal.age })
```
2. **Kotlin에서는 메소드 호출 시 맨 마지막 인자가 람다식이라면, 람다식을 괄호 밖으로 뺄 수 있다는 문법**이 있다.
   * 람다를 괄호 밖으로 빼내어 람다식임을 명시할 수도 있고, 괄호 안에 두어 메소드의 인자임을 명시해도 좋다.
   * 그러나 인자가 여럿인 경우 람다식만 밖으로 빼내면 람다식의 존재 의의를 바로 알아보기 어렵다.
   * 때문에 일반적인 경우, 괄호 내부에 람다를 두어 사용하는 일반적인 호출 구문이 바람직하다.
```
animals.maxByOrNull(){ animal -> animal.age }
```
3. 람다가 어떤 함수의 유일한 인자이고, 괄호 뒤에 람다를 빼낸 경우에는 괄호가 비어 있게 되므로 이를 생략할 수 있다.
```
animals.maxByOrNull { animal -> animal.age }
```
4. **람다의 인자 명칭의 디폴트 값은 it이며, 람다의 인자 갯수가 하나이면서 컴파일러에 의해 추론될 수 있다면 it을 바로 사용할 수 있다**.
   * 람다식의 인자명을 따로 지정하지 않은 경우에만 it 이라는 이름이 자동으로 생성된다.
   * **it을 사용하는 관습은 코드를 간결하게 만들어주지만, 남용하지 않아야 한다**.
     * 예를 들어 람다식이 중첩되는 경우, 모두 it을 사용하면 각각의 it이 가리키는 대상을 유추하기 어려워진다.
     * **가독성 측면에서 도움이 되는 경우, 인자 타입 또는 인자명을 명시하도록 한다**.
```
animals.maxByOrNull { it.age }
```
* **람다식을 변수에 저장하는 경우, 인자 타입을 추론할만한 문맥이 존재하지 않으므로 인자 타입을 생략할 수 없다**.
* 람다식은 반드시 한 줄로 작성되리라는 법은 없으며, 여러 줄일 수 있다.
  * 이 때, 람다식의 결과는 다른 블록 식과 마찬가지로 마지막 식이 된다.
```
fun main(args: Array<String>) {
     val lambda = { num1: Int, num2: Int ->
          println("num1: $num1")
          println("num2: $num2")
          num1 + num2
     }
     println(lambda(4, 3))
}
```

## 2022-02-12 Sat
### 람다와 포획된 변수
* Java의 경우, 메소드 안에 정의된 무명 내부 클래스는 메소드의 로컬 변수에 접근할 수 있다.
* Kotlin의 경우, 메소드 안에 저으이된 람다는 메소드의 파라미터와 로컬 변수에 접근하여 읽거나 쓸 수 있다.
  * 또한 Kotlin에서는 Java와 달리 final이 아닌 변수도 읽고 쓸 수 있다.
* 이렇듯 **람다 내부에서 사용하는 외부 메소드의 변수를 람다가 포획한 변수라고 한다**.
  * 아래의 예시에서, 람다는 메소드의 변수인 suffix, prefix, length에 접근하고 있다.
```
fun main(args: Array<String>) {
     val animals = listOf(
          Animal("Animal", 5),
          Animal("Bnimal", 15),
          Animal("Cnimal", 3),
          Animal("Dnimal", 20),
          Animal("Enimal", 12),
          Animal("Fnimal", 7),
          Animal("Gnimal", 9),
     )

     describe(animals, ", and")
}
data class Animal(val name: String, val age: Int)

fun describe(animals: List<Animal>, suffix: String) {
     var length: Int = 0
     val prefix: String = "next element is: "
     // prefix, suffix에 접근하고 있다.
     animals.forEach {
          // 메소드의 변수를 쓸 수도 있다.
          length++
          println("$prefix$it$suffix")
     }
     println("length: $length")
}
```
* 참고로, **forEach는 간결함 외에는 for문보다 나은 것이 없으므로 항상 forEach만을 사용할 필요는 없다**.
* 기본적으로 함수 내부에서 정의된 로컬 변수의 생명 주기는 함수가 반환되는 시점에 함께 종료된다.
  * 그러나 **어떤 함수가 로컬 변수를 포획한 람다를 반환하거나, 람다를 다른 변수에 저장하는 경우 로컬 변수와 함수의 생명 주기는 달라질 수 있다**.
* 아래의 예시에서, **getLambda는 람다를 반환하고 종료되지만 반환된 람다식은 name과 plus 변수의 값을 포획한 상태를 유지**한다.
  * 변수의 final 여부에 따라, 변수 포획은 내부적으로 다음과 같이 이루어진다.
  1. final 변수: 변수의 값과 람다 코드를 함께 저장한다.
  2. final이 아닌 변수: 변수를 읽고 변경할 수 있는 특별한 래퍼 클래스로 감싼 후, 래퍼의 참조를 람다 코드와 함께 저장한다. 
```
fun main(args: Array<String>) {
     val lambda = getLambda("ingnoh", 10)
     println(lambda(3))
     println(lambda(4))
     println(lambda(2))
}

fun getLambda(name: String, plus: Int) = { number: Int ->
          println("name: $name")
          number + plus
     }
```

### 멤버 참조
* forEach와 같은 예시에서, 람다를 활용해서 코드 블록 자체를 다른 함수에 인자로 넘기도록 했다.
  * 그러나 넘기려는 코드 블록이 이미 함수로 정의되어 있다면, 람다 내부에서 함수를 호출하는 것은 코드 중복이 된다.
* **이러한 중복 문제를 해결하고자 함수를 직접 넘기려는 경우에 멤버 참조를 사용할 수 있다**.
* **Kotlin에서는 Java 8의 경우와 마찬가지로 멤버 참조를 활용하여 함수를 값으로 바꿀 수 있으며, 이중 콜론을 사용**한다. 
  * 자세한 형식은 [클래스명]::[멤버명]이 된다.
  * **매 번 인스턴스를 참조하는 변수를 활용하지 않고, 클래스명과 멤버 명을 활용할 수 있으므로 편리**하다.
  * **클래스에 정의된 확장 함수 역시 멤버 참조 구문을 활용할 수 있다**.
```
// 이름을 얻는 동작을 getName 변수에 저장한다.
val getAge = Animal::age
// 람다식으로 정의된 동작을 마치 변수처럼 전달한다.
val oldest = animals.maxByOrNull(getAge)
println(oldest)
```
* **멤버 참조는 프로퍼티나 메소드를 단 하나만 호출하는 함수 값을 만드는 역할**을 한다.
  * 예를 들어, 상술한 Animal::name은 { animal: Animal -> animal.name }을 간결화한 것이다.
* 이를 이용하여 최상위 함수를 호출할 수도 있다.
  * 아래 예시에서, run은 인자로 받은 람다식을 실행하는 역할을 수행한다.
```
// 최상위 함수 정의
fun helloWorld() = println("Hello world!")
fun main(args: Array<String>) {
     // 최상위 함수 호출
     run(::helloWorld)
}
```
* **멤버 참조를 활용하면 생성자를 참조하여 클래스 생성 작업을 연기하거나 저장해둘 수 있다**.
  * 아래 예시에서, getAnimalInstance 변수에 Animal 인스턴스를 만드는 동작을 값으로써 저장한다.
```
fun main(args: Array<String>) {
     // Animal 인스턴스를 만드는 동작을 저장한다.
     val getAnimalInstance = ::Animal
     // Animal 인스턴스를 만들고자 하는 경우에 호출한다.
     println(getAnimalInstance("Hello", 0))
     println(getAnimalInstance("World", 1))
}
data class Animal(val name: String, val age: Int)
```
* 람다가 여러 파라미터를 받는 다른 함수를 호출하기만 하는 역할을 수행하는 경우, 람다를 별도로 정의하지 않고 위임 함수에 대한 참조를 제공할 수 있다.
  * 이 경우, 람다는 다른 함수에 작업을 위임하는 역할만을 수행하므로 굳이 함수 호출만을 위한 람다식을 작성하지 않아도 된다.
```
fun main(args: Array<String>) {
     // describe 메소드에 작업을 위임하는 람다식의 예시이다.
     val redundant = { animal: Animal, suffix: String ->
          describe(animal, suffix)
     }
     // 람다식을 작성하는 대신 멤버 참조를 활용한다.
     val better = ::describe
}
data class Animal(val name: String, val age: Int)

fun describe(animal: Animal, suffix: String) {
     println("$animal $suffix")
}
```

### 컬렉션의 함수형 API
* 컬렉션을 다루는 과정에서 함수형 프로그래밍 스타일을 적용하면 코드를 간결하게 만들며, 편리할 수 있다.
* 예를 들어 filter와 map은 컬렉션 활용의 기반이 되는 함수이며, 대부분의 컬렉션 연산은 이 두 함수로 표현이 가능하다.

### filter 함수
* filter는 함수를 걸러내는 역할을 하며, 컬렉션을 순회하며 주어진 조건에 맞는 원소만을 모은다.
* **filter 연산의 결과는 입력 컬렉션 원소 중 조건을 만족하는 원소만으로 구성된 새로운 컬렉션**이다.
* filter는 컬렉션에서 필요하지 않은 원소는 제거할 수 있지만, 원소의 값을 변환할 수는 없다.
```
val animals = listOf(
     Animal("Animal", 5),
     Animal("Bnimal", 15),
     Animal("Cnimal", 3),
     Animal("Dnimal", 20),
     Animal("Enimal", 12),
     Animal("Fnimal", 0),
     Animal("Gnimal", 2),
     Animal("Hnimal", 13),
     Animal("Inimal", 29),
     Animal("Jnimal", 6),
     Animal("Knimal", 19),
)

fun main(args: Array<String>) {
     println(animals.size)
     val filtered = animals.filter { it.age < 10 }
     println(filtered.size)
     val notFiltered = animals.filter { it.age >= 10 }
     println(notFiltered.size)
}
data class Animal(val name: String, val age: Int)
```

### map 함수
* map 함수는 요소를 변환하는 역할을 하며, 컬렉션을 순회하며 주어진 동작을 적용하여 새로운 원소를 만든다.
* **map 함수의 결과는 변경된 요소들을 포함하는 새로운 컬렉션**이다.
  * 따라서 결과 컬렉션의 요소 개수는 원본의 요소 개수와 같지만, 내용이 다르다.
```
fun main(args: Array<String>) {
     val mapped = animals.map { it.name }
     // 멤버 참조를 활용해도 결과는 같다.
     // val mapped = animals.map(Animal::name)
     println(mapped)
}
```

### filter와 map의 메소드 체이닝
* filter 연산과 map 연산의 결과는 새로운 컬렉션이므로, 이어서 연산을 수행할 수 있다.
```
fun main(args: Array<String>) {
     val result = animals
          .filter { it.age < 10 }
          .map { it.name }
     println(result)
}
```
* 이러한 연쇄는 가독성이 좋지만, 잘 못 작성된 경우 내부적인 로직은 매우 비효율적일 수 있다.
* 예를 들어, 아래의 filter는 매 순회마다 maxByOrNull을 호출한다.
  * 미리 최대값을 변수에 저장하고, 변수를 활용하는 연산으로 수정하는 것이 좋다.
  * 이렇듯 **map과 filter 함수를 적용한 경우 내부적으로 어떻게 처리되는지를 이해하는 것은 중요**하다.
```
fun main(args: Array<String>) {
     val result = animals
          .filter { it.age == animals.maxByOrNull(Animal::age)!!.age }
     
     // 아래처럼 수정하는 것이 더 효율적이다.     
     // val max = animals.maxByOrNull(Animal::age)!!.age
     // val result = animals
     //      .filter { it.age == max }     
          
     println(result)
     
}
```

### Map에 filter와 map 함수 적용
* filter와 map 메소드는 Map에도 적용이 가능하며, 호출해야 하는 메소드의 이름이 다음과 같이 키와 값 각각에 대해 존재한다.
1. filter
   * filterKeys: Map의 각 요소를 순회하며 키를 기준으로 걸러낸다.
   * filterValues: Map의 각 요소를 순회하며 값을 기준으로 걸러낸다.
2. map
   * mapKeys: Map의 각 요소를 순회하며 키를 변환한다.
   * mapValues: 각 요소를 순회하며 값을 변환한다.
```
fun main(args: Array<String>) {
     val map = mapOf(1 to "one", 2 to "two", 3 to "three")
     println(map.mapValues { it.value.uppercase(Locale.getDefault()) })
}
```

### all, any, count, find
* all과 any는 컬렉션의 모든 요소가 임의의 조건을 만족하는지 확인하는 함수이다.
  * all: 모든 요소가 조건을 만족하는 경우 true를 반환한다.
  * any: 조건을 만족하는 요소가 하나라도 있는 경우 true를 반환한다.
* count는 조건을 만족하는 요소의 개수를 반환하는 함수이다.
* find는 컬렉션 요소 중 조건을 만족하는 첫 요소를 반환하는 함수이다.
  * 조건을 만족하는 요소가 없다면 null이 반환된다.
```
fun main(args: Array<String>) {
     val lessThan20 = { animal: Animal -> animal.age < 20 }
     val all = animals.all(lessThan20)
     val any = animals.any(lessThan20)
     val count = animals.count(lessThan20)
     val find = animals.find(lessThan20)
     println(all)
     println(any)
     println(count)
     println(find)
}
```
* all과 any의 경우, boolean이므로 !를 붙이는 경우가 있다.
  * 만약 !를 붙이는 경우 가독성이 떨어질 수 있을 것 같다면, all과 any를 서로 바꾸어 사용하거나 조건의 부정을 활용하는 것이 좋다.
  * 이 때, 다음과 같은 드 모르간의 법칙을 적용해볼 수 있다.
    1. **어떤 조건의 !all은 어떤 조건의 부정의 any와 같다**. 
    2. **어떤 조건의 !any는 어떤 조건의 부정의 all과 같다**.
* **대부분의 경우, 가독성 측면에서는 !all또는 !any를 사용하지 않는 것이 좋다((.
```
fun main(args: Array<String>) {
     val lessThan20 = { animal: Animal -> animal.age < 20 }
     val all = animals.all(lessThan20)
     // 아래와 같은 식의 사용은 가독성이 떨어진다.
     val notAll = !animals.all(lessThan20)
     println(all)
     println(notAll)
     
     val greaterThan20 = { animal: Animal -> animal.age >= 20 }
     // 드 모르간의 법칙을 적용하여 가독성을 높인다.
     val likeThis = animals.any(greaterThan20)
     println(likeThis)
}
```
* 아래의 식에서, count와 컬렉션의 size 결과는 같다.
  * 그러나 중간 처리 과정은 다르며, filter를 적용하는 경우 불필요한 중간 연산에 의해 조건을 만족하는 컬렉션이 생성된다.
  * **count 함수는 중간 컬렉션을 생성하지 않고 개수만을 추적하므로, 이 경우에는 count 함수를 사용하는 것이 훨씬 더 바람직**하다.
```
fun main(args: Array<String>) {
     // filter 함수에 의해 불필요한 중간 연산 결과가 생성된다.
     println(animals.filter { it.age < 10 }.size)
     // 중간 컬렉션을 생성하지 않고 개수를 추적한다.
     println(animals.count { it.age < 10 })
}
```