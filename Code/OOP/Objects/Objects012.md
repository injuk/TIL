# Objects
## 2022-04-05 Tue

## 다형성
* 앞서 다룬 바와 같이 코드 재사용을 목적으로 상속을 사용하면 변경이 어렵고 유연하지 않은 설계를 얻기 쉽다.
  * 즉, **상속의 목적은 재사용이 아니다**!
* **상속은 타입 계층을 구조화하기 위해서만 사용해야 하며, 이렇게 정의된 타입 계층은 객체지향 프로그래밍의 중요한 특성인 다형성의 기반을 제공**한다.
```
> 상속을 이용한 클래스를 추가하는 경우, 반드시 상속의 용도를 자문하도록 한다.
> 코드 재사용을 목적으로 한다면, 상속을 사용하지 않아야 한다.
> 클라이언트 관점에서 동일하게 행동하는 인스턴스들을 그룹화하기 위해서라면, 상속을 사용해도 무방하다.
```

### 다형성의 이해
* 컴퓨터 과학에서의 다형성이란, 하나의 추상 인터페이스에 코드를 작성하고 해당 추상 인터페이스에 대해 서로 다른 구현을 연결할 수 있는 능력으로 정의된다.
  * 즉, 다형성은 여러 타입을 대상으로 동작할 수 있는 코드를 작성할 수 있는 능력이다.
* 객체지향 프로그래밍에서 사용되는 다형성은 크게 다음과 같은 갈래로 세분화된다.
  1. 유니버설 다형성
     1. 매개변수 다형성
     2. 포함 다형성
  2. 임시 다형성
     1. 오버로딩 다형성
     2. 강제 다형성
* 오버로딩 다형성이란, 하나의 클래스 안에 동일한 이름의 메소드가 여럿 존재하는 경우를 말한다.
* 강제 다형성이란, 언어가 지원하는 자동적인 타입 변환이나 사용자가 구현한 타입 변환을 통해 동일한 연산자를 다양한 타입에 사용할 수 있는 경우를 말한다.
  * Java의 예로 들어, + 연산자는 정수 연산과 문자열이 포함된 연산에서 다르게 동작한다.
* 매개변수 다형성이란 제네릭과 관련이 깊으며, 다양한 타입의 요소를 다루기 위해 동일한 오퍼레이션을 사용하는 경우를 말한다.
  * 클래스의 인스턴스 변수나 메소드 매개변수 타입을 임의의 타입으로 선언한 후, 실제로 사용하는 시점에 구체적인 타입을 지정하는 방식이다.
* 포함 다형성이란 서브타입 다형성으로도 불리우며, 동일한 메시지가 수신 객체에 따라 실제로 수행되는 기능이 달라지는 경우를 말한다.
  * **가장 널리 알려진 형태의 다형성이므로, 일반적으로 다형성이라는 말이 가리키는 것은 포함 다형성**이다.

### 포함 다형성
* 포함 다형성을 구현하는 가장 일반적인 방법은 상속을 활용하는 것이다.
  * 자식 클래스에서 부모 클래스의 메소드를 오버라이딩한 후 클라이언트가 부모 클래스를 참조하도록 한다.
* **포함 다형성은 서브타입 다형성이라고도 불리우며, 이는 포함 다형성을 위한 전제로서 자식 클래스가 부모 클래스의 서브타입이어야 한다는 점을 의미**한다.
  * **상속의 진정한 목적은 코드 재사용이 아닌, 다형성을 위한 서브타입 계층을 구축하는 것**이다!
* 포함 다형성을 위해 상속을 사용하는 가장 큰 이유는 상속이 클래스들을 계층화한 후, 상황에 따라 적절한 메소드를 선택하는 메커니즘을 제공하기 때문이다.

### 상속의 용도
* **객체지향 패러다임의 근간을 이루는 아이디어는 데이터와 행동을 객체라는 하나의 단위로 묶는 것**이다.
  * 즉, 객체지향 프로그램을 작성하려면 항상 데이터와 행동 관점을 함께 고려해야 한다.
