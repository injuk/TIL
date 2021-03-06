# CleanCode
## 2022-01-31 Mon

## 경계
* 모든 소스 코드를 직접 개발하는 경우는 드물다.
* 어떤 식으로든 시스템에 외부 코드가 포함되며, 외부의 코드는 우리의 코드에 깔끔하게 통합되어야 한다.

### 외부 코드의 사용
* 인터페이스 제공자와 사용자 사이에는 특유의 긴장이 존재하며, 이로 인해 시스템 경계에서의 문제가 생길 수 있다.
* 예를 들어, java.util.Map은 Map을 사용하는 모든 메소드가 clear를 호출하여 Map의 내용을 지워버릴 수 있다.
  * 즉, 메소드별 권한 제어가 불가능하다. 필요하지 않은 기능까지 제공하는 것을 막을 수 없다.
* 또한 Map의 인스턴스를 반환값으로, 인수로 여기저기 넘기는 구조라면 Map이 변경되었을 때 수정되어야할 부분이 많다.
  * 실제로 JDK 1.5에서 Map 인터페이스는 변경이 있었다.
* 이러한 문제는 Map의 사용을 위한 래퍼 클래스를 만드는 것으로 깔끔하게 정리할 수 있다.
  * 아래는 Sensors 클래스에서 경계 인터페이스인 Map을 관리하고 변환하는 예시이다.
```
public class Sensors {
    private Map sensors = new HashManp();
    public Sensors getById(String id) {
        return (Sensor) sensors.get(id);
    }
    // 생략...
}
```
* 위 방식의 장점은 다음과 같이 생각해볼 수 있다.
  1. 제네릭을 사용하지 않은 경우의 명시적 형변환 책임을 클라이언트로 넘기지 않아 깔끔하다.
  2. 필요한 기능, 인터페이스만 public 메소드로 정의하여 노출할 수 있다.
     * 결과 코드의 이해는 쉬워지고, 사용에 주의가 필요한 메소드의 오용을 막을 수 있다. 
* 상술한 방식은 **경계 인터페이스인 Map을 클래스 Sensors 내부로 숨긴** 것이다.
  * 때문에 **Map 인터페이스가 수정되더라도 프로그램의 나머지에는 영향을 주지 않는다**.
  * **제네릭 역시 Sensors 내부에서 관리되므로, Map을 사용하는 메소드 호출자는 이를 신경쓸 필요가 없다**.
* **Map과 같은 Collection을 사용할 때마다 래퍼 클래스를 만들어야 한다는 의미는 아니다**.
  * **중요한 것은 외부 코드로부터 포함된 경계 인터페이스를 여기저기 넘겨가며 사용하지 말아야한다는 점을 기억**하자.
  * **Map과 같은 경계 인터페이스를 사용하는 경우, 이를 이용하는 클래스나 클래스 계열 밖으로 public하게 노출되지 않도록 해야** 한다.
    * 예를 들어, Map 인스턴스를 public한 API의 인수로 넘기거나 반환하지 말아야 한다.

### 경계 익히기
* 외부에서 가져온 패키지를 사용하면 적은 시간에 빠르게 개발할 수 있는 장점이 있다.
* 그러나 가져온 패키지에 포함된 코드를 익히고, 통합하는 것을 병행하는 작업은 비용이 많이 드는 작업이다.
* 외부 패키지의 코드를 익히기 위해, 우리는 학습 테스트 기법을 활용해볼 수 있다.
  * 학습 테스트는 간단한 테스트 케이스를 작성하여 외부 코드를 테스트하는 방식이다.
  * 우리가 **외부 코드를 사용하려는 방식대로 테스트 케이스를 만들어 외부 API를 호출하며, 통제된 환경에서 API를 제대로 이해하고 있는지 확인**한다.
