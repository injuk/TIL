# EssenceOfObject-Orientation
## 2022-12-12 Mon

### 추상화를 활용한 복잡성 극복
```
> 현대적인 지하철 노선도는 승객이 꼭 알아야 하는 사실만 표현하고, 알 필요 없는 정보는 무시하는 추상화 기법을 통해 직관적인 형태를 얻게 되었다.
```
* **너무나 복잡한 현실에 비해 사람의 인지 능력은 보잘 것 없으므로, 인간은 본능적으로 이해하기 쉽고 예측 가능한 수준까지 현실을 분해하고 단순화**한다.
  * 예를 들어 현대적인 지하철 노선도는 불필요한 정보를 제거함으로써 단순함을 달성한 추상화의 좋은 예시이다.
  * 이렇듯 **추상화는 현실에서 출발하되, 불필요한 부분은 과감히 제거해가며 사물의 본질을 드러내는 과정**이라고 볼 수 있다.
* 즉, **추상화의 목적은 불필요한 부분을 고의적으로 무시하는 것으로 현실의 복잡성을 극복하는 데에 있다**.
  * 또한 추상화는 너무도 복잡한 현실을 단순화하기 위해 사용하는 인간의 가장 기본적인 인지 수단으로 이해할 수 있다.
* 현대적인 지하철 노선도와 같이, **좋은 추상화는 반드시 본래 목적에 부합해야 한다**.
  * 반면, 현대적인 지하철 노선도는 역의 실제 위치와 역 간의 실제 거리 등 사실적인 지형 정보를 얻는 데에는 적합하지 않을 수 있다.
  * 이렇듯 **어떠한 추상화도 의도된 목적이 아닌 다른 목적으로 사용되는 경우에 오인될 수 있으며, 추상화의 수준과 이익 및 가치는 목적에 의존**한다.
* 상술한 내용을 토대로, 추상화는 다음과 같이 정의할 수 있다.
```
> 어떠한 양상이나 세부 사항, 구조를 더 명확히 이해하기 위해 특정한 절차나 물체를 의도적으로 생략하거나 감추어 복잡도를 극복하는 방법이다.
```
* 나아가 이러한 복잡성을 다루기 위한 추상화는 다음과 같은 두 차원에서 진행될 수 있다.
  1. **구체적인 사물들 사이의 공통점을 취하더ㅚ, 차이는 버리는 일반화를 활용하여 단순화**하기
  2. **중요한 부분을 강조하기 위해 불필요한 세부 사항은 제거하여 단순화**하기
* 무엇보다, **모든 경우에 추상화의 목적은 복잡성을 이해할 수 있는 수준까지 단순화하는 데에 있다**.
* 이렇듯 **객체지향 패러다임은 객체라는 추상화를 통해 현실의 복잡성을 극복**한다.
  * 또한, 객체지향을 통해 **유용하고 아름다운 애플리케이션을 개발하기 위해서는 추상화의 두 차원을 올바르게 이해하고 적용할 수 있어야** 한다.

### 개념으로 그룹화하기
* **객체지향 패러다임에서, 명확한 경계를 가지고 서로 구별할 수 있는 구체적인 사물을 객체라고 지칭**한다.
  * 이러한 객체지향 패러다임에서 구체적이고 실제적인 객체는 수많지만, 이들을 개별 단위로 취급하기에 인간의 인지 능력은 보잘 것 없다.
  * 때문에 **사람은 본능적으로 공통된 특성을 기준으로 객체를 여러 그룹으로 묶으며, 이를 통해 동시에 다룰 가짓수를 줄여 상황을 단순화**한다.
* 이렇듯 **공통점을 토대로 객체를 묶기 위한 분류가 개념이며, 이는 또한 일반적으로 사람이 인식하는 다양한 사물 또는 객체에 적용할 수 있는 관념을 의미**한다.
  * 예를 들어 하늘을 나는 교통수단을 지칭하는 개념은 비행기가 된다.
* **개념을 활용하면 객체를 여러 그룹으로 분류할 수 있으므로, 개념은 또한 공통점을 기반으로 객체를 분류할 수 있도록 하는 일종의 체**와 같다.
* 결국 **각각의 객체는 특정한 개념이라는 그룹의 일원으로 기능하며, 이렇듯 어떤 객체에 임의의 개념을 적용할 수 있을 때 인스턴스라는 표현을 사용**할 수 있다.
  * 예를 들어, 어떤 객체가 임의의 개념 그룹의 일원인 경우 해당 객체를 그 개념의 인스턴스라고 정의할 수 있다.