* **상속 역시 해당 원칙의 예외가 될 수 없으며, 이에 따라 다음과 같은 목적을 가지리라고 오해**할 수 있다.
  1. 데이터 관점의 상속: 부모 클래스의 모든 데이터를 자식 클래스가 자동으로 포함한다.
  2. 행동 관점의 상속: 부모 클래스에 정의된 일부 메소드를 자식 클래스에 포함시킬 수 있다.
* 데이터와 행동 관점에서만 바라본 상속은 마치 코드 재사용에 목적이 있는 것 처럼 보이지만, 이 경우 이해하기 어렵고 유지보수성이 떨어지는 코드를 만들기 쉽다.
```
> 상속은 프로그램을 구성하는 개념들을 기반으로 다형성을 가능케하는 타입 계층을 구축하기 위해 존재하는 기능이다.
```

### 데이터 관점의 상속
* **개념적으로 자식 클래스를 인스턴스화했을 경우, 인스턴스 내부에 부모 클래스의 인스턴스가 포함되는 식으로 생각하는 것이 유용**하다.
  * 실제로 객체를 메모리에 생성하는 방식이나 구조는 언어나 실행 환경에 따라 다르다!
* 인스턴스를 참조하는 변수는 자식 클래스를 가리키므로, 특별한 방법을 사용하지 않으면 자식 클래스 내부에 포함된 부모 클래스의 인스턴스 직접 접근할 수는 없다.
* 이렇듯 **데이터 관점에서의 상속은 자식 클래스의 인스턴스 안에 부모 클래스의 인스턴스를 포함하는 것**이다.
  * 때문에 자식 클래스의 인스턴스는 자동으로 부모 클래스에 정의된 모든 인스턴스 변수를 내부에 포함하게 된다.

### 행동 관점의 상속
* 상술했듯, 데이터 관점의 상속은 자식 클래스의 인스턴스 안에 부모 클래스의 인스턴스를 포함하는 개념이다.
* 그러나 **행동 관점의 상속은 부모 클래스가 정의한 일부 메소드를 자식 클래스의 메소드에 포함시키는 것을 의미**한다. 
* 일반적으로 부모 클래스의 모든 퍼블릭 메소드는 자식 클래스의 퍼블릭 인터페이스에 포함된다.
  * 때문에 외부의 객체가 부모 클래스의 인스턴스에 전송할 수 있는 모든 메시지는 자식 클래스의 인스턴스에도 전송할 수 있다.
* 부모 클래스에 정의되어 있지만 자식 클래스에는 정의하지 않은 메소드를 호출하는 경우, 호출 대상 메소드는 런타임에 결정된다.
  * **자식 클래스에 정의되지 않은 메소드는 시스템이 런타임에 부모 클래스 안에서 탐색하는 식으로 동작**한다.
* 객체의 경우 서로 다른 상태를 저장할 수 있도록 인스턴스 별로 독립된 메모리 공간을 할당받아야만 한다.
* 반면 **메소드의 경우, 동일한 클래스의 인스턴스끼리 공유가 가능하므로 클래스는 하나만 메모리에 로드**한다.
  * 이후 **각 인스턴스는 클래스를 가리키는 포인터를 갖는다**.
* **각 객체는 자신의 클래스의 위치를 가리키는 class라는 이름의 포인터를 갖고, 이 포인터를 활용하여 자신의 클래스 정보에 접근할 수 있다**.
* 또한 각 클래스는 자신의 부모를 가리키는 parent라는 이름의 포인터도 가지며, 이를 통해 클래스의 상속 계층에 따라 부모 클래스의 정의로 이동할 수 있다.
* 상술한 정보를 토대로, 자식 클래스의 인스턴스를 통해 부모 클래스에 정의된 메소드를 호출할 경우 다음과 같은 순서로 실행된다.
  1. **메시지를 수신한 객체는 class 포인터로 연결된 자신의 클래스에서 적절한 메소드를 찾는다**.
  2. **메소드가 존재하지 않는 경우, 자신의 클래스에 포함된 parent 포인터를 활용하여 부모 클래스의 메소드를 차례로 탐색**한다.
     * 이러한 탐색 흐름은 최상위 부모 클래스인 Object에 이르기까지 반복된다.
  3. 적절한 메소드가 있는 경우, 실행한다.

