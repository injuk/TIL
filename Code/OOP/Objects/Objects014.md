# Objects
## 2022-04-07 Thu

## 일관성 있는 협력
* 객체는 협력을 위해 존재하며, 협력은 객체가 존재하는 이유인 동시에 문맥을 제공하는 역할을 수행한다.
* **객체지향 설계의 목표는 적절한 책임을 수행하는 객체들의 협력을 기반으로 결합도가 낮고, 재사용 가능한 코드 구조를 만드는 것**이다.
* 반면, 애플리케이션의 개발 과정에서는 유사한 요구사항을 반복적으로 추가하거나 수정하게 되는 경우가 잦다.
  * 가능하다면 유사한 기능을 구현하기 위해 유사한 협력 패턴을 사용한다.
  * 재사용을 위해 객체들의 협력 방식을 일관성 있게 정의해야 한다.
* 유사한 기능을 구현하기 위해 유사한 협력 방식을 따를 경우, 코드를 이해하기 한 층 더 쉬워질 수 있다.

### 일관성 없는 협력
* 유사한 기능을 수행하는 객체들이 정책을 구현하는 방식이 완전히 다른 경우가 있을 수 있다.
  * 예를 들어, 개념적으로는 연관된 기능들이 구현 방식에 대해서는 완전히 제각각인 경우에 해당한다.
* 이러한 구현의 비일관성은 다음과 같은 상황에서 특히 어려움을 겪게 한다.
  1. 새로운 구현을 추가해야하는 경우,
  2. 또는 기존의 구현을 이해해야하는 경우
* 애석하게도 개발자로서 우리가 수행하는 대부분의 활동은 새로운 구현을 추가하거나 기존의 구현을 이해하는 과정이다.
* 일관성 없는 코드는 크게 다음과 같은 문제를 갖는다.
  1. 새로운 기능을 추가할수록 코드 사이의 일관성이 틀어진다.
  2. 코드를 이해하기가 어렵다.
* 유사한 요구사항을 서로 다른 구조로 구현하는 코드는 코드를 이해하는데에 심리적인 장벽을 만든다.
* 따라서 **유사한 기능은 유사한 방식으로 구현해야 하며, 절대 서로 다른 방식으로 구현하지 않아야 한다**.
  * **객체지향에서 기능을 구현하는 유일한 방법은 객체 간의 협력을 만드는 것이므로, 유지보수 가능한 시스템을 만들기 위해 협력은 일관성이 있어야 한다**.

### 설계에 일관성 부여하기
* 일관성 있는 설계를 만들기 위한 조언은 크게 다음과 같다.
  1. 다양한 설계 경험을 익히도록 한다.
  2. **널리 알려진 디자인 패턴을 학습하고, 변경이라는 문맥에서 적용해본다**.
* 풍부한 설계 경험을 갖는 개발자는 어떤 변경이 중요하고, 변경을 어떻게 다루어야 하는지 통찰력을 얻을 수 있다.
  * 따라서, 경험이 풍부할수록 어떤 위치에서 일관성을 보장하고, 일관성을 제공하기 위해 어떤 방법을 사용해야할지 직관적으로 결정할 수 있다.
* 반면, **디자인 패턴은 특정한 변경에 대해 일관성 있는 설계를 만들 수 있는 경험을 모아놓은 일종의 템플릿**이다.
  * 따라서, 디자인 패턴을 학습하는 것으로 빠른 시간 안에 전문가의 경험을 흡수할 수 있다.
* 그러나 모든 경우에 적용 가능한 패턴을 찾을 수 있는 것은 아니므로, 협력을 일관성 있게 구성하기 위해 다음과 같은 지침을 따를 수 있다.
  1. **변하는 개념을 변하지 않는 개념으로부터 분리**한다.
  2. **변하는 개념은 캡슐화**한다.
