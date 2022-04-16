# Refactoring
## 2022-04-15 Fri

## 기본적인 리팩토링
* 가장 많이 사용되는 리팩토링은 함수 추출하기와 변수 추출하기이다.
  * 이러한 함수 구성과 이름 짓기는가 가장 기본적인 저수준의 리팩토링에 해당한다.
  * 또한, 반대로 진행하는 함수 인라인하기와 변수 인라인하기도 자주 사용한다.
* 추출은 결국 명명과 같으며, 코드의 이해도가 높아지다 보면 이름을 바꾸어야 하는 경우가 많다.

### 함수 추출하기
* 코드 조각을 찾아 무슨 일을 수행하는지 파악한 후, 독립된 함수로 추출하고 적절히 명명하는 리팩토링 기법에 해당한다.
  * 가장 자주 사용되는 리팩토링 기법이기도 하다.
* **코드 조각의 목적과 구현을 분리하는 방식은 독립된 함수로 구분하는 좋은 기준**이다.
  * 즉, **코드를 보고 무슨 일을 하는지 파악하기가 오래 걸리는 코드는 함수로 추출하고 '무슨 일'에 맞는 이름을 붙여준다**.
  * 이를 통해 함수의 내부 구현은 신경 쓰지 않고, 함수의 이름만으로 목적을 파악할 수 있다.
  * 내부적으로 reverse()만을 호출하는 highlight()의 예시처럼, **코드의 목적과 구현 사이의 차이가 클수록 함수로 추출했을 때 얻는 이점은 크다**.
* 짧은 함수를 많이 만들면 성능에 영향을 주리라고 걱정할 수도 있으나, 짧은 함수는 오히려 캐싱하기가 쉬우므로 컴파일러가 최적화하는 데에는 유리하다.
* **짧은 함수의 이점은 명명이 잘 되었을 때에만 발휘되므로, 반드시 명명에는 신경을 써주어야 한다**.
  * 긴 함수에는 코드 덩어리마다 주석이 달려 있을 확률이 높으며, 해당 코드 덩어리를 추출하는 함수의 이름을 지을 때 주석의 내용을 참고하여 명명할 수 있다.

### 함수 추출하기 - 절차
1. 함수를 새로 만들고 목적이 잘 드러나는 이름을 붙여준다.
   * 이 때, **함수의 이름은 어떻게보다는 무엇을 하는지 잘 드러나야 한다**.
   * 적절한 이름이 떠오르지 않는다면 일반적으로 함수로 추출하면 안 된다는 신호이다.
   * **중첩 함수를 지원하는 언어의 경우, 추출된 함수를 기존 함수 안에 중첩**시킨다.
2. 추출할 코드를 원본 함수에서 복사하여 새로운 함수에 붙여 넣는다.
3. 추출한 코드 중 원본 함수의 지역 변수를 참조하거나, 추출한 함수의 유효 범위를 벗어나는 변수를 매개변수로 전달한다.
   * **중첩 함수로 추출한 경우, 내부 함수는 외부 함수의 변수에 접근할 수 있으므로 이러한 문제가 생기지 않는다**.
   * 가장 간단한 경우는 변수를 사용하지만 값을 수정하지 않는 경우이며, 이 때에는 그냥 매개변수로 넘겨주면 된다.
   * 반면 지역 변수의 값을 수정하면서도 추출한 함수 외부에서도 사용해야 하는 경우, 해당 값을 반환해야 한다. 
   * **값을 반환할 변수가 여럿이라면, 되도록이면 함수가 값 하나만을 반환하도록 여러 함수로 나눈다**.
4. **우선 컴파일 해보고 아직 미처 처리하지 못한 변수가 있는지 확인**한다.
5. 원본 함수에서 추출한 코드 부분을 새로 만든 함수를 호출하는 문장으로 변경한다.
   * 추출한 함수를 최상위 수준 문맥으로 이동해야 하는 경우, 실제로 문맥을 옮겨보기 전에는 알 수 없는 것이 많다.
   * 때문에 **중첩 함수로 추출할 수 있더라도 최소한 원본 함수와 같은 수준의 문맥으로 먼저 추출해보는 것이 바람직**하다.
   * 이를 통해 코드를 제대로 추출했는지 즉각 판별할 수 있게 된다.
