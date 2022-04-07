# Objects
## 2022-03-29 Tue

## 책임의 할당
* 앞서 다뤘듯 데이터 중심의 설계는 행동보다 데이터를 먼저 결정하고, 협력이라는 문맥을 벗어나 고립된 객체의 상태에 초점을 맞추므로 문제가 발생했다.
  * 예를 들어, 캡슐화를 위반하므로 요소 간의 결합도가 높아지고 응집도는 낮아져 변경이 어려워진다.
* 이러한 **문제점을 해결하기 위해서는 데이터보다 책임을 초점에 맞추어야 한다**.
* 그러나 책임에 초점을 맞춰 설계하는 것은 어떤 객체에게 어떤 책임을 할당할지 결정하는 것이 어렵다는 사실이다.
  * 책임 할당 과정은 끊임없는 트레이드오프이며, 어떤 방법이 최선인지는 문맥과 상황에 따라 달라질 수 있다.
* **올바른 책임을 할당할 수 있으려면 우선 다양한 관점에서 설계를 평가할 수 있는 역량이 뒷받침되어야 한다**.

### 데이터보다 행동을 먼저 결정하기
* 데이터 중심 설계에서 책임 중심 설계로 전환하는 과정에서, 다음의 원칙을 따라야만 한다.
  1. 데이터보다는 행동을 우선 결정하기
  2. 협력이라는 문맥 안에서 책임을 결정하기
* **상술한 두 원칙은 모두 설계 과정에서 데이터보다 객체의 책임, 협력에 초점을 맞추는 것을 강조**한다.
* **객체에게 중요한 것은 데이터가 아닌 외부에 제공하는 행동**이다.
  * 클라이언트 관점에서 객체가 수행하는 행동은 곧 객체의 책임을 의미한다.
  * **객체는 협력에 참여하기 위해 존재하며, 협력 안에서 수행하는 책임이 객체의 존재의의**가 된다.
* 필요한 것은 객체의 데이터보다 행동에 초점을 맞추기 위한 기법이며, 기본적인 해결 방법은 다음과 같이 자문하는 것이다.
  1. 객체가 수행해야하는 책임은 무엇인가?
  2. 객체가 책임을 수행하기 위해 필요한 데이터는 무엇인가?
* 이렇듯 **책임 중심 설게에서는 객체의 행동을 대변하는 책임을 결정한 후에 객체의 상태를 결정**한다.
  * 때문에 **객체지향 설계에서 가장 중요한 것은 적절한 객체에게 적절한 책임을 할당하는 능력**이다.

### 협력을 문맥삼아 책임을 결정하기
* 객체에게 할당된 책임의 품질은 협력에 적합한 정도로 결정된다.
  * 따라서 할당된 책임이 객체에게 어울리지 않는다면 나쁜 책임이며, 어색하더라도 협력에 적합하다면 책임은 좋은 것이다.
  * **책임은 객체의 입장이 아닌 협력의 관점에서 적합해야 한다**.
  * **협력을 시작하는 주체는 메시지 전송자이므로, 이는 즉 협력에 적합한 책임이라는 말이 메시지 전송자에게 적합한 경우를 의미**한다.
* **메시지를 전송하는 클라이언트의 의도에 적합한 책임을 객체에 할당해야하며, 이를 위해 메시지를 결정한 후에 객체를 선택**해야 한다.
  * **메시지가 존재하기 때문에 메시지를 처리할 객체가 필요한 것이므로, 객체를 결정한 후에 메시지를 선택하는 것은 지양**해야 한다.
  * 객체가 메시지를 선택하는 것이 아닌, 메시지가 객체를 선택해야 한다.
* 메시지는 클라이언트의 의도를 표현하며, 객체를 결정하기 전에 객체가 수신할 메시지를 먼저 결정해야 한다.
* **클라이언트는 어떤 객체가 메시지를 수신할지는 모르지만, 어떤 객체가 자신의 메시지를 받아 적절히 처리해줄 것을 믿고 의도를 표현한 메시지를 전송**한다.
  * 이 때, 메시지를 수신하기로 결정된 객체는 메시지를 처리할 책임을 할당 받게 된다.
* **메시지를 우선 결정하므로, 메시지 송신자는 메시지 수신자에 대한 그 어떤 세부사항도 전제할 수 없다**.
  * 즉, 전송자의 관점에서 수신자는 캡슐화된다.