* 상술한 두 원칙은 훌륭한 구조를 설계하기 위해 따라야 하는 기본적인 원칙이기도 하다.
  * **객체지향과 관련된 모든 원칙과 개념은 대부분 변경의 캡슐화라는 목표**를 갖는다.
```
> 애플리케이션에서 변경될 수 있는 부분을 찾아 변경되지 않는 부분으로부터 분리하는 것은 설계의 첫 번째 원칙이다.
> 다시말해, 코드에서 새로운 요구사항이 있을 때마다 바뀌는 부분이 있다면 해당 행동은 바뀌지 않는 부분으로부터 골라내서 분리해야 한다.
```

### 조건 로직과 객체 탐색
* 전통적인 절차지향 프로그램에서는 변경을 처리하기 위해 조건문의 분기를 추가하거나, 개별 분기 로직을 수정하곤 한다.
  * 이러한 방식은 새로운 조건이 추가될 때마다 분기를 추가하는 식으로 수정해야 한다.
* **객체지향은 다른 접근법을 취하며, 변경을 다루는 전통적인 방법은 조건 로직을 객체 사이의 이동으로 바꾸는 것**이다.
* **다형성은 이러한 조건 로직을 객체 사이의 이동으로 바꾸기 위해 객체지향이 제공하는 설계 기법**이다.
  * 이 경우, **클라이언트 코드는 상세한 구현을 판단하지 않고 참조를 통해 메시지의 전송만을 수행**한다.
  * **실제로 실행할 메소드를 결정하는 것은 순전히 메시지를 수신한 객체의 책임**이다.
* **결론적으로 객체지향적인 코드는 조건을 판단하지 않으며, 단지 다음 객체로 이동할 뿐**이다.
* **조건 로직을 객체 사이의 이동으로 대체하기 위해서는 커다란 클래스를 더 작은 클래스들로 분리**해야 한다.
  * 이 때, **클래스를 분리하는 중요한 기준은 변경의 이유와 변경의 주기**이다.
  * 클래스는 명확히 하나의 이유만으로 변경되어야 하고, 변경되는 경우에는 클래스 내부의 모든 코드가 함께 변경되어야 한다.
  * 즉, **클래스는 단일 책임 원칙을 준수하도록 분리되는 것이 이상적**이다.
* 커다란 메소드에 뭉쳐있던 조건 로직들을 변경의 압력에 맞추어 작은 클래스로 분리하면 인스턴스 간의 협력 패턴에 일관성을 부여하기가 더 쉬워진다.
  * **유사한 행동을 수행하는 작은 클래스들은 자연스레 역할이라는 추상화에 의해 묶이며, 역할 사이의 협력은 전체 설계의 일관성을 유지**하도록 한다.
* **이 과정에서도 핵심은 훌륭한 추상화를 찾아 추상화에 의존하도록 만드는 것**이다.
  * 추상화에 대한 의존은 결합도를 낮추고, 결과적으로 대체 가능한 역할로 구성된 협력을 설계할 수 있도록 한다.
  * 즉, **선택하는 추상화의 품질은 곧 캡슐화의 품질을 결정**한다.
```
> 변경에 초점을 맞추고, 캡슐화 관점에서 설계를 바라보면 일관성 있는 협력 패턴을 얻을 수 있다.
> 또한, 디자인 패턴은 이와 같은 접근법을 통해 얻을 수 있는 훌륭한 설계에 대한 경험의 산물이다.
```

### 캡슐화 돌아보기
* 대부분의 개발자는 캡슐화가 데이터 은닉을 말하는 것으로 이해한다.
  * 데이터 은닉이란, 오직 외부에 공개된 메소드를 통해서만 객체의 내부에 접근할 수 있도록 제한하는 기법이다.
  * 따라서 객체 내부의 상태 구현은 숨겨지며, 해당 클래스의 메소드만이 내부 상태를 나타내는 인스턴스 변수에 접근이 가능하다.
