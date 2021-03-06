# UnitTesting
## 2022-05-09 Mon

## 무엇을 테스트할 것인가?
* 코드만 봐서는 숨어 있는 모든 버그를 찾아내는 것이 불가능하다.
* **우리에게 필요한 것은 무엇을 테스트하는 것이 중요한지 결정해주는 가이드이며, Right-BICEP은 무엇을 테스트할지 쉽게 선별할 수 있도록 돕는 지침**이다.
  1. Right: 결과가 올바른가?
  2. B oundary conditions: 경계 조건은 맞는가?
  3. I nverse relationship: 역 관계를 검사할 수 있는가?
  4. C ross check: 다른 수단을 통해 교차 검사할 수 있는가?
  5. E rror conditions: 오류 조건을 강제로 발생시킬 수 있는가?
  6. P erformance characteristics: 성능 조건은 기준에 부합하는가?

### Right - 결과가 올바른가?
```
> 테스트 코드는 무엇보다도 기대한 결과를 산출해낼 수 있는지 검증할 수 있어야 한다.
```
* 앞선 ScoreCollection 클래스의 테스트를 예로 들어, 5와 7울 넣어 6을 반환하는지 확인하는 단위 테스트를 작성했다.
  * **더 많은 숫자나 더 큰 수를 넣어 테스트를 강화할 수도 있으나, 이는 어디까지나 행복 경로 테스트에 해당**한다.
* **행복 경로 테스트는 코드가 정상적으로 동작하는 경우에 이를 개발자가 알 수 있도록 하는 역할을 수행**한다.
  * 반면, 어떠한 작은 코드 영역에 대해 행복 경로 테스트를 할 수 없다면 아직 그 내용을 완전히 이해하지 못한 것과 같다.
  * **이 경우, 코드가 정상적으로 동작하는지 개발자가 알 수 있을 때까지 추가 개발을 보류하는 것이 바람직**하다.
* 그러나 모든 요구사항을 알 수 없는 상황에서 단지 단위 테스트를 완료하기 위해 기능 구현을 중단하는 것은 현실적이지 못한 것으로 보일 수도 있다.
  * 예를 들어, 요구사항이 모호하거나 불완전한 경우에는 모든 요구사항이 확정될 때까지 개발을 멈추어야 하는가?
* 사실 **모든 요구사항이 확정될 때까지 기다릴 필요는 없으며, 지금 가능한 최선의 판단을 하되 요구사항이 확정되었을 때 개선하는 방향이 현실적**이다. 
  * **대부분의 경우 어떻게든 요구사항은 변경되며, 단위 테스트는 단지 그 당시의 선택을 문서화하는 역할**을 맡는다.
  * 단위 테스트를 통해 다른 개발자들은 현재까지 코드가 어떻게 동작했는지 알 수 있으며, 변경이 발생한 경우에 이를 참고할 수 있다.

### B - 경계 조건은 맞는가?
```
> 클래스가 외부에서 호출되는 공개 API며, 클라이언트를 완전히 신뢰할 수 없다면 나쁜 데이터에 대한 보호가 필요하다.
```
* 코드 상에 나타나는 **행복 경로는 입력된 값의 양 극단을 다루는 경계 조건에 걸리지 않을 수 있다**.
* 개발자가 마주치는 **대부분의 결함은 이러한 모서리 사례에서 드러나므로, 테스트 코드로 이들을 처리**할 수 있어야 한다.
* 반드시 생각하여 테스트해야 하는 경계 조건은 크게 다음과 같다.
  1. 모호하고 일관성이 없는 입력 값: 예를 들어, 특수 문자가 포함되는 파일의 이름
  2. 잘못된 양식의 데이터: 예를 들어, com, net과 같은 최상위 도메인이 누락된 이메일 주소
  3. 수치적으로 오버플로우를 일으키는 계산: 예를 들어, int형에 Integer.MAX_VALUE를 여럿 삽입하는 경우
  4. 비거나 빠져있는 값: 예를 들어, 0, 0.0, "", null, 등
  5. 이성적인 기댓값을 훨씬 벗어나는 값: 예를 들어, 200세의 나이
  6. 중복을 허용해서는 안되는 목록에 중복된 값이 포함된 경우
  7. 정렬이 안 된 정렬 리스트, 또는 그 반대의 경우: 예를 들어, 정렬 알고리즘에 이미 정렬된 값이나 역순 정렬된 데이터를 삽입하는 경우
  8. 시간 순서가 맞지 않는 경우: 예를 들어, POST 메소드의 결과를 OPTIONS 메소드의 결과보다 먼저 반환하는 경우
