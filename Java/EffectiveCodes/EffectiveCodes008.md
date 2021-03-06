# EffectiveCodes
## 2022-03-12 Sat

## 일반적인 원칙
### 지역변수의 범위는 가능한 한 최소화하기
* **지역변수의 범위를 최소화하면 코드 가독성과 유지보수성은 향상되고, 잠재적인 오류 발생 가능성은 낮아진다**.
* **지역변수의 범위를 줄이는 가장 강력한 방법은 처음 사용될 때 선언하기**이다.
  * **사용하기까지 한참 남은 변수를 미리 선언해두면 코드가 어수선해지고, 변수를 실제로 사용하는 시점에서 타입과 초기값을 잊을 수도** 있다.
* 거의 모든 지역변수는 선언과 동시에 초기화해야 한다.
  * **초기화에 필요한 정보가 충분하지 않다면, 충분해질 때까지 선언을 미뤄야** 한다.
  * 예외적으로, **변수를 초기화하는 표현식에서 검사 예외가 발생할 수 있다면 예외가 메소드 전체에 전파되지 않도록 try 블록 안에서 초기화**한다.
* 부득이하게 try 블록 밖에서도 사용해야 하는 변수 값은 try 블록 밖에서 선언하고 블록 내부에서 초기화한다.
* for 문에서는 반복 변수의 값이 적용되는 범위가 for 키워드와 몸체 내부의 괄호로 재한된다.
  * **따라서 반복 변수의 값을 반복문 종료 후에도 사용해야하는 경우가 아니라면 while 보다는 for 문을 사용**한다.
* **지역변수의 범위를 최소화하는 또 하나의 방법은 메소드를 가능한 한 작게 유지하고, 하나의 기능에만 집중하도록 정의**하는 것이다.
  * 만약 메소드가 여러 기능을 처리한다면, 단순히 메소드를 기능 별로 쪼개도록 한다.

### 전통적인 for 문보다는 향상된 for 문을 사용하기
* 컬렉션을 예로 들어, 반복자나 전통적인 for 문을 활용하는 방식은 인덱스 변수와 반복자를 불필요하게 사용하게 된다.
  * **컬렉션 순회에서 개발자가 정말 원하는 것은 원소의 값 뿐이기 때문**이다.
  * 또한, **매 반복마다 반복자 또는 인덱스를 여러 번 접근해야 하므로 컴파일러가 잡아주지 못하는 잠재적인 오류 발생 가능성이 크다**.
  * 마지막으로, 처리 대상의 종류가 컬렉션과 배열 중 어느 것인지에 따라 코드 형태가 달라진다.
* **이러한 문제는 향상된 for 문을 사용하여 모두 해결**할 수 있다.
  * 반복자와 인덱스를 사용하지 않으므로 가독성이 높아지고, 오류도 발생하지 않는다.
  * 또한, 컬렉션과 배열 어떤 컨테이너를 사용하더라도 같은 문법을 사용하므로 간단하다.
* **향상된 for 문을 사용하더라도 순회 속도는 유지**된다.
* 그러나 **향상된 for 문을 사용할 수 없는 경우도 다음과 같이 분명히 존재**한다.
  1. 컬렉션을 순회하면서 선택한 원소를 제거해야 하는 경우
     * Java 8부터는 Collection의 removeIf를 활용하여 컬렉션을 명시적으로 순회하지 않아도 된다.
  2. 리스트나 배열을 순회하며, 원소의 값 일부 또는 전체를 교체해야 하는 경우에는 반복자나 인덱스를 활용해야 한다.
  3. 여러 컬렉션을 병렬로 순회해야 하는 경우, 각 컬렉션의 반복자나 인덱스 변수를 활용하여 엄격하고 명시적으로 제어해야 한다.
  * 상술한 세 가지 경우에 속하는 경우, 고전적인 for 문을 사용하되 고전적인 for 문의 단점을 경계해야 한다.
* **향상된 for 문은 컬렉션과 배열은 물론이거니와, Iterable 인터페이스를 구현한 객체는 무엇이든 순회가 가능**하다.
  * Iterable 인터페이스는 객체의 원소를 순회하는 반복자를 반환하는 메소드인 iterator 만이 포함된다.