6. 테스트한다.
7. 다른 코드에 추출한 함수와 같은 코드가 있는지 확인하고, 있다면 추출한 함수를 호출하도록 수정할지 검토한다.

### 함수 인라인하기
* 때로는 함수 본문이 이름만큼 명확한 경우도 있으므로, 이러한 경우에는 추출된 함수 본문을 다시 인라인한다.
  * 간접 호출은 유용할 수도 있지만, 쓸데없는 간접 호출은 오히려 거슬리는 경우가 많다.
  * 리팩토링 과정에서 잘못 추출된 함수들도 인라인 대상이 될 수 있다.
* 이번에도 **핵심은 항상 단계를 잘게 나누어서 처리하는 데에 있으며, 함수를 평소에 충분히 작게 만들어두었다면 인라인을 한 번에 처리할 수 있는 경우가 많다**. 

### 함수 인라인하기 - 절차
1. 다형 메소드인지 확인하며, 서브클래스에서 오버라이드하는 메소드라면 인라인하지 않아야 한다.
2. 인라인할 함수를 호출하는 모든 지점을 확인한다.
3. 각 호출 지점을 모두 함수 본문으로 교체한다.
4. 하나씩 교체할 때마다 테스트를 진행한다.
   * 이렇듯 **인라인 작업은 한 번에 모두를 처리할 필요는 없다**.
5. 모든 함수 호출 지점의 교체가 완료되었다면 인라인 대상 함수를 삭제한다.

### 변수 추출하기
* 복잡한 식으로 인해 이해하기 어려워지는 경우가 있을 수 있으며, 이럴 때 지역 변수를 추가하여 표현식을 쪼갤 수 있다.
  * 이렇게 쪼개진 식은 복잡한 로직을 구성하는 단계마다 이름을 붙일 수 있으므로 관리하기도 쉽고, 이해하기도 쉽다.
  * 또한, 추가된 지역 변수는 중단점을 지정할 수 있으므로 디버깅에도 도움이 될 수 있다.
* **변수 추출을 고려한다는 점은 곧 적절한 이름을 붙이고 싶다는 뜻이기도 하므로, 문맥을 고려하여 명명**해야 한다.
  * 현재 함수 안에서만 의미가 있다면 변수로 추출한다.
* **추출 대상 식이 함수를 벗어난 문맥에서까지 의미를 고려해야 한다면, 넓은 문맥에 맞춘 이름을 생각**해야 한다.
  * 즉, 이 경우에는 변수보다는 함수로 추출해야 한다.
  * 그러나 이름이 통용되는 문맥을 넓힐 때에는 잠정적으로 할 일이 늘어난다는 단점이 존재한다.
* 이 때, **객체는 특정한 로직과 데이터를 외부와 공유하려 할 때 공유할 정보 자체를 설명해주는 적당한 크기의 문맥이 되어줄 수 있는 큰 장점이 존재한다**.
  * 때문에 덩치가 큰 클래스에서는 공통 동작을 별도 이름으로 뽑아내어 추상화하면 해당 객체를 다룰 때 쉽게 사용할 수 있어 유용하다.

### 변수 추출하기 - 절차
1. 추출하려는 표현식에 부작용이 있는지 확인한다.
2. 불변 변수를 선언하고, 이름을 붙일 표현식을 복사하여 대입한다.
3. 원본 표현식을 새로 만든 불변 변수로 대체한다.
4. 테스트를 진행한다.
5. 표현식을 여러 곳에서 사용한다면 각각 새로 만든 불변 변수로 교체하며, 하나 교체할 때마다 테스트를 진행한다.

### 변수 인라인하기
* 변수는 함수 안에서 표현식을 가리키는 이름으로 사용될 수 있으며, 대부분 긍정적인 효과를 낸다.
* 그러나 변수를 너무 과하게 추출한 경우, 변수의 이름이 원래의 표현식과 크게 다를 바가 없을 수도 있다.
  * 또는 변수 자체가 주변 코드의 리팩토링에 방해가 될 수도 있다.