* 클래스를 설계하는 과정에서 잠재적인 정수 오버플로우 등의 경계 조건을 고려할지의 여부는 전적으로 개발자의 선택이다.
  * 클라이언트가 개발자와 같은 팀의 소속이라면 여러 경계 조건에 대한 보호절을 제거하고 이를 알려도 무방하다.
  * 이는 코드에서 불필요하게 과도한 인자를 검사하는 코드를 줄여줄 수 있는 선택이다.
  * **보호절을 제거하는 경우에는 클라이언트 개발자에게 주석으로 경고할 수도 있지만, 더 좋은 것은 코드 제한 사항을 문서화하는 테스트를 추가하는 것**이다.

### 경계 조건과 CORRECT
* CORRECT라는 두문자 약어는 잠재적인 경계 조건을 기억하는 데에 도움을 줄 수 있다.
* **각 항목에 대해 유사한 조건을 테스트하는 메소드에서 이를 고려해야 하며, 이러한 조건을 위반했을 때 어떠한 일이 발생할 수 있는지 숙고할 수 있어야 한다**.
  1. C onformance: 값이 기대한 양식을 '준수'하고 있는가?
  2. O rdering: 값의 집합이 적절히 정렬되었거나, 그렇지 않은가? 
  3. R ange: 이성적인 최소값과 최대값의 범위 내부에 있는 값인가?
  4. R eference: 코드 자체에서 통제할 수 없는 외부 참조를 포함하는가?
  5. E xistence: 값이 실제로 존재하는가?
     * 예를 들어 null이거나, 0이거나, 집합에 존재하지 않는 것은 아닌가?
  6. C ardinality: 정확히 충분한 값들만 존재하는가? 
  7. T ime: 모든 것이 정확한 시간에, 정시에 순서대로 발생하는가?

### I - 역 관계를 검사할 수 있는가?
* 수학에서 나눗셈을 곱셈으로 검증하고, 덧셈을 뺄셈으로 검증하는 것처럼 논리적인 역 관계를 통해 행동을 검증할 수 있다.
* 이 과정에서, **각 루틴이 공통된 코드를 호출한다면 운영 코드와 역 행동 코드 모두 같은 결함을 가질 수 있음에 주의**해야 한다.
  * 때문에 데이터베이스에 데이터를 넣는 코드를 테스트하는 경우, 테스트에서 직접적인 JDBC 쿼리를 사용하는 것도 하나의 방법이다.
* 컬렉션 필터링을 테스트하는 경우, ==와 !=의 필터링을 통해 반환된 새로운 컬렉션의 길이 합이 원본 컬렉션의 길이와 같은 값을 갖는지 확인할 수도 있다.

### C - 다른 수단을 통해 교차 검사할 수 있는가?
* '흥미로운' 문제에는 무수한 해법이 존재하며, 일반적으로 그 중 성능이 좋거나 악취가 없는 1등 해법이 선택된다.
  * 즉, **그 외의 버려진 모든 해법은 운영 코드를 교차 검증하는 데에 활용할 수 있는 패배자 해법**이 된다.
* **패배자 해법은 운영 환경에서 활용하기에는 너무 느리거나 유연하지 않을 수는 있지만, 믿을 수 있고 참을 보장하는 경우에 교차 검증에 활용할 수 있다**.
* 예를 들어, 제곱근을 구하는 메소드를 새로 개발했다고 하자.
  * 해당 메소드는 제곱근 값 자체로 검증할 수도 있지만, 같은 기능을 수행하는 Math.sqrt 메소드의 결과와 근사하는지 확인하는 것으로 교차 검증할 수도 있다.

### E - 오류 조건을 강제로 발생시킬 수 있는가?
* 행복 경로가 있다는 것은 불행 경로도 있다는 것을 의미한다.
  * 예를 들어 디스크가 꽉 차거나, 네트워크가 중단되거나 하는 실전 문제로 프로그램은 중단될 수 있다.
  * **개발자는 테스트 코드만으로도 이러한 모든 실전 문제를 우아하고 이성적으로 다루고자할 수 있다**.
  * 이를 위해서는 **테스트 자체가 각 오류들을 강제로 발생시킬 수 있어야 한다**.
