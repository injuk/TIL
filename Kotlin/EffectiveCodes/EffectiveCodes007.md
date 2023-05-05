# EffectiveCodes
## 2023-05-01 Mon
### 코드 효율성을 바라보는 관점
* 현대에는 메모리 등 하드웨어 자원이 저렴해졌고, 개발자의 몸값은 올랐기에 코드 효율성을 관대하게 바라보곤 한다.
* 그러나 코드의 효율성은 장기적인 관점에서 중요하며, 효율성을 높이는 것으로 장기적으로는 비용을 줄이고 효율을 높일 수 있게 된다.
* 반면, **코드 최적화는 난이도가 높은 축에 속하며 프로젝트의 초기 단계에서부터 도입할 경우 잃는 것이 더 많을 수 있다**.
  * 그럼에도 **거의 잃는 것 없이도 코드의 효율성을 높일 수 있는 고정된 규칙은 존재하며, 이를 토대로 비용 없이 성능을 향상**시킬 수 있다.
* 나아가 가독성과 성능 사이에는 일종의 트레이드 오프 관계가 존재하며, 개발자는 컴포넌트에서 어떤 것이 더 중요한지 스스로 결정할 수 있어야 한다.

### 객체 생성과 비용
* **객체 생성은 언제나 크거나 작은 비용을 포함하므로, 불필요한 객체 생성을 지양하는 것은 최적화 관점에서 언제나 바람직한 일**이다.
  * **객체 생성을 최소화하는 예로, JVM은 한 번 이상 사용되는 동일한 문자열은 재사용하며 Int 역시 -128 ~ 127 범위에서 캐시**해둔다.
* 어떤 객체, 또는 래퍼 객체를 사용할 경우 크게 다음과 같은 비용을 고려할 수 있다.
  1. 원시 타입에 비해 객체 자체가 더 많은 용량을 차지하며, 64비트 JDK의 경우 객체는 최소 16바이트에서 시작하여 8의 배수만큼의 용량을 차지한다.
  2. 요소가 캡슐화되어 있다면 이에 접근하기 위한 추가적인 함수 호출이 필요하며, 함수 처리 자체는 굉장히 빠르지만 누적되기 쉽다.
  3. 객체는 사용을 위해 우선 생성될 필요가 있으므로, 생성과 메모리 영역 할당 및 레퍼런스 생성 등등의 추가 작업을 필요로 한다.
* 상술한 **세 가지 비용은 모두 자체적으로는 굉장히 적은 비용이지만, 크게 누적되기 쉬운 분류에 속한다**.
  * 때문에 객체를 제거할 경우 이러한 비용을 모두 회피할 수 있으며, 최소한 객체를 재사용하는 것으로 첫 번째와 세 번째 비용을 회피할 수 있다.

### 객체의 재사용 - 객체를 선언하기
* 매 순간 새로운 객체를 생성하는 대신 재사용하는 가장 간단한 방법은 object 한정자를 활용하는 객체 선언으로, 이는 싱글톤이라는 용어로도 지칭할 수 있다.
* **빈번히 재사용되는 객체는 싱글톤으로 정의할 수 있으며, 제네릭 타입의 경우 모든 타입의 서브타입인 `Nothing`을 유용하게 활용**할 수 있다.

### 객체의 재사용 - 팩토리에 캐시를 활용하기
* 일반적으로 객체는 생성자를 토대로 생성되지만 팩토리 메소드를 활용하는 경우도 많으며, 이 때 캐시 기능을 도입할 수 있다.
  * 예를 들어, stdlib의 `emptyList()` 메소드는 캐싱을 활용하여 항상 같은 객체를 반환하도록 구현되어 있다.
* 여러 객체를 두어 그 중 하나를 사용하는 경우에는 객체 풀의 정의를 고려할 수 있으며, 쓰레드 풀과 DB 커넥션 풀이 좋은 예시이다.
  * 이러한 **객체 풀 방식은 무거운 객체를 생성히거나, 동시에 여러 개의 mutable 객체를 사용해야 하는 경우에 유용**하다.