### 올바른 책임 중심 설계의 흐름
* 아래와 같이 협력이라는 문맥 안에서 객체가 수행해야 할 책임에 초점을 맞추어 설계하도록 한다.
  1. 객체에게 적절한 책임을 할당하기 위해, 우선 협력이라는 문맥을 고려한다.
  2. 이 때, 협력이라는 문맥에서 적절한 책임이란 곧 클라이언트 관점에서 적절한 책임임을 이해한다.
  3. 클라이언트가 전송할 메시지를 결정한 후에야 비로소 객체의 상태를 저장하는데 필요한 데이터 구조를 고민한다.

### 책임 주도 설계와 책임 중심 설계
* 책임 중심 설계는 앞서 다루었던 책임 주도 설계와 방법이 거의 동일하다.
* **책임 주도 설계의 핵심은 책임을 결정한 후에 책임을 할당할 객체를 결정하는 것**이다.
* 또한, 협력에 참여하는 객체들의 책임이 어느 정도 수준까지 정리될 때까지는 객체 내부 상태에 대해 관심을 갖지 않아야 한다.

### GRASP 패턴
* 객체지향이 성숙함에 따라 많은 책임 할당 기법이 고안되었으나, 대중적으로는 패턴 형식으로 제안된 GRASP 패턴이 널리 알려졌다.
  * General Responsibility Assignment Software Pattern, 일반적인 책임 할당을 위한 소프트웨어 패턴의 약자이다.
  * GRASP는 객체에 책임을 할당할 때 지침으로 삼을 수 있는 원칙들을 패턴 형식으로 정리한 개념이다.

### 1. 도메인 개념에서 시작하기
* 설계를 시작하기 전에 도메인에 대한 개략적인 모습을 그려보는 것은 유용하다.
* 도메인 내부에는 무수히 많은 개념들이 존재하기 마련이다.
  * 이러한 도메인 개념들을 책임 할당의 대상으로 사용했을 때 코드에 도메인의 전체적인 모습을 투영하기가 더 수월해진다.
  * 때문에 어떤 **책임을 할당해야 하는 경우, 가장 먼저 고민해야 하는 유력한 후보는 도메인 개념**이다.
* **설계를 시작하는 단계에서는 개념의 의미와 관계가 정확하거나 완벽할 필요는 없으며, 단지 출발점의 역할만 수행할 수 있다면 충분**하다.
  * 설계를 위해 작성한 도메인과 도메인 개념 사이의 관계는 참고 자료 정도로만 간주하되, 이를 정리하는 데에 너무 많은 시간을 쏟을 필요는 없다.
  * **중요한 것은 설계를 시작하는 것이므로, 빠르게 설계와 구현을 진행하고 넘어가도록 한다**.

### 참고 - 올바른 도메인 모델이란?
* 올바른 구현을 이끌어낼 수 있는 도메인 모델은 모두 옳다.
* 도메인 모델은 도메인을 개념적으로 표현한 것이며, 내부적으로 포함된 개념과 관계는 구현의 기반이 되어야 한다.
* 이는 즉 **도메인 모델은 구현을 염두에 두고 구조화되어야 하며, 구현에 실제로도 도움이 되는 것이 바람직함을 의미**한다.
  * 필요한 것은 도메인을 그대로 투영하는 모델이 아닌, 구현에 도움이 되는 실용적인 모델이다.

### 2. 정보 전문가에게 책임을 할당하기 - 정보 전문가 패턴
* 책임 주도 설계 방식은 크게 다음과 같은 순서로 진행된다.
  1. 우선 애플리케이션의 목적을 기반으로 도메인 개념 구조를 설정한다.
  2. 애플리케이션이 제공하는 기능을 애플리케이션 자체의 책임으로 본다.
  3. 이 책임을 애플리케이션에게 전송된 메시지로 간주하여 첫 번째 메시지를 결정한다.
     * 이 때, 메시지를 전송할 객체는 무엇을 원하는지 메시지에 반영한다.
  4. 메시지를 책임질 첫 번째 객체를 선택한다.
     * 2.의 과정에서 결정된 메시지를 수신하기에 적합한 객체를 선택한다.
  5. 첫번째 객체가 메시지를 처리하는 데에 필요한 절차와 구현을 간단하게 고려한다.
     * 해당 과정은 매우 세세한 구현까지 정의할 필요는 없으며, 해당 겍체가 할 수 있는 일과 할 수 없는 일을 간단하게 구분하는 정도면 충분하다.
  6. 스스로 처리할 수 없는 작업이 있다면 외부의 다른 객체에게 협력을 요청한다.
     * 이것이 새로운 메시지가 되고, 이제부터 해당 요청을 책임으로서 처리해줄 객체를 찾는 작업을 반복한다.
  7. 목적을 달성할 수 있는 협력 공동체가 완성될 때까지 반복한다.
