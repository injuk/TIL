# Refactoring
## 2022-04-14 Thu

## 테스트 구축
* **리팩토링은 혼자서는 존재할 수 없으며, 제대로 된 리팩토링을 위해서는 반드시 견고한 테스트 스위트가 뒷받침되어야 한다**.
* 무엇보다, 좋은 테스트 코드를 작성하는 일은 테스트 코드 작성에 드는 시간을 포함하더라도 개발 효율을 높여준다.

### 자가 테스트 코드의 가치
* 개발자들의 업무를 살펴보면, 실제로 코드를 작성하는 시간의 비중은 크지 않다.
  * 현재 상황을 파악하거나, 설계를 고민하거나, 대부분의 시간은 디버깅에 사용하곤 한다.
  * 버그 역시 수정 자체는 오래 걸리지 않으나, 버그를 찾는 시간은 끔찍하고, 길고, 고통스럽다.
  * 최악의 경우, 버그를 수정하는 과정에서 새로운 버그를 심기도 한다.
* 잘 작성된 테스트 코드가 뒷받침되는 경우, 다음과 같은 이유에서 디버깅 시간은 크게 줄어들고 생산성이 늘어난다.
  1. 모든 테스트는 완전히 자동화되며, 결과까지 스스로 검사한다.
  2. **테스트를 자주 수행하던 와중 테스트에 통과하지 못한 경우, 이전 작업 분량에서부터 추가된 코드 사이에 버그가 존재함**을 알 수 있다.
  3. 테스트를 자주 수행하므로 의심되는 코드의 양은 많지 않고, 기억도 생생하니 금방 디버깅할 수 있다.
* 이렇듯 **자가 테스트 코드 자체 뿐만 아니라, 테스트를 자주 수행하는 습관도 버그를 찾는 강력한 도구가 되어준다**.
```
> 모든 테스트는 완전히 자동화되고, 결과까지 스스로 검사할 수 있어야 한다.
> 테스트 스위트는 강력한 버그 검출 도구이며, 디버깅에 드는 시간을 대폭 줄여줄 수 있다.
```

### 테스트 코드 작성의 어려움
* 테스트 코드를 작성하기 어려운 요인 중 하나는 다른 개발자를 설득하는 일이며, 크게 다음과 같은 사유가 있다.
  1. 테스트를 작성하려면 소프트웨어 자체적인 기능 외의 코드를 상당량 작성해야 한다.
  2. 테스트 작성법을 배운 적이 없는 개발자가 많다.
* 물론 수동으로 진행하는 테스트는 지겨운 일이지만, 자동화하는 데에 성공한다면 테스트 코드를 작성하는 재미를 느낄 수 있다.

### 테스트 주도 개발
```
> 테스트 주도 개발이란 테스트 코드부터 작성하는 습관을 바탕으로 창시된 기법이다.
```
* **테스트 코드를 작성하기 가장 좋은 시점은 프로그래밍을 시작하기 전**이다.
* 테스트 코드를 작성하다 보면 원하는 기능을 추가하기 위해 무엇이 필요할지 고민하게 된다.
  * 즉, **구현보다 인터페이스에 집중하게 된다는 장점이 수반**된다.
* 또한 테스트 코드를 모두 통과하는 시점은 개발이 완료되는 시점이므로, 코딩이 완료되는 시점을 정확히 판단할 수 있다.
* 테스트 주도 개발에서는 크게 다음과 같이 분류된 과정을 따라 개발을 진행한다.
  1. 처음부터 통과하지 못한다는 사실이 자명한 테스트 코드부터 작성한다.
  2. 테스트를 통과할때까지 코드를 작성한다.
  3. 테스트에 통과했다면, 결과 코드를 적절히 리팩토링한다.
  4. 상술한 과정을 짧은 주기로 반복하여 개발한다.
* **상술한 과정은 테스트 - 코딩 - 리팩토링 과정을 짧은 시간에도 여러 차례 반복적으로 수행하므로, 코드를 생산적이면서도 차분하게 작성해나갈 수 있다**.

