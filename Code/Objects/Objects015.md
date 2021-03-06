# Objects
## 2022-04-08 Fri

## 디자인 패턴과 프레임워크
* 디자인 패턴이란, 설계에서 반복적으로 발생하는 문제를 해결하기 위한 방법이며 설계를 재사용하는 데에 그 목적이 있다.
* 디자인 패턴이 설계를 재사용하기 위한 것이라면, 프레임워크는 설계와 코드를 함께 재사용하는 데에 목적이 있다.
* **디자인 패턴과 프레임워크는 모두 일관성 있는 협력을 만들기 위해 사용할 수 있는 방법**이다.

### 소프트웨어 패턴
```
> 패턴이란 하나의 실무 컨텍스트에서 유용하게 사용되어 왔고, 다른 실무 컨텍스트에서도 유용할 것이라고 예상되는 무언가이다.
```
* 패턴은 언제나 소프트웨어 개발의 중요한 화두였으며, 그 개념이 제창된 이후로 수많은 패턴이 발견되었다.
* 수 많은 패턴 속에서 길을 잃지 않기 위해, 패턴의 정의보다는 패턴이라는 용어 자체가 풍기는 뉘앙스를 이해하는 것이 중요하다.
* 일반적으로 패턴이 갖는 핵심적인 특징은 크게 다음과 같다.
  1. 패턴은 반복적으로 발생하는 문제와 해법의 쌍으로 정의된다.
  2. 패턴을 사용하여 이미 알려진 문제와 해법을 문서화할 수 있고, 이를 토대로 다른 개발자와 소통할 수 있다.
  3. 패턴은 추상적인 원칙과 실제 코드 작성 사이의 간극을 메워주며, 실질적인 코드의 작성을 돕는다.
  4. 패턴은 실무에서 탄생했다는 점이 중요하다.
* **패턴이 지니는 가장 큰 가치는 경험을 통해 축적된 실무 지식을 효과적으로 요약하여 전달할 수 있다는 것**이다.
  * 패턴은 치열한 실무 현장에서 검증되고 입증된 자산이며, 경험의 산물이다.
  * 실무 경험이 적은 주니어 개발자 역시 패턴을 익히고 반복적으로 적용하는 것으로 유연하고 품질이 높은 소프트웨어를 개발하는 방법을 익힐 수 있다.
* **패턴의 이름은 개발자 커뮤니티가 공유할 수 있는 중요한 어휘이며, 높은 수준의 대화를 가능하게 하는 가장 중요한 요소**이다.
* 패턴은 소프트웨어 개발과 직접적인 연관이 있는 영역으로 한정되지 않으며, 반복적인 규칙을 발견할 수 있는 모든 영역이 패턴의 대상이 될 수 있다.
  * 예를 들어, 프로젝트 조직의 구성 방법이나 일정 추산법, 백로그를 활용하는 요구사항 관리 모두 패턴의 대상이 될 수 있다.

### 패턴의 분류
* 패턴을 분류하는 가장 일반적인 방법은 패턴의 범위나 적용 단계에 따라 다음과 같이 나누는 것이다.
  1. 아키텍쳐 패턴
  2. 분석 패턴
  3. 디자인 패턴
  4. 이디엄
* 아키텍쳐 패턴은 디자인 패턴의 상위에 위치하며, 소프트웨어의 전체적인 구조를 결정하기 위해 사용할 수 있다.
  * 이는 서브시스템들 사이의 관계를 조직화하는 규칙과 가이드라인을 포함한다.
* 디자인 패턴은 특정 상황에서의 일반적인 설계 문제를 해결하며, 협력하는 컴포넌트 사이에서 반복적으로 발생하는 구조를 서술한다.
  * 중간 규모의 패턴이며, 특정 설계 문제를 해결하는 것에 의의가 있다.
  * 아키텍쳐 패턴과 디자인 패턴은 프로그래밍 패러다임 또는 프로그래밍 언어에 독립적이다.
