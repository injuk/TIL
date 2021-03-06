# UnitTesting
## 2022-05-08 Sun

## 테스트 조직하기
* 밑바닥부터 시작하는 테스트는 새롭고 익숙하지 않은 환경에서 잘못된 행동을 하거나, 진행하는 과정 자체가 어려울 수 있다.
* 때문에 어떻게 테스트를 시작할지 시간을 들여 생각하는 과정은 필수적이며, 여러 junit의 기능을 활용하여 테스트 코드를 잘 조직하고 구조화할 수 있어야 한다.

### AAA로 테스트 일관성 유지하기
* 앞서 다룬 ScoreCollection 테스트에서, 테스트 코드는 준비, 실행, 단언의 세 단계를 통해 명시적으로 구분되었다.
  * 이를 준수하는 경우, 각 단계를 식별하기 위해 작성하는 주석은 더 이상 필요가 없다.
* AAA는 다음과 같은 준비, 실행, 단언의 세 단계로 구성된다.
  1. 준비: 테스트 코드를 실행하기 전에 시스템이 적절한 상태를 갖는지 확인하는 단계이다.
     * 객체를 생성하거나, 생성된 객체와 소통하거나, 다른 API를 호출하는 과정이 포함될 수 있다.
     * 시스템이 이미 필요한 상태를 만족하고 있는 경우, 해당 단계는 생략될 수도 있다.
  2. 실행: 테스트 대상 코드를 실제로 실행하는 단계이며, 일반적으로 단일 메소드 호출 형태를 따른다.
  3. 단언: 실행된 코드가 기대한 대로 동작했는지 확인한다.
     * 실행한 코드의 반환값 또는 그 외의 필요한 객체들의 새로운 상태를 검사한다.
     * 또, 테스트한 코드와 다른 객체들 사이의 의사소통 자체를 검사할 수도 있다.
* **테스트 코드 상에 작성된 각 단계를 구분하는 개행은 전체 테스트 코드를 훨씬 빠르게 이해하도록 하는 필수적인 도구**이다.
* 때에 따라 준비, 실행, 단언 이외의 네 번째 단계인 '사후' 단계가 필요하기도 하다.
  1. 사후: **테스트를 실행하는 과정에서 어떠한 자원을 할당했다면, 해당 자원이 테스트 이후에 잘 정리되었는지 확인**한다.

### 동작 테스트와 메소드 테스트
```
> 테스트를 작성하는 경우, 집중하여 테스트해야할 것은 클래스의 동작이다.
> 개별 메소드를 테스트한다는 마음가짐은 지양하는 것이 바람직하다.
```
* **단위 테스트를 작성하는 경우, 우선 전체적인 관점에서 시작**해야 한다.
* **개별 메소드를 테스트하는 것이 아닌, 클래스의 종합적인 동작을 테스트할 수 있도록 해야 한다**.

### 테스트와 운영 코드의 관계
```
> 테스트 코드는 운영 코드에 의존하지만, 운영 코드는 테스트 코드의 존재를 모른다.
> 그러나 이것은 테스트를 작성하는 행위 자체가 운영 시스템의 설계에 전혀 영향을 주지 못함을 의미하는 것은 아니다.
```
* junit의 테스트는 검증 대상인 운영 코드와 같은 프로젝트에 위치할 수 있다.
  * 그러나 **테스트는 주어진 프로젝트 안에서 반드시 운영 코드와 분리되어야 한다**.
* 더 많은 단위 테스트를 작성할수록 설계를 변경했을 때 테스트 작성이 훨씬 용이해지는 경우가 늘어날 수 있다.
* **개발자의 삶은 테스트 친화적인 설계를 채택할수록 편해지며, 설계 자체도 좋아진다**.

### 테스트와 운영 코드의 분리
* 운영 소프트웨어를 배포할 때 테스트를 함께 포함할 수도 있지만, 대부분의 프로젝트는 이 방식을 따르지 않는다.
  * 이 경우, 로딩하는 JAR 파일은 부풀려지고 코드 베이스의 공격 표면 역시 늘어난다.