* 메시지는 메시지를 수신하여 처리하는 객체가 아닌, **메시지를 전송하는 객체의 의도를 반영하여 결정**해야 한다.
* 또한 **메시지를 책임지기에 적합한 객체를 선택하기 위해, 객체가 상태와 행동을 하나의 단위로 캡슐화하는 단위라는 사실을 명심**한다.
  * 때문에 **기본적으로 필요한 정보인 상태와 행동을 갖고 있는 객체를 찾아 책임을 할당하는 것이 바람직**하다.
  * GRASP에서 이러한 패턴은 정보 전문가 패턴으로 구분된다.
* 정보 전문가 패턴이란 쉽게 말해 책임을 객체에게 할당하는 일반적인 원리로서 **책임을 수행하기 위해 필요한 정보를 갖는 객체에게 할당**하는 것을 말한다.
  * 객체는 자율적인 존재이므로, 정보를 알고 있는 객체만이 책임을 어떻게 수행할지 스스로 판단하고 결정할 수 있다.
  * 정보 전문가 패턴을 따를 경우, 정보와 행동은 가까이에 배치되므로 높은 수준의 캡슐화를 유지할 수 있다.
* 다시 말해, **정보 전문가 패턴은 객체가 자신이 소유하는 정보와 관련된 작업을 수행해야 한다는 일반적인 직관을 표현한 패턴**이다.
* 중요한 것은, **정보 전문가 패턴에서 말하는 정보가 반드시 인스턴스 변수와 같은 데이터일 필요는 없다는 사실**이다.
  * 정보 전문가로서의 객체는 책임을 수행하기 위해 필요한 정보를 반드시 가지고 있을 필요 없으며, 그 때 그 때 계산하거나 다른 객체와 협력할 수 있다.

### 정보 전문가 패턴의 결론
* **정보 전문가 패턴은 객체에게 책임을 할당할 때 가장 기본이 되는 책임 할당 원칙**이다.
* 정보 전문가 패턴은 객체가 상태와 행동을 함께 가지는 단위라는 객체지향의 기본 원리를 충실히 따르며, 이를 책임 할당 관점에서 표현한다.
  * 정보 전문가 패턴은 책임을 수행하기에 적절한 정보를 갖는 객체를 최우선적으로 선택하는 원칙이기 때문이다.
* 정보 전문가 패턴을 따르는 것 만으로도 자율성이 높은 객체들로 구성되는 협력 공동체를 구성할 수 있는 가능성이 매우 높아진다.

### 3. 낮은 결합도, 높은 응집도를 유지하기 - 낮은 결합도 패턴과 높은 응집도 패턴
* 설계는 연속된 트레이드오프 활동이므로, 동일한 기능을 구현할 수 있는 많은 대안이 존재한다.
  * 때문에 실제로 설계를 진행하다 보면 설계 중에서 적절한 하나를 선택해야 하는 경우가 빈번하게 발생한다.
  * 이러한 경우, 올바른 책임 할당을 위해 정보 전문가 패턴 이외의 책임 할당 패턴을 함께 고려할 필요가 있다.
* **높은 응집도와 낮은 결합도는 객체에 책임을 할당할 때 반드시 고려해야 하는 기본 원리 중 하나**이다.
  * 따라서 책임을 할당할 수 있는 다양한 대안들이 존재한다면, 응집도와 결합도 측면에서 이점이 있는 대안을 선택하는 것이 좋다.
* GRASP에서는 이러한 패턴을 각각 낮은 결합도 패턴과 높은 응집도 패턴으로 분류한다.
* 낮은 결합도 패턴: **의존성을 낮추고 변화의 범위를 좁히며 재사용성을 증가시키기 위해, 설계의 전체적인 결합도는 낮게 유지되도록 책임이 할당되어야 한다**.
  * 따라서 여러 대안적인 설계가 존재할 경우, 낮은 결합도를 유지할 수 있는 설계를 선택하는 것이 바람직하다.