* 그러나 **캡슐화는 데이터 은닉 이상의 가치**를 갖는다.
* **캡슐화는 단순히 데이터만을 감추는 것이 아닌, 소프트웨어 상에서 변경할 수 있는 모든 개념을 감추는 것**이다.
  * 즉, **설계라는 맥락에서 캡슐화는 항상 변경이라는 주제가 녹아 들어 있는 용어**이다.
* 캡슐화의 가장 대표적인 예시는 객체의 퍼블릭 인터페이스와 구현을 분리하는 것이다.
  * 객체 개발자는 필요한 시점에 내부 구현을 변경할 수 있기를 바라며, 클라이언트 개발자는 퍼블릭 인터페이스가 변하지 않기를 바란다.
  * 따라서, **자주 변경되는 내부 구현은 안정적인 퍼블릭 인터페이스 뒤로 숨겨야**만 한다.
* 캡슐화는 다음과 같이 다양한 종류가 공존한다.
  1. 데이터 캡슐화: 클래스 내부에서 관리하는 데이터를 캡슐화한다.
  2. 메소드 캡슐화: 클래스의 내부 행동을 캡슐화한다.
  3. 객체 캡슐화: 합성을 의미하며, 객체와 객체 사이의 관계를 캡슐화한다.
  4. **서브타입 캡슐화: 다형성의 기반이 되며, 서브타입의 종류를 외부로부터 감춘다**.
* **캡슐화란 단순한 데이터 은닉이 아닌, 코드 수정으로 인한 영향을 제어할 수 있는 모든 기법이 캡슐화의 일종**이다.
  * 일반적으로 데이터, 메소드 캡슐화는 개별 객체에 대한 변경을 관리하기 위해 사용한다.
  * 일반적으로 객체, 서브타입 캡슐화는 협력에 참여하는 객체들의 관게에 대한 변경을 관리하기 위해 사용한다.
* **변경을 캡슐화할 수 있는 다양한 방법이 존재하지만, 협력을 일관성 있게 만들기 위해 가장 일반적으로 사용하는 방법은 서브타입, 객체 캡슐화의 조합**이다.
  * 이 때, **서브타입 캡슐화는 인터페이스 상속을 사용하지만 객체 캡슐화는 합성을 사용**한다.

### 서브타입 캡슐화와 객체 캡슐화를 적용하기
* **변하는 부분을 분리해서 타입 계층을 작성**한다.
  1. 변하지 않는 부분으로부터 변하는 부분을 분리하고, 변하는 부분들의 공통적인 행동은 추상 클래스나 인터페이스로 추상화한다.
  2. 변하는 부분은 해당 추상 클래스나 인터페이스를 상속받게 한다.
  3. 이를 통해 변하는 부분은 변하지 않는 부분의 서브타입이 될 수 있다.
* 즉, **변하는 부분을 우선 분리하고 > 변하는 부분의 공통점을 뽑아 추상화하고 > 변하지 않는 부분이 이 추상화에 의존하도록 하는 것으로 캡슐화가 가능**하다.
  * 서브타입 캡슐화: 변하는 부분의 실제 내부 구현은 추상화된 변하는 부분을 구현하는 서브타입이 되도록 한다.
  * 객체 캡슐화: 변하지 않는 부분은 합성 관계로서 변하는 부분의 추상화에 의존하도록 한다.
* **변하지 않는 부분의 일부로 타입 계층을 합성**한다.
  1. **앞서 구현한 타입 계층을 변하지 않는 부분에 합성**한다.
  2. 변하지 않는 부분은 변경되는 구체적인 사항에 결합되지 않아야 한다.
  3. **특히 의존성 주입과 같이 결합도를 느슨하게 유지하는 방법을 이용하여 오직 추상화에 의존**하도록 한다.
  4. 이를 토대로 변하는 부분은 변하지 않는 부분에 대해 분리되며, 변하는 부분의 구체적인 내용은 알 수 없으므로 변경이 캡슐화될 수 있다.

