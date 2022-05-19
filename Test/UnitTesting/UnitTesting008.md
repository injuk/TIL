# UnitTesting
## 2022-05-10 Tue

## 깔끔하게 리팩토링하기
* 코드의 중복은 유지 보수 비용과 변경에 대한 리스크를 늘리는 등 비용을 늘리므로, 시스템에서 중복의 양은 최소화되어야 한다.
  * 깔끔하고 좋은 구조를 갖는 코드는 수 분 내에 변경이 가능하지만, 복잡하고 지저분한 코드는 몇 시간을 들여야 한다.
* 낮은 중복성과 높은 명확성이라는 두 마리 토끼는 단위 테스트를 통해 합리적인 비용으로 잡을 수 있다.

### 작은 리팩토링
* 리팩토링이란, 기존 기능은 그대로 유지하면서도 코드의 하부 구조를 건강하게 변형하는 것이다.
  * 즉, 리팩토링은 코드를 옮긴 후에도 시스템이 정상 동작함을 보장하는 것이다.
* 이 때, 단위테스트는 리팩토링 과정에서의 적절한 보호 장치가 되어준다.
* **리팩토링에서 가장 중요한 도구는 명명이며, 그 대상은 클래스와 메소드, 모든 종류의 변수를 가리지 않는다**.
  * 명확성은 코드의 의도를 선언하는 것이므로, 좋은 이름은 코드의 의도를 전달하는 가장 좋은 수단이다.
* 저수준의 세부사항이 코드의 의도를 어지럽히고, 고수준의 정책을 이해하고 있다면 저수준의 세부사항을 잘 명명된 별도의 메소드로 추출할 수 있다.
  * 이를 통해 코드를 읽어내려가면서 관심은 분산되지 않을 수 있다.
* 코드를 이리저리 옮기는 과정에서 기존 기능들은 쉽게 깨지므로, 자신감을 갖고 코드를 변경할 수 있도록 돕는 수단이 필요하다.
* 테스트 코드가 이러한 역할을 수행해줄 수 있으며, 작은 코드 변경이 있을 때 테스트 집합을 빠르게 실행하여 재확인할 수 있다.
  * 이렇듯 코드를 안전하게 옮길 수 있도록 하는 능력은 단위 테스트의 가장 중요한 이점이다.
  * **새로운 기능을 안전하게 추가할 수 있고, 좋은 설계를 유지하면서도 변경**할 수 있다.
* **충분한 테스트가 없으면 코드를 변경하기 어려우며, 이러한 변경은 수반되는 리스크도 매우 높다**.

### 임시 변수의 용도
* 임시 변수는 일반적으로 다음과 같은 용도로 사용된다.
  1. 값비싼 비용의 계산 값을 캐싱
  2. 메소드 바디에서 변경되는 것들을 수집
* **임시 변수의 또 다른 용도는 적절한 명명을 통해 코드의 의도를 명확히하는 것**이다.
  * 이 경우, 임시 변수가 한 번만 사용된다고 해도 유효한 선택에 해당한다.

### 자동 리팩토링의 활용
* 대부분의 IDE는 수십 가지의 리팩토링 자동화 기능을 내장하고 있으며, 이것들은 배우고 사용할 만한 가치가 충분하다.
* **자동 리팩토링을 활용하는 경우, 코드를 변형하는 것은 물론 수동으로 수정했을 때의 실수를 미연에 방지하여 시간을 절약**해준다.
  * 자동화된 리팩토링 기능은 너무나 훌륭하므로, 지저분한 일은 컴퓨터에게 맡기고 개발자는 코드의 정상 동작만 확인하는 데에 집중하는 것이 바람직하다.
* 어떠한 경우라도 수동 리팩토링은 실수로 인한 문제가 발생하기 쉬우므로, 가능하다면 자동화된 리팩토링 도구를 활용해야 한다.
* 또한 **어떠한 경우라도 메소드를 추출하는 등 리팩토링을 적용할 때에는 기존 동작에 문제가 없는지 확인**해야 한다.
  * 개발자는 이 과정에서 단위 테스트를 통해 행복해질 수 있으므로, 리팩토링 과정에서는 항상 테스트를 실행하는 것이 바람직하다.

### 과한 리팩토링?
* 리팩토링 과정에서 한 번의 반복으로 끝나는 코드가 같은 반복을 여러 번 순회하는 형태로 변경될 수 있다.
* 일반적으로, 개발자는 이러한 반복을 무의미한 반복으로 보고 리팩토링 자체에 의문을 가질 수 있다.
* 이러한 리팩토링이 적용되었을 때의 장점은 **메소드가 즉시 이해될 수 있을 정도로 깔끔하게 정리**된다는 것이다.
  * 이전 버전의 코드는 훨씬 더 주의를 기울여야 하고, 메소드의 의도를 혼동하기 쉬운 가능성을 내포하는 경우가 많다.
  * 리팩토링을 통해 각 세부 구현 사항은 도우미 메소드에 숨겨지며, 도우미 메소드는 명확하고 고립된 방식으로 잘 표현된다.
* 반면, 우려되는 단점은 실행 시간의 증가로 성능이 떨어질 수 있다는 점이다.
  * 그러나 일반적으로 데이터가 적은 수준이라면 성능은 상상하는 것만큼 나빠지지 않는다.
* **성능이 즉시 문제가되지 않는 경우, 어설픈 최적화를 적용하는 것으로 시간을 낭비하는 것보다는 코드를 깔끔하게 유지하는 것이 바람직**하다.
  * 최적화된 코드는 일반적으로 가독성이 낮고, 유지 보수 비용이 증가하고, 유연하지 않은 설계를 갖는 경우가 많다.
  * 반대로 **깔끔한 설계는 성능을 위해 최적화를 시작하는 과정에서 즉시 대응할 수 있도록 하는 최선의 준비에 해당**한다.
  * 깔끔한 설계는 코드를 이동시킬 수 있는 유연성을 제공함은 물론, 최적화를 위해 다른 알고리즘을 적용하는 데에도 수월하다.
* 성능이 당장 문제되는 것이 확인되었다면, 우선 문제가 얼마나 심각한지 실제 성능을 측정하도록 한다.
  * 예들 들어 작은 테스트 코드를 작성하여 예전 코드의 속도가 얼마나 빠르고, 리팩토링한 코드는 얼마나 느린지 파악하여 비교할 수 있다.

## 결론
* 대량의 코드를 빠르게 작성하는 것은 쉽지만, 결과의 가독성과 유지보수성은 떨어지는 경우가 많다.
* **단위 테스트는 겉보기 동작을 깨트리지 않고서도 코드를 깔끔하게 유지해주는 보호 장치를 제공**한다. 