* 상술한 바를 토대로, 객체는 다음과 같이 정의될 수 있다.
```
> 객체는 특정한 개념을 적용할 수 있는 구체적인 사물이며, 개념이 객체에 적용되었을 때 객체는 해당 개념의 인스턴스가 된다.
```
* **개념은 세상의 수많은 객체들을 거르는 정신적인 체로서 기능하며, 이를 통해 복잡한 객체 세상을 단지 몇 개의 개념만으로 단순화**할 수 있다.
  * 이렇듯 개념은 객체를 분류할 수 있는 틀을 제공하며, 복잡한 객체들은 단지 몇 가지 개념의 인스턴스에 지나지 않게 된다.

### 개념의 세 가지 관점
* 상술했듯 개념은 객체가 어떠한 그룹에 속할지를 결정하며, 어떤 객체에 임의의 개념이 적용되었다는 것은 객체가 그 개념이 부가하는 의미를 만족시켰음을 의미한다.
  * 이 경우, 비로소 객체는 다른 객체들과 마찬가지로 해당 개념의 일원이 될 수 있다.
* 또한, 개념을 이야기 하는 경우에는 다음과 같은 세 관점을 함께 고려할 수 있다.
  1. 심볼: 개념을 가리키는 간략한 이름을 의미한다.
  2. 내연: **개념의 완전한 정의의자 개념의 의미이며, 내연의 의미를 활용하여 객체가 개념에 속하는지 판별**할 수 있다.
  3. 외연: 개념에 속하는 모든 객체의 집합을 의미한다.
* 이 때, **중요한 것은 내연이 개념을 객체에 적용할 수 있는지 여부를 판단하기 위한 조건이라는 점**이다.
* 이렇듯 **개념을 구성하는 세 관점은 객체의 분류 방식에 대한 지침이 되어줄 수 있으나, 사실은 개념을 활용하여 객체를 분류할 수 있다는 사실이 더 중요**하다. 

## 2022-12-13 Tue
### 객체를 분류하기 위한 틀
* **외연의 관점에서, 어떤 객체에 임의의 개념을 적용하는 것은 곧 해당 객체 집합에 새로운 객체를 포함시키는 것**과 같다.
  * 즉, 어떤 객체에 적용할 개념을 결정하는 것은 결국 해당 객체를 이미 개념이 적용된 객체 집합에 추가하는 것이다.
  * 때문에 **객체에 어떤 개념을 적용할지 결정하는 것은 즉 객체를 개념에 따라 분류하는 것**과 같다.
* **객체의 분류는 특정한 객체를 어떤 개념을 적용하는 것으로, 결국 임의의 개념을 따르는 객체 집합에 해당 객체를 포함시키거나 포함시키지 않는 것**과 같다.
* **분류는 객체지향 패러다임의 중요한 개념 중 하나이며, 이는 곧 객체지향 애플리케이션의 품질과 직결**된다.
  * 예를 들어, 객체를 적절한 개념에 따라 분류하지 못한 애플리케이션은 유지보수가 어렵고 변화에 대응하지 못한다.
* 나아가 **적절한 객체 분류 체계는 애플리케이션을 관리하는 개발자가 객체를 쉽게 찾고 조작할 수 있는 정신적인 지도로서 기능**할 수 있다.

### 개념과 추상화의 두 차원
* **개념을 활용하여 객체를 분류하는 과정은 기본적으로 다음과 같은 추상화의 두 차원을 모두 활용**한다.
  1. 고체적인 사물들의 공통점은 취하고, 차이는 버리는 일반화를 활용한 단순화
  2. 사물의 중요한 부분을 강조하기 위해 불필요한 세부 사항은 제거하는 단순화
* 개념은 객체들의 복잡성을 극복하기 위한 추상화 도구이며, 인간은 매 순간 무수한 사물을 개념으로 분류하며 세상을 단순화한다.
  * 즉, **추상화를 통해 사람은 세상의 복잡성을 제어 가능한 수준까지 단순화**할 수 있게 된다.

### 타입과 개념
```
> 타입은 개념이다.
```
* **타입이란 개념을 대체하는 수학적 용어이며, 그 정의는 개념의 정의와 완전히 동일**하다.
  * 즉, **타입은 공통점을 기반으로 객체들을 분류하기 위한 틀**이다.
  * 또한 타입은 개념과 마찬가지로 심볼과 내연, 그리고 외연으로 서술될 수 있다.
  * 나아가 **타입에 속하는 객체는 타입의 인스턴스가 되며, 인스턴스는 타입을 구성하는 외연으로서의 객체 집합의 일원**이 된다.