### 협력 패턴의 설계
* 변하는 부분과 변하지 않는 부분을 분리하고, 변하는 부분을 추상화하였다면 변하는 부분을 생략하고 변하지 않는 부분만으로 객체 간 협력을 고민할 수 있다.
  * 즉, **변하지 않는 요소와 추상화만으로 표현된 모델을 통해 재사용 가능한 협력 패턴이 선명하게 드러나게 된다**.
  * 이렇듯 **변하지 않는 요소와 추상화된 추상적인 요소만으로도 임의의 기능에 필요한 전체적인 협력 구조를 설명**할 수 있다.
* 핵심은 **변하는 것과 변하지 않는 것을 분리한 후 변하는 것을 캡슐화하면 변하지 않는 것과 추상화에 대한 의존성으로 전체 협력을 구현할 수 있다는 사실**이다.
  * 변하는 것은 추상화 뒤에 캡슐화되어 숨겨지므로, 전체적인 협력에 영향을 주지 않는다.
* 그러나 **실제로 협력이 동작하기 위해서는 추상화된 부분이 구체적인 컨텍스트로 확장되어야할 필요**가 있다.
  * 바꿔 말해, 구체화된 클래스는 협력 안에서 추상화된 부분을 대체할 수 있어야 한다.
* **유사한 기능을 서로 다른 방식으로 구현하는 경우 협력의 일관성이 유지되지 않으므로, 이해하기 어렵고 유지보수성이 낮은 코드가 만들어질 수 밖에 없다**.
  * 따라서 유사한 기능은 가능한 한 협력의 일관성을 유지하는 것이 이상적이다.

### 일관성 있는 협력의 효과
* 상술한 내용을 토대로 변경을 캡슐화하고 협력을 일관성 있게 만드는 데에 성공했다면, 다음과 같은 장점을 누릴 수 있다.
1. 변하는 부분을 변하지 않는 부분으로부터 분리했으므로, 변하지 않는 부분은 재사용이 가능하다.
   * 이를 토대로 코드의 재사용성은 향상되고, 테스트 대상 코드는 줄어든다.
2. 새로운 기능을 추가하기 위해서는 오직 변하는 부분만 구현하면 그만이다.
   * 이는 신규 기능을 추가할 경우 따라야하는 구조를 강제하므로, 기능을 추가하거나 변경할 때에도 설계의 일관성이 무너지지 않는다.
3. 기능을 추가하기 위해 규칙을 지키는 것보다 어기는 것이 더 어려운 구조를 갖는다.
   * 따라서, 개발자는 일관성 있는 협력을 통해 확장 포인트를 강요받으므로 정해진 구조를 우회하기가 더 어렵다.
4. **변하지 않는 부분은 모든 기본 정책에서 공통적이므로, 하나의 기능에서 이해한 지식을 다른 코드를 이해하는데에 그대로 적용**할 수 있다.
   * 이는 공통된 코드의 구조와 협력 패턴이 모든 기본 정책에 걸쳐 동일하기 때문이다.
   * 따라서 **코드를 한 번만 이해하면 다른 구현체에서도 해당 지식의 재사용이 가능하며, 이는 일관성이 떨어지는 협력에서 누리지 못하는 큰 장점**이다.
* 이렇듯 **유사한 기능에 대해 유사한 협력 패턴을 적용하는 것은 객체지향 시스템에서 말하는 개념적 무결성을 유지하는 가장 효과적인 방법**이다.
  * 개념적 무결성의 유지는 일관성 유지와 동일한 뜻으로 간주할 수 있다.
* 시스템이 몇 개의 일관성 있는 협력 패턴으로 구성된다면 시스템을 이해하고, 수정하고, 확장하기 위해 필요한 노력과 시간을 아낄 수 있다.
  * 이 과정에서 개념적으로 불필요한 구현체가 추가되더라도, 시스템의 전체적인 개념적 무결성을 무너트리는 것보다는 약간의 부조화를 기꺼이 수용하는 것이 좋다.