### 업캐스팅과 동적 바인딩
* 코드 안에 선언된 참조 타입과 무관하게 실제로 메시지를 수신하는 객체의 타입에 따라 실행되는 메소드는 달라질 수 있다.
  * 이러한 동작을 뒷받침하는 것은 업캐스팅과 동적 바인딩이다.
* 업캐스팅이란, 부모 클래스 타입으로 선언된 변수에 자식 클래스의 인스턴스를 할당하는 것이다.
* 동적 바인딩이란, 선언된 변수의 타입이 아닌 실제로 메시지를 수신하는 객체의 타입에 따라 실행되는 메소드가 결정되는 것이다.
  * 이는 **객체지향 시스템이 메시지를 처리할 메소드를 컴파일타임이 아닌 런타임에서 결정하기 때문에 가능**하다.
* 업캐스팅은 서로 다른 클래스의 인스턴스를 같은 타입에 할당하는 것을 가능케하므로, 부모 클래스에 작성된 코드를 수정하지 않고도 자식 클래스에 적용할 수 있다.
* **동적 메소드 탐색은 부모 클래스의 타입에 대해 메시지를 전송했더라도, 실행 시에는 실제 클래스를 기반으로 실행될 메소드가 선택되게 한다**.
  * 따라서, 코드를 수정하지 않고도 실행되는 메소드를 변경할 수 있다.

### 업캐스팅
* 상속을 이용하면 부모 클래스의 퍼블릭 인터페이스가 자식 클래스와 합쳐지므로, 부모 인스턴스에게 전달할 수 있는 메시지는 자식에게도 절달할 수 있다.
* 컴파일러는 명시적인 타입 변환 없이 자식 클래스가 부모 클래스를 대체할 수 있도록 허용한다.
  * 이러한 특징을 활용할 수 있는 대표적인 두 방식은 대입문과 메소드의 파라미터 타입이다.
  * 이렇듯 **모든 객체지향 언어는 명시적인 타입 변환 없이도 부모 클래스 타입의 참조 변수에 자식 클래스의 인스턴스를 대입할 수 있도록 한다**.
* 반대로 부모 클래스의 인스턴스를 자식 클래스 타입으로 변경하기 위해서는 명시적인 타입 캐스팅이 필요하며, 이는 다운캐스팅에 해당한다.
* 컴파일러 관점에서, 자식 클래스는 아무런 제약 없이 부모 클래스를 대체할 수 있다.
  * 따라서 부모 클래스와 협력하는 클라이언트는 다양한 자식 클래스의 인스턴스와 협력할 수 있다.
  * **여기서 자식 클래스란, 현자 상속 계층에 포함된 클래스 뿐만 아니라 앞으로 추가될지도 모르는 미래의 자식 클래스까지 포함**한다.
* 이러한 특징 덕에 **업캐스팅은 어떠한 자식 클래스와도 협력을 가능케하는 무한한 확장 가능성을 갖고, 따라서 설계는 유연하고 확장 가능**하다.

### 동적 바인딩
* 전통적인 언어들은 코드를 작성하는 시점에 호출될 함수가 결정되므로, 컴파일타임에 함수를 결정한다.
  * 이렇듯 컴파일타임에 호출할 수 있는 함수를 결정하는 방식은 정적 바인딩 또는 컴파일타임 바인딩이라고 부른다.
* 반면 객체지향 언어에서는 메시지를 수신했을 때 실행될 메소드가 런타임에 결정된다.
  * 이렇듯 실행될 메소드가 런타임에 결정되는 방식을 동적 바인딩 또는 지연 바인딩이라고 부른다.