* 학습 테스트는 다음과 같은 순서로 진행할 수 있다.
  1. 패키지를 내려받는다.
  2. 문서를 자세히 읽기 전에 간단한 테스트 케이스를 작성한다.
  3. 원하는 동작이 나올 때까지 문서를 읽고, 구글링하여 테스트 케이스를 수정한다.
     * 3.의 과정에서 외부 코드가 돌아가는 원리를 상당히 이해하게 된다.
  4. 학습 테스트 과정에서 얻은 지식을 간단한 단위 케이스 몇 가지로 표현하도록 한다.
  5. 학습 테스트를 통해 충분한 지식을 얻었다면, **습득한 지식을 총동원하여 독자적인 클래스로 캡슐화한다**.
     * 이를 통해 **나머지 프로그램은 외부로부터 가져온 패키지의 코드인 경계 인터페이스에 대해 알 필요가 없다**.

### 학습 테스트의 이점
* **학습 테스트는 비용보다 결과가 더 크다**. 즉, 공짜 이상이다.
  * 어차피 외부 API를 익혀야하므로, 사실상 비용이 없는 셈이다.
* 학습 테스트를 통해 필요한 지식만 손쉽게 확보하며, 외부 코드와 기능에 대한 이해도를 높일 수 있다.
* **외부 패키지의 새로운 버전이 나온다면 학습 테스트를 돌려 차이가 있는지 확인**한다.
  * 외부 코드가 우리의 코드에 통합된 이후로도 평생 호환되는 것을 보장할 수는 없다.
    * 외부 패키지는 새로운 버전이 나올 때마다 코드를 통합한 우리에게 새로운 위험이 된다. 
  * 학습 테스트는 학습 뿐만 아니라 패키지가 예상한대로 동작하는지 검증하는데에 재활용된다.
    * 새 버전이 우리의 코드와 호환되지 않으면 학습 테스트를 통해 이를 금방 밝혀낼 수 있다.
* 학습 테스트를 통한 **학습이 필요치 않더라도 외부 경계 인터페이스에 대한 테스트 코드는 필요**하다.
  * 이러한 경계 테스트 코드의 작성을 통해 패키지를 새로운 버전으로 업그레이드하기 쉬워진다.
  * **경계 테스트 코드를 작성하지 않는다면 프로젝트가 낡은 버전의 패키지에 종속**될 수 있다.

### 존재하지 않는 코드의 사용
* 다른 팀과 협업하다 보면, 다른 팀의 코드와 우리 팀의 코드가 서로를 호출하는 경계가 발생한다.
* 만약 상대 팀에서 인터페이스도 정의하지 못한 상태라면, 다음과 같은 흐름으로 개발을 진행해볼 수 있다.
  1. 우선 경계로부터 먼 부분부터 개발한다.
  2. 충분히 개발이 진행되었지만 상대 팀에서 인터페이스를 정의하지 못했다면, 자체적인 인터페이스와 추상 메소드를 정의한다.
     * 우리가 바라는 형태의 인터페이스를 설계하므로, 인터페이스를 전적으로 통제할 수 있다.
     * 따라서 코드 가독성이 높아지고 의도가 분명해진다.
     * 또한 API를 올바르게 사용하고 있는지 확인하기 위해 해당 인터페이스를 구현하는 Mock 클래스를 정의하여 경계 테스트 케이스를 작성해둘 수 있다.
  3. 상대 팀의 인터페이스 설계 및 API 개발이 완료된다면, 2.의 인터페이스와 상대 팀의 API 간극을 Adapter 패턴을 통해 메꾼다.
     * **Adapter 패턴을 통해 상대 팀의 API를 사용하는 코드가 캡슐화되므로 상대 팀의 수정사항에 대응할 코드를 하나의 클래스로 모을 수 있다**.

### 결론
* 경계는 언제나 변경을 함께 생각해야만 한다.
* 잘 설계된 소프트웨어는 변경에 많은 비용이 들지 않는다.
* 우리가 통제할 수 없는 코드를 사용한다면, 추후에 있을 변경에 대응하는 비용이 너무 커지지 않을 수 있도록 신중히 대비해야 한다.
  1. 경계에 위치하는 코드는 깔끔하게 분리한다.
  2. 외부 코드의 세부사항을 알 필요가 없도록, 동작의 기대사항을 정의하는 테스트 케이스를 작성한다.
  3. 외부 패키지를 호출하는 코드는 가능한한 줄여 경계를 관리한다.
  * 이는 다음과 같은 두 가지 방식 중 하나로 경계를 관리할 수 있다.
    1. java.util.Map의 예시와 같이 **래퍼 클래스로 경계를 감싸거나**,
    2. **Adapter 패턴을 적용하여 우리가 원하는 인터페이스를 외부 코드가 제공하는 인터페이스로 변환**한다.
    * **두 방식 모두 코드 가독성이 높아지고, 경계 인터페이스를 사용하는 일관성이 높아지고, 외부 패키지 변경에 대한 비용이 줄어**든다.