* 또한 모든 순수 함수에도 캐싱을 적용할 수 있으며, 이는 메모이제이션이라는 용어로 지칭된다.
* 캐시 방식은 두 번째 객체 접근부터는 좋은 성능을 보이나, 캐싱 기능 자체로 인해 더 많은 메모리를 사용하는 단점 역시 존재한다.
* 때문에 메모리 문제가 발생한다면 이를 해제할 필요가 있으며, 이 경우애는 GC가 자동으로 메모리를 해제하는 `SoftReference`를 사용하는 것이 바람직하다.
  * 이 때, `WeakReference`는 GC가 값을 정리하는 것을 막지 않으므로 다른 레퍼런스가 이를 참조하지 않으면 곧바로 제거한다.
  * 반면, `SoftReference`는 GC가 값을 정리하거나 하지 않을 수 있으며, 일반적인 JVM 구현의 경우 메모리가 부족한 경우에만 정리한다.
* 이렇듯 **캐시 기능의 구현을 위해서는 `SoftReference` 방식이 더 권장되며, 캐시는 언제나 메모리와 성능의 트레이드 오프가 발생**할 수 밖에 없다.
  * 때문에 반드시 상황을 잘 고려하여 캐싱 기능의 도입을 결정하는 것이 바람직하다.

## 2023-05-02 Tue
### 무거운 객체 또는 작업은 외부 스코프로 옮기기
```
> 무거운 객체나 무거운 작업을 외부 스코프로 옮기는 것은 성능 상 매우 유용한 트릭이다.
```
* 에를 들어 컬렉션 처리 과정에서 반복적으로 호출되는 무거운 연산은 컬렉션 처리 함수 내부가 아닌 외부로 빼내는 것이 성능 측면에서 큰 이득을 볼 수 있다.
  * 이러한 원칙은 일견 당연한 것으로 보일 수 있으나, 실제로는 많은 개발자들이 실수하거나 간과하는 부분에 해당한다.
* 또 다른 예로 **정규 표현식 패턴을 활용하는 함수의 경우, 함수 내부의 변수에 정규 표현식을 할당하는 것보다는 최상위로 끌어올리는 것이 바람직**하다.
  * **정규 표현식 패턴을 컴파일하는 과정은 복잡한 연산에 속하므로, 함수를 호출할 때마다 정규 표현식을 할당하는 것은 성능 상 득보다 실**이 더 많다.
* 나아가 **정규 표현식을 활용하는 함수가 다른 함수와 같은 파일에 존재하여 함수 자체가 잘 사용되지 않는다면, 지연 초기화를 적용하는 것이 바람직**하다.
  * 이 경우, 정규 표현식 패턴이 컴파일된다는 것 자체가 낭비이므로 `val REGEX by lazy { "some_regex".toRegex() }`와 같이 정의할 수 있다.
  * 이렇듯 프로퍼티를 지연 초기화하는 것은 무거운 클래스를 사용해야 하는 상황에서 더 유용하다.
* **해당 방식은 성능 자체를 향상시켜줄 수 있을 뿐더러, 적절한 명명을 토대로 가독성까지도 향상시킬 수 있으므로 적극적으로 고려하는 것이 권장**된다.

### 지연 초기화 활용하기
* 무거운 클래스를 생성하는 경우, 지연되게 생성하는 것은 다음과 같은 장점과 단점이 모두 존재한다.
  1. 장점: 내부에서 참조하는 인스턴스들을 지연 초기화하는 것으로 무거운 객체의 생성 자체를 가볍게할 수 있다.
  2. 단점: **클래스가 무거운 인스턴스들을 참조하지만 이를 활용한 메소드 호출은 빨라야 하는 경우에는 적절하지 못하다**.
* 일반적으로 백엔드 애플리케이션은 전체 실행 시간은 크게 중요하지 않으므로, 첫 호출 시 응답 시간에 악영향을 주는 지연 초기화는 적절하지 않을 수 있다.

### 가능한 한 원시 타입을 사용하기
* JVM은 숫자나 문자 등의 기본 요소를 표현하기 위한 기본 내장 자료형을 가지며, 이를 원시 타입이라고 지칭한다.
  * **코틀린과 JVM 컴파일러 역시 내부적으로는 최대한 원시 타입을 활용하는 식으로 동작**한다.