* **객체지향 언어가 제공하는 업캐스팅과 동적 바인딩을 활용하면 부모 클래스 참조에 대한 메시지 전송을 자식 클래스에 대한 메소드 호출로 변환**할 수 있다.

## 2022-04-06 Wed
### 동적 메소드 탐색
* 객체지향 시스템은 다음과 같은 순서에 따라 실행할 메소드를 찾는데 성공하여 탐색을 종료할 때까지 참조를 계속한다.
  1. 메시지를 수신한 객체는 먼저 자신을 생성한 클래스에 적합한 메소드가 존재하는지 검사한다.
     * **메소드를 찾는 데에 성공했다면 메소드를 실행한 후 탐색을 종료**한다.
  2. 메소드를 찾지 못한 경우, 바로 위의 부모 클래스에서 탐색을 계속한다.
  3. 적합한 메소드를 발견할 때까지 탐색을 게속한다.
  4. 상속 계층의 최상위 클래스에서도 메소드를 발견하지 못한 경우, 예외를 발생시키며 탐색을 중단한다.
* **객체가 메시지를 수신하면 컴파일러는 self 참조라는 임시 변수를 자동으로 생성한 후, 메시지를 수신한 객체를 가리키도록 설정**한다. 
  * **self 참조는 메시지 탐색과 관련한 중요한 변수이며, Java에서의 this에 해당**한다.
* **동적 메소드 탐색은 self가 가리키는 객체의 클래스에서 시작하며, 상속의 역방향으로 탐색을 진행한 후 탐색이 종료되었을 때 self가 자동으로 소멸**된다.
  * **역방향으로 진행하므로, 항상 자식 클래스에 선언된 메소드가 부모 클래스에 선언된 메소드보다 높은 우선순위**를 갖는다.
    * 메소드 오버라이딩은 이러한 특징 탓에 벌어지는 현상이다.
  * 이렇듯 시스템은 class, parent 포인터와 self 참조를 조합하여 메소드를 탐색해나간다.
* 동적 메소드 탐색은 다음과 같은 두 가지 원리로 구성된다.
  1. 자동 메시지 위임: 자식 클래스는 자신이 이해하지 못하는 메시지의 처리를 자동으로 상속 계층을 따라 부모에게 위임한다.
     * 해당 **위임 과정은 개발자의 개입 없이 상속 계층을 따라 자동으로 진행**된다.
  2. 동적 문맥 사용: **메시지를 수신한 경우 실제로 어떤 메소드를 실행할지 결정하는 것은 컴파일타임이 아닌 런타임**이다.
     * 이 경우, 메소드를 탐색하는 경로는 self 참조를 이용하여 결정한다.
* 때문에 메시지가 처리되는 문맥을 이해하는 것은 정적인 코드를 분석하는 것만으로 충분하지 않다.
  * **런타임에 실제로 메시지를 수신한 객체가 어떤 타입인지 추적해야하며, 객체의 타입에 따라 메소드 탐색 문맥이 동적으로 결정**된다. 
  * 또한, 이러한 탐색 과정에서 self 참조를 사용하게 된다.
* **동적 메소드 탐색에서 적절한 메소드를 찾는 기준은 메소드 시그니처이므로, 메소드의 이름이 같더라도 시그니쳐가 다르다면 동적 탐색의 대상이 되지 않는다**.
  * 이러한 특징은 메소드 오버로딩을 가능케한다.
* 동적 메소드 탐색 규칙은 언어마다 다를 수 있다. 
  * 예를 들어, C++은 같은 클래스의 오버로딩은 허용하지만 부모 클래스의 메소드는 기본적으로 오버로딩할 수 없다.
  * C++의 자식 클래스에서 부모 클래스의 메소드를 오버로딩하고자 하는 경우, 특수한 방법을 사용해야 한다.
```
> 메소드 오버라이딩은 메소드를 감추지만, 메소드 오버로딩은 함께 공존한다.
```