* 이러한 경우에는 변수를 다시 인라인하는 것이 바람직하다.

### 함수 선언 바꾸기
* 함수는 프로그램을 작은 부분으로 나누는 주된 수단이며, 소프트웨어 시스템의 구성 요소를 조립하는 연결부 역할을 수행한다.
* 이러한 **연결부에서 가장 중요한 요소는 함수의 이름**이다.
  * 이름이 좋은 함수는 굳이 구현 코드를 살펴볼 필요도 없다.
* 이름이 잘 못된 함수를 발견하면 그 나은 이름이 떠오르는 즉시 바꾸어주어야 한다.
* 함수의 매개변수 역시 마찬가지이며, 매개변수는 함수가 외부 세계와 어우러지는 방식을 정의한다.
  * 즉, 매개변수는 함수를 사용할 문맥을 설정한다.
* 매개변수를 올바르게 선택하는 것은 단순히 규칙 몇개로 표현할 수 없으므로, 함수의 매개변수로 전달될 객체를 정하는 것은 정답이 없다.
  * 때문에 **어떻게 연결하는 것이 더 좋은 방식인지 이해하게될 때마다 코드를 그에 맞게 개선할 수 있도록 함수 선언 바꾸기 리팩토링에 익숙해져야 한다**.
* 함수의 이름이나 매개변수 중 무엇을 바꾸든지 절차는 같다.
* **상속 구조 속에 있어 다형성을 구현한 클래스의 메소드를 변경할 때에는 다형 관계인 다른 클래스들 역시 변경**해야 한다.
  * 이 경우, 상황이 복잡하므로 전달 메소드를 새로 정의하여 원래 함수를 호출하도록 활용할 수 있다.
* 공개 API를 리팩토링하는 경우, 새 함수를 추가한 다음 리팩토링을 잠시 멈출 수 있다.
  * 이 상태에서 기존 함수를 deprecated 처리하고, 모든 클라이언트가 새 함수로 이전했다는 확신이 들면 예전 함수를 삭제한다.
  * 이렇듯 **함수 선언 바꾸기는 공개 API, 즉 직접 고칠 수 없는 외부 코드가 사용하는 부분을 리팩토링하기에 적절**하다.

### 함수 선언 바꾸기 - 절차
* 함수 선언 바꾸기는 간단한 절차로도 완료될 수 있지만, 세분화된 절차를 적용해야하는 경우도 있을 수 있다.
* 때문에 함수 선언 바꾸기 리팩토링을 적용하는 경우, 변경 사항을 우선 살펴보고 함수 선언과 호출문들을 한 번에 고칠 수 있는지 판단해야 한다.
  * 가능하다면 간단한 절차를 사용하고, 그렇지 않은 경우에는 복잡한 절차를 통해 점진적으로 수정해나가야 한다.
* 간단한 절차는 다음과 같다.
1. 매개변수를 제거하기에 앞서 함수 본문에서 제거 대상 매개변수를 참조하는 위치가 있는지 확인한다.
2. 메소드 선언을 원하는 형태로 변경한다.
3. 기존 메소드 선언을 참조하는 부분을 모두 찾아 수정한다.
4. 테스트를 진행한다.
* 반면, 복잡한 절차는 다음과 같다.
1. 함수의 본문을 적절히 리팩토링한다.
2. 함수 본문을 새로운 함수로 추출하되, 우선 검색하기 쉬운 이름을 적당히 명명한다.
3. 추출한 함수에 매개변수를 추가해야 한다면 간단한 절차를 따라 추가한다.
4. 테스트를 진행한다.
5. 기존 함수를 인라인한다.
   * 호출문은 반드시 한 번에 하나씩 변경하고 테스트하도록 한다.
6. 이름을 임시로 명명했다면 함수 선언 바꾸기를 한 번 더 적용하여 원래 이름으로 되돌린다.
7. 테스트를 진행한다.