* **통제가 불가능한 외부 코드보다 통제가 가능한 우리의 코드에 의존하는 것이 언제나 바람직**하다. 

## 단위 테스트
### TDD의 세 가지 법칙
* TDD는 실제 코드를 작성하기 전에 단위 테스트를 작성하는데에 더해 세가지 법칙을 갖는다.
  1. 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
  2. 컴파일은 실패하지 않지만, 실행을 실패하는 정도로만 단위 테스트를 작성한다.
  3. 현재의 실패하는 테스트를 통과하는 정도로만 실제 코드를 작성한다.
* 이를 통해 매일 수십 개에 달하는 테스트 케이스가 나오며, 사실상 실제 코드를 전부 테스트하는 케이스가 나오게 된다.
  * 그러나 이는 실제 코드와 맞먹는 방대한 코드가 나오게 되어 관리 상의 문제를 유발하기도 한다.

### 깨끗한 테스트 코드의 유지
* 일회용 테스트 코드를 작성하다가 새삼스럽게 자동화된 단위 테스트 수트를 작성하는 것은 어렵다.
* 일부 개발 조직은 이러한 이유에서, 빠르게 개발하기 위해 지저분한 테스트 케이스를 작성하기로 결정하기도 한다.
  * 그러나 **지저분한 테스트 코드는 테스트를 안하는 것과 큰 차이가 없다**!
* **실제 코드가 변경되면 테스트 코드도 함께 변경되어야 한다**.
* 하지만 **지저분한 테스트 코드는 변경 사항의 반영을 반영하기가 어려울 것**이다.
  * 테스트 코드가 복잡하다면 실제 코드를 작성하기보다 테스트 코드를 작성하는 시간이 더 오래 걸리게 된다.
  * 점점 개발자 사이에서 테스트 수트 자체가 불만으로 자리잡으며, 결과 테스트 수트가 폐기될 수 있다.
* 그러나 **테스트 수트가 없어진다면 개발자는 자신의 코드의 정상성을 확인할 방법이 없다**!
  * 요구사항을 반영하기 위한 수정이 안전하다는 검증이 불가능하며, 결함율이 점차 높아진다.
  * 시스템을 수정할 때마다 발생하는 결함으로 인해 개발자는 점차 수정을 주저하게 된다.
* 상술한 예시에서, 문제는 테스트 코드 자체가 아니다.
  * 오히려 문제는 **테스트 코드는 깔끔하게 작성하지 않아도 좋다고 생각한 최초의 판단**이다.
* **테스트 코드는 실제 코드만큼 중요하며, 실제 코드만큼 사고와 설계를 신중히 하여 깨끗하게 작성하여야 한다**.
  * **테스트 코드는 이급 시민이 아니다**.

### 테스트 코드의 이점
* 이렇듯 테스트 코드는 깨끗하게 유지하지 않으면 잃게 된다.
  * 테스트 케이스가 없다면 모든 변경 사항은 잠정적인 버그이다.
* 테스트 케이스가 없다면 개발자는 아키텍쳐가 아무리 유연해도 버그의 발생을 우려하여 변경을 주저하게 된다.
  * **테스트 케이스가 있고, 코드 커버리지가 높을수록 변경에 대한 공포는 사실상 사라진다**.
  * 심지어 아키텍쳐와 설계까지 안심하고 개선할 수 있게 된다.
* 실제 코드를 점검하는 자동화된 단위 테스트 수트는 설계와 아키텍쳐를 깨끗하게 보존할 수 있도록 하는 열쇠이다.
  * 테스트 케이스가 있다면 변경이 쉬워지므로, 코드에 유연성, 유지보수성, 재사용성을 제공한다.