* **테스트의 배포 여부를 고려하기보다는 테스트를 운영 코드와 같은 프로젝트에 정의할지 결정하는 것이 바람직**하다.
  1. 테스트를 운영 코드와 같은 디렉토리, 및 같은 패키지에 작성하기
     * 구현하기 쉽지만, 어느 누구도 실제로 이 방식을 채택하지 않는다.
     * 이러한 정책을 사용하는 경우, 실제 배포시 테스트 코드를 제거하는 스크립트가 필요해진다.
  2. 테스트를 별도 디렉토리로 분리하되, 운영 코드와 같은 패키지 구조를 따르기
     * 대부분의 프로젝트에서 채택되는 방식이며, test 디렉토리의 구조는 운영 코드의 디렉토리 구조를 반영한다.
     * **각 테스트는 검증 대상과 같은 패키지를 갖게 되므로, 테스트 클래스는 패키지 수준의 접근 권한을 갖게 된다**.
  3. 테스틀 별도 디렉토리로 분리하되, 운영 코드와 유사한 패키지 구조를 따르기
     * **테스트 코드를 운영 코드의 패키지와 다르게 정의하는 경우, 공개 인터페이스만 활용하여 테스트 코드를 작성**해야 한다.
     * 많은 개발자가 설계 시 의도적으로 해당 정책을 채택하곤 한다.

### 내부 데이터 노출과 내부 동작 노출
```
> 일부 개발자들은 테스트를 작성하는 과정에서 운영 코드의 공개 인터페이스만 사용해야한다고 믿는다.
```
* 공개되지 않은 메소드를 테스트 코드에서 호출하는 것은 정보 은닉 원칙을 위배하는 것으로 보일 수 있다.
* 이렇듯 **비공개 코드를 호출하는 테스트는 그 자체로 구현의 세부사항과 결합**된다.
  * 즉, 세부사항이 변경되면 기술적으로는 공개된 메소드라고 해도 테스트가 깨질 수 있다.
* 이렇게 세부사항을 테스트하려는 시도는 다음과 같은 이유에서 소프트웨어의 저품질로 이어질 수 있다.
  1. 세부사항을 모두 테스트하기 위해 많은 테스트 코드를 작성하였으며, 각 코드는 과도하게 세부적인 구현 사항을 알게 된다.
  2. 세부사항 중 코드 몇몇이 일부분 수정된다.
  3. 수많은 테스트가 깨지고, 개발자는 이를 고치기 위해 많은 비용을 들인다.
  4. 테스트가 더 많이 깨지는 것을 경험하는 과정 속에서, 개발자는 점차 리팩토링을 꺼리게 된다.
  5. **결과적으로 리팩토링이 줄어들며, 이는 코드 품질의 퇴화로 직결**된다. 
* 상술한 강한 결합도 때문에 단위 테스트에 많은 시간을 투자하는 것을 거부하는 팀들 역시 존재한다.
* 이 밖에도 개발자는 테스트를 작성하기 위해 객체에 대해 많은 세부사항을 묻게 되는 경우가 많다.
  * 예를 들어, 내부 필드에 대한 단언을 위해 없던 게터를 작성하는 경우가 있다.
* **테스트를 위해 내부 데이터를 노출하는 것은 테스트와 운영 코드 사이의 과도한 결합도를 초래**한다.
  * 이 과정에서 내부의 동작이 노출되는 것은 또 다른 문제이다. 
* **커다란 클래스는 수많은 복잡한 private 메소드를 포함하기도 하며, 개발자는 이를 테스트하고 싶은 충동을 느끼기 쉽다**.
  * 예를 들어 같은 패키지에 있다면 해당 클래스에 대해 패키지 수준으로 접근하거나, 다른 패키지에 있다면 Java 리플렉션을 활용할 수 있다.
  * **그러나, 둘 다 하지 않는 것이 좋다**!
* 이렇듯 **내부의 행위에 대한 테스트 충동은 그 자체로 설계에 문제가 있음을 시사**한다.
  * 테스트하고 싶은 흥미로운 내부의 행동은 대부분의 경우에 단일 책임 원칙을 위배한다.
  * **단일 책임 원칙은 클래스가 작고 단일 목적을 지녀야함을 의미하므로, 가장 좋은 해결책은 흥미로운 private 메소드를 별도의 클래스로 이동하는 것**이다.
  * 이를 통해 해당 메소드는 새로 정의된 클래스의 유용한 public 메소드가 될 수 있다.

### 집중적인 단일 목적 테스트의 가치
* 짧은 여러 테스트를 하나의 큰 테스트로 합치고 싶은 충동에 빠질 수 있다.
  * 이 경우, 하나의 큰 케이스로 합치되 각각의 테스트 케이스를 주석으로 구분한다.
  * 이로 인해 테스트를 분리했을 때 실행되는 반복적인 초기화의 부담을 줄여줄 수 있다.
* 그러나 **이는 junit이 제공하는 테스트 고립의 이점을 완전히 잃게 된다**.
  * **다수의 케이스는 별도의 junit 테스트 메소드로 분리하되, 각각 검증하는 동작을 표현할 수 있는 이름을 붙여주는 것이 바람직**하다.