### 데이터 타입의 시작
```
> 데이터가 어떤 타입에 속하는지 결정하는 것은 해당 데이터에 적용할 수 있는 작업의 종류이다.
```
* **타입이 없는 체계 안에서, 애플리케이션을 구성하는 모든 데이터는 메모리 상에 비트 스트림으로 존재**한다.
  * 또한, 어떠한 메모리 조각에 포함된 값의 의미는 여러 애플리케이션과 같이 자신의 용도에 맞게 사용하는 외부 해석기에 따라 결정된다.
* **타입이 없는 체계에서 0과 1의 비트 스트림은 조작할 방법이 없으므로, 사람들은 자신이 다룰 데이터의 용도와 가능한 행동에 따라 이들을 분류하기 시작**했다.
  * 이렇듯 **컴퓨터 내부에 자리 잡은 데이터를 목적에 따라 분류하기 시작하며 타입 시스템이 탄생**한다.
  * **타입 시스템은 메모리에 저장된 수많은 0과 1에 대해 수행 가능한 작업과 불가능한 작업을 분류하며, 데이터가 잘못 사용되는 것을 방지**한다.
  * 즉, **본질적으로 타입 시스템의 목적은 데이터가 잘못된 방법으로 사용되지 않을 수 있도록 제약을 부과**하는 데에 있다.
* 타입은 또한 다음과 같은 두 가지 특징을 갖는다.
  1. **타입은 데이터가 어떻게 사용되느냐에 대해 이야기**한다.
  2. **타입에 속한 데이터가 메모리 상에서 어떻게 표현되는지는 외부로부터 철저히 감춰진다**.
* 우선, **데이터가 어떤 타입에 속하는지 결정하는 것은 곧 데이터에 적용할 수 있는 작업**이 된다.
  * 즉, **어떤 데이터에 어떤 연산자를 적용할 수 있느냐에 따라 데이터의 타입이 결정**된다.
* 또한, **개발자는 로우 레벨에서 데이터 타입의 표현 방식을 모르더라도 데이터를 사용하는 데에는 전혀 지장이 없다**.
  * **대신 개발자는 해당 데이터 타입을 사용하기 위해서 단지 적용 가능한 연산자만을 기억**하면 된다.
  * 예를 들어, 숫자형 데이터에 적용 가능한 산술 데이터 타입을 이해하고 있다면 메모리 상에 숫자 데이터가 어떻게 저장되는지까지는 알 필요가 없다.
* 이렇듯 **데이터 타입은 메모리 상에서 저장된 데이터의 종류를 분류하기 위해 사용하며, 일종의 메모리 데이터 집합에 대한 메타데이터로서 기능**한다.
  * 때문에 **데이터에 대한 분류는 암묵적으로 해당 데이터에 대해 어떠한 종류의 연산이 수행될 수 있는지 결정**한다.

### 객체와 데이터 타입
* 상술한 전통적인 데이터 타입에서 말하는 타입과 객체지향 패러다임에서 말하는 타입 사이에는 깊은 연관이 존재한다.
  * 예를 들어, 객체지향 애플리케이션을 작성하는 개발자는 객체를 일종의 데이터처럼 사용한다.
  * 때문에 **객체를 타입에 따라 분류하고 타입을 명명하는 것은 곧 애플리케이션에서 사용할 새로운 데이터 타입을 선언하는 것과 같다**고 볼 수 있다.
* 그러나 여전히 객체에서 가장 중요한 것은 객체의 행동이며, 객체의 상태는 행동의 결과로 초래된 사이드 이펙트를 쉽게 표현하기 위해 도입된 추상적인 개념이다.
  * 즉, 객체를 창조할 때 가장 중요한 것은 객체가 협력을 위해 어떤 행동을 할지 결정하는 것이다.
  * 때문에 **객체지향 설계의 핵심은 객체가 협력을 위해 어떠한 책임을 지닐지 결정하는 데에 있다**.
* 이러한 관점에서, 상술한 전통적 타입의 두 가지 특징은 객체의 타입에도 동일하게 적용될 수 있다.
  1. **객체가 임의의 타입에 속하는지 결정하는 것은 객체가 수행하는 행동으로, 임의의 객체 집합이 동일한 행동을 수행한다면 이들은 동일한 타입**일 수 있다.
  2. **객체의 세부 사항은 외부로부터 철저히 감춰지며, 객체의 행동을 효율적으로 수행할 수 있다면 객체의 내부 상태는 어떤 식으로든 표현**될 수 있다.