* **개발자는 코드를 테스트하기 위해 도입할 수 있는 오류의 종류, 또는 다른 환경적인 제약 사항을 생각할 필요**가 있다.
* 예를 들어, 다음의 시나리오와 같은 오류 상황을 고려해볼 수 있다.
  1. 메모리가 가득 찬 경우
  2. 디스크가 가득 찬 경우
  3. 서버와 클라이언트 간의 시간이 달라지는 경우
  4. 네트워크의 가용성 및 관련된 오류들의 경우
  5. 시스템 로드
  6. 제한된 색상 팔레트
  7. 너무 높거나 낮은 해상도
  * 이렇듯 특정한 오류를 시뮬레이션하기 위해서는 특별한 기법이 필요하다.
* **좋은 단위 테스트는 단지 코드에 존재하는 로직 모두에 대한 커버리지를 달성하는 것이 아니다**.
  * 때때로 가장 끔찍한 결함들은 전혀 예상하지 못한 곳에서 나오기 쉽다.

### P - 성능 조건은 기준에 부합하는가?
```
> 병목 지점은 놀라운 곳에서 발생하므로, 실제로 병목이 어디에서 발생하는지 증명되기 전까지는 짐작만으로 코드를 건드리지 말아야 한다.
```
* 많은 개발자는 성능 문제의 위치와 최적의 해법을 추측하지만, 이러한 추측이 항상 들어맞지는 않는다.
* **추측만으로 성능 문제에 즉각 대응하기보다는 단위 테스트를 설계하고, 진짜 문제가 어디에 있고 예상한 변경 사항으로 어떠한 차이가 생기는지 파악**해야 한다.
* 성능을 테스트하는 경우, 다음과 같은 항목을 충분히 고려할 수 있어야 한다.
  1. 코드를 충분한 횟수만큼 실행하여 타이밍과 CPU 클록 주기와 관련된 이슈를 제거한다.
     * 성능을 측정하는 경우, 충분한 횟수를 반복해야 결과가 일정해진다.
     * for 문 등의 내부에서 몇 번만 반복하는 경우 수치는 들쑥날쑥해지기 쉽다. 
  2. 반복하는 코드 부분을 JVM이 최적화하지 못하는지 확인해야 한다.
  3. 최적화되지 않은 테스트는 일반적인 테스트에 비해 매우 느리므로, 느린 테스트들을 빠른 테스트들과 분리해야 한다.
     * 일반적으로, 이러한 성능 테스트는 야밤에 한 번만 수행하는 것으로 충분하다.
  4. 동일한 머신에서도 실행 시간은 시스템 로드와 같은 여러 요소에 따라 달라질 수 있음을 기억한다.
* 일반적으로 유일한 해법은 가능한 한 운영 환경과 동일한 머신에서 실행하는 것이다.
* 주의할 것은 성능 요구 사항은 일반적으로 end to end 기능을 기반으로 한다는 사실이다.
  * 이러한 **기능의 성능을 단위 테스트로 검증하는 것은 마치 사과와 오렌지를 비교하는 것과 같다**.
* **단위 성능 측정을 잘 활용하는 방법은 변경 사항을 정의할 때 기준점으로 활용하는 것**이다.
  1. 코드 최적화를 수행하기 전에 우선 기준점으로 현재 방식의 경과 시간을 측정하는 성능 테스트를 우선 작성한다.
  2. 이를 수 회 실행하고, 평균을 계산한다.
  3. 코드를 변경한 후에 성능 테스트를 다시 실행하여 결과를 비교한다.
  4. **상대적인 개선량을 찾되, 실제 숫자에 집착하지 않아야 한다**.
* 이렇듯 **모든 성능 최적화 시도는 실제 데이터를 기반으로 해야 하며, 추측을 기반으로 하지 않는 것이 바람직**하다.
* 나아가 성능이 핵심적인 고려 사항이라면 단위 테스트보다는 더 고수준에서 문제를 검증하고 싶을 수 있다.
  * 이 경우, 적절한 서드 파티 도구를 찾아 도입하는 것을 고려할 수도 있다.

## 결론
* Right-BICEP 약어를 활용하여 행복 경로와 경계 조건, 오류 조건을 다루는 테스트를 작성해야 한다.
* 나아가 **결과를 교차 검사하고, 역 관계를 살펴보며 테스트 코드의 유효성을 강화**할 수 있다.