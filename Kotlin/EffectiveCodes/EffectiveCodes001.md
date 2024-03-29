# EffectiveCodes
## 2023-04-03 Mon
### 코틀린을 왜 사용하는가?
```
> 프로젝트에 코틀린을 도입하기 위한 대표적인 이유 중 하나는 안정성에 있다.
```
* 코틀린은 다양한 설계 및 지원을 토대로 안전한 언어가 되었지만, 정말 안전하게 사용하기 위해서는 개발자가 이를 뒷받침할 필요가 있다.
  * 즉, 코틀린을 토대로 애플리케이션을 제작하는 개발자는 오류가 덜 발생하는 코드를 만들고자 노력해야 한다.

### 가변성이란?
* 코틀린 애플리케이션을 구성하는 모듈은 클래스 또는 객체 및 함수 등 다양한 요소로 구성될 수 있으며, 이 중 일부는 가변적인 `상태`를 가질 수 있다.
  * 예를 들어 var 키워드로 설정된 프로퍼티 또는 mutable 객체가 그렇다.
* 이러한 가변적인 `상태`는 변화하는 요소를 표현하는 데에 유용하지만, **`상태`를 적절히 관리하는 것은 생각보다 어려운 일**이 될 수 밖에 없다.
  * 예를 들어 멀티 스레드의 경우 적절한 동기화가 필요하며, 테스트 관점에서는 가능한 모든 `상태`를 테스트하기도 어렵다.
* 이렇듯 **가변성은 시스템의 `상태`를 나타내기 위해 중요한 방법이나, 변경이 일어나는 부분은 반드시 신중하고 확실하게 결정된 후에 사용**되어야 한다.

## 2023-04-04 Tue
### 코틀린의 가변성 제한 방법
* 코틀린은 가변성을 제한할 수 있도록 설계되어 있으며, 그 중 가장 자주 사용되는 방식은 크게 다음과 같다. 
  1. val 키워드를 활용하여 읽기 전용 프로퍼티를 정의하기
  2. 가변 컬렉션과 읽기 전용 컬렉션을 구분하기
  3. data 클래스의 copy 메소드 활용하기
* 읽기 전용 프로퍼티의 경우 마치 값처럼 동작하며 일반적인 방법으로는 값이 변화하지 않는다.
  * 그러나 읽기 전용 프로퍼티가 mutable 객체를 참조하고 있는 경우, 내부적으로는 상태가 변경될 수 있다.
  * 이렇듯 코틀린의 프로퍼티는 기본적으로 캡슐화되어 있으므로 코틀린은 API를 변경하거나 정의할 때에 굉장히 유용하다.
* 코틀린에서 읽기 전용 프로퍼티와 가변 프로퍼티가 구분되듯, 컬렉션 역시 읽기 전용 컬렉션과 가변 컬렉션으로 구분될 수 있다.
  * 이 때, **읽기 전용 프로퍼티를 가변적으로 만들 수 있는 방법이 없는 것은 아니지만 반드시 읽기 전용으로만 사용하도록 강제**할 수 있어야 한다.
  * 때문에 읽기 전용 컬렉션을 가변 컬렉션으로 다운 캐스팅하지 않아야 하며, 대신 내부적으로 새로운 컬렉션을 만드는 `toMutableList()` 등을 활용한다.
* 불변 객체는 자신의 일부를 수정하는 새로운 객체를 만들어낼 수 있어야 하며, 코틀린에서 이러한 기능은 `data` 키워드를 통해 쉽게 적용할 수 있다.
  * 해당 키워드는 내부적으로 copy 메소드를 정의해주며, 이를 통해 모든 기본 생성자 프로퍼티가 같은 별개의 객체를 생성할 수 있다.
  * 또한, **해당 키워드를 토대로 정의된 데이터 모델 클래스는 불변 객체의 특성**을 갖게 된다.

### 변경 가능한 지점을 제어하기
```
> 변경 가능한 컬렉션보다는 변경 가능한 프로퍼티와 불변 컬렉션을 활용하는 것이 바람직하다.
```
* 변경 가능한 리스트를 정의하는 경우, 다음과 같이 두 가지 방식을 고려할 수 있다.
  1. mutable 컬렉션으로 정의하기
  2. var 키워드로 immutable 컬렉션을 정의하기