* 테스트를 여러 케이스로 분리하는 것으로 다음의 이점을 얻을 수 있다.
  1. 단언이 실패했을 때 실패한 테스트의 이름이 표시되므로, 어느 동작에서 문제가 발생했는지 빠르게 파악할 수 있다.
  2. 실패한 테스트를 해독하는데에 들어가는 시간을 줄일 수 있다.
     * 이는 junit이 각 테스트를 별도의 인스턴스로 실행하기 때문이다.
     * 또한, **현재 실패한 테스트에 대해 다른 테스트의 영향을 제거**할 수 있다.
  3. **모든 케이스가 실행됨을 보장할 수 있다**.
     * 단언이 실패한 경우, 현재의 테스트 메소드는 중단된다.

### 문서로서의 테스트
```
> 단위 테스트는 개발자가 작성한 클래스에 대한 지속적이고 신뢰할 수 있는 문서 역할을 수행할 수 있어야 한다.
> 테스트는 코드 자체로는 쉽게 설명할 수 없는 것들을 알려줄 수 있다.
```

### 일관성 있는 이름으로 테스트 문서화하기
* **테스트 케이스를 커다란 단일 메소드로 결합할수록 테스트의 이름 역시 너무 일반화되어 의미를 잃어 간다**.
  * 더 작은 테스트로 이동할수록 각각의 테스트는 분명한 행동에 집중할 수 있게 된다.
  * 또한, 각각의 테스트에 명명한 이름에 더 많은 의미를 부여할 수 있게 된다.
* **테스트에 대한 맥락을 제시하기보다, 어떠한 맥락에서 호출된 일련의 행동이 어떠한 결과를 반환하는지 명시하는 것이 이상적**이다.
  * 더 분명한 이름은 개발자가 테스트 내용이 무엇인지 파악하는지 도울 수 있다.
* **합리적인 테스트의 이름은 단어 일곱 개 정도로 구성되며, 너무 긴 테스트 이름은 설계의 결함을 의미할 수도 있다**.
* 멋지고 설명적인 이름은 다음의 양식을 따를 수 있다.
  1. doingSomeOperationGeneratesSomeResult: 어떠한 동작을 하면 어떠한 결과가 나온다.
  2. someResultOccursUnderSomeCondition: 어떠한 결과는 어떠한 동작에서 발생한다.
  3. givenSomeContextWhenDoingSomeBehaviorThenSomeResultOccurs: 행위 주도 개발에서 가리키는 given when then 양식을 따른다.
     * 그러나 너무 길고 복잡하므로, given을 제거하여 whenDoingSomeBehaviorThenSomeResultOccurs 정도로 줄일 수 있다.
     * 이는 doingSomeOperationGeneratesSomeResult 명명과 일치하는 방식에 해당한다.
* **어떠한 명명법을 사용해도 무방하지만, 중요한 것은 일관성을 유지하여 다른 사람에게 의미 있는 테스트 코드를 작성하는 것**이다.

### 테스트를 의미 있게 하기
* 다른 사람이 테스트의 역할 또는 동작을 파악하기 어려워하는 경우, 다음과 같은 방식을 통해 테스트 코드를 개선해야 한다.
  1. 테스트의 이름을 개선하기
  2. 지역 변수의 이름을 개선하기
  3. 의미 있는 상수를 도입하기
  4. 햄크레스트 단언을 활용하기
  5. 커다란 테스트를 작게 나누어 보다 집중적인 테스트로 만들기
  6. 테스트의 복잡한 코드들을 도우미 메소드나 @Before 메소드로 이동하기
* 즉, **테스트의 이름과 코드를 리팩토링하여 주석 없이도 테스트가 시사하는 바를 알 수 있도록 하는 것이 바람직**하다.

### 공통 초기화의 @Before
* 연관된 행동들에 대해 더 많은 테스트를 추가하는 경우, 상당한 테스트 코드가 같은 초기화 부분을 공유하곤 한다.
* 이 경우, **@Before 메소드를 적절히 활용하는 것으로 중복된 코드를 상당 부분 줄일 수 있다**. 
* **@Before 메소드는 다른 테스트 메소드의 실행에 앞서 실행되며, 여러 메소드가 존재할 수 있다**.
  * 예를 들어 각 테스트 전에 어떤 파일을 삭제해야 하는 경우, 두 개의 @Before 메소드를 작성할 수 있다.
  * 그러나 **파일을 삭제하는 메소드와 인스턴스를 생성하는 메소드 사이의 순서는 보장되지 않는다**.
  * @Before 메소드에 포함된 로직의 순서가 중요한 경우, 단일 @Before 메소드에 작성하도록 한다.