* 높은 응집도 패턴: **복잡성을 관리할 수 있는 수준으로 유지하기 위해, 높은 응집도를 유지하는 책임을 할당**해야 한다.
  * 낮은 결합도와 마찬가지로 높은 응집도 역시 여러 설계 대안 중 하나를 선택하기 위한 기준이 되어줄 수 있다.
* **낮은 결합도 패턴과 높은 응집도 패턴은 모두 설계 과정 전체에서 책임과 협력의 품질을 검토하기 위해 반드시 염두에 두어야 할 평가 기준**이다.
  * 책임을 할당하고 코드를 작성하는 순간마다 두 관점에서 전체적인 설계 품질을 검토하는 것으로 유연성과 재사용성, 그리고 단순한 설계를 얻을 수 있게 된다.

### 4. 객체 생성 책임을 할당하기 - 창조자 패턴
* 협력에 참여하는 임의의 객체에는 반드시 인스턴스를 생성할 책임이 할당되어야 한다.
  * GRASP의 창조자 패턴은 이러한 경우에 사용할 수 있는 책임 할당 패턴이다.
* **창조자 패턴은 객체를 생성할 책임은 어떤 객체에 할당해야하는지에 대한 다음과 같은 지침을 제공**한다.
* 객체 A를 생성해야 하는 경우, 다음과 같은 조건 중 가장 많은 항목을 만족하는 객체 B에게 생성 책임을 할당한다:
  1. A를 포함 또는 참조하거나,
  2. A를 기록하거나,
  3. A를 긴밀하게 사용하거나,
  4. A를 초기화하는 데에 필요한 모든 데이터를 갖는 경우
     * 이 경우, 객체 B는 객체 A에 대한 정보 전문가가 된다.
* **창조자 패턴의 주된 의도는 어떤 방식으로든 생성되는 객체와 연결되거나, 관련되어야할 필요가 있는 객체에게 객체 생성 책임을 할당하는 것**이다.
  * **생성될 객체에 대해 잘 알고 있거나, 해당 객체를 사용해야 하는 객체는 어떤 방식으로든 생성될 객체와 긴밀하게 결합**된다.
* **이미 결합되어 있는 객체에게 생성 책임을 할당하는 것은 전체적인 설계 결합도에 영향을 주지 않는다**.
  * 창조자 패턴은 이미 존재하는 객체 간의 관계를 이용하므로, 설계가 낮은 결합도를 유지할 수 있도록 한다.

### 책임 분배와 코드 작성
* 현재까지 작성한 원칙들을 활용하면 대략적인 책임을 객체에게 할당할 수 있다.
* 그러나 이러한 책임 분배는 설계를 시작하기 위해 필요한 대략적인 스케치에 지나지 않는다.
* **실제 설계는 코드를 작성하는 과정에서 이루어짐을 반드시 명심**한다.
  * 협력과 책임이 제대로 동작하는지 확인하는 유일한 방법은 개발을 진행하는 것 뿐이므로, 올바른 설계인지 확인하려면 반드시 코드를 작성해야만 한다.  

### 변경에 취약한 클래스
* 변경에 취약한 클래스는 코드를 수정해야하는 이유를 하나 이상 갖는 클래스다.
  * 변경의 이유가 많은 클래스는 그만큼 응집도가 낮아진다.
* **이러한 클래스는 낮은 응집도로 인한 문제가 발생하기 전에 변경의 이유에 따라 클래스를 분리해주는 것이 바람직**하다.
* 일반적인 설계 개선 작업은 변경의 이유가 하나 이상인 클래스를 찾아내는 것에서 시작하는 것이 좋다.
* 변경의 이유는 다음과 같은 기준을 통해 코드로부터 파악할 수 있다.
  1. 인스턴스 변수가 초기화되는 시점이 제각각인 경우
     * 응집도가 높은 클래스는 생성 시점에 모든 속성을 초기화한다.
     * 반면 응집도가 낮은 클래스는 객체 속성 중 일부만 초기화하고, 일부는 초기화하지 않은 상태로 남겨두곤 한다.
     * 이 경우, 한 번에 초기화되는 속성을 기준으로 클래스를 분리하는 것이 바람직하다.
  2. 메소드들이 인스턴스 변수를 제각각 사용하는 경우
     * 응집도가 낮은 클래스는 메소드들이 사용하는 속성에 따라 그룹화가 가능하다.
     * 모든 메소드가 객체의 모든 속성을 사용하는 것이 가장 이상적이며, 그렇지 못한 경우 속성 그룹 별 메소드를 기준으로 분리하는 것이 바람직하다.