* 그러나 다음과 같은 두 상황에서는 래퍼 클래스가 사용되며, 원시 타입을 사용할 수 없다는 점에 주의해야 한다.
  1. nullable 타입: 원시 타입은 null일 수 없으므로, 래퍼 클래스가 사용된다.
  2. 제네릭 타입 인자: 원시 타입은 제네릭 타입 인자로 사용될 수 없으므로, 래퍼 클래스가 사용된다.
* 상술한 항목을 예로 들어, Int 형 변수는 원시 타입인 `int`를 사용하는 반면 Int?나 List<Int>는 래퍼 클래스인 `Integer`를 사용한다.
* 이를 토대로 **래퍼 클래스 대신 원시 타입을 사용하도록 코드를 최적화할 수 있으나, 일반적으로 이는 숫자에 대한 큰 작업이 반복될 때 의미**를 갖는다.
  * 기본적으로 숫저와 관련된 연산은 어떤 자료형을 사용하더라도 성능적으로 큰 차이를 보이지 않는 경향이 있다.
  * 때문에 **이러한 최적화는 굉장히 큰 컬렉션에 대한 작업을 처리할 때에서야 유효한 성능 차이를 보일 수 있으므로, 성능이 중요할 때에만 적용해도 무방**하다.

### 가능한 한 원시 타입을 사용하기?
* 이러한 최적화는 성능이 극단적으로 중요하지 않은 경우에는 큰 의미가 없는 축에 속하므로, 성능이 중요한 코드 또는 라이브러리 구현에만 적용을 고려해야 한다.
* 때문에 **해당 최적화 기법의 구현에 많은 비용이 들거나, 다른 코드에 많은 영향을 줄 수 있는 상황이라면 최적화를 미루는 선택지도 고려**할 수 있다.

## 2023-05-03 Wed
### 고차 함수에는 inline 한정자를 고려하기
* 코틀린 stdlib의 `repeat()`과 같은 고차 함수의 경우, 대부분 inline 한정자가 명시되어 있다.
* 이 때, **inline 한정자는 컴파일 시점에 함수 호출부가 inline 한정자가 명시된 함수 내부에 정의된 본문으로 대체되는 식으로 동작**한다.
  * 일반적인 함수를 호출할 경우 함수 본문으로 점프한 후, 함수 본문의 모든 문장을 호출한 후에 원래 위치로 다시 점프하게 된다.
  * 그러나 **inline 한정자를 활용할 경우 함수 호출부 자체가 함수 본문으로 대체되므로, 상술한 점프 과정 자체가 생략되는 이점**이 있다.
* 또한 inline 한정자를 함수에 명시할 경우, 크게 다음과 같은 장점을 얻을 수 있다.
  1. 타입 인자에 reified 한정자를 명시할 수 있다.
  2. 함수 타입을 매개 변수로 갖는 고차 함수의 동작이 훨씬 빨라질 수 있다.
  3. 비지역적인 반환(=non-local return)을 사용할 수 있다.

### 타입 인자와 reified 한정자
* **초기 Java에는 제네릭 자체가 존재하지 않았으므로, 코드 상에서 제네릭을 사용하더라도 JVM 바이트 코드 상에서는 이를 표현할 수 없는 문제**가 있다.
  * 이러한 이유에서 컴파일 후 제네릭과 관련된 내용은 모두 제거되기에 제네릭 인자를 타입으로서 다루는 코드는 기본적으로 사용이 불가능하다.
* 그러나 함수에 inline 한정자를 명시할 경우 함수 호출이 본문으로 대체되므로, reified 한정자를 추가로 명시하여 이러한 제약으로부터 벗어날 수 있다.
  * 예를 들어 아래의 예시와 같이, 타입 인자를 사용하는 부분이 컴파일 시점에 실제 타입으로 대체될 수 있다.
```kotlin
inline fun <reified T> printType() {
    print(T::class.simpleName)
}

printType<Int>()

// 상술한 코드는 컴파일 시점에 아래와 같이 표현된다.
// print(Int::class.simpleName)
```