### 향상된 for 문의 결론
* 전통적인 for 문과 비교하여, 향상된 for 문은 가독성이 좋고, 유연하고, 버그 발생 가능성이 낮으며, 성능 저하도 없다.
* **가능한 모든 경우에 전통적인 for 문보다는 향상된 for 문을 적용하는 것이 바람직**하다.

### 라이브러리를 공부하고, 사용하기
* 표준 라이브러리를 활용하는 경우, 다음의 이점을 얻을 수 있다.
  1. **표준 라이브러리의 기능을 사용하면 해당 코드를 작성한 전문가의 지식과, 자신보다 먼저 해당 라이브러리를 사용한 개발자들의 경험을 활용**할 수 있다.
  2. 핵섬적인 업무와 크게 관련 없는 문제를 해결하기 위해 시간을 허비할 필요가 없다.
  3. 별도로 노력을 기울이지 않아도 성능이 지속적으로 개선된다.
  4. 사용할 수 있는 기능이 점점 많아진다.
  5. 표준 라이브러리를 활용하여 작성한 코드가 다른 개발자들에게 낯익은 코드가 되어 유지보수성이 높아진다.
* 이러한 장점에도 불구하고, 많은 개발자들은 라이브러리에 어떤 기능이 존재하는지 몰라 기능을 직접 구현하여 사용하고 있다.
* 메이저 릴리즈마다 주목할만한 새로운 기능들이 라이브러리에 추가되며, 개발자는 이를 능동적으로 확인해볼 필요가 있다.
* 라이브러리가 너무 방대하여 모든 기능을 공부하기 어렵겠지만, Java 개발자라면 적어도 다음과 같은 라이브러리에 익숙해져야 한다.
  1. java.lang
  2. java.util
  3. java.io
  4. java.util.concurrent
  5. 컬렉션 프레임워크
  6. 스트림 API
* 이외의 라이브러리들은 필요한 시점에 익히는 것으로 충분하다.
* 라이브러리가 필요한 기능을 충분히 제공하지 못하는 경우, 다음과 같은 흐름으로 문제 해결을 시도한다.
  1. 우선 표준 라이브러리를 사용하여 문제를 해결해본다.
  2. 불가능한 경우, 고품질의 서드 파티 라이브러리를 활용하여 문제를 해결해본다.
  3. 불가능한 경우, 문제 해결 방법을 직접 구현한다.

### 라이브러리 사용의 결론
* 바퀴의 재발명을 하지 말아야 한다.
* 특별한 나만의 기능이 아니라면 누군가는 이미 라이브러리에 추가해두었을 것이며, 이러한 코드는 우리의 코드보다 품질이 좋고 지속적으로 개선된다.
* 코드 품질에도 규모의 경제가 적용되며, 라이브러리 코드는 많은 개발자들에게 주목을 받으므로 코드 품질이 높아질 수 밖에 없다.

### 정확한 답이 필요한 경우에는 float과 double을 지양하기
* float과 double 타입은 넓은 범위의 수를 빠르게 근사치로 계산하도록 설계된, 과학과 공학 계산용 타입이다.
* 따라서 **정확한 결과가 필요한 경우에는 사용하지 말아야 하며, 특히 금융 관련 계산과는 잘 맞지 않는다**.
  * **금융 계산에는 BigDecimal, int, long 등의 타입을 사용**해야 한다.
* BigDecimal을 활용하면 정상적인 결과가 나오지만, 다음과 같은 단점도 존재한다.
  1. 기본 타입보다 훨씬 사용하기 불편하다.
  2. 기본 타입보다 훨씬 느리다.
* 때문에 BigDecimal 대신 int와 long 타입의 사용을 고려할 수 있다.
  * 이 경우, 다룰 수 있는 값의 크기가 제한되고 소수점을 직접 관리해야 하는 단점이 있다.

### float과 double의 결론
* 정확한 답이 필요한 계산에는 float과 double을 사용하지 않아야 한다.
* 소수점 추적은 시스템에 맡기고, 개발 과정에서의 불편함과 성능 저하를 신경쓰지 않아도 좋다면 BigDecimal을 활용한다.
* 성능이 중요하고, 소수점 추척을 직접 수행하며 관리해야하는 수의 범위가 너무 크지 않다면 int와 long을 사용한다.
  * 관리해야 하는 수의 범위가 int와 long의 범위보다 큰 경우에는 BigDecimal을 사용해야만 한다.