* **두 방식 모두 변경 가능한 지점이 존재하나, 전자는 컬렉션 내부에 위치하는 반면 후자는 프로퍼티 자체가 변경 가능한 지점인 차이가 존재**한다.
  * 당연히 **후자의 방식이 멀티스레드 환경에서 상대적으로 높은 안정성을 보일 것이므로, 결국 mutable 프로퍼티를 사용하는 것이 변경을 제어하기 더 쉽다**.

### 변경 가능한 지점은 노출하지 않기
* 상태를 나타내기 위한 mutable 객체를 직 / 간접적으로 public하게 공개하는 것은 매우 위험하므로 다음과 같은 방법을 활용하는 것이 바람직하다.
  1. 방어적 복제: mutable 객체를 복제하여 반환하며, 이 과정에서 `data` 키워드와 copy 메소드를 활용할 수 있다.
  2. 업캐스팅: **가변성을 제한하기 위해 컬렉션을 반환하는 시점에 읽기 전용 슈퍼타입으로 업캐스팅**한다.
* 이렇듯 **코틀린은 가변성을 제한하기 위한 다양한 선택지를 열어두므로, 상술한 여러 방법을 통해 가변 지점을 제한할 수 있도록 주의**를 기울여야 한다.
  * 예를 들어, 변경이 필요한 대상을 정의하는 경우 `data` 키워드를 적용하여 불변 데이터 클래스와 copy 메소드를 활용할 수 있다.
* 반면, 효율성을 위해 고의적으로 가변 객체를 활용할 수도 있으나 이러한 최적화 기법은 성능이 중요한 경우에만 적용하는 것이 바람직하다.

## 2023-04-05 Wed
### 변수의 스코프는 최소화하기
```
> 가능한 한 클래스의 프로퍼티보다 지역 변수를 사용하며, 최대한 좁은 스코프를 활용하는 것이 바람직하다.
```
* 코틀린 스코프는 기본적으로 중괄호 블록을 통해 정의되며, 다른 프로그래밍 언어와 마찬가지로 요소의 가시성을 제한한다.
* **스코프를 좁혀야 할 이유 중 가장 중요한 것은 유지보수성이며, 이를 통해 애플리케이션을 추적하고 관리하기 쉽도록** 만들 수 있다.
  * 애플리케이션이 간단할수록 가독성과 안정성은 높아지며, 이는 가변 프로퍼티보다 불변 프로퍼티를 선호하는 논리와 유사하다.
  * 또한, 애플리케이션의 추적이 쉬워질수록 코드를 수정하기도 쉬워진다.
* **변수는 읽기 전용이거나 가변적임과는 무관하게 가능한 한 정의되는 시점에 초기화되는 것이 바람직**하다.
  * 이를 위해 if 절 또는 when, try-catch 절과 엘비스 연산자 또는 구조 분해 할당을 적극적으로 활용할 수 있다.

### 플랫폼 타입의 사용은 지양하기
* **플랫폼 타입이란 Java 등의 다른 프로그래밍 언어로부터 전달된 타입으로, nullable 여부를 알 수 없는 특징**을 갖는다.
  * 때문에 이는 본질적으로 널 안정성으로 대두되는 코틀린의 주요 철학을 위협하는 요소일 수 밖에 없다.
* 플랫폼 타입은 언제든지 동작이 변경될 가능성이 있으므로, 반드시 주석 또는 어노테이션을 활용한 명시적 표현이 수반되어야 한다.
  * 예를 들어, Java를 코틀린과 병용하는 경우 Java 코드를 수정할 수 있다면 `@Nullable` 또는 `@NotNull` 어노테이션을 활용한다.
  * 이렇듯 여러 어노테이션의 지원을 받을 수 있으나, 그럼에도 여전히 플랫폼 타입은 가능한 한 제거되는 것이 바람직하다.
  * 또한 플랫폼 타입은 해당 코드 부분 뿐만 아니라 이를 사용하는 다른 코드에도 영향이 전파될 수 있으며, 이는 기본적으로 추적하기 매우 어려운 축에 속한다.