## 2022-12-14 Wed
### 다시, 행동과 객체
```
> 결국 객체의 타입을 결정하는 것은 객체의 행동 뿐이다.
> 반면, 객체가 어떠한 데이터를 갖는지는 타입 결정에 아무런 영향을 줄 수 없다. 
```
* 상술한 바에 따라, 다음과 같은 두 가지 원칙을 이끌어낼 수 있다.
  1. **객체가 어떠한 행동을 하느냐에 따라 객체의 타입을 결정**할 수 있다.
  2. **객체의 타입은 객체의 내부 표현과는 완전히 무관**하다.
* 즉, **객체의 내부 표현 방식이 완전히 다르더라도 어떠한 객체 집합이 모두 동일하게 행동한다면 그 객체들은 동일한 타입**에 속한다.
  * 이는 **다시 말해 동일한 책임을 수행하는 일련의 객체들이 동일한 타입**에 속함을 의미한다.
* 이렇듯 **임의의 객체를 다른 객체와 동일한 타입으로 분류하는 기준은 해당 객체가 타입에 속한 다른 객체 일원들과 동일한 행동을 하는지의 여부**가 된다.
  * 이는 곧 타입을 나누는 과정에서 객체가 갖는 데이터는 전혀 영향을 주지 못함을 의미한다.
  * 바꿔 말해, 객체가 다른 객체와 동일한 데이터를 갖더라도 행동이 다르다면 이러한 객체 집합은 다른 타입으로 분류되어야 한다.

### 다형성의 시작
* 상술한 바와 같이, 같은 타입에 속한 객체는 행동만 동일하다면 서로 다른 데이터를 가져도 무방하다.
  1. 이 때, **동일한 행동이란 동일한 책임을 의미**한다.
  2. 나아가 **동일한 책임이란, 동일한 메시지 수신을 의미**한다.
  3. 때문에 **동일한 타입에 속하는 객체는 내부의 데이터 표현 방식이 다르더라도 동일한 메시지를 수신하고 이를 처리**할 수 있다.
* 그러나 **동일한 타입에 속하는 객체들은 각각 내부의 데이터 표현 방식이 다르므로, 동일한 메시지를 수신할 수 있더라도 이를 처리하는 방법은 서로 달라진다**.
* **이는 곧 다형성을 의미하며, 다형성이란 동일한 요청에 대해 서로 다른 방식으로 응답할 수 있는 능력을 의미**한다.
  * 즉, 동일한 메시지를 서로 다른 방식으로 처리하기 위해서는 다형적인 객체들이 동일한 메시지를 수신할 수 있어야 한다.
  * 때문에 **다형적인 객체들은 동일한 타입 또는 동일한 타입 계층에 속할 수 밖에 없다**.

### 캡슐화의 시작
```
> 데이터가 캡슐화되지 않고 객체의 인터페이스를 오염시킬 경우, 설계의 유연성은 급격하게 떨어지게 된다.
```
* **데이터의 내부 표현 방식 대신 행동만이 타입 분류의 고려 대상이라는 말은 곧 데이터를 외부로부터 숨겨야한다는 의미로 해석**할 수도 있다.
  * 때문에 **좋은 객체지향 설계는 외부에 행동만을 제공하고, 데이터는 행동을 경계로 숨기는 것이 바람직**하다.
* 이렇듯 **데이터를 행동 뒤로 숨기는 원칙을 캡슐화라고 지칭하며, 이는 또한 객체를 행동에 따라 분류하기 위해 지켜야 할 기본적인 원칙**이다.
* 또한, **행동에 따라 객체를 분류하기 위해서는 객체가 내부적으로 관리할 데이터 대신 외부에 제공할 행동을 다음과 같이 우선적으로 고려**할 수 있어야 한다.
  1. **객체가 외부에 제공할 책임을 최우선적으로 결정**한다.
  2. **결정된 책임을 수행하기 위해 적합한 데이터를 결정**한다.
  3. **데이터를 책임을 수행하기 위해 필요한 외부 인터페이스 뒤로 캡슐화**한다.
* 반면, 데이터를 우선 결정한 후에 이에 맞추어 객체에 책임을 할당하는 데이터 주도 설계 방식은 유연하지 못한 설계를 낳게 된다.