### 리팩토링과 테스트 코드
* **리팩토링에는 테스트가 반드시 필요하므로, 제대로 된 리팩토링을 위해서는 테스트 코드를 반드시 작성**해야 한다.
  * 코드를 작성할 때에는 반드시 테스트 코드도 함께 작성한다.
  * 만약 **테스트 코드가 갖추어지지 않은 코드를 리팩토링해야 한다면, 리팩토링하기에 앞서 자가 테스트 코드를 작성**하도록 한다.

### 테스트 코드 작성하기
```
> 테스트는 최대한 자주 테스트해야 한다.
> 현재 작업 중인 테스트는 최소한 몇 분 간격으로 테스트하며, 적어도 하루에 한 번은 전체 코드를 테스트한다.
```
* 테스트 코드를 작성할 때에는 크게 픽스쳐 작성과 검증의 두 단계로 나눌 수 있다.
  * **픽스쳐란, 테스트에 필요한 데이터 또는 객체를 의미**한다.
  * 픽스쳐의 검증을 위해서도 별도의 어서션(assertion) 라이브러리를 사용할 수 있다.
* 각각의 테스트 결과를 확인하며, 보다 확실하게 테스트하기 위해 테스트 코드에 의도적으로 오류를 주입하여 실패하는 테스트를 확인하는 것도 좋다.
* **테스트 라이브러리는 어떠한 것을 사용해도 무방하며, 중요한 것은 테스트의 성공 실패를 빠르게 체크할 수 있어야한다는 점**이다. 

### 테스트 진행
```
> 테스트를 완벽하게 만들기 위해 아무것도 못하는 것보다, 불완전한 테스트라도 작성하여 실행하는 것이 바람직하다.
```
* 테스트는 위험 요인을 중심으로 작성해야 하며, 단순히 필드를 읽고 쓰기만 하는 메소드는 테스트할 필요가 없다.
  * **테스트의 목적은 현재 또는 향후에 발생할 수도 있는 버그를 찾는 데에 있다**.
  * 즉, 잘못될까봐 걱정되는 영역을 집중적으로 테스트해야 한다.
* 하나의 테스트 케이스에서 공유되는 픽스쳐가 존재하더라도, 하위 테스트 케이스가 하나의 객체 참조를 공유하도록 하지 말아야 한다.
  * 테스트끼리 상호작용하는 공유 픽스쳐는 테스트와 관련된 버그 중 가장 지저분한 유형에 속한다.
  * 차라리 beforeEach 등의 구문을 통해 각 테스트 케이스가 수행되기 전에 새로운 픽스쳐를 생성하도록 하는 것이 좋다.
  * 이렇듯 **개별 테스트를 실행할 때마다 픽스쳐를 새로 만드는 것은 모든 테스트를 독립적으로 구성하는 첫걸음**이다.
* 개별 테스트마다 새로운 픽스쳐를 생성하더라도 테스트 코드는 눈에 띄게 느려지지 않는다.
  * 테스트 성능에 악영향을 끼쳐 문제가 되는 경우, 픽스쳐를 불변으로 만들어 공유하는 것이 바람직하다.
  * **사실 불변 픽스쳐는 공유해도 무방**하다.
* **beforeEach와 같은 구문을 사용하는 것은 테스트 케이스로부터 중복 코드를 제거하고 표준 픽스쳐를 사용한다는 사실을 다른 개발자들에게 명시**한다.

### 경계 조건 검사하기
```
> 문제가 생길 수 있는 경계 조건을 생각해보고, 해당 부분을 집중적으로 테스트하는 것이 바람직하다.
```
* 언제나 사용자가 코드를 의도대로 사용하는, 꽃길만 걷는 테스트는 큰 의미가 없다.
* 따라서 **의도적으로 사용자의 입력 범위를 벗어나는 경계 지점에서 발생한 문제에 대해 어떤 일이 발생하는지 확인하는 테스트도 작성하는 것이 바람직**하다.
  * 예를 들어 양수 필드에 음수를 입력하거나, 문자열을 입력해보는 식의 테스트를 작성할 수 있다.