## 2022-03-13 Sun
### 박싱된 기본 타입보다는 기본 타입을 사용하기
* Java의 데이터 타입은 크게 기본 타입과 참조 타입으로 나뉘며, 모든 기본 타입에는 대응되는 참조 타입이 존재한다.
  * 이를 박싱된 기본 타입이라고 한다.
* **오토박싱과 오토언박싱 기능 덕분에 둘을 구분하지 않고 사용할 수 있지만, 둘 사이의 차이가 전혀 존재하지 않는다는 의미는 아니다**.
  * 오토박싱이 번거로움을 줄여주지만, 위험성까지 모두 없애주지는 못한다.
* 기본 타입과 박싱된 기본 타입의 주효한 차이점은 다음과 같다.
  1. 기본 타입은 값만 갖지만, 박싱된 기본 타입은 값과 식별성을 함께 갖는다.
     * 즉, 박싱된 기본 타입의 두 인스턴스는 값이 같음에도 다르다고 식별될 가능성이 있다.
  2. 기본 타입은 언제나 유효한 값을 갖지만, 박싱된 기본 타입은 null을 가질 수 있다.
  3. 기본 타입이 박싱된 기본 타입보다 시간과 메모리 측면에서 더 효율적이다.
* 이러한 차이점이 존재하기 때문에, 두 타입을 구분하지 않고 사용한다면 다음과 같은 문제가 발생할 수도 있다.
  1. 두 인스턴스의 식별성 비교에서 같은 값이어도 false가 반환될 수 있다.
  2. 두 타입을 혼용하는 경우, 연산 과정에서 박싱된 기본 타입의 언박싱 결과가 null인 경우 NPE가 발생할 수 있다.
  3. 기본 타입을 박싱 / 언박싱하는 과정에서 성능이 떨어질 수 있다.
* 개발자가 기본 타입과 박싱된 기본 타입의 차이를 무시하면 오류나, 심각한 성능 문제가 발생할 수 있다.
* **기본적으로는 기본 타입을 사용하되, 다음과 같은 경우에 박싱된 기본 타입을 사용**한다.
  1. 컬렉션의 원소, 키, 값으로는 기본 타입을 사용할 수 없으므로 박싱된 기본 타입을 사용한다.
  2. 매개변수화 타입이나 매개변수화 메소드에는 기본 타입을 사용할 수 없으므로, 박싱된 기본 타입을 사용한다.
  3. 리플렉션을 통해 메소드를 호출하는 경우, 박싱된 기본 타입을 사용해야 한다.

### 다른 타입이 적절한 경우에는 문자열 사용을 지양하기
* 문자열은 텍스트를 잘 표현하지만, 원래 의도와 다른 용도로 사용되는 경향이 있다.
* 다음과 같은 경우, 문자열 대신 적합한 대체제를 사용하여야 한다.
  1. 문자열은 다른 값 타입을 대체하기에는 적절하지 않다.
  2. 문자열은 상수 표현에 있어서 열거 타입을 대체하기에는 적절하지 않다.
  3. 문자열은 혼합된 타입을 대체하기에는 적절하지 않다.
     * 차라리 전용 타입의 클래스를 새로 설계하여 작성하는 것이 낫다.
  4. 문자열은 권한을 표현하기에는 적절하지 않다.
* 더 적절한 데이터 타입이 있거나, 새로운 클래스를 작성하여 대체할 수 있는 경우에는 문자열을 사용하지 말아야 한다.
* **문자열을 기본 타입, 열거 타입, 혼합 타입을 대체하여 잘못 사용하는 경우에는 사용성이 어렵고, 유연하지 않고, 느리고, 잠재적인 오류 발생 가능성이 크다**.

### 문자열 연결은 주의해서 사용하기
* 덧셈 연산자인 + 는 문자열 연결 연산자로도 사용되며, 여러 문자열을 하나로 합치기 위해 편리하게 사용할 수 있다.
* 그러나 문자열 연결 연산자는 문자열 n 개에 대해 n의 제곱에 비례하며, 많이 사용할수록 성능 저하가 뒤따를 수 밖에 없다.
  * **이는 문자열이 불변인 특성을 갖기 때문으로, 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사하는 식으로 동작하기 때문**이다.