* 이디엄은 특정 프로그래밍 언어에만 국한되는 하위 레벨 패턴이며, 언어의 기능을 활용하여 컴포넌트 또는 컴포넌트 간의 특정 측면을 구현하는 방법을 서술한다.
  * 이디엄은 언어에 종속적이므로, 특정 언어의 이디엄은 다른 언에서는 적용 불가능한 경우가 발생할 수 있다.
  * 아키텍쳐 패턴과 디자인 패턴, 이디엄은 일반적으로 기술적인 문제를 해결하는 데에 초점을 맞추는 패턴이다.
* 분석 패턴은 도메인 내에서 발생하는 개념적인 문제를 해결하는 데에 초점을 맞추는 패턴이다.
  * 분석 패턴은 업무 모델링 시에 발견되는 공통적인 구조를 표현하는 개념의 집합이다.

### 패턴과 책임 주도 설계
```
> 객체지향 설계에서 가장 중요한 일은 올바른 책임을 올바른 객체에게 할당하고, 객체 간의 유연한 협력 관계를 구축하는 것이다.
```
* 책임과 협력을 결정하는 작업은 대부분의 경우 훌륭한 품질의 설계를 얻기까지 많은 시간과 노력을 들여야만하는 일이다.
* **패턴은 공통으로 사용할 수 있는 역할과 책임, 협력의 템플릿이며, 반복적으로 발생하는 문제를 해결하기 위해 사용할 수 있는 좋은 예제인 템플릿을 제공**한다.
* **중요한 것은 각 패턴의 세부적인 내용이 아닌, 특정 상황에 적용할 수 있는 설계를 패턴을 통해 쉽고 빠르게 떠올릴 수 있다는 사실**이다. 
  * 특정 상황에 맞는 패턴을 잘 알고 있는 경우, 책임 주도 설계의 절차를 모두 따르지 않고서도 구현해야할 객체들의 역할, 책임, 협력 관계를 구성할 수 있다.
* **패턴의 구성 요소는 클래스가 아닌 역할임에 주의**해야 한다.
  * 예를 들어, Composite 패턴의 구성 요소인 Component, Composite, Leaf는 클래스의 이름이 아닌 역할이다.
  * 따라서, **각 역할이 필요로하는 기능을 제공하는 모든 객체는 해당 역할을 수행할 수 있다**.
* 패턴을 구성하는 요소가 클래스가 아닌 역할이라는 사실은 패턴의 템플릿을 구현할 수 있는 다양한 방법이 존재한다는 의미와 같다.
  * 이 때, 역할은 같은 오퍼레이션에 응답할 수 있는 책임의 집합이므로 하나의 객체가 여러 역할을 수행하거나, 다수의 클래스가 동일한 역할을 구현할 수도 있다.
* **디자인 패턴의 구성요소가 클래스와 메소드가 아닌 역할과 책임이라는 사실을 반드시 이해**해야 한다.
  * 예를 들어, 어떤 구현이 임의의 디자인 패턴을 따르는 경우 역할, 책임, 협력 관점에서 유사성을 공유한다는 의미이다.
  * **디자인 패턴은 역할, 책임, 협력의 템플릿을 제공할 뿐 구체적인 구현 방법을 제공하지 않으므로, 절대 특정한 구현 방식이 강제되는 것은 아니다**.
```
> 패턴을 적용하기 위해서는 패턴에서 제시하는 구조를 그대로 표현할 것이 아니라, 패턴의 기본 구조에서 출발하여 요구사항에 맞게 구조를 수정해야 한다.
```

### 캡슐화와 디자인 패턴
* 알려진 대부분의 디자인 패턴은 협력을 일관성 있고 유연하게 만드는 데에 그 목적이 있다.
  * 때문에 **각 디자인 패턴은 특정한 변경을 캡슐화하기 위한 독자적인 방법을 제공**한다.