* 이렇듯 **junit은 소스 코드에 명시된 것과는 무관한 순서로 테스트 메소드를 실행**한다.
* @Before 메소드는 클래스 내부에 존재하는 모든 테스트에 대해 적용되므로, 해당 클래스에 있는 모든 테스트에 앞서 실행되어야 하는 코드만 작성해야 한다.

### 공통 정리의 @After
* 드물게 @After 메소드가 필요한 경우도 있으며, 이는 각 테스트를 실행한 후에 호출된다.
  * 또한, **테스트가 실패했더라도 호출**된다.
* @After 메소드는 데이터베이스 연결과 같이 각 테스트에서 발생하는 부산물들을 정리하는 역할을 수행한다.

### BeforeClass와 AfterClass 어노테이션
* 일반적으로 테스트 수준의 초기화인 @Before로 충분하지만, 매우 드물게 테스트 클래스 수준의 초기화가 필요할 수 있다.
* @BeforeClass 어노테이션은 이 경우에 사용할 수 있으며, 클래스에 존재하는 어떠한 테스트를 처음 실행하기 전에 최초 한 번만 호출된다.
  * @AfterClass는 반대로 해당 클래스의 테스트가 종료된 후 한 번만 호출된다.

### 테스트를 의미 있게 유지하기
* 개발자는 일반적으로 모든 테스트가 항상 통과하는 것을 기대하며, 버그가 존재하는 경우에는 테스트 한 두개는 반드시 실패하기 마련이다.
* 실패하는 테스트가 발견된 경우, 곧바로 이를 고쳐서 모든 테스트가 항상 통과하도록 유지해야 한다.
  * **항상 녹색으로 유지하는 습관이 운영 코드를 변경해야하는 경우 코드에 오류가 없도록 지켜줄 수 있다**.

### 테스트를 빠르게 유지하기
```
> 대부분의 단위 테스트는 매우 빨라야 한다.
```
* 테스트를 실행하여 항상 녹색을 유지하는 것에 집착하여, 필요하다고 판단하는 테스트만 실행하는 경우가 있다.
* 그러나 **실행되는 테스트 개수를 한정하는 것은 더 큰 문제로 이어질 수 있다는 단점이 수반**된다.
  * 예를 들어, 전체 테스트 스위트에서 피드백을 받지 않는 기간이 길어질수록 애플리케이션에서 어떤 부분을 깨트리는 코드를 작성할 확률이 높아진다.
  * 이러한 유형의 문제는 즉시 대응하는 상황과 비교했을 때 찾는 과정에 상당한 비용이 소모되기 쉽다.
* **테스트 코드에 데이터베이스처럼 느린 자원에 접근하는 부분이 없는 경우, 전체 테스트를 항상 실행하는 것은 크게 어렵지 않다**.
  * 일부 개발자는 한 걸음 더 나아가 단위 테스트는 빛처럼 빨라야한다고 주장하기도 한다.
  * 때문에 목 객체를 활용하여 느린 테스트를 빠르게 의존하는 방법을 택할 수 있다.
* **프로젝트 내부에 정의된 모든 테스트의 실행을 도저히 기다릴 수 없는 경우, 한 단계 내려와 패키지에 있는 모든 테스트를 실행해도 무방**하다.
* **테스트는 외부 자원에 접근하는 경우가 많아질수록 반드시 느려지며, 느려지는 테스트의 개수는 주의를 기울여 최소화하는 것이 이상적**이다.
* **개발자는 자신이 견딜 수 있는 선에서 최대한 많은 테스트를 실행해야 한다**.

### 테스트 제외하기
* 코드를 개발하다 보면 현재의 테스트 코드가 적색이 될 수도 있지만, 이는 전혀 문제가 되지 않는다.
  * 오히려 여러 개의 테스트 실패를 동시에 처리하는 번거로움을 피할 수도 있다.
* 다수의 실패를 다루는 해결책 중 하나는 진짜 문제가 있는 테스트에만 집중하고, 나머지 실패 테스트는 주석 처리하는 것이다.
* 그러나 **junit은 주석 처리보다 좋은 @Ignore 어노테이션을 제공하며, 무시된 이유를 설명하는 메시지를 선택적으로 작성할 수도 있다**.
  * junit 테스트 러너는 하나 이상의 제외된 테스트를 알려준다.
  * 주석 처리는 쉽게 잊혀지므로, junit과 같이 제외된 테스트 케이스를 따로 볼 수 있는 기능을 활용하는 것이 바람직하다.