### 자동적인 메시지 위임
* **동적 메소드 탐색 관점에서, 상속 계층은 메시지를 수신한 객체가 이해할 수 없는 메시지의 처리를 부모 클래스에게 위임하기 위한 물리적인 경로를 정의**한다.
  * 상속을 이용하는 경우 개발자는 메시지 위임과 관련된 코드를 명시적으로 작성할 필요가 없으며, 메시지는 상속 계층을 따라 부모 클래스에게 자동으로 위임된다.
  * 즉, 상속 계층을 정의하는 것은 메소드 탐색 경로를 정의하는 것과 같다.
* Java의 경우 메시지를 위임하는 과정에서 상속을 사용하지만, 자동적인 메시지 위임은 프로그래밍 언어마다 다르게 제공될 수 있다.

### 동적인 문맥과 self 전송
* 상술한 바와 같이, 메시지 전송 코드만으로는 어떤 클래스의 어떤 메소드가 실행될지 알 수 없다.
* **중요한 것은 메시지를 수신한 객체가 무엇인지에 따라 메소드 탐색을 위한 문맥이 동적으로 바뀐다는 점**이다.
  * 이 때, 이 **동적인 문맥을 결정하는 것이 메시지를 수신한 객체를 가리키는 self 참조**이다.
* 동일한 코드더라도 self 참조가 가리키는 객체의 실체가 무엇인지에 따라 메소드 탐색을 위한 상속 계층의 범위는 동적으로 변할 수 있다.
  * 즉, **self 참조가 가리키는 객체의 타입을 변경하는 것으로 객체가 실행될 문맥 역시 동적으로 바꿀 수 있다**. 
* **객체가 메소드를 호출하는 과정에서 자신의 또 다른 메소드를 호출하기 위해 self 참조에게 메시지를 전송하는 것을 self 전송**이라고 한다.
  * **self 전송은 단순히 자신의 메소드를 호출하는 것이 아니며, 실제로는 self 참조에게 새로운 메시지를 전송하여 동적 탐색을 다시 시작하는 것**이다.
  * 즉, **자신의 메소드를 호출한 시점에서 self가 가리키는 인스턴스인 자신으로부터 동적 메소드 탐색을 다시 시작**된다.
* self 전송은 동적 탐색을 self 참조로부터 다시 시작하게 한다.
  * 극단적인 경우, **self 전송이 깊은 상속 계층과 계층 중간 중간에 숨겨진 메소드 오버라이딩과 조합되는 경우 이해하기 어려운 코드가 만들어질 수도 있다**.

### 이해할 수 없는 메시지의 처리
* 객체가 메시지를 이해할 수 없는 경우에 대한 대응은 프로그래밍 언어의 종류에 따라 다음과 같이 분류된다.
  1. 정적 타입 언어의 경우, 컴파일타임에 상속 계층을 모두 순회하여 이해할 수 없는 메시지는 컴파일 에러로 처리한다.
  2. **동적 타입 언어의 경우, 컴파일 단계가 존재하지 않으므로 실제로 수행해보기 전에는 메시지의 처리 가능성을 판단할 수 없다**.
* Java는 대표적인 정적 타입 언어이므로 컴파일타임에 동적 메소드 탐색을 진행하고, 이해할 수 없는 메소드는 컴파일 에러를 발생시켜 개발자에게 알린다.
* 동적 타입 언어의 경우 런타임에 동적 메소드 탐색을 진행하며, 이해할 수 없는 메소드의 경우에 대응되는 새로운 메시지를 전송한다.
  * 예를 들어, **Ruby의 경우 메소드 탐색에 실패한 경우 self.method_missing 메시지를 전송**한다.
  * 개발자는 method_missing 메시지에 응답할 수 있는 메소드를 구현할 수 있으며, 이를 통해 순수한 관점에서 객체지향 패러다임을 구현할 수 있다.
* 정적 타입 언어는 다소 유연성이 떨어지는 대신, 컴파일타임에 메소드 탐색을 수행하는 것으로 안정성을 얻을 수 있다.
  * 즉, 컴파일타임에 오류가 발생할 수 있는 가능성을 체크하는 것으로 런타임 에러의 발생 가능성을 낮춘다.