### inline 한정자와 고차 함수의 성능
```
> 함수 타입 매개 변수를 사용하는 유틸리티 함수에는 inline 한정자를 명시하는 것이 바람직하며, 실제로도 stdlib의 고차 함수는 대부분 inline으로 정의된다.
```
* **단적으로 말해 모든 함수는 inline 한정자를 명시하는 것으로 조금 더 빠르게 동작하므로, stdlib의 간단한 함수는 대부분 해당 한정자를 명시**한다.
  * 이는 함수의 호출과 반환을 위해 점프하거나 백스택을 추적하는 과정이 생략되기 때문이다.
* 그러나 함수 타입을 매개 변수로 갖지 않는 일반 함수에서는 이러한 차이가 큰 영향을 주지 않으므로, IntelliJ와 같은 IDE는 이러한 상황에 경고를 표시한다.
* 이는 **고차 함수가 JVM의 익명 클래스 또는 일반 클래스를 기반으로 객체화되기 때문이며, 기본적으로 람다 표현식은 클래스로 컴파일**된다.
  * 이 과정에서 고차 함수를 표현하기 위해 필요한 모든 인터페이스는 내부적으로 코틀린 컴파일러에 의해 생성된다.
  * 앞서 다룬 바와 같이 **함수 본문을 굳이 객체로 래핑하는 과정에서 코드의 속도는 느려질 것이므로, inline 한정자를 명시한 함수의 실행이 더 빠르다**.
* 나아가 **inline 한정자가 누락된 람다 표현식 내부 스코프에서는 외부의 지역 변수를 사용할 수 없으며, 이 역시 컴파일 과정에서 객체로 래핑**된다.
  * 다시 말해, 람다 표현식 내부에서는 스코프 외부의 지역 변수가 한 번 레퍼런스 객체로 래핑된 후에 이를 참조하는 식으로 컴파일된다.
  * 결국 함수가 객체 형태로 컴파일된 후에 지역 변수가 래핑되므로 더 많은 비용을 필요로 하고, 이러한 문제가 누적될수록 성능 상의 이슈로 이어지기 쉽다.

### 비지역적 반환이란?
* inline 한정자를 명시하지 않은 고차 함수의 경우, 함수 블록 내부에서 return 문을 명시할 수 없다.
* 이는 함수 리터럴이 컴파일될 때 객체로 래핑되기 때문으로, 함수가 다른 클래스에 위치하기에 return 문을 명시하더라도 이전 함수로 복귀할 수 없기 때문이다.
  * 반면, **inline 한정자를 명시한 함수는 함수 본문 자체가 호출 부를 대체하므로 함수 블록 내부에서 return 문을 명시하는 것이 가능**하다.
  * 이러한 특징으로 인해 inline 한정자가 명시된 함수는 마치 제어문이나 반복분처럼 보여질 수 있으며, 간접적으로 가독성 역시 향상된다.

## 2023-05-04 Thu
### inline 한정자의 비용
* inline 한정자가 명시된 함수는 재귀적으로 호출될 수 없으며, 여러 가시성 제한을 사용할 수 없다는 단점이 존재한다.
  * 즉, inline 함수는 구현을 숨길 수 없으므로 클래스에서는 거의 사용되지 않는다.
  * 나아가 **해당 한정자를 남발할수록 코드의 크기는 기하급수적으로 커질 수 있으므로 위험**할 수 있다.

### crossinline, 그리고 noinline
* 함수를 인라인으로 적용하되 일부 함수 타입 파라미터는 inline을 적용하고 싶지 않은 경우에 다음과 같은 한정자를 적용할 수 있다.
  1. crossinline: 인자로 inline 함수를 전달받되 비지역적 반환하는 함수는 전달받을 수 없도록 제한한다.
  2. noinline: 인자로 inline 함수를 전달받을 수 없게 하며, inline 함수를 제외한 인자를 전달받고자 하는 경우에 사용할 수 있다.