* **성능에 민감한 경우, 문자열을 직접 연결하기보다는 적절한 크기로 초기화된 StringBuilder를 사용하는 것이 바람직**하다.
  * 적절한 크기로 초기화하지 않고 기본 값을 사용하더라도 여전히 단순한 연결 연산자를 사용한 경우보다 훨씬 빠르다.

### 객체는 되도록 인터페이스로 참조하기
* **적절한 인터페이스가 있다면, 매개변수와 반환값, 변수, 필드 모두 인터페이스 타입으로 선언하는 것이 좋다**.
  * **객체의 실제 클래스를 사용해야하는 경우는 오직 생성자로 생성할 때 뿐임을 기억**하자.
* **인터페이스를 타입으로 사용하는 습관을 길러두면 프로그램은 훨씬 유연**해진다.
  * 예를 들어, 추후 실제 구현 클래스를 교체해야하는 경우가 생기더라도 생성자 또는 정적 팩토리 코드만 교체해주면 된다.
  * 주변 코드는 교체 이전의 구현 클래스의 실체에 대해 전혀 알지 못하므로, 이러한 변화에 아무런 영향을 받지 않는다.
* 만약 기존 구현 클래스가 인터페이스의 규약 이외의 기능을 제공하고, 주변 코드가 해당 기능에 의존하는 경우 새로운 클래스도 같은 기능을 제공해야 한다.
  * 덧붙여, 실제 구현 타입을 변경하려는 이유는 일반적으로 다음과 같다.
  1. 기존 구현 타입보다 더 성능이 좋기 때문.
  2. 또는 기존 구현 타입이 제공하지 못하는 새로운 기능을 제공하기 때문.
* 적절한 인터페이스가 없는 경우는 다음과 같이 나누어볼 수 있으며, 이들은 당연히 클래스로 참조해야 한다.
  1. 값 클래스.
  2. 클래스 기반으로 작성된 프레임워크가 제공하는 객체들.
  3. 인터페이스에는 없는 메소드를 제공하는 클래스들.
* **값 클래스는 여러 형태로 구현될 수 있다고 가정하고 설계하는 경우가 거의 없다**.
  * 따라서 값 클래스는 대부분 final이며, 대응되는 인터페이스가 존재하지 않는 경우가 많다.
  * 이러한 **값 클래스는 인터페이스 없이 매개변수, 변수, 필드, 반환값으로 사용해도 무방**하다.
* OutputStream 등 java.io 패키지의 클래스들은 클래스 기반으로 작성된 프레임워크가 제공하는 객체들의 예시이다.
  * **이 경우에도 가능하다면 특정한 구현 클래스보다는 추상 클래스와 같은 기반 클래스로 참조하는 것이 바람직**하다.
* PriorityQueue와 같은 클래스는 Queue 인터페이스에는 존재하지 않는 comparator 메소드를 제공한다.
  * 인터페이스 대신 클래스 타입을 직접 사용하는 경우는 이러한 추가 메소드를 사용해야만 하는 경우로 최소화한다.
  * 이는 절대 남발하지 말아야 하는 경우에 해당한다.
* 상술한 경우는 인터페이스 대신 구현 클래스 타입을 사용해도 되는 예시이며, 모든 상황을 설명하지는 못한다.
* **적절한 인터페이스가 없는 경우에는 클래스 계층구조에서 필요한 기능을 만족하는 가장 추상적인 클래스를 타입으로 사용하는 것이 바람직**하다.

### 리플렉션보다는 인터페이스를 사용하기
* 리플렉션을 가용하면 프로그램에서 임의의 클래스에 접근할 수 있다.
* 그러나 이러한 리플렉션은 다음과 같은 단점을 갖는다.
  1. 컴파일 시점의 타입 검사가 주는 이점을 전혀 누릴 수 없다.
  2. 코드가 지저분하고 장황해져 가독성이 떨어진다.
  3. 성능이 훨씬 떨어진다.