## 2022-04-16 Sat
### 변수 캡슐화하기
* 리팩토링은 결국 프로그램의 요소를 조작하는 일이며, 이러한 관점에서 함수는 데이터보다 다루기가 쉽다.
* 함수를 사용한다는 것은 호출한다는 뜻이며, 함수의 이름을 바꾸거나 다른 모듈로 옮기는 것은 어렵지 않다.
* 반면 데이터는 함수보다 다루기가 까다로우며, 참조하는 모든 부분을 한 번에 바꾸어야 코드가 정상 동작하는 특징이 있다.
* 때문에 **접근할 수 있는 범위가 넓은 데이터를 옮기는 경우에는 우선 데이터로의 접근을 독점하는 함수를 만드는 식으로 캡슐화하는 것이 바람직**하다.
  * 이는 즉 **데이터 재구성이라는 어려운 작업을 함수 재구성이라는 쉬운 작업으로 변환하는 것**과 같다.
* 데이터 캡슐화는 데이터를 변경하거나 사용하는 코드를 감시할 수 있는 통로가 되어주므로, 변경 전 검증이나 변경 후의 추가 로직을 쉽게 추가할 수 있다.
* **데이터는 그 유효범위가 넓을수록 캡슐화**해야 한다.
  * 예를 들어, **유효범위가 함수 하나보다 넓은 가변 데이터는 모두 캡슐화**해야 한다.
* 반면 불변 데이터는 가변 데이터보다 캡슐화해야할 이유가 적다.
  * **데이터가 변경될 일이 없으므로 검증 로직을 추가할 필요가 없으며, 애초에 불변 데이터는 옮길 필요 없이 복제**하면 된다.
* **기본 캡슐화 기법으로는 데이터 구조로의 접근이나 구조 자체를 다시 대입하는 행위를 제어할 수 있지만, 필드 값을 변경하는 일은 제어할 수 없다**.
  * 기본 캡슐화 기법은 데이터를 참조하는 부분만 캡슐화하지만, 변수 뿐만 아니라 변수에 담긴 내용을 변경하는 것까지 제어할 수 있도록 캡슐화하고 싶을 수 있다.
  * 이 경우, Object.assign()과 같은 메소드를 통해 데이터의 복제본을 전달한다.
  * 반환값이 객체라면, new 키워드를 통해 새로운 인스턴스를 만들어 반환할 수도 있다.
* 이렇듯 변경을 감지하거나 막는 구체적인 방법은 언어마다 다를 수 있으므로, 사용하는 언어에 맞추어 처리한다.
* **복제본을 만드는 과정이 번거로울 수 있지만, 대체로 복제가 성능에 주는 영향은 미미**하다.

### 변수 캡슐화하기 - 절차
1. 변수로의 접근과 갱신을 담당하는 캡슐화 함수를 작성한다.
2. 정적 검사를 수행한다.
3. 변수를 직접 참조하던 부분을 모두 적절한 캡슐화 함수로 수정한다.
   * 이 과정에서, 하나씩 바꿀 때마다 테스트를 진행한다.
4. 변수의 접근 범위를 제한한다.
   * Javascript의 경우, 변수와 접근자 메소드를 하나의 파일로 옮기고 접근자만 export하는 것으로 가시 범위를 제한할 수 있다.
5. 테스트를 진행한다.
6. 변수 값이 레코드라면 레코드 캡슐화하기 기법을 적용할지 고려한다.

### 변수 이름 변경하기
* **명확한 프로그래밍의 핵심은 명명이며, 개발자의 작업에 대해 많은 것을 설명해주는 변수는 이름을 잘 지었을 때에 한정**된다. 
* 특히 이름의 중요성은 사용 범위에 영향을 많이 받는다.
  * 예를 들어, 간결한 람다식 또는 함수의 매개변수 이름은 짧게 지어도 무방하다.
  * 반면 **함수 호출 한 번으로 끝나지 않고 값이 영속되는 필드라면 이름에 더 신경**써야 한다.
* 가장 간단한 예시는 함수 하나에 국한된 임시 변수나 인수이며, 이는 참조하는 코드를 찾아 하나씩 바꾸는 것으로 작업을 쉽게 진행할 수 있다.
  * 그러나 **함수 밖에서도 참조되는 변수라면 조심해서 변경을 진행**해야 한다. 