## 2023-04-06 Thu
### inferred 타입으로 리턴하지 않기
```
> 추론된 타입은 프로젝트가 성숙할수록 예측하지 못한 결과를 반환하기 쉽다.
> 때문에 타입을 확실하게 지정해야하는 경우, 반드시 타입 정보를 명시하는 원칙을 유지하는 것이 바람직하다.
```
* **inferred(추론된 타입)은 절대로 부모 타입이나 인터페이스로 추론되지 않으며, 항상 할당문 오른쪽의 피연산자에 정확히 맞는 타입으로 설정**된다.
* 추론된 타입을 반환하는 것은 모든 코드가 개발자의 제어 아래 있다면 상관이 없으나, 라이브러리 등 서드 파티 코드를 사용하는 경우에 문제가 될 수 있다.
* **반환형은 해당 API를 사용하는 개발자에게 정보를 명시적으로 전달할 수 있는 방법이므로, 반환형은 외부에서도 확인할 수 있도록 명시하는 것이 바람직**하다.

### 코드에 꼭 필요한 제한이 있는 경우, 예외를 활용하기
* 임의의 코드가 반드시 어떠한 형태로 동작해야하는 경우, 다음과 같은 예외를 활용하여 제한을 명시하는 것이 바람직하다.
  1. require 블록: 매개변수에 제한을 적용할 수 있다.
  2. check 블록: 상태와 관련된 동작에 대해 제한을 적용할 수 있다.
  3. assert 블록: 테스트 모드에서만 동작하며, 어떠한 것이 true인지 확인할 수 있도록 지원한다.
  4. return 또는 throw와 함께 적용된 엘비스 연산자
* 이러한 **제한 사항의 명시를 통해 개발자는 문서 없이도 코드를 더 잘 이해할 수 있으며, 문제가 있는 경우 함수는 이상 동작하는 대신 예외를 반환**하게 된다.
  * 당연스럽게도 예상치 못한 동작을 수행하는 것은 예외를 던지는 것보다 훨씬 위험하다.
  * 나아가 제한 사항의 명시를 통해 불필요한 단위 테스트와 타입 캐스팅 코드를 줄일 수 있게 된다.
* require 함수는 매개 변수에 대한 제한을 적용하기 위해 사용하며, 위배되는 경우 예외를 throw한다.
  * 이 때, 블록 내부에 문자열을 작성하여 예외 메시지를 명시할 수 있으며 throw 되는 예외는 `IllegalArgumentException`에 해당한다.
* check 함수는 임의의 객체 상태와 관련된 제한을 명시하며, 조건을 만족하지 못하는 경우에 `IllegalStateException`을 throw 한다.
  * 또한, **해당 제약은 사용자가 규칙을 위배하고 사용하지 말아야할 곳에서 함수를 호출하고 있을 것으로 의심되는 경우에 명시**할 수 있다.
  * require 함수와 병용하는 경우, 일반적으로 check 블록이 더 나중에 실행되도록 배치한다.
* 나아가 require 함수는 nullable 여부를 확인할 때 굉장히 유용하며, 이를 지원하는 `requireNotNull` 또는 `checkNotNull` 함수가 존재한다.
  * 무엇보다 **이러한 함수들은 스마트 캐스트를 지원하며, 변수의 언패킹을 위해 명시하여 스마트 캐스트를 유발**시킬 수 있다.
* 또한 **nullable한 변수와 엘비스 연산자, 그리고 return 문 또는 throw 문을 병용하는 것으로 제한 사항을 명시**할 수 있다.
  * 반면, 변수가 null일 때 어떠한 처리를 수행한 후 함수를 종료시키는 경우에는 run 블록과 return 문을 조합할 수 있다.
  * 이렇듯 return과 throw를 활용하는 엘비스 연산자는 nullability를 확인하기 위해 자주 사용되며, 일반적으로는 함수 앞부분에 명시하는 것이 바람직하다.
