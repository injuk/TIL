# Kotlin
## 2022-02-13 Sun

## lazy 컬렉션 연산 (지연 계산)
* map, filter와 같은 컬렉션 함수는 결과 컬렉션을 즉시 생성한다.
* **따라서 컬렉션 함수를 여럿 연쇄하면 매 단계마다 계산의 중간 결과를 새로운 컬렉션에 중간 저장하게 된다**.
* 이러한 중간 저장은 리스트의 요소가 적은 경우 큰 문제가 되지 않지만, **요소의 개수가 많아질수록 성능에 영향을 준다**.
```
fun main(args: Array<String>) {
     val upperCaseNames = animals
          .filter { it.age < 10 } // 새로운 중간 컬렉션이 생성된다.
          .map { it.name.uppercase() } // 새로운 중간 컬렉션이 생성된다.
     println(upperCaseNames)
}
```
* 컬렉션 연산을 여럿 사용해야 하는 경우, 시퀀스를 사용할 수 있다.
  * **상술한 연산이 컬렉션을 직접 사용하는 대신 시퀀스를 사용하면 더 효율적인 구현이 가능**하다.
  * **리스트를 시퀀스로 변경하고, 결과를 다시 리스트화하기 위해 asSequence()와 toList()를 활용**한다.
  * 시퀀스 역시 리스트와 같이 filter, map 함수를 사용할 수 있다.
```
fun main(args: Array<String>) {
     val upperCaseNames = animals
          .asSequence() // 리스트를 시퀀스로 변경한다.
          .filter { it.age < 10 } // 새로운 중간 컬렉션이 생성되지 않는다.
          .map { it.name.uppercase() } // 새로운 중간 컬렉션이 생성되지 않는다.
          .toList() // 시퀀스를 다시 리스트로 변경한다.
     println(upperCaseNames)
}
```
* 시퀀스를 활용하는 경우, 중간 결과를 저장하는 컬렉션이 새로 생성되지 않으므로 요소가 많을수록 성능이 좋아진다.

### Sequence 인터페이스
* Kotlin의 지연 계산 시퀀스를 아래와 같은 Sequence 인터페이스를 활용한다.
  * 해당 인터페이스는 한 번에 하나씩 열거될 수 있는 원소의 시퀀스를 표현하며, 원소 순회를 위한 메소드 하나만을 포함한다.
  * 해당 메소드인 iterator를 통해 시퀀스로부터 원소의 값을 얻어올 수 있다.
```
public interface Sequence<out T> {
    public operator fun iterator(): Iterator<T>
}
```
* Sequence 인터페이스의 강점은 인터페이스에서 구현된 연산이 계산을 수행하는 방법 때문이다.
  * **시퀀스의 원소는 필요한 시점에 계산되므로 중간 처리 결과를 저장하지 않고 효율적인 계산이 가능**하다.
* **원소를 순서대로 순회해야 하는 경우 시퀀스가 적절하지만, 인덱스를 활용한 접근 등의 경우 컬렉션을 그대로 사용하는 것이 바람직**하다.
* **커다란 컬렉션에 대한 연쇄적인 연산이 필요한 경우, 반드시 시퀀스를 활용**해야 한다.
  * **컬렉션이 일반적인 크기인 경우, 중간 컬렉션을 생성하더라도 즉시 계산을 적용하는 컬렉션의 연산이 더 효율적**이다.
* 시퀀스에 대한 연산은 지연되므로, 실제로 연산을 적용하고 싶다면 다음과 같은 마무리가 필요하다.
  1. 원소를 하나씩 순회한다.
  2. 최종 결과를 리스트로 변환한다.

### 시퀀스의 연산
* 시퀀스의 연산은 중간 연산과 최종 연산으로 나뉜다.
  1. 중간 연산: 다른 시퀀스를 반환하며, 각 시퀀스는 최초 시퀀스의 변환 방법을 알고 있다.
  2. 최종 연산: 최초 컬렉션에 대해 변환을 적용한 시퀀스로부터 일련의 연산을 통해 얻을 수 있는 컬렉션이나 원소, 숫자, 객체 등의 결과를 반환한다.