* 대부분의 **디자인 패턴이 갖는 의의는 특정한 변경을 캡슐화하는 것으로 유연하고 일관성 있는 협력을 설계할 수 있는 경험을 공유하는 것**이다.
* 디자인 패턴에서 중요한 것은 디자인 패턴의 구현 방법이나 설계가 아니며, 어떤 디자인 패턴이 어떤 변경을 캡슐화하는지 이해하는 것이 중요하다.
  * 또한, 각 디자인 패턴이 변경을 캡슐화하기 위해 어떤 방법을 사용하는지 반드시 이해해야 한다.

### 출발점으로서의 패턴
* **패턴은 출발점이며, 설계의 목표라는 목적지가 되어서는 안 된다**.
* **전문가들은 널리 요구되는 유연성이나 공통적으로 발견되는 특정한 설계 이슈를 해결하기 위해 적절한 디자인 패턴을 이용하여 설계를 시작**한다.
  * 패턴은 단지 목표로 향하는 설계에 이를 수 있는 나침반에 지나지 않는다.
  * 따라서, **현재 요구사항이나 기술 스택, 프레임워크에 적합하지 않다면 패턴을 목적에 맞게 수정**해야 한다.
* **패턴을 사용하며 마주치는 대부분의 문제는 패턴을 무분별하게 적용하는 경우에 발생**한다.
  * 이는 패턴 입문자가 빠지기 쉬운 함정으로, 패턴을 적용하는 컨텍스트의 적절성을 무시하고 패턴의 구조에만 초점을 맞춘 경우에 발생한다.
* 해결하려는 문제가 아니라 패턴이 제시하는 구조를 맹목적으로 따르는 것은 불필요하게 복잡하고, 유지보수성이 낮은 시스템을 낳는다.
  * 때문에 **부적절한 상황에서 적절하지 않게 선택된 패턴으로 인해 소프트웨어의 엔트로피는 계속해서 증가하는 부작용으로 이어지기 쉽다**.
* **패턴을 남용하지 않기 위해, 다양한 트레이드 오프 관계 속에서 패턴을 적용하고 사용해 본 경험**이 필요하다.
* 패턴에 있어 초심자와 전문가의 차이는 크게 다음과 같다.
  1. 어떤 문제를 해결하기 위해 과거의 경험을 통해 어떤 컨텍스트에 어떤 패턴을 적용할지 결정할 수 있는지,
  2. 어떤 컨텍스트에 무엇을 적용하지 말아야 하는지에 대한 감각.
* 패턴에 입문한 초심자는 그 강력함에 매료되어 모든 코드에 패턴을 적용하려할 수 있다.
  * 그러나 명확한 트레이드오프 없이 남용된 팩토리는 설계의 복잡성만 높이는 일이다.
  * 타당한 이유 없이 적용된 패턴은 패턴에 익숙한 개발자들로 하여금 의도를 이해시킬 수 없다. 
  * 나아가 패턴을 알지 못하는 사람은 불필요하게 설게를 따르는 데에서 시간을 낭비하게 된다.
* 정당한 이유 없이 사용된 모든 패턴은 설계를 복잡하게 하는 원흉이다.
  * 때문에, **패턴은 복잡성의 가치가 단순성을 넘어설 때에만 적용**해야 한다. 
* 패턴을 적용하는 대신, 우선 설계를 조금 더 단순하고 명확하게 만들 수 없는지 고민해야 한다.
  * 또한, 코드를 공유하는 모든 개발자가 적용된 패턴에 대해 알고 있어야만 한다.