* 리플렉션은 아주 제한된 형태로만 사용해야 단점을 피하고 이점을 누릴 수 있다.
* **리플렉션은 인스턴스 생성에만 사용하도록 하고, 이렇게 생성된 인스턴스는 인터페이스나 상위 클래스로 참조하여 사용하도록 해야 한다**.
* 드물지만, 리플렉션은 런타임에 존재하지 않을 수도 있는 클래스, 메소드, 필드 의존성을 관리할 때 적합하다.
  * 이러한 요구사항은 버전이 여럿 존재하는 외부 패키지를 다룰 때 유용하다.
  * 예를 들어, 동작 가능한 최소한의 환경을 갖는 오래된 버전을 지원하도록 컴파일한 후 이후 버전의 클래스 등은 리플렉션으로 접근할 수 있다.

### 리플렉션의 결론
* 리플렉션은 복잡하고 특수한 시스템을 개발할 때 필요한 강력한 기능이지만, 수반되는 단점도 많다.
* 리플렉션은 되도록 객체 생성에만 사용하고, 생성된 객체를 사용할 때에는 컴파일 시점에 알 수 있는 적절한 상위 타입으로 형변환하여 사용한다.

### 네이티브 메소드는 신중하게 사용하기
* JNI는 Java 프로그램이 네이티브 메소드를 호출할 수 있도록 제공되는 기능이다.
  * 네이티브 메소드란, C나 C++ 등의 네이티브 프로그래밍 언어로 작성된 기능이다.
* 일반적으로 네이티브 메소드의 쓰임은 다음과 같은 세 가지의 경우이다.
  1. 레지스트리 등의 플랫폼 특화 기능을 사용하는 경우.
  2. 네이티브 코드로 작성된 기존 라이브러리를 사용하는 경우.
  3. 성능 개선을 목적으로 성능에 큰 영향을 주는 기능을 네이티브 언어로 작성하는 경우.
* 플랫폼 특화 기능을 사용하는 경우 네이티브 메소드를 활용해야 하지만, Java가 성숙해짐에 따라 하부 플랫폼의 기능을 점차 흡수하고 있다.
  * 따라서, 해당 요구사항에서 네이티브 메소드의 필요성은 점차 줄어들고 있다.
* 대체할만한 Java 라이브러리가 없는 네이티브 라이브러리는 여전히 네이티브 코드를 사용해야 한다.
* 성능을 개선할 목적으로 사용하는 네이티브 메소드는 일반적으로 권장되지 않는다.
  * Java 초기라면 이러한 요구사항에 대처할 수 없었으나, JVM은 엄청난 속도로 발전해왔다.
  * 현재의 Java는 다른 플랫폼에도 견줄만한 성능을 보여줄 수 있다.
* 네이티브 메소드는 안전하지 않고, 이식성이 낮고, 디버깅이 어렵고, 자동으로 GC되지 않으며, 접착 코드를 필수로 작성해야 하므로 가독성도 떨어진다.
  * 즉, 네이티브 메소드가 성능을 개선해주는 일은 많지 않지만 수반되는 단점은 많다.
  * 어쩔수 없는 경우에도 네이티브 코드는 최소한으로만 사용하고, 철저히 테스트해야 한다.

## 2022-03-14 Mon
### 최적화는 신중하게 진행하기
* **최적화는 좋은 결과보다는 해로운 결과를 낳기 쉽고, 섣불리 진행할수록 더욱 그렇다**.
* 성능 때문에 견고히 설계된 구조를 희생하지 않아야 한다.
  * **빠른 프로그램보다는 좋은 프로그램을 작성하도록 해야 한다**.
* **좋은 프로그램이지만 성능이 떨어진다면, 아키텍쳐 자체가 최적화할 수 있는 길을 가이드할 수 있다**.
  * 좋은 프로그램은 정보 은닉 원칙을 따르므로, 개별 구성 요소의 내부를 독립적으로 설계할 수 있다.
  * 즉, 시스템의 나머지에 영향을 주지 않고도 각 요소를 재설계할 수 있다.
* **이는 프로그램이 완성될 때까지 완전히 성능 문제를 무시하라는 의미가 아닌, 설계 단계에서부터 성능을 염두에 둔 설계를 진행해야 한다는 의미**이다.
  * 아키텍쳐적인 결함이 프로그램의 성능을 저하시키는 경우, 전체를 갈아엎지 않고서는 문제를 해결할 수 없는 경우가 발생할 수 있다.