### inline 한정자의 결론
* 함수 자체가 너무나도 자주 사용되는 경우, inline 한정자의 명시를 고려할 수 있다.
* 또한 타입 인자로 reified 타입을 전달받는 경우에도 inline 한정자의 명시를 고려할 수 있다.
* 함수 타입 인자를 전달 받는 톱레벨 고차함수를 정의하는 경우 inline 한정자를 유용히 활용할 수 있으며, 특히 컬렉션 처리와 같은 헬퍼 함수에 유용하다. 
* 나아가 임의의 inline 함수가 다른 inline 함수를 자주 호출할수록 코드는 기하급수적으로 늘어날 수 있으므로 반드시 주의를 기울여야 한다.

### inline 클래스를 고려하기
* **함수뿐만이 아니라 단일 프로퍼티만을 갖는 객체 역시 inline 한정자를 명시할 수 있으며, 이 경우 해당 객체를 사용하는 모든 코드가 프로퍼티로 교체**된다.
  * 또한, inline 클래스의 모든 메소드는 정적 메소드로 정의된다.
* **inline 클래스는 다른 자료형을 래핑하는 새로운 자료형을 정의할 때 주로 사용되며, 이 과정에서 어떠한 오버헤드도 발생하지 않는다**.
  * 때문에 inline 클래스는 측정 단위를 표현하거나, 타입 오용으로부터 기인하는 문제를 방지하기 위해 주로 사용된다.

### 측정 단위를 표현하기 위한 inline 클래스
* 측정 단위의 혼용은 큰 문제를 초래할 수 있으므로, 파라미터 타입에 제한을 주는 것은 언제나 유용하다.
  * 이 과정에서 **더 효율적인 클래스의 정의를 위해 inline 한정자를 명시할 수 있으며, 이로 인해 올바른 타입이나 측정 단위를 사용하도록 강제**할 수 있다.

## 2023-05-05 Fri
### 타입 오용으로 인한 문제를 방지하기 위한 inline 클래스
* 예를 들어 id와 같은 필드가 Int이거나 String과 같은 간단한 자료형인 경우, 실수로 잘못된 값을 사용하기가 쉬워 문제가 발생할 수 있다.
* 이러한 타입 오용으로 인한 문제를 미연에 방지하기 위해 inline 클래스를 정의할 수 있으며, 컴파일 시점에 타입이 대체되므로 문제도 발생하지 않는다.
  * 즉, **inline 클래스를 도입하는 것으로 안전을 위해 새로운 타입을 추가하더라도 별도의 오버헤드가 발생하지 않을 수 있다**.

### 인터페이스를 구현하는 inline 클래스?
* inline 클래스 역시 다른 클래스들과 같이 인터페이스를 구현할 수 있으나, 이 경우 inline 클래스를 사용하는 장점을 전혀 얻지 못한다.
  * 이는 인터페이스를 구현하는 과정에서 객체를 래핑하기 때문으로, 인터페이스를 구현하는 inline 클래스는 아무런 의미가 없다.

### 타입 별칭의 한계
* 이미 **존재하는 타입에 적용할 수 있는 타입 별칭은 `typealias` 키워드를 통해 할당할 수 있으며, 이는 길고 반복적인 타입의 가독성을 높여준다**.
  * 예를 들어, 자주 사용되는 함수 타입에 대해 타입 별칭을 할당할 수 있다.
* 그러나 타입 별칭은 둘 이상의 타입 별칭을 사용할 때 발생하기 쉬운 사용자의 실수를 방지할 수 없으며, 그저 안전한 코드를 작성한다는 착각만을 심어주기 쉽다.
* 때문에 **Int 등으로 표현되는 단위 따위를 표현하기 위해서는 타입 별칭보다는 매개 변수의 이름이나 클래스를 적용하는 것이 바람직**하다. 
  * 매개 변수 이름은 적은 비용을 소모하며, 클래스는 기본적으로 안전하므로 inline 클래스를 통해 비용과 안전이라는 두 장점을 모두 취할 수 있다.

### inline 클래스 - 결론
* **inline 클래스를 토대로 성능 오버헤드 없이도 임의의 타입을 래핑할 수 있으며, 타입 시스템을 활용하여 휴먼 에러를 미연에 방지**할 수 있다.
* inline 클래스를 적절히 활용하는 것으로 코드의 안정성은 높아질 수 있으며, 가독성 측면에서도 의미가 명확하지 않은 타입 등을 표현하기 적절하다.