* 반면 다른 코드베이스에서 참조하는 변수는 외부에 공개된 변수이므로, 이러한 리팩토링을 적용할 수 없다.

### 변수 이름 변경하기 - 절차
1. 폭넓게 사용되는 변수라면 변수의 캡슐화를 고려한다.
   * 캡슐화한 후에는 접근자 함수 내부에서 반환하는 변수의 이름을 자유롭게 변경할 수 있다.
   * 그러나 변수의 이름을 변경한 후에 다시 접근자 함수를 없애고 접근자 함수 본문을 원래 자리에 인라인하는 것은 지양하는 것이 바람직하다.
   * **이름을 바꾸기 위해 캡슐화해야할 정도로 널리 사용되는 변수라면, 추후를 위해서라도 함수 안에 캡슐화된 채로 두는 것이 좋을 수 있기 때문**이다.
2. 이름을 변경할 변수를 참조하는 모든 곳을 찾아 하나씩 변경한다.
   * 변수의 값이 변하지 않는다면 다른 이름으로 복제본을 만들어서 하나씩 변경하며, 하나씩 바꿀 때마다 테스트하는 것이 바람직하다.
3. 테스트를 진행한다.

## 2022-04-17 Sun
### 매개변수 객체 만들기
* 여러 **데이터 항목이 여러 함수에서 활용되는 경우, 데이터 뭉치로 취급하여 하나의 데이터 구조로 모아줄 수 있다**.
  * 예를 들어, **범위를 나타내는 개념은 객체 하나로 묶어 표현하기에 적절한 대표적인 예시**이다.
  * **데이터 구조로 묶인 데이터 뭉치는 데이터 사이의 관계가 명확**해진다.
  * 또한, 함수에 전달되는 매개변수의 수를 줄일 수 있다.
* 해당 리팩토링 과정에서 발견된 새로운 데이터 구조가 있다면, 해당 데이터 구조를 활용하는 형태로 프로그램의 동작을 재구성할 수 있다.
  * 즉, 해당 리팩토링 기법에는 코드를 더 근본적으로 바꾸어주는 힘이 있다.
  * 예를 들어, 데이터 구조에 포함되는 데이터에 공통으로 적용되는 동작을 추출하여 함수로 만들거나 새로운 클래스를 작성할 수 있다.
  * 이렇게 새로 만들어진 데이터 구조가 문제 영역을 간결하게 표현하는 새로운 추상 개념으로 격상되어 코드의 개념적인 그림을 다시 그릴 수도 있다.

### 매개변수 객체 만들기 - 절차
1. 적당한 데이터 구조가 아직 없는 상황이라면 새로 작성한다.
   * 이 경우, **일반적으로 클래스로 만드는 것이 좋다**.
   * 이는 추후에 동작까지 추가하기가 좋기 때문이며, 데이터 구조는 주로 값 객체로 만들어질 수 있다.
2. 테스트를 진행한다.
3. 함수 선언 바꾸기 리팩토링을 통해 새로운 데이터 구조를 매개변수로 추가한다.
4. 테스트를 진행한다.
5. 함수 호출시 새로운 데이터 구조 인스턴스를 넘기도록 수정하며, 하나씩 변경할 때마다 테스트를 진행한다.
6. 기존 매개변수를 사용하던 코드는 모두 새 데이터 구조의 원소를 사용하도록 변경한다.
7. 모두 변경한 후에 기존의 매개변수를 제거하고 테스트를 진행한다.

### 여러 함수를 클래스로 묶기
* 클래스는 대다수의 프로그래밍 언어가 제공하는 기본적인 요소이다. 
  * 클래스는 데이터와 함수를 하나의 공유 환경으로 묶고, 다른 요소와 어우러질 수 있도록 그 중 일부를 외부에 제공하는 역할을 수행한다.
