# UnitTesting
## 2022-05-07 Sat

## junit 사용해보기
### 어떤 테스트를 작성할 수 있는가?
* 복잡한 메소드를 작성한 경우, 개발자는 이것이 원하는대로 동작하는지 확인하고 싶을 수 있다.
  * 테스트 코드는 이럴 때 작성할 수 있다.
* **복잡한 메소드에서의 테스트 코드를 적게는 수십, 많게는 수백 개 작성해야할 수 있다**.
  * 예를 들어, 코드 상에 존재하는 분기점이나 null, 0과 같은 데이터의 변형을 고려할수록 테스트 코드는 늘어난다.
  * 이러한 **코드의 분기 또는 데이터 변형을 파악하여 테스트를 작성하는 경우, 개발자는 코드의 실제 동작을 더욱 잘 이해할 수 있게 된다**.
* 이렇게 나뉜 여러 조건 중 일부는, 또 다른 조건을 충족했을 경우에만 필요하므로 종속되는 조건들을 하나의 테스트로 묶을 수도 있다.
* **자신이 작성한 코드에 대해 테스트 코드를 작성하는 경우, 개발자는 이미 코드에서 가장 흥미롭고 위험한 영역이 어디인지 대략적으로 알고 있다**.
```
> 새로 작성한 코드에 테스트 코드를 작성하는 경우에는 코드 상에서 가장 신경쓰이는 부분이 무엇인지 알고 있어야 한다.
```

### 단일 경로 커버하기
* 단일 경로를 커버하는 단위 테스트는 다음의 절차를 거쳐 작성할 수 있다.
  1. 메소드 상에서 '흥미로운' 로직을 찾아낸다.
  2. 테스트 메소드의 이름은 테스트의 의도를 담을 수 있는 적절한 이름으로 명명한다.
  3. 로직을 분석하여 반드시 필요한 객체를 파악한다.
  4. 파악한 내용을 토대로 테스트의 '준비' 단계를 작성한다.
  5. 테스트 대상 메소드를 분석한 지식을 바탕으로 실행문을 작성한다.
  6. 테스트 대상 메소드를 분석한 지식을 바탕으로 단언문을 작성한다.
* 이렇듯 테스트 대상 메소드에 대해 파악한 지식을 바탕으로 의도대로 동작하는지 검증하는 테스트를 작성할 수 있다.
  * **코드를 충분히 이해하지 못한 경우, 시간을 들여 코드의 동작을 천천히 이해하고 실질적인 테스트로 점차 진화**시켜야 한다.
* **테스트를 작성했다고 끝이 아니며, 반드시 테스트의 유지보수성을 고려**해야 한다.
```
> 자신이 보기엔 길지 않은 테스트 코드도, 해당 기능을 전혀 모르는 사람이 볼 수도 있다는 사실을 고려해야 한다.
```

### 두 번째 junit 테스트
* junit에서 실행되는 각각의 단위 테스트는 고유한 맥락을 갖는다.
  * 즉, **junit은 결정된 순서로 테스트를 실행하지 않는다**.
  * **모든 테스트는 다른 테스트의 결과에 영향을 받지 않는다**.
  * **junit은 각각의 테스트마다 별도의 테스트 인스턴스를 생성**한다.
* **동일 클래스에 작성한 두 번째 테스트 메소드의 로직이 상당 부분 첫 번째 테스트 메소드와 겹치는 경우, 중복을 제거하기 위해 리팩토링을 적용**한다.
  * 이를 통해 이후의 잠재적인 테스트 코드 중복을 방지할 수 있으며, 결과적으로 유지보수성을 높일 수 있다.

### @Before 어노테이션으로 테스트 초기화하기
* **단일 테스트 클래스의 모든 테스트 코드에 중복되는 공통적인 초기화 코드는 @Before 어노테이션이 할당된 메소드에 옮겨져야 한다**.
  * **각각의 junit 테스트를 실행할 때마다 @Before 어노테이션이 표시된 메소드를 먼저 실행**하게 된다.
  * 이렇듯 공통된 기능을 추출한 초기화용 메소드의 이름은 create 등으로 적절히 명명하도록 한다.
* 초기화 로직을 @Before 메소드로 옮기는 것으로 가독성과 유지보수성을 높일 수 있다.

### junit의 테스트 동작 순서
* junit은 테스트 클래스에서 @Test 어노테이션이 명시된 테스트를 실행하며, 이 과정에서 다음의 순서를 따른다.
  1. 테스트 중 처음에 실행할 테스트를 선정한다.
  2. **새로운 테스트 클래스를 만들되, 해당 클래스의 멤버 변수는 초기화하지 않는 상태**로 둔다.
  3. @Before 메소드를 호출하여 멤버 변수를 적절하게 초기화한다.
  4. 테스트를 실행하고, 실행 결과를 표기한다.
  5. **다른 테스트가 남은 경우, 테스트 클래스를 새로이 생성**한다.
  6. 3.과 4.의 과정을 실행한다.
* **중요한 것은 junit이 매 테스트 마다 테스트 클래스를 신규 생성한다는 사실**이다.
  * junit은 이러한 방식을 통해 각각의 테스트를 독립적으로 만든다.
  * 만일 동일한 인스턴스에서 모든 테스트가 실행되는 경우, 공유되는 테스트 대상 객체의 상태를 정리하는 것도 신경써주어야 했을 것이다.
* 개발자는 임의의 테스트 코드가 다른 테스트에 영향을 주는 것을 최소화하고 싶어하며, 실제로도 그래야만 한다.
  * 때문에 **테스트 클래스에서는 static 필드를 지양**해야 한다.

### 테스트 코드 리팩토링하기
* 테스트 코드 역시 다른 개발자가 볼 수 있다고 생각하고, 언제나 가독성과 유지보수성에 신경써야 한다.
* **테스트 코드는 리팩토링을 통해 준비, 실행, 단언 코드를 한 두줄로 압축하는 것이 이상적**이다.
  * 필요한 경우 @Before 메소드를 체크할 수 있으나, 이는 일반적인 주요 관심사가 아니다.

## 결론
* 테스트는 추후에 더 단순하게 작성될 수 있도록 지속적으로 정리되고 리팩토링되어야 한다.
* 잘 정리된 테스트 코드는 추가적인 단위 테스트를 쉽게 구현할 수 있도록 한다.
  * 이로 인해 **테스트 코드를 늘려가는 것은 어렵지 않을 뿐더러, 나아가 테스트 대상 메소드가 기대한대로 동작하리라는 자신감을 높여줄 수 있다**.
* 좋은 단위 테스트는 단지 assert 문만 작성하는 것으로 끝나는 일이 아니며, 더 많은 훈련이 필요하다.
  * junit은 이를 뒷받침할 작고 많은 기능을 개발자에게 제공한다.