* 나아가 클래스의 변경의 이유가 많은 경우 응집도가 낮음을 의미하므로, 변경의 이유를 기준으로도 클래스를 분리해야 한다.
* 일반적으로 응집도가 낮은 클래스는 변경의 이유가 많은 문제를 포함한 상술한 세 개의 특징을 모두 갖는 경우가 많다.
  * 또한 메소드의 크기가 너무 커 문제가 명확히 보이지 않을 수도 있으며, 이러한 경우에는 메소드를 응집도 높은 작은 메소드로 분리해나가야 한다.

### 5. 객체의 타입에 따라 변하는 행동을 책임으로 할당하기 - 다형성 패턴
* **객체의 암시적인 타입에 따라 행동을 분기해야 하는 경우, 암시적인 타입 자체를 명시적인 타입으로 정의하고 행동을 나누도록 한다**.
  * 즉, 객체의 타입에 따라 변하는 행동은 타입을 분리하고 변화하는 행동은 각 타입의 책임으로 할당한다.
* 예를 들어 두 객체가 동일한 책임을 수행하지만 내부 구현이 다른 경우, 역할 개념을 떠올릴 수 있다.
* 동일한 책임을 수행한다는 것은 동일한 역할을 수행할 수 있으므로, 역할은 협력 안에서 객체의 대체 가능성을 의미한다.
* 이러한 경우, 역할을 사용하여 객체의 구체적인 타입을 추상화하는 것이 바람직하다.
* 이를 통해 협력이 필요한 객체에 수많은 if 분기 없이 상대 객체가 어떤 역할을 수행할 수 있고, 어떤 메시지를 이해할 수 있는지만 알면 협력이 가능해진다.
  * 조건에 따른 변화는 프로그램의 기본 논리이다. 
  * 그러나 프로그램을 if 또는 switch 등의 조건 논리를 활용하여 설계하는 경우 변경 사항 발생시 조건 논리를 수정해야 한다.
  * 이러한 방식은 프로그램의 수정 가능성을 낮추고, 변경에 취약하게 만든다.
* **다형성 패턴에 따를 경우 객체의 타입 검사와 타입 별 분기 코드를 작성하는 대신 다형성을 활용하여 새로운 변화를 다루기 쉽도록 확장**하게 된다.

### 6. 변경으로부터 보호하기 - 변경 보호 패턴
* GRASP에서 변경 보호 패턴은 변경을 캡슐화하도록 책임을 할당하는 것을 의미한다.
  * 즉, **변경 보호 패턴은 책임 할당의 관점에서 캡슐화를 설명**한다.
  * 설계에서 변하는 것을 찾아내고, 변하는 개념은 캡슐화해야 한다.
* 설계 단계에서 **변화가 예상되는 지점들을 식별하고, 그 주위에 안정된 인터페이스들을 형성하도록 책임을 할당하는 것이 이상적**이다.

### 다형성 패턴과 변경 보호 패턴의 조합
* 클래스를 변경에 따라 분리하고 인터페이스로 캡슐화하는 것은 설계의 결합도와 응집도를 향상시키는 강력한 기법이다.
  * **하나의 클래스가 여러 타입의 행동을 구현하는 것처럼 보인다면, 분리하고 다형성 패턴을 적용하여 책임을 분산**한다.
  * **예측 가능한 변경 사항으로 인해 여러 클래스가 불안정해진다면, 변경 보호 패턴에 따라 안정적인 인터페이스를 삽입하고 그 뒤로 변경을 캡슐화**한다.
* 적시에 두 패턴을 조합한다면 코드 수정이 끼치는 영향의 범위를 조절할 수 있고, 변경과 확장에 유연하게 대처 가능한 설계를 정의할 수 있게 된다.

### 바람직한 객체지향 설계
* 바람직한 책임 중심의 객체지향 설계를 통해 얻을 수 있는 이점은 다음과 같다.
  1. 모든 클래스의 내부 구현은 캡슐화된다.
  2. 모든 클래스는 변경의 이유를 오직 하나만 갖게 된다.
  3. 모든 클래스는 응집도가 높고, 다른 클래스와는 최대한 느슨하게 결합된다.
  4. 모든 클래스는 작고, 하나의 일만 수행한다.
  5. 모든 클래스에 책임은 적절하게 분배된다.