* 중간 연산은 항상 지연 계산되며, 최종 연산이 없으면 연산이 적용되지 않는다.
  * 아래와 같이 최종 연산을 주석 처리한 경우, kotlin.sequences.TransformingSequence@38af3868와 같은 의미 없는 값이 출력된다.
  * **주석을 다시 해제하면 원하는 결과가 출력되며, 이는 최종 연산을 호출해야 지연된 계산이 모두 적용됨을 의미**한다.
```
fun main(args: Array<String>) {
     val upperCaseNames = animals
          .asSequence()
          .filter { it.age < 10 } // 새로운 중간 컬렉션이 생성된다.
          .map { it.name.uppercase() } // 새로운 중간 컬렉션이 생성된다.
//          .toList()
     println(upperCaseNames)
}
```

### 컬렉션과 시퀀스 연산의 차이
* map, filter 연산을 컬렉션과 시퀀스 각각에 대해 연쇄적으로 적용하는 경우,
  1. 컬렉션은 map을 적용한 결과 컬렉션을 생성하고, 새로운 컬렉션에 대해 filter를 적용한다.
  2. **시퀀스는 모든 원소를 순차적으로 순회하며, 원소마다 map > filter를 적용**한다.
* 때문에 **시퀀스에 대한 지연 계산은 원소마다 연산을 적용하다 결과가 얻어진 시점에서 그 이후의 연산을 수행하지 않을 수도 있다**.
  * 아래의 예시에서, .asSequence()를 주석 처리하는 경우 map 연산에 의해 animals 의 모든 요소가 출력된다.
  * 주석 해제하는 경우, **find 호출에 의해 시퀀스의 모든 원소에 map > find를 각각 적용**하며 결과가 반환되는 시점 이후의 요소는 출력되지 않는다.
```
fun main(args: Array<String>) {
     val result = animals
          // .asSequence()
          .map { animal: Animal ->
               println(animal)
               animal
          }
          .find { it.age == 20 }

     println("result: $result")
}
```
* **컬렉션에 대한 중간 연산의 순서는 성능에 영향을 미친다**.
  * 예를 들어, 일반적으로 map > filter보다 filter > map이 더 적은 요소를 검사하도록 하므로 성능이 좋다.
  * 이는 filter 연산에 의해 불필요한 원소가 연산에서 제외되기 때문이다.

### 시퀀스의 생성
* 상술한 예시에서는 컬렉션에 대해 asSequence()를 호출하여 시퀀스를 생성한다.
* generateSequence() 함수를 사용하면 이전의 원소를 인자로 받아 다음 원소를 계산하는 시퀀스를 생성할 수 있다.
  * 즉, generateSequence()의 인자로 첫 번째 원소를 지정하고,
  * 시퀀스의 앞 원소로부터 다음 원소를 계산하는 방법을 제공함으로써 시퀀스를 생성한다.
* 아래의 예시는 0부터 99까지 포함된 리스트를 생성한다.
  * generateSequence(0)은 최초 원소인 0 부터 시작해서 다음 원소를 계산하기 위한 조건인 + 1을
  * takeWhile 에서 99까지 계산한다.
    * 이 시점에서, 최종 연산이 수행되지 않았으므로 각 연산은 지연된 상태이다.
  * toList()라는 최종 연산에 의해 sequence가 계산된다.
```
fun main(args: Array<String>) {
     // 최초 원소와 다음 원소를 계산하는 방법을 정의한다.
     val sequence = generateSequence(0) { it + 1 }
          // 시퀀스를 적용할 범위를 정의한다.
          .takeWhile { it < 100 }
     // 최종 연산인 toList()를 호출하여 시퀀스의 각 원소에 계산을 적용한다.
     println(sequence.toList())
}
```

### Stream vs Sequence
* Java 8에 도입된 Stream API와 Kotlin의 Sequence API는 유사하다.
  * 개념적으로는 같다!
* **Kotlin은 단순히 Stream API가 없는 Java 8 이전의 버전을 사용하는 환경을 위해 Sequence API를 제공**한다.
* Java 8 이후의 버전을 사용하고 있다면, 아직 Sequence가 제공하지 못하는 기능을 Stream API에서 누릴 수 있다.
* 즉, 개발자의 필요와 Java 버전에 따라 Sequence와 Stream 중 적절한 API를 선택하여 개발한다.