* 반면, **테스트 코드가 지저분하면 코드를 변경하기 어려워지며 구조 개선도 어렵다**.
  * 따라서 **테스트 코드가 지저분할수록 실제 코드도 함께 지저분**해진다.
  * 이러한 상황이 지속되면 결국 테스트 코드를 잃게 되며, 실제 코드도 망가진다.

### 깨끗한 테스트 코드의 작성
* 가독성은 깨끗한 테스트 코드를 만들기 위해 최우선적으로 고려되어야 한다.
  * 테스트 코드는 실제 코드보다도 가독성이 더 중요하다.
* 테스트 코드는 명료하고 단순하지만 풍부한 표현력을 갖고 **최소의 표현으로 많은 것을 나타내야만 한다**.
* 지저분한 테스트 코드는 일반적으로 불필요한 테스트 코드의 중복이나 테스트 코드와 무관한 사전 작업이 포함된 잡음에 의해 발생한다.
  * 이러한 테스트 코드는 읽는 사람으로 하여금 잡다하고 관련 없는 코드를 이해한 후에야 테스트 케이스를 이해할 수 있도록 한다.
* 깨끗한 테스트 코드를 위해 build operate check 패턴을 적용해볼 수 있다.
  * 각 테스트 케이스는 명확한 세 부분으로 나뉜다.
  1. build: 테스트를 위한 자료를 만들고,
  2. operate: 테스트 자료를 조작하고,
  3. check: 조작 결과가 올바른지 확인한다.
* **테스트 코드에서는 잡다하고 세세한 코드를 최대한 제거하며, 반드시 필요한 자료형과 함수만 사용해야** 한다.
  * 코드를 읽는 사람으로 하여금 잡다한 세부사항에 주눅들지 않고도 코드의 기능을 빠르게 이해할 수 있도록 해야 한다.

### 테스트 코드에서 DSL을 사용하기
* 일반적으로 사용되는 시스템 조작 API 대신 API 위에 함수와 유틸리티를 구현하고, 그 함수와 유틸리티를 사용하여 테스트 코드를 작성한다.
  * 테스트 코드에서만 사용하기 위해 기존 테스트 케이스에서 메소드 추출 기법을 통해 빼낸 새로운 메소드를 DSL로 취급할 수 있다.
* 이렇게 구현된 함수와 유틸리티는 테스트 코드에서 사용하는 특수한 API가 된다.
  * 즉, 테스트 코드 개발자와 코드를 읽는 개발자를 도와주는, 테스트만을 위한 언어가 된다.
  * 이러한 API는 처음부터 설계된 API가 아니며, 세부사항으로 점철된 코드를 계속해서 리팩토링하는 과정에서 진화된 API이다.
  * 개발자라면 자신의 코드를 간결하고 표현력이 풍부한 코드로 리팩토링할 수 있어야 한다.

### 이중 표준
* **테스트 API 코드에 적용하는 표준과 실제 코드에 적용되는 표준은 확실히 다르다**.
  * 간결하고 표현력이 좋아야하지만, 실제 코드만큼 효율적이지는 않아도 좋다.
  * 이는 테스트 API가 운영 환경이 아닌 테스트 환경에서 동작하는 코드여야 하기 때문이다.
    * 실제 환경과 테스트 환경의 요구사항은 완전히 다르다.
* **이중 표준은 실제 환경에서는 절대로 불가능한 방식도 테스트 환경에서는 문제 없는 방식이라면 채택할 수 있다**.
  * 예를 들어, 임베디드 환경에서의 시스템과 같이 메모리나 CPU 효율이 제한되어 있는 경우에 불가능한 방식도 테스트 환경에서는 가능할 수 있다.
  * 이러한 **이중 표준의 본질은 코드의 꺠끗함과는 무관하며, 단순한 효율성만을 따진 이야기**이다.

### 테스트 케이스 당 assert문의 개수
* 함수마다 assert 문을 하나로 제한하는 것은 가혹하지만, 코드의 이해가 빠르다는 측면에서 확실한 장점이 있다.
  * 함수(=테스트 케이스)마다 결론이 하나가 되기 때문에 그렇다.