* **경계를 확인하는 테스트를 작성하는 것으로 프로그램에서 특이 케이스를 어떻게 처리해야하는지 다시 한 번 생각**해볼 수 있다.
* **경계 조건을 벗어나는 코드를 처리하는 방법은 많지만, 유효성 검사 코드가 너무 많으면 이미 확인한 것을 중복으로 검증하는 문제가 발생할 수도 있다**.
  * 어떠한 방식으로 유효성을 검증하더라도 경계 조건을 테스트하다 보면 이러한 점을 놓치지 않고 다시 한 번 확인하는 장점이 있다.
* 반면, 이러한 테스트는 리팩토링 전에는 작성하지 않아도 무방하다.
  * 리팩토링은 겉보기 동작에 영향을 주지 않아야 하지만, 경계 조건 테스트는 애당초 겉보기 동작에 포함되지 않는다.
  * 따라서, 경계 조건에 대응하는 동작이 리팩토링에 의해 변하는지 신경 쓸 필요도 없다.

### 테스트 수준
* 아무리 테스트 코드를 많이 작성하더라도 완벽히 버그 없는 프로그램을 만들 수는 없으며, 장황한 테스트 케이스는 의욕을 떨어트릴 수 있다.
  * 그러나 테스트는 전체적인 프로그래밍의 속도를 높여준다는 사실에는 변함이 없다.
* 때문에 **모든 부분에 대해 테스트 코드를 작성하는 것보다는 위험한 부분에 집중하는 것이 바람직**하다.
* **코드에서 처리 과정이 특히 복잡한 부분과, 함수의 동작에서 오류가 생길만한 부분을 집중적으로 테스트**한다.
  * 이렇게 작성된 테스트가 모든 부분을 체크하지는 못할지언정, 안심하고 리팩토링할 수 있도록 하는 보호막은 되어줄 수 있다.
* 이렇게 작성된 테스트 코드를 기반으로 리팩토링을 진행하며 프로그램을 더욱 깊게 이해할 수 있게 되고, 이는 더 많은 버그를 찾아내는 선순환으로 이어진다.
  * **테스트 스위트를 갖춘 후부터 리팩토링을 진행하되, 리팩토링 도중에도 계속해서 테스트 케이스를 추가하는 것이 바람직**하다.

### 올바른 테스트 습관
```
> 버그를 발견한 즉시 그 버그를 잡아낼 수 있는 단위 테스트 케이스를 작성하자.
```
* **리팩토링 못지 않게 테스트도 중요한 주제이며, 리팩토링에 반드시 필요한 토대일 뿐만 아니라 그 자체로도 프로그래밍에 지대한 기여를 하는 기반**이다.
  * 또한, 테스트는 뛰어난 개발자라면 최우선으로 관심을 갖는 주제이기도 하다. 
* 일반적으로 한 번에 완벽한 테스트 스위트를 얻어낼 수는 없으므로, 다른 프로그래밍 활동과 마찬가지로 테스트 역시 반복적으로 진행해야 한다.
* **제품 운영 코드 못지 않게 테스트 코드 역시 지속적으로 작성되고, 추가되고, 보강되어야 한다**.
* **버그를 발견했거나 버그 리포트를 받았다면, 버그를 고치기 전에 해당 버그를 명확히 잡아내는 테스트 코드를 작성하는 습관을 들이는 것이 바람직**하다.
* 충분한 테스트 코드를 판단하는 기준은 없으며, 테스트 커버리지 역시 테스트하지 않은 영역을 찾아낼 수 있을 뿐 테스트 케이스 자체의 품질을 판단할 수는 없다.
  * 즉, 테스트 스위트가 충분한지 평가하는 기준은 늘 주관적이다.
  * 때문에 리팩토링 후 테스트를 모두 통과한 것만으로도 리팩토링 과정에서 버그가 발생하지 않았음을 확신할 수 있다면 이미 충분한 테스트 스위트라고 할 수 있다.
* 테스트 코드를 너무 많이 작성하는 경우도 충분히 발생할 수 있다. 
  * 예를 들어 **제품 코드보다 테스트 코드의 수정에 시간을 더 쏟아 전체적인 개발 속도가 더뎌진다면 테스트 코드가 너무 많은 것은 아닌지 의심**하자.
  * 그럼에도 테스트 코드가 너무 많은 경우보다는 너무 적은 경우가 훨씬 많다!