* 상술한 내용을 토대로, 애플리케이션의 제한을 훨씬 더 쉽고 빠르게 확인할 수 있으면서도 안정성을 유지할 수 있게 된다.
  * 또한, **코드를 잘못된 용도로 사용하는 것을 방지하거나 필요한 위치에서 스마트 캐스트를 유발시킬 수도 있다는 이점 역시 존재**한다.

## 2023-04-07 Fri
### 최대한 표준 오류를 사용하기
* **애플리케이션을 개발하는 과정에서 필요에 따라 직접 사용자 정의 예외를 추가할 수 있으나, 가능하다면 표준 라이브러리 오류를 사용하는 것이 바람직**하다.
  * 표준 라이브러리 오류는 대부분의 개발자가 알고 있으므로, 이를 활용할수록 애플리케이션의 가독성을 높일 수 있다.

### null 또는 Failure를 적극적으로 활용하기
```
> 예외는 개발자가 예상하지 못한, 잘못된 부분을 알려 주기 위해 사용하는 장치이다.
```
* 네트워크 오류 등으로 인해 함수가 원하는 결과를 만들 수 없는 경우가 있으며, 일반적으로 이러한 상황은 예외를 던져 처리한다.
* 그러나 **예외는 정보를 전달하는 방법으로 사용되지 않아야 하며, 반드시 잘못된 특별한 상황만을 표현하기 위해 사용되어야 한다**.
  * 무엇보다, **예외는 정말 특수한 상황을 처리하기 위해 존재하므로 코드를 활용한 명시적인 검사만큼 빠를 수 없다**.
* 때문에 **대체제로서 null 또는 Failure를 사용할 수 있으며, 이를 통해 명시적이고 효율적인 방식으로 오류를 표현**할 수 있다.
  * 즉, 충분히 예측 가능한 오류는 null과 Failure를 우선적으로 고려한다.
  * 반면, 예측이 어려운 예외적인 범위의 오류 상황은 예외를 던져 처리할 수 있다.
* **상술한 처리 방식은 고전적인 try-catch 블록보다 효율적이면서 사용법이 간단하고 명시적인 장점**이 있다.
  * 예를 들어 예외는 놓칠 수 있으며, 심한 경우 전체 애플리케이션을 중지시킬 수 있지만 상술한 두 방식은 명시적인 처리를 코드 차원에서 요구한다.
* Failure와 같은 sealed 클래스와, null 의 용도 차이는 크게 다음과 같다.
  1. Failure: **처리에 필요한 정보를 함께 반환할 수 있으므로, 실패와 함께 추가적인 정보를 전달해야하는 경우에 활용**한다.
  2. null: 1.에 해당하지 않는 경우에 주로 사용된다.
* 나아가 **개발자는 항상 자신이 모든 요소를 안전하게 추출할 것을 기대하므로, 절대 nullable을 리턴하지 않아야 한다**.

## 2023-04-08 Sat
### 방어적 프로그래밍, 공격적 프로그래밍
* 방어적 프로그래밍이란 발생할 수 있는 예외 상황들에 대해 안정성을 높이고자 하는 방법들을 지칭하는 포괄적인 용어이다.
* 공격적 프로그래밍이란 예상치 못한 상황이 발생했을 때 이를 개발자에게 알려 수정하도록 유도하는 방법을 의미한다.
  * 예를 들어, require / check / assert 등은 공격적 프로그래밍을 위해 사용하는 도구이다.
* **둘은 상충되는 개념이 아니며, 오히려 코드의 안전성을 끌어올리기 위해 병용되어야 하는 개념**이다.
  * 예를 들어 상황을 처리할 수 있는 올바른 방법이 있다면 방어적 프로그래밍을 사용하되, 예상치 못한 상황은 공격적 프로그래밍으로 대비할 수 있다.

### null은 적절하게 처리하기
* 코틀린에서 nullable 타입을 처리하는 방법은 크게 다음과 같이 분류할 수 있다.
  1. ?.과 스마트 캐스팅, 엘비스 연산자 등을 활용하여 안전하게 처리한다.
  2. 또는 예외를 던진다.
  3. 또는 함수 및 프로퍼티를 리팩토링하여 nullable 타입이 반환되지 않도록 한다.