* **패턴을 적용할 때 함께 작업하는 사람들이 패턴에 익숙한지 확인하고, 그렇지 못하다면 설계 지식과 더불어 패턴 지식도 함께 공유해야 한다**.
* **훌륭한 소프트웨어 설계가 발전해온 과정을 공부하는 것이 훌륭한 설계 자체를 공부하는 것보다 훨씬 중요**하다.
  * 패턴은 출발점이며, 공통적인 문제에 대한 적절한 해법을 제공한다.
  * 이러한 공통 해법은 우리가 직면한 문제와 맞지 않을 수도 있으며, 문제를 분석하고 창의력을 발휘하여 패턴을 현재 상황에 맞도록 수정해야 한다.
```
> 패턴이 현재의 문제에 맞지 않는다고 해도, 참조할 수 있는 모범적인 역할과 책임의 집합을 알고 있는것은 큰 도움이 될 것이다.
```

### 코드 재사용과 설계 재사용
* 디자인 패턴은 프로그래밍 언어에 독립적으로 재사용 가능한 설계 아이디어를 제공하는데에 목적이 있다.
  * 그러나 디자인 패턴은 적용을 위해 설계 아이디어를 프로그래밍 언어에 맞추어 가공해야하고, 매 번 구현 코드를 재작성 해야한다는 단점이 존재한다.
* 재사용 관점에서 설계 재사용보다 더 좋은 방법은 코드 재사용이지만, 이를 토대로 대두된 컴포넌트 기반의 재사용은 현실성이 떨어지는 지적을 받아왔다.
* **가장 이상적인 형태의 재사용 방법은 설계와 코드의 재사용을 각각 적절한 수준으로 조합하는 것**이다.
  * 이렇듯 **설계를 재사용하면서도 유사한 코드를 반복적으로 구현하는 문제를 피하기 위해 제안된 방법이 프레임워크**이다.

### 프레임워크
* 프레임워크란, 추상 클래스나 인터페이스를 정의하고 인스턴스 사이의 상호작용을 토대로 시스템의 전체 혹은 일부를 구현하는 재사용 가능한 설계이다.
  * 또는, 개발자가 현재의 요구사항에 맞추어 커스터마이징할 수 있는 애플리케이션의 골격이기도 하다.
  * **상술한 두 문장은 각각 프레임워크의 구조적인 측면과, 코드와 설계의 재사용이라는 프레임워크 사용 목적에 초점을 맞추는 설명에 해당**한다.
* **프레임워크는 코드를 재사용하는 것으로 설계 아이디어를 재사용**한다.
* 프레임워크는 애플리케이션 아키텍쳐를 제공하며, 문제 해결에 필요한 설계 결정과 기반 코드를 함께 제공한다.
  * 또한 애플리케이션을 확장할 수 있도록 부분적으로 구현된 추상 클래스와 인터페이스 집합, 또는 추가 작업 없이 재사용 가능한 다양한 컴포넌트도 제공한다.

### 프레임워크 II
* 프레임워크는 클래스와 객체들의 분할, 전체 구조, 클래스와 객체 간 상호작용, 클래스와 객체 간 조합 방법, 제어 흐름에 대해 미리 정의한다.
  * 즉, 프레임워크는 애플리케이션의 아키텍쳐를 제공한다.
* 프레임워크는 설계의 가변성을 미리 정의하므로 애플리케이션 설계자나 구현자는 애플리케이션에 종속된 부분에 대해서만 설계하는 것으로 작업을 진행할 수 있다.
  * 프레임워크는 애플리케이션 영역에 걸쳐 공통의 클래스들을 정의하여 일반적인 설계 결정을 미리 내려둔다.
* **프레임워크는 즉시 업무에 투입할 수 있는 구체적인 서브클래스를 제공하지만, 코드의 재사용성보다는 설계 자체의 재사용을 중요시**한다.

### 상위 정책과 하위 정책의 패키지 분리
* 프레임워크의 핵심은 추상 클래스나 인터페이스와 같은 추상화이며 이러한 특징이 프레임워크의 재사용성을 향상시킨다.
  * 이는 추상 클래스와 인터페이스가 일관성 있는 협력을 만드는 핵심 재료이기 때문이다.
  * 협력을 일관성 있고 유연하게 만들기 위해, 추상화를 이용하여 변경을 캡슐화해야 한다.
  * 또한, **협력을 구현하는 코드 안의 의존성은 되도록 추상 클래스나 인터페이스와 같은 추상화를 향해야 한다**.