* 가능하다면 데이터보다 책임을 중심으로 설계하는 것이 좋다.
  * **객체에게 중요한 것은 상태가 아니라 행동이며, 객체지향 설계의 기본은 책임과 협력에 초점을 맞추는 것**이다.
* **또한 중요한 것은 구현을 가이드할 수 있는 도메인 모델을 선택하는 것**이다.
  * 객체지향은 도메인의 개념과 구조를 반영한 코드를 만들 수 있기 때문에 도메인의 구조가 코드의 구조를 이끌어내는 것은 자연스럽다.
  * 좋은 도메인 모델에는 도메인 안에서 변하는 개념과, 이들 사이의 관계가 투영되어 있어야 한다.
  * 이렇게 투영된 직관은 설계가 가져야하는 유연성을 이끌도록 가이드할 수 있다.

### 변경과 유연성
* 설계를 주도하는 것은 변경이며, 개발자로서 변경에 대비할 수 있는 방법은 크게 다음과 같이 나누어볼 수 있다.
  1. 코드를 이해하고 수정하기 쉽도록 단순하게 설계한다.
  2. 코드를 수정하지 않고도 변경을 수용할 수 있도록 유연하게 설계한다.
* **대부분의 경우 단순한 코드가 좋지만, 유사한 변경이 반복적으로 발생하는 상황에서는 유연성을 부여하는 설계가 바람직**하다.

### 책임 주도 설계의 대안
* 책임 주도 설계에 익숙해지려면 부단한 시간과 노력이 필요하지만, 숙련된 설계자에게도 적절한 책임과 객체를 선택하는 일은 여전히 어려울 수 있다.
* 책임과 객체의 위치를 결정하지 못하는 경우, 돌파구를 얻기 위해 최대한 빠르게 동작하는 코드를 작성한 후 리팩토링을 수행하는 방식을 활용할 수 있다.
  * 오히려 객체지향 설계에 익숙하지 않은 개발자는 이러한 방식이 더 훌륭한 결과물을 얻을 수도 있다.
* **중요한 것은 캡슐화와 결합도, 응집도를 이해하고 훌륭한 객체지향 원칙을 적용하기 위해 노력해야 한다는 마음가짐**이다.

### 메소드의 응집도 높이기
* 너무 긴 메소드는 다양한 측면에서 유지보수에 악영향을 미치므로, 메소드를 작게 분해하여 응집도를 높일 필요가 있다.
* 클래스와 마찬가지로 응집도 높은 메소드 역시 변경의 이유는 단 하나여야 한다.
  * 따라서, 객체로 책임을 분배할 때 가장 먼저 할 일은 메소드를 응집도 있는 수준까지 분해하는 것이다.
* 이로 인해 클래스의 길이가 길어지더라도, 대부분의 경우 명확성이 갖는 가치가 클래스의 물리적인 길이보다 더 크다.
  * 코드를 작은 메소드들로 분해하면 전체적인 흐름을 이해하기도 쉬워진다.
* **작고, 명확하고, 하나의 일만 충실히 수행하는 응집도 높은 메소드는 변경 가능한 설계를 이끌어내는 기반**이 된다.
  * 이러한 메소드들이 하나의 변경의 이유만 갖도록 개선된다면, 결론적으로 응집도 높은 클래스가 만들어진다.

### 객체를 자율적으로 만들기
* 상술한 과정에서 분할된 메소드들이 객체와 어울리지 않는다면, 메소드를 적절한 객체로 이동시킬 필요가 있다.
* 이 때, 적절한 객체를 결정하는 기준은 자율적인 객체를 만드는 기준과 같다.
  * 즉, **메소드가 사용하는 데이터를 저장하는 클래스로 메소드를 이동**시켜야 한다.
* 이 과정에서, **가능하다면 메소드들의 접근 제한은 다 제한적인 방향으로 수정되어 내부 구현을 캡슐화하는 것이 이상적**이다.
  * **메소드의 위치를 변경할 때마다 반드시 캡슐화, 응집도, 결합도 측면에서 이동시킨 메소드의 적절성을 판단**해야 한다.
* 메소드를 데이터를 갖는 객체로 이동시키는 작업은 결과적으로 메시지를 통해서만 협력하고, 내부 구현은 캡슐화되는 좋은 설계를 낳는 기반이 된다.