```
> 협력을 설계하고 있다면, 시스템의 개념적 무결성을 지키기 위해 항상 기존의 협력 패턴을 따를 수는 없는지 고민하는 것이 바람직하다.
> 이 과정에서 설계를 약간 비틀어 비교적 이상한 구조를 얻더라도, 전체적인 일관성을 유지할 수 있는 설계를 선택하는 것은 현명하다.
> 하나로 통합된 일련으 설계 아이디어를 반영하는 시스템은 서로 독립적이고 조화롭지 못한 아이디어들로 구성된 시스템보다 항상 좋다.
```

### 지속적으로 개선하기
* 처음에는 일관성을 유지하던 협력 패턴이 프로젝트가 성숙함에 따라 새로운 요구사항이 추가됨에 따라 일관성이 깨지는 경우를 자주 볼 수 있다.
  * 이는 **협력을 설계하는 초기 단계에 모든 요구사항을 예측하는 것이 불가능에 가까우므로 자연스러운 현상이며, 절대 잘 못된 것이 아니다**.
* 이러한 상황은 오히려 새로운 요구사항을 수용할 수 있는 협력 패턴으로 설계를 진화시킬 수 있는 좋은 기회로 삼는 것이 바람직하다.
* 협력은 고정된 것이 아니며, 현재의 협력 패턴이 변경의 압박을 견뎌내기 어렵다면 변경을 수용할 수 있는 협력 패턴이 되도록 기꺼이 리팩토링해야 한다.
  * 즉, **요구사항이 변경됨에 따라 협력 역시 지속적으로 개선**되어야 한다.
```
> 중요한 것은 현재의 설계에 맹목적으로 일관성을 맞추는 것이 아닌, 달라지는 변경의 방향에 맞추어 지속적으로 코드를 개선하려는 태도이다.
```

### 패턴을 찾아내기
* 상술한 바와 같이, 일관성 있는 협력의 핵심은 변경을 분리하고 캡슐화하는 것이다.
  * 변경을 캡슐화하는 방법은 협력에 참여하는 객체들의 역할과 책임을 결정한다.
  * 이렇게 결정된 협력은 코드의 구조를 결정한다.
* 따라서, 훌륭한 설계자가 되기 위해서는 변경의 방향을 파악하는 감각을 익히고 변경에 탄력적으로 대응할 수 있는 다양한 캡슐화 방법과 설계를 익혀야 한다.
* 애플리케이션에서 유사한 기능에 대한 변경이 지속적으로 발생한다면, 변경을 캡슐화할 수 있는 적절한 추상화를 찾아 변하지 않는 공통적인 책임을 할당한다.
  * 만약 현재의 구조가 변경을 캡슐화하기에 적합하지 않다면, 코드 수정 없이도 변경을 수용할 수 있도록 협력과 코드를 기꺼이 리팩토링해야 한다.
  * 이렇듯 변경을 수용할 수 있는 적절한 역할과 책임을 찾아나가다 보면 협력의 일관성이 서서히 윤곽을 드러낼 수 있게 된다.
* 협력을 일관성 있게 만드는 과정은 유사한 기능을 구현하기 위해 반복적으로 적용할 수 있는 협력의 구조를 찾아가는 긴 여정이다.
  * 즉, 협력을 일관성 있게 만드는 것은 유사한 변경을 수용할 수 있는 협력 패턴을 발견하는 것과 같다.
```
> 객체지향 설계는 객체의 행동과 이를 지원하는 구조를 계속해서 수정하는 작업을 반복해 나가며 다듬어진다.
> 객체, 역할, 책임은 계속해서 진화해 나가야하는 개념이다. 
> 이러한 과정을 통해 객체들의 통신 경로는 더욱 효율적이게 되어 주어진 작업을 수행하는 표준 방안이 정착되며, 비로소 협력 패턴이 드러나게 된다.
```