* 상위 정책은 상대적으로 변경에 안정적이며, 세부사항은 자주 변경되는 특징을 갖는다.
* 때문에 상위 정책이 세부사항에 비해 재사용될 가능성이 높고, 실제로도 그래야 한다.
  * 만약 **상위 정책이 세부사항에 의존하는 경우, 상위 정책을 재사용할 때 항상 세부사항도 함께 존재하므로 재사용성이 낮아지게 된다**.
  * 이러한 **문제를 해결하기 위해 고안된 것이 의존성 역전 원칙이며, 상위 정책과 세부사항 모두 추상화에 의존하도록 만들 수 있다**.
  * 이 때, 의존성 역전 원칙의 관점에서 세부사항은 곧 변경을 의미한다.
* 프레임워크는 여러 애플리케이션에 걸쳐 재사용이 가능해야 하므로, 변하는 것과 변하지 않는 것을 서로 다른 주기로 배포할 수 있도록 배포 단위를 분리해야 한다.
  * 이를 위한 **첫걸음은 변하는 부분과 변하지 않는 부분을 별도의 패키지로 분리하는 것**이다.
* **중요한 것은 패키지 간의 의존성 방향이며, 의존성 역전 원칙에 따라 추상화에만 의존하도록 의존성의 방향을 조정**해야 한다.
  * 이를 위해 추상화를 경계로 패키지를 분리하므로, 세부사항을 구현하는 패키지는 항상 상위 정책을 구현하는 패키지에 의존하게 된다.
  * 이를 토대로 상위 정책을 구현하는 패키지는 세부사항을 구현하는 패키지로부터 완벽하게 분리된다.
  * 즉, **상위 정책을 구현하는 패키지는 다른 애플리케이션에 의해 재사용될 수 있는 컨텍스트 독립성을 보장**받는다.
* 나아가 **패키지가 충분히 안정적이고 성숙된다면, 상위 정책 패키지를 하위 정책 패키지로부터 완벽히 분리된 별도의 배포 단위로 만들 수 있다**.
  * 이 시점에 재사용 가능한 기능을 제공하는 새로운 프레임워크가 탄생하게 된다!

### 일관성 있는 협력과 프레임워크의 관계
* 프레임워크는 여러 애플리케이션에 걸쳐 일관성 있는 협력을 구현할 수 있도록 뒷받침한다.
  * 따라서, 일관성 있는 협력이 제공하는 다양한 장점들을 프레임워크에서도 적용할 수 있게 된다.
* 예를 들어, **동일한 프레임워크를 사용하는 여러 애플리케이션에 걸쳐 일관성 있는 코드를 설계하고 구현할 수 있으며, 이해하기도 쉬워진다**.
* 나아가 설계와 함께 코드 자체를 재사용할 수 있게 된다.

### 제어 역전 원리와 프레임워크
* **상위 정책을 재사용할 수 있다는 것은 결국 도메인에 존재하는 핵심적인 개념 사이의 협력 관계를 재사용하는 것**과 같다.
* **객체지향 설계의 재사용성은 개별 클래스가 아닌 객체 간의 공통적인 협력 흐름에서 기인하며, 이 뒤에는 항상 의존성 역전 원리라는 메커니즘이 존재**한다.
  * **의존성 역전 원리는 전통적인 설계 방법과 객체지향을 구분하는 가장 핵심적인 원리**이다.