* 아키텍쳐 설계 단계에서 반드시 다음의 내용을 고려해야 한다.
  1. 성능을 제한하는 설계는 지양한다.
     * 프로그램의 완성 후 변경하기 가장 어려운 설계 요소는 컴포넌트 간, 또는 외부 시스템괴의 소통 방식이다.
     * 예를 들어, API, 프로토콜, 데이터 저장 형식 등이 있다.
  2. API 설계시 성능에 주는 영향을 고려한다.
     * API의 설계가 성능에 주는 영향은 다분히 현실적이며, 일반적으로 잘 설계된 API는 성능도 좋다.
     * 때문에 성능을 위해 API를 왜곡하는 것은 바람직하지 않은 접근 방식이다.
* **신중하게 설계하여 멋진 구조를 갖춘 프로그램을 완성한 후, 성능이 만족스럽지 못한다면 그제서야 최적화를 고려**할 수 있다.
* 각각의 최적화 시도 전후로 반드시 성능을 측정해야 한다.
  * 시도한 최적화 기법이 성능을 높이지 못하는 경우도 많고, 더 나빠지게 하는 경우도 많다.
  * **일반적으로 프로그램 실행 시간중 9할은 1할의 코드에서 사용되므로, 최적화한 코드가 성능에 영향을 주지 않는 부분이라면 시간만 허비**하게 된다.
* 프로파일링 도구 역시 최적화 시도를 어디에 집중해야할 지 찾는데에 도움을 주며, 시스템의 규모가 커질 수록 중요한 역할을 맡는다.
  * 이를 통해 최적화를 시도할 지점과 알고리즘 교체의 필요성을 눈치챌 수 있다.
  * 너무 느린 알고리즘을 사용한 경우, 알고리즘을 교체하는 것으로 다른 최적화를 진행할 필요가 없어지기도 한다.
* Java는 다른 언어에 비해 성능 모델이 덜 정교하며, 기본 연산에 드는 비용을 덜 명확하게 정의한다.
  * 따라서 개발자의 코드와 CPU에서 수행하는 명령 사이의 추상화 격차가 더 크므로 최적화로 인한 성능 변화를 일정하게 예측하기 더 어렵다.
  * 즉, Java에서의 최적화 전 후의 성능 측정 및 비교는 더욱 중요한 절차에 해당한다.
* **프로그램을 여러 Java 플랫폼이나 여러 하드웨어 플랫폼에서 구동하는 경우, 최적화의 효과는 각각의 환경에서 측정**되어야 한다.

### 최적화의 결론
* 빠른 프로그램을 작성하려고 안달할 필요는 없으며, 좋은 프로그램을 작성하다보면 성능은 자연스레 따라온다.
* 그러나 시스템 설계 시 API, 프로토콜, 저장 형식 등을 선택하는 경우에는 성능을 염두에 두어야 한다.
* 프로그램이 완성되었다면 그제서야 성능을 측정하고, 충분히 빠르다면 추가적인 최적화를 수행할 필요가 없다.
* 성능에 만족하지 못하는 경우, 프로퍼일링 도구를 활용하여 문제의 원인을 찾아 최적화를 수행한다.
* 최적화 과정에서는 우선 어떤 알고리즘을 사용했는지 확인하고, 이를 더 빠른 알고리즘으로 교체하는 과정을 거치도록 한다.
  * **애초에 알고리즘의 선택이 잘못되었다면 다른 저수준 최적화는 소용이 없는 경우가 많다**.
* 만족스러운 성능이 나올 때까지 이를 반복하고, 모든 변경 후에는 최적화 전후의 성능을 비교한다.

### 일반적인 명명 규칙을 따르기
* Java 플랫폼은 명명 규칙이 잘 정립되어 있고, 대부분은 Java 언어 명세에 기술되어 있다.
* Java의 명명 규칙은 크게 철자와 문법으로 나뉘며 특별한 이유가 없는 한 반드시 따르는 것이 좋다.
  * 일반적으로 철자 규칙은 직관적이므로 따르기 쉬운데 반해, 문법 규칙은 복잡하고 느슨하다.
* 객체를 생성할 수 있는 클래스는 명사 또는 명사구를 사용하고, 생성할 수 없는 클래스는 복수형 명사로 정의한다.
  * 예를 들어, **Collections, Collectors 등은 객체를 생성할 수 없는 클래스이므로 복수형 명사를 사용**하였다.
* 표준 명명 규칙은 자연스럽게 사용이 가능하도록 체득해야 한다.