* 동적 타입 언어는 이해할 수 없는 메시지를 처리할 수 있는 능력을 가짐으로써 메시지가 선언된 인터페이스와 메소드가 정의된 구현을 분리할 수 있다.
  * 그러나 이러한 동적 타입 언어의 유연성과 동적인 특성은 코드를 이해하고 수정하는 과정과 디버깅을 복잡하게 만들 수도 있다.

### 다형성의 기반
* 다형성의 기반은 업캐스팅과 동적 바인딩을 지원하는 언어적 특성에 더해 실행 시점에 적절한 메소드를 선택하는 동적 메소드 탐색이다.
  * 세 가지 메커니즘을 혼합하는 것으로 동일한 코드를 이용하여 서로 다른 메소드를 실행하는 다형성의 구현이 가능하다.
* 즉, **객체지향 프로그래밍 언어는 이러한 메커니즘의 도움을 받아 동일한 메시지에 대해 서로 다른 메소드를 실행할 수 있는 다형성을 구현**한다.

### super 참조
* **self 참조의 가장 큰 특징은 동적이라는 점이며, 메시지를 수신한 객체의 클래스에 따라 메소드 탐색을 위한 문맥을 런타임에 결정**한다.
* 반면, 자식 클래스에서 부모 클래스의 구현을 재사용해야하는 경우에 대비하여 대부분의 객체지향 언어에서는 super 참조라는 내부 변수를 제공한다.
* **super 참조를 활용한 메소드 호출은 실제로는 부모 클래스에게 해당 메시지를 전송하는 식으로 동작**한다.
  * **self 전송은 self 참조로부터 메소드 탐색을 재시작하는 반면, super 참조는 부모 클래스를 가리키므로 부모 클래스로부터 동적 탐색을 시작**한다.
* **super 참조에 의해 호출되는 메소드는 바로 상위의 부모 클래스 메소드가 아닌, 상속 계층의 더 상위에 위치한 클래스의 메소드일 수도 있다**.
  * 이러한 특징은 메소드가 반드시 부모 클래스에 위치하지 않아도 되는 유연성을 제공한다.
* 부모 클래스의 메소드를 호출하는 것과 부모 클래스로부터 메소드 탐색을 시작하는 것은 의미가 완전히 다르다.
  1. 부모 클래스의 메소드를 호출하는 것은 반드시 부모 클래스에 해당 메소드가 정의되어 있어야 한다.
  2. **부모 클래스로부터 메소드 탐색을 시작하는 것은, 상속 계층 상 상위에 위치한 클래스 중 어딘가에 하나라도 메소드가 정의되어 있으면 된다**.
* self 전송과 마찬가지로, super 참조를 활용한 메소드 호출 역시 부모 클래스의 인스턴스에게 메시지를 전송하는 것처럼 보이므로 super 전송이라고 부른다.
```
> super 참조의 용도는 부모 클래스의 메소드를 호출하는 것이 아닌, 한 단계 부모 클래스로부터 동적 메소드 탐색을 시작하기 위함이다.
> self 참조, super 참조는 동적 바인딩과 함께 상속을 이용하여 다형성을 구현하고 코드를 재사용하기 위한 핵심적인 재료이다.
```

### self 전송과 super 전송
* self 전송은 메시지를 수신하는 객체의 클래스에 따라 메소드를 탐색할 시작 위치를 동적으로 결정한다.
  * super 전송은 항상 메시지를 전송하는 클래스의 부모 클래스에서부터 시작한다.
* self 전송은 어떤 클래스에서 메시지 탐색이 시작될지 알 수 없다.
  * super 전송은 항상 해당 클래스의 부모 클래스에서부터 메소드 탐색을 시작한다.