* **일반적으로 함수 호출 시 인수로 전달되는 공통 데이터를 중심으로 긴밀하게 엮여 동작하는 함수 무리는 클래스 하나로 정의하는 것이 바람직**하다.
* **클래스로 묶인 공통 데이터는 함수들이 공유하는 공통 환경을 명확히 표현하며, 각 함수에 전달되는 인수를 줄여 객체 내부에서의 함수 호출을 간결**히 한다.
* 클래스로 묶는 경우의 두드러지는 장점은 클라이언트가 객체의 핵심 데이터를 변경할 수 있고, 파생 객체들을 일관성 있게 관리할 수 있다는 점이다.
  * 또한, **프로그램의 다른 부분에서 데이터를 갱신할 가능성이 종종 있는 경우에는 클래스로 묶는 것이 큰 도움이 되어줄 수 있다**.

### 여러 함수를 클래스로 묶기 - 절차
1. 함수들이 공유하는 공통 데이터 레코드를 캡슐화한다.
   * 공통 데이터가 레코드 구조로 묶여 있지 않다면 우선 매개변수 객체로 만들어 데이터를 묶는 레코드를 작성한다.
2. 공통 레코드를 사용하는 함수 각각을 새로운 클래스로 옮긴다.
   * 이 때, **공통 레코드의 멤버는 함수 호출문의 인수 목록에서 제거**한다.
3. 데이터를 조작하는 로직들은 함수로 추출하여 새로운 클래스로 옮긴다.

### 여러 함수를 변환 함수로 묶기
* **데이터를 입력받아 여러 정보를 도출하는 작업은 한 곳으로 몰아둘 수 있고, 모아두면 검색과 갱신을 일관된 장소에서 처리하여 로직 중복을 막아줄 수 있다**.
* **변환 함수는 원본 데이터를 입력받아 필요한 정보를 모두 도출해낸 후, 각각을 출력 데이터의 필드에 추가하여 반환**한다.
* 클래스로 묶기 리팩토링과 유사하지만 다음과 같은 큰 차이점이 존재한다.
  * **원본 데이터가 코드 안에서 갱신되어야 하는 경우, 클래스로 묶는 편이 훨씬 좋다**.
  * 이는 **변환 함수로 묶은 경우 가공한 데이터를 새로운 레코드에 저장하므로, 원본 데이터가 갱신되었을 때 일관성이 깨질 수 있기 때문**이다.
  * 때문에 불변성을 제공하는 언어로 개발을 진행하는 경우, 변환 함수로 묶기 기법을 적용하는 비중이 높아진다.
    * 반면, **불변성을 지원하지 않는 언어이더라도 읽기전용 문맥에서 사용하는 경우에는 변환 함수 방식을 사용할 수 있다**.
* 여러 함수를 하나의 변환 함수로 묶어주는 이유 중 하나는 도출 로직의 중복을 방지하기 위해서이다.
  * 로직을 함수로 추출하는 것으로 유사한 효과를 볼 수도 있으나, 데이터 구조와 이를 사용하는 함수가 근처에 있지 않으면 함수를 발견하기 어려울 수 있다.
  * 해당 리팩토링 기법을 통해 다양한 파생 정보 계산 로직을 모두 하나의 변환 단계로 모아줄 수 있다.

### 여러 함수를 변환 함수로 묶기 - 절차
1. 변환할 레코드를 입력받아 값을 그대로 반환하는 변환 함수를 작성한다.
   * **해당 작업은 일반적으로 깊은 복사로 처리하여 원본 레코드를 변경하지 않아야 한다**.
   * 또한, **변환 함수가 원본 레코드를 변경하지는 않는지 검사하는 테스트를 마련해두면 큰 도움이 될 수 있다**.
2. 묶을 함수 중 하나를 골라 본문 코드를 변환 함수로 옮기고, 처리 결과를 레코드의 새로운 필드에 기록한다.
   * 옮길 로직이 복잡한 경우, 함수 추출하기 기법을 우선 적용할 수 있다.
   * **변환 함수 내부에서는 데이터의 유효범위가 좁으므로, 결과 객체를 매번 복제할 필요 없이 마음껏 변경해도 무방**하다.
3. 클라이언트 코드가 해당 필드를 사용하도록 수정한다.
4. 테스트를 진행한다.
5. 나머지 함수도 상술한 과정에 따라 처리한 후, 테스트를 진행한다.