* 시스템이 진화하는 방향에는 반드시 의존성 역전 원리를 따르는 설계가 존재해야 하며, 그렇지 못한 경우 변경을 적절하게 수용하기 어려울 수 있다.
* **훌륭한 객체지향 설계는 의존성이 역전된 설계이며, 애플리케이션의 의존성이 역전되어 있다면 이는 객체지향 설계를 갖는 것**과 같다.
  * 의존성 역전 원칙은 객체지향 기술에서 다양하게 요구되는 많은 이점 뒤의 하위 수준에 위치하는 기본 메커니즘이다.
* 또한 **의존성 역전 원리는 프레임워크의 가장 기본적인 설계 메커니즘에 해당**한다.
* 이는 **의존성 역전 원칙은 의존성의 방향 뿐만 아니라 제어 흐름의 주체까지도 역전시키기 때문**이다.
  * 예를 들어, 의존성을 역전시킨 객체지향 구조에서는 프레임워크가 애플리케이션에 속하는 서브클래스의 메소드를 호출한다.
  * 따라서 **프레임워크를 사용하는 경우 개별 애플리케이션에서 프레임워크로 제어 흐름의 주체가 이동**하게 된다.
* **이러한 원리를 Inversion of Control(제어 역전) 원리, 또는 Hollywood 원리**라고 한다.
  * 할리우드 원리는 할리우드 캐스팅 담당자가 오디션 대상자에게 우리가 연락할테니 먼저 연락하지 말라고 전달하는 데에서 비롯되었다.
* 이렇듯 프레임워크에서는 일반적인 해결책만 제공하고, 애플리케이션에 따라 달라질 수 있는 세부사항은 비워둔다.
  * 이렇게 **비워진 동작은 훅이라고 부르며, 훅의 구현 방식은 애플리케이션의 컨텍스트에 따라 달라진다**.
  * **훅은 프레임워크 코드에서 호출하는 프레임워크의 특정 부분이며, 재정의된 훅은 제어 역전 원리에 따라 프레임워크가 원하는 시점에 호출**된다.
* **협력을 제어하는 것은 프레임워크이며, 개발자는 이제 프레임워크가 적시에 호출할 것으로 예상되는 코드를 작성하는 역할을 수행**하게 된다.
  * 이렇듯 **과거와 달리 객체지향의 시대에서는 프레임워크가 호출하는 코드를 개발자가 작성하도록 제어가 역전되어, 프레임워크가 제어권을 갖게 된다**.
  * 프레임워크를 사용하는 우리의 코드는 수동적인 존재이며, 프레임워크가 우리의 코드를 호출할 때까지 아무것도 할 수 없이 기다리는 수 밖에 없다.
* 이러한 제어의 역전은 프레임워크로 제어 흐름이 역전되어 개발자로 하여금 불안감에 빠져들도록 할 수도 있다.
  * 그러나 **제어의 역전은 프레임워크의 핵심 개념이며, 코드의 재사용을 가능케하는 강력한 힘이라는 사실을 이해**해야 한다.

### 제어 역전 원리의 결론
* 설계 수준의 재사용은 애플리케이션과, 기반이 되는 소프트웨어 사이의 제어를 바꾸게 한다.
  * 예를 들어, 라이브러리를 사용하는 애플리케이션은 애플리케이션 자체가 라이브러리의 호출을 제어하게 된다.
  * 그러나 프레임워크의 경우, 프레임워크가 제공하는 메인 프로그램을 재사용하고 메인 프로그램에 의해 호출되는 코드를 개발자가 작성하게 된다.
  * 즉, **언제 자신이 작성한 코드가 호출될지 개발자 스스로는 제어할 수 없게 된다**.
* 이는 **제어의 주체가 개발자 자신으로부터 프레임워크로 넘어가는 제어의 역전을 의미**한다.
* 제어가 역전된 환경에서 개발자는 이미 이름과 호출 방식이 결정된 오퍼레이션의 세부사항을 작성해야 한다.
  * 그러나 **이로 인해 결정해야하는 설계는 줄고, 애플리케이션 별로 구체적인 오퍼레이션의 구현 작업만을 수행하면 된다**.