* self 전송은 메시지 탐색을 시작하는 클래스가 미정이므로, 반드시 실행 시점에 동적으로 결정해야만 한다.
  * super 전송은 메시지 탐색을 시작하는 클래스가 미리 정해져있으므로, 컴파일 시점에 미리 결정해둘 수 있다.
  * 언어에 따라 super 참조를 런타임에 결정하는 경우도 있으나, 대부분의 객체지향 언어에서 상속을 사용하는 경우의 super 참조는 컴파일타임에 결정된다.

### 상속과 self 참조
* 자식 클래스의 인스턴스를 생성하는 경우, 개념적으로 자식 클래스의 내부에 부모 클래스의 인스턴스가 생성된다.
  * 이 때, 자식 클래스의 인스턴스 입장에서 self 참조는 자기 자신을 가리킨다.
* **자식 클래스의 인스턴스에 포함되어 생성된 부모 클래스의 인스턴스 입장에서, self 참조는 역시 자식 클래스의 인스턴스**를 가리킨다.
* 따라서 **동적 메소드 탐색 과정에서 자식 클래스의 인스턴스와 부모 클래스의 인스턴스는 동일한 self 참조를 공유**한다.

### 위임과 포워딩
* 위임이란, 자신이 수신한 메시지를 다른 객체에게 동일하게 전달하여 처리를 요청하는 것이다.
* 위임은 본질적으로 자신이 정의하지 않거나, 처리할 수 없는 속성 또는 메소드의 탐색 과정을 다른 객체로 이동시키기 위해 사용한다.
  * 때문에 **위임은 항상 현재의 실행 문맥을 가리키는 self 참조를 인자로 전달**한다.
* 이러한 관점에서, 포워딩과 위임의 차이는 다음과 같다.
  1. 포워딩은 다른 객체에게 요청 처리를 전달할 때 self 참조를 전달하지 않는다.
  2. 위임은 다른 객체에게 요청 처리를 전달할 때 현재의 실행 문맥인 self 참조를 전달한다.
* **포워딩은 요청을 전달받았던 최초의 객체에게 다시 메시지를 전송할 필요 없이, 단순히 코드를 재사용하고 싶은 경우에 사용**한다.
* 반면, **위임은 클래스를 이용한 상속 관계를 객체 사이의 합성 관계로 대체하여 다형성을 구현하기 위해 사용**한다.
* 상속은 개발자가 위임 기능을 직접 구현하지 않고도 자동으로 제공받을 수 있게 하는 데에 의의가 있다. 
  * **실행 시에 인스턴스 간의 self 참조는 알아서 전달되며, 결과 자식 클래스의 인스턴스와 부모 클래스의 인스턴스 사이에 동일한 실행 문맥을 공유**한다. 
```
> 상속 관계로 연결된 클래스 사이에서는 자동적인 메시지 위임이 발생한다.
> 상속은 동적으로 메소드를 탐색하기 위해 현재의 실행 문맥을 갖는 self 참조를 전달하며, 객체들 간 메시지 전달 과정은 자동으로 이뤄진다.
```

### 위임의 결론
* 객체지향은 객체를 지향하는 것이며, 클래스는 객체를 편리하게 정의하고 생성하기 위해 제공되는 프로그래밍 구성 요소에 불과하다.
  * **중요한 것은 언제나 메시지와 협력**이다.
* 클래스 없이도 객체 사이의 협력 관계를 구축할 수 있으며, 상속 없이도 다형성을 구현할 수 있다.
* 대부분의 객체지향 언어는 클래스에 기반하므로 다형성을 위해 클래스 기반의 상속이 널리 사용되곤 한다.
  * 그러나 프로토타입 언어처럼 위임을 통한 객체 수준의 상속을 구현하는 언어들도 분명히 존재한다.
  * 심지어, 클래스 기반의 객체지향 언어에서도 클래스라는 제약을 벗어나기 위해 위임 메커니즘을 사용할 수 있다.
* **중요한 것은 클래스 기반의 상속과 객체 기반의 위임 사이에서 기본 개념과 메커니즘이 공유된다는 사실**이다.