* 그러나 assert 문이 하나가 되도록 무분별하게 테스트 케이스를 분할하면 함수마다 중복되는 코드가 늘어나는 단점이 있다.
* 때문에 **assert 문은 최소화하되, 필요한 경우에는 테스트 케이스에 여러 assert 문을 넣는 것을 주저하지 않는 방식이 바람직**하다.

### 테스트 당 개념 하나
* 또한 **중요한 것은 테스트 함수마다 하나의 개념만 테스트해야한다는 규칙**이다.
* 이것 저것 잡다한 개념을 연속으로 테스트해야 하는 긴 테스트 함수의 작성은 지양한다.
* 상술한 내용을 토대로 가장 좋은 규칙은 다음과 같다.
  1. **개념당 assert 문의 숫자를 최소화**한다.
  2. **테스트 함수 하나는 하나의 개념만 테스트**해야 한다.

### FIRST 규칙
* 깨끗한 테스트가 되기 위해 지켜야 하는 다섯 가지 규칙이 있다.
  1. Fast: **테스트는 빠르게 돌아야** 하며, 자주 돌릴 수 있어야 한다.
     * 테스트가 느리면 자주 돌리기 싫어지며, 따라서 초반에 문제를 잡지 못하게 된다.
     * 때문에 코드를 마음껏 정리하지도 못하고 코드 품질이 망가지는 결과를 낳게 된다.
  2. Independent: 각 **테스트 케이스는 서로 의존하지 않아야** 한다.
     * 하나의 테스트가 다음 테스트의 환경을 준비해서는 안된다.
     * 모든 테스트는 서로 독립적이고, 어떤 순서로 실행되더라도 문제가 없어야 한다.
     * 테스트 케이스가 서로 의존하면 하나의 테스트 케이스가 실패했을 때 다른 케이스들이 연쇄적으로 실패하며, 실패의 원인을 찾기 힘들다.
  3. Repeatable: **테스트는 어떠한 환경에서도 반복이 가능해야** 한다. 
     * 환경은 운영 환경, QA 환경 뿐만 아닌 네트워크가 단절된 개인 노트북 환경도 포함한다.
     * 테스트가 동작하지 않는 환경이 존재한다면 테스트의 실패 이유에 대한 변명거리가 생기게 된다.
       * 또한 환경 상의 문제로 테스트를 돌릴 수 없는 상황도 생길 수 있게 된다!
  4. Self-validating: **테스트는 Bool로 결과가 반환되어야** 한다.
     * 즉, 테스트의 결과는 성공 또는 실패이다.
     * 테스트의 결과를 알기 위한 지루한 수작업이 생겨서는 안 된다.
       * 이러한 경우는 테스트 결과 판단이 주관적이고, 지루한 수작업이 된다.
     * 테스트 코드 스스로가 성공과 실패를 결정지을 수 있어야 한다.
  5. Timely: **테스트는 적시에 작성되어야** 한다.
     * 단위 테스트는 테스트 대상인 실제 코드를 구현하기 직전에 작성해야 한다.
     * 실제 코드를 구현한 후에 테스트 코드를 작성한다면 실제 코드를 테스트하기 어렵거나, 할 수 없는 상황에 직면할 수도 있다.

### 결론
* 테스트 코드는 실제 코드만큼이나, 어쩌면 실제 코드보다도 프로젝트 품질에 중요할 수 있다.
* 깨끗한 테스트 코드는 실제 코드의 유연성, 유지보수성, 재사용성을 보존하고 강화하므로, 지속적으로 품질이 관리되어야 한다.
* 깨끗한 테스트 코드의 작성과 유지를 위해,
  1. 지속적으로 표현력을 높이고 간결하게 정리하여 깨끗하게 관리한다.
  2. 테스트용 API를 구현하는 것으로 DSL을 만들어 쉽게 테스트 코드를 작성한다.
* **테스트 코드가 방치되면 실제 코드도 망가진다는 사실을 명심**해야 한다.