* ?. 연산자를 활용하는 안전 호출은 사용자 관점에서 가장 안전한 방법이며, 다른 방식에 비해 더 광범위하게 활용된다.
  * 또한 많은 객체가 nullable과 관련된 처리를 지원하며, 예를 들어 컬렉션의 경우 `Collection<T>?.orEmpty()`를 활용할 수 있다.
* 응당 정상적으로 동작해야할 만한 코드들은 throw나 !! 및 requireNotNull / checkNotNull 등을 활용할 수 있다.
  * 이 경우 당연히 동작해야할 것으로 보이는 코드들이므로, 오류가 발생한다면 이를 강제로 개발자에게 알릴 수 있도록 상술한 방법을 사용한다.
* 프로퍼티가 처음 사용되기 전에 반드시 초기화될 것으로 예상되는 경우, lateinit 한정자를 명시할 수 있다.
  * 이 경우 !! 문법을 통해 언패킹할 필요가 없으며, 추후에 null을 대입하여 nullable한 프로퍼티처럼 취급할 수도 있는 장점이 존재한다.

### not null 단언
```
> 코틀린을 사용하는 개발 팀은 대부분 !! 문법을 사용하지 못하도록 강제하는 정책을 준수한다.
```
* !! 기호를 활용한 not null 단언문은 사용하기 쉽지만, 좋은 해결책으로 볼 수는 없다.
  * 해당 문법은 일반적으로 nullable 타입이지만 사실상 null이 되는 상황이 거의 없는 경우에 사용할 수 있다.
  * 그러나 **현재에 확실하다고 해서, 미래의 모든 순간에도 항상 확실하다고 볼 수는 없다**.
* 명시적인 예외를 던지는 것은 지양해야하지만, 최소한 generic NPE보다 더 많은 정보를 제공해줄 수 있으므로 !! 단언보다는 좋다.
* **해당 문법이 의미 있는 상황은 매우 드물며, 일반적으로는 nullability가 제대로 표현되지 않는 라이브러리를 사용할 때에만 적용을 고려**해볼 수 있다.

### 무의미한 nullability는 지양하기
```
> nullability는 반드시 처리되어야 하며, 이는 곧 추가 비용이 발생함을 의미한다.
```
* null은 중요한 메시지를 전달하기 위해 사용되는 반면, 바꿔 말해 의미가 없어 보이는 상황에는 null을 사용하지 않는 것이 바람직하다.
  * 때문에 nullability를 처리하기 위해 `List<T>`의 get 또는 getOrNull과 같은 확장 함수를 만들어 제공하는 것을 고려할 수 있다.

## 2023-04-09 Sun
### Closable과 AutoClosable 구현체는 use를 활용하여 처리하기
* Input / OutputStream과 SqlConnection 등은 더 이상 필요하지 않을 때 use 메소드를 명시하여 닫아주어야 할 필요가 있다.
  * 물론 이러한 **리소스 역시 더 이상 참조되지 않을 경우 쓰레기 수집의 대상이 될 수 있으나, 이는 매우 느리며 리소스 유지 비용도 크게 소모**한다.
* 전통적으로는 try-finally 블록을 명시하여 리소스를 더 이상 사용하지 않는 시점에 닫아주었으나, 이는 다음과 같은 문제점을 극복하지 못한다.
  1. 리소스를 닫는 과정에서 예외가 발생할 수 있으나, 이를 따로 처리하기 어렵다.
  2. try 블록과 finally 블록 각각에서 발생한 예외는 항상 둘 중 하나만 전파된다.
* **코틀린은 이러한 상황에 사용할 수 있도록 쉽고 안전한 use 함수를 제공하며, 이는 모든 Closable 구현체에 대해 적용이 가능**하다.
  * 또한, 텍스트 파일을 스트림을 토대로 한 줄씩 읽어 처리할 수 있도록 useLines 함수 역시 제공된다.