### 객체를 객체답게 만드는 것
* 객체를 결정하는 것은 행동이며, 데이터는 단지 행동을 따르는 존재에 불과하다.
  * 또한, **이는 객체를 객체답게 만드는 가장 핵심적인 원칙**이기도 하다!

## 2022-12-15 Thu
### 일반화와 특수화
* 예를 들어 포유류와 고래라는 각각의 객체가 존재할 경우, 이들은 공통된 행동을 갖기에 개별 타입으로 구분될 수 있다.
  * 반면, **고래 객체는 포유류 객체가 할 수 있는 모든 행동을 할 수 있는 데에 더해 추가적인 행동이 가능**하다.
  * 때문에 **고래 겍체는 포유류 객체의 일종이지만, 일반적인 포유류 객체에 비해서는 더 특화된 기능을 수행할 수 있는 포유류 객체**가 된다.
* **외연의 관점에서 모든 고래 객체는 포유류 객체이므로, 모든 고래 객체는 곧 포유류**이기도 하다.
  * 즉, 고래 타입에 속하는 객체는 포유류 타입에도 속해야 한다.
* 다시 말해 **포유류 객체는 고래 객체를 포괄하는, 더 일반적인 개념이라면 고래 객체는 포유류 객체보다 더 특화된 행동을 하는 특수한 개념**이다.
  * 이렇듯 **고래 객체와 포유류 객체의 관계를 일반화와 특수화 관계라고 지칭**할 수 있다.
* 포유류 객체와 고래 객체의 예시처럼, **타입과 타입 사이에는 일반화와 특수화 관계가 존재할 수 있으며 이들은 항상 동시에 발생**한다.

### 일반화 / 특수화와 객체의 행동
```
> 일반화와 특수화 관계를 결정하는 것은 객체의 상태인 데이터가 아니라 행동이다.
```
* **두 타입 사이에서 일반화와 특수화 관계가 성립하기 위해서는 한 타입이 다른 타입보다 더 특수하게 행동**해야 한다.
  * 반대로, 다른 타입은 반대 타입보다 더 일반적인 행동을 가져야 한다.
  * 이렇듯 **일반화와 특수화 관계에서도 중요한 것은 객체의 내부 상태를 의미하는 데이터가 아닌 객체가 외부에 제공하는 행동**이다.
* 상술한 내용을 토대로, **일반적인 타입이란 특수한 타입이 갖는 모든 행동 중 일부만을 갖는 타입**이 된다.
  * 반대로, **특수한 타입은 일반적인 타입이 갖는 모든 행동에 더해 자신만의 특수한 행동을 제공**한다.
* 즉, 일반화와 특수화는 어디까지나 행동에 관한 것이며 일반적인 타입은 특수화된 타입보다 더 적은 행동을 갖는다.
  * 이 때, **특수한 타입은 일반적인 타입이 할 수 있는 모든 행동을 동일하게 제공할 수 있어야 한다**.

### 일반화 / 특수화 관계와 내연 / 외연
* **타입의 내연은 간접적으로 행동의 개수를 의미하는 반면, 타입의 외연은 객체 집합의 크기를 의미**한다.   
* 이 때, **일반화 및 특수화 관게에서 내연과 외연은 서로 반비례한다는 특징이 존재**한다.
  * 즉, **일반적인 타입은 특수한 타입보다 적은 수의 행동을 갖지만 더 큰 크기의 외연 집합**을 갖는다.
  * 반면, **특수한 타입은 더 많은 행동을 제공하지만 더 작은 크기의 외연 집합**을 갖는다.

### 서브타입과 슈퍼타입
* 일반화 및 특수화 관계는 타입 간의 관계이며, 일반적으로 더 일반화된 타입을 슈퍼타입이라고 지칭한다.
  * 반면, 더 특수한 타입은 서브타입이 된다.
* 두 타입 간의 **슈퍼타입 및 서브타입 관계는 행동에 의해 결정되며, 서브타입은 슈퍼타입에 대해 반드시 행위적 호환성을 만족해야 한다**.
  * 다시 말해, **서브타입은 슈퍼타입의 행위와 호환되므로 서브타입은 슈퍼타입을 대체할 수 있어야 한다**.

### 일반화 / 특수화 관계와 추상화
* **일반화 및 특수화 계층은 객체지향 패러다임에 있어 추상화의 두 번째 차원, 즉 중요한 내용을 강조하기 위한 세부 사항의 생략을 활용하는 예시**이다.
* 나아가 **객체지향 패러다임에서 세상을 바라보는 대부분의 경우에 분류 및 일반화 / 특수화 기법은 동시에 적**용된다. 