### 단위 테스트를 작성하기
```
> 코틀린은 코드를 안전하게 다룰 수 있도록 하는 다양한 방법을 지원하지만, 사실 가장 궁극적인 방법은 다양한 종류의 테스트를 작성하는 것이다.
```
* 일반적으로 개발자가 아닌 관리자에게 인식되는 테스트란, 사용자 관점에서 애플리케이션 외부적인 기능의 동장을 확인하는 데에 목표를 둔다.
  * 그러나 이러한 종류의 테스트는 분명 유용하지만 개발자에게는 충분하지 않으며, 어떤 요소가 정상적으로 동작한다는 사실을 보장받을 수도 없다.
  * 이러한 **문제를 해결하기 위해서는 단위 테스트를 작성하며, 이는 개발자에게 유용한 테스트에 해당**한다.
* 단위 테스트는 개발자가 작성하되, 일반적으로 다음과 같은 내용을 확인하는 데에 그 목적을 둔다.
  1. happy path: 일반적인 유즈케이스이며, 테스트 대상이 사용될 일반적인 목적을 테스트한다.
  2. 일반적인 오류 또는 잠재적인 문제: **테스트 대상이 제대로 동작하지 않을 것처럼 보이는 부분, 또는 과거에 이미 문제가 발생했던 부분을 테스트**한다.
  3. edge case: **Int형 변수에 대한 `Int.MAX_VALUE`나 nullable 참조에 대한 null 전달 등을 테스트하는 데에 목적**을 둔다.
* 이렇게 작성된 단위 테스트는 개발자가 작성 중인 대상의 동작성을 빠르게 피드백해주며, 계속해서 축적되어 다음과 같은 이점을 가져다준다.
  1. 테스트가 잘 된 대상을 신뢰할 수 있도록 만들어주므로 자신감을 불어넣는다.
  2. 거침없이 리팩토링할 수 있으므로, 테스트가 잘 준비된 애플리케이션은 코드 수준이 점점 발전한다.
  3. **수동 테스트보다 더 빠르며, 이를 토대로 코드의 작성과 테스트의 반복인 피드백 루트가 빨라져 버그 발견 속도와 개발 자체의 재미를 높여준다**.
* 그러나 단위 테스트는 다음과 같은 종류의 단점을 갖기도 하며, 단위 테스트 작성 시 반드시 이 점을 유념해야 한다.
  1. 단위 테스트는 장기적으로 디버깅 시간을 줄여주지만, 초기에는 그만큼 더 많은 시간을 소모한다.
  2. **테스트 대상에 대해 단위 테스트를 수행할 수 있도록 코드를 조정**해야 하며, 이를 통해 더 좋은 아키텍쳐를 적용하는 것이 강제될 수 있다.
  3. **좋은 단위 테스트를 만드는 작업 자체가 어려운 축에 속하며, 남은 개발 과정에 대한 확실한 이해를 요구하는 성향**이 있다.
* 때문에 **잘못 만들어진 단위 테스트는 득보다 실이 더 크며, 이러한 상황을 방지하기 위해서는 반드시 올바른 단위 테스트 방식을 익혀야 한다**.

### 단위 테스트 대상
* 일반적으로 다음과 같은 코드는 중요한 축에 속하므로, 반드시 단위 테스트를 적용하는 방법을 알고 있어야 한다.
  1. 내용 자체가 복잡한 경우
  2. 계속해서 수정되거나 리팩토링될 만한 경우
  3. 비즈니스 로직 자체를 포함하는 경우
  4. 공용으로 사용되는 API인 경우
  5. 버그가 자주 발생하는 부분인 경우
  6. 운영 상에서 버그가 발생하여, 반드시 수정해야 하는 경우
* **상술한 모든 내용을 통해 안정적이고 올바르게 동작하는 애플리케이션을 작성할 수 있으나, 가장 중요한 것은 정말로 올바르게 동작하는지 확인하는 데에 있다**.
  * 이것이 테스트이며, 그 중에서도 개발 과정에서 가장 효율적으로 활용할 수 있는 테스트는 단위 테스트에 해당한다.