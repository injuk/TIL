# DomainDrivenDesign
## 2022-04-30 Sat

## 의존성 제어
```
> 소프트웨어를 만드는 과정에서 객체 간의 의존 관계를 막을 방법은 없다.
> 중요한 것은 의존 자체를 피하는 것이 아닌, 잘 제어하는 것이다.
```
* 소프트웨어에서 말하는 의존이란, 임의의 객체가 다른 객체를 참조할 때 발생한다.
* 즉, 의존이 생기는 것 자체는 자연스러운 일이지만 이를 주의하여 다루지 않는 경우에 유연하지 못한 소프트웨어가 만들어질 수 있다.
  * 소프트웨어에서 중심적인 지위를 갖는 객체를 변경하는 경우, 하나의 변경이 여러 객체에 영향을 미치게 된다.
  * 정교하게 엮인 코드를 수정하는 작업은 그 자체로 복잡하며, 개발자를 크게 압박할 수 있다.
* **변경에 유연한 소프트웨어를 만들기 위해서는 특정 기술에 대한 의존은 피하고, 변경의 주도권을 추상 타입에 두어야 한다**.

## 의존성이란?
* 의존은 다음과 같이 임의의 객체가 다른 객체를 참조하는 경우에 발생한다.
  * 아래의 코드에서, TempB가 존재하지 않는다면 TempA는 성립할 수 없으므로 TempA는 TempB에 의존한다.
```
public class TempA {
  private TempB tempB;
}
```
* 의존 관계는 참조를 통해서만 발생하지는 않으며, 인터페이스와 구현체 사이에서도 생길 수 있다.
  * 이러한 일반화 관계는 다이어그램 상에서 속이 빈 화살표를 통해 나타난다.
  * 때문에 **의존 관계는 소프트웨어를 제작하는 과정에서 자연스럽게 발생할 수 밖에 없다**.
* 애플리케이션 서비스가 리포지토리에 의존하는 경우를 예로 들어, 리포지토리가 특정 데이터베이스 기술을 구현한 경우 소프트웨어는 그 기술 자체에 의존하게 된다.
  * 이로 인해 로컬에 반드시 데이터베이스 또는 테이블을 준비해두어야 하는 등, 테스트가 어려워질 수 있다.
  * **소프트웨어가 건강하게 성장하려면 아무 때나 부담 없이 코드를 실행해볼 수 있어야 하지만, 이러한 경우에는 코드의 실행 자체에 비용**이 들게 된다.
* 이러한 **문제를 해결하기 위해 애플리케이션 서비스가 특정 리포지토리에 의존하기보다, 리포지토리 인터페이스에 의존하는 방식을 사용할 수 있다**.
  * 이로 인해 서비스는 추상 리포지토리 타입을 구현한 클래스라면 어떠한 구현체라도 인자로 받을 수 있으며, 특정한 데이터베이스 기술에 묶이지 않게 된다.
* 이렇듯 추상 타입을 적절히 활용하면 의존 관계가 언제나 추상 타입을 향하도록 제어할 수 있으며, 이를 의존성 역전 원칙이라고 한다.
```
> 의존 관계의 방향을 제어하여 모든 모듈이 추상 타입에 의존하도록 만드는 것으로 비즈니스 로직은 특정한 세부사항으로부터 해방될 수 있다.
```

## 의존성 역전 원칙
* 의존성 역전 원칙은 크게 다음과 같이 분류하여 정의할 수 있다.
  1. **추상화 수준이 높은 모듈은 낮은 모듈에 의존하지 않아야 하며, 두 모듈 모두 추상 타입에 의존**해야 한다.
  2. **추상 타입이 어떠한 세부사항에 의존해서는 안되며, 구현의 세부사항이 추상 타입에 의존**해야 한다.
* 의존성 역전 원칙은 소프트웨어를 유연하게 하며, 기술적인 세부사항이 비즈니스 로직을 침범하지 못하도록 하기 위해서 필수적인 원칙이다.

### 추상 타입에 의존하기
* 소프트웨어에서 말하는 추상화 수준은 입력, 또는 출력으로부터의 거리를 뜻한다.
  1. 추상화 수준이 낮다: 기계와 가까운 구체적인 거리를 말한다.
  2. 추상화 수준이 높다: 사람과 가까운 추상적인 처리를 말한다.
* 애플리케이션 서비스가 특정 기술에 종속된 리포지토리에 의존하는 경우, 다음과 같은 논리에 의해 의존성 역전 원칙을 위배한다.
  1. **리포지토리는 특정한 데이터베이스 기술에 종속되므로, 해당 내용은 기계와 더 가까우므로 추상화 수준이 낮다**.
  2. 애플리케이션 서비스는 리포지토리를 사용하여 기능을 제공하므로, 추상화 수준이 상대적으로 높다.
  3. 즉, 추상화 수준이 높은 애플리케이션 서비스가 추상화 수준이 낮은 리포지토리에 의존하므로 의존성 역전 원칙의 첫 번째 정의를 위배한다.
* 이 경우, **가장 추상화 수준이 높은 추상 리포지토리를 도입하여 두 모듈이 모두 추상 리포지토리를 의존하게 만드는 것으로 문제를 해결**할 수 있다.
  1. 애플리케이션 서비스와 리포지토리 모두 자신보다 추상화 수준이 높은 추상 리포지토리에 의존하게 된다.
  2. 애플리케이션 서비스와 리포지토리 모두 추상 타입에 의존하게 된다.
* 즉, **추상 타입을 도입한 것으로 인해 기존에 구체 클래스를 향하던 의존성 방향이 추상 타입을 의존하게 되며, 의존성 관계가 역전**된다. 
* **일반적으로 추상 타입은 자신을 사용할 클라이언트가 요구하는 기능에 맞추어 정의**된다.
  * 즉, 추상 리포지토리는 사실 애플리케이션 서비스를 위해 존재한다.
  * 때문에 **특정 기술에 의존하는 구체 리포지토리 클래스가 추상 리포지토리를 구현하는 경우, 결과적으로 애플리케이션 서비스에게 주도권을 넘기는 것**이 된다.
  * 의존성 역전 원칙의 두 번째 정의는 이러한 형태로 준수된다.

### 주도권을 추상 타입에 두기
* 전통적인 소프트웨어는 추상화 수준이 높은 모듈이 낮은 모듈에 의존하는 경향이 있으며, 추상 타입이 세부사항에 의존하는 경우가 잦았다.
  * 이 경우, 특정한 세부사항에 의해 발생한 수정사항이 높은 추상화 수준의 모듈에까지 영향을 미치게 되는 상황이 발생하게 된다.
  * **중요도가 높은 도메인 규칙은 항상 추상화 수준이 높은 쪽에 정의되므로, 극단적으로는 세부사항에 의한 변경이 비즈니스 로직을 변경하게 만들 수 있다**.
* 이러한 문제를 방지하기 위해 **언제나 주체가 되는 것은 추상화 수준이 높은 모듈인 추상 타입이어야 한다**.
* **일반적으로 추상화 수준이 높은 모듈은 추상화 수준이 낮은 모듈을 사용하는 클라이언트이며, 클라이언트가 해야할 일은 어떠한 처리를 호출하는 것**이다.
  * **인터페이스는 구현할 처리를 클라이언트에 선언하는 것이므로, 주도권 또한 인터페이스를 사용하는 클라이언트에 넘기게 된다**.
```
> 추상화 수준이 낮은 모듈을 인터페이스와 함께 구현하는 것으로 중요도가 높은 고차원적인 개념에 주도권을 넘길 수 있다.
```

## 의존 관계의 제어
* 상황에 따라 애플리케이션 서비스는 테스트용 인메모리 리포지토리를 사용하거나, 운영용 관계형 데이터베이스 리포지토리를 사용해야할 수도 있다.
  * 이 때, **중요한 것은 소프트웨어가 실제로 어떠한 세부사항을 사용하느냐가 아닌 원하는 것을 선택하도록 제어하는 것**이다.
* 애플리케이션 서비스가 리포지토리를 사용하는 다음의 예시는 추상 타입에 의존하지만, 원하는 리포지토리를 사용하기 위해 항상 수정을 필요로 하는 단점이 있다.
  * 이러한 구현 방식은 수정 자체는 매우 단순하지만, 번거로운 반복 작업이며 실수하기도 쉽다.
  * 또한, 환경에 따라 매 번 바꾸어주어야 하므로 수정을 필요로하는 빈도 역시 높다.
```
public class ApplicationService {
  private IRepository repository;
  public ApplicationService() {
    // 환경이 바뀔 때마다 아래와 유사한 형태의 주석을 모두 수정해야 한다.
    // this.repository = new InMemoryRepository();
    this.repository = new Repository();
  }
}
```
* 이러한 문제점은 Service Locator 패턴 또는 IoC Container 패턴으로 해결할 수 있다.

### Service Locator 패턴
* **서비스 로케이터 패턴은 서비스 로케이터 객체에 의존 대상 객체를 미리 등록하고, 필요한 시점에 서비스 로케이터로부터 인스턴스를 가져오는 패턴**이다.
  * 즉, 생성자 메소드에 필요한 의존 대상을 직접 명시하지 않고 서비스 로케이터에 필요한 인스턴스 요청을 보내게 된다.
  * 이 경우, 실제 인스턴스는 시작 스크립트 등을 통해 미리 등록해둔 인스턴스에 해당한다.
* 서비스 로케이터 패턴을 활용하면 의존 관계가 설정된 시작 스크립트만 수정하는 것으로 의존 대상 객체를 간단하게 교체할 수 있다.
* 즉, **서비스 로케이터가 의존성 해결을 전담하므로 애플리케이션 핵심 로직을 수정할 필요 없이 의존 대상 객체를 손쉽게 교체할 수 있게 된다**.
  * 이 경우, 의존 관계 설정은 운영용과 테스트용 등 환경에 맞추어 나누어 일괄 관리하면 더더욱 쉽게 사용이 가능하다.
  * 시작 스크립트 등에서 프로젝트 구성 설정을 키로 삼아 용도에 맞는 의존 관계 설정으로 교체되도록 한다.
* **서비스 로케이터 패턴은 처음부터 커다란 설정을 만들 필요가 없으므로 최초 도입이 쉽다는 장점**이 있다.
* 반면, 다음과 같은 이유에서 서비스 로케이터 패턴을 안티 패턴으로 볼 수도 있다.
  1. 의존 관계를 외부에서 확인하기 어렵다.
  2. 테스트를 유지하기 어렵다.
* 의존 관계를 외부에서 확인하기 어렵다:
  * 서비스 로케이터 패턴을 적용한 경우, 대부분 생성자가 하나이므로 이를 사용하는 클라이언트를 개발하는 개발자는 하나의 생성자에 맞추어 인스턴스를 생성한다. 
  * 그러나 서비스 로케이터에 의존성 해결 설정을 하지 않았으므로, 컴파일은 실패한다.
  * 다시 말해 **의존성 해소를 위한 서비스 로케이터 설정을 해야한다는 사실을 바로 알 수 없으며, 이는 바람직한 상황**이 아니다.
* 테스트를 유지하기 어렵다: 
  * 테스트는 개발자의 실수를 미연에 방지하는 도구지만, 테스트 역시 시간이 지남에 따라 변화가 필요하다.
    * 이렇듯 개발자의 입장에서 테스트는 유용하지만, 끊임없는 관리가 필요한 도구이기도 하다.
  * 때문에 실제 코드에 새로운 의존성이 추가되었더라도 테스트 코드에 이 사실을 반영하는 것을 잊는다면 테스트는 깨지게 된다.
  * **문제는 테스트 코드를 변경하는 것이 아닌, 테스트를 실행하기 전까지는 테스트가 깨진 것을 알 수 없다는 사실**이다. 
  * **올바른 테스트를 유지하려면 일정한 수준의 강제력이 필요하지만, 의존성이 수정되었을 때 테스트의 수정을 강제할 수 없다면 테스트는 유지하기 어려워진다**.

### IoC Container 패턴
* IoC 컨테이너를 이해하려면 우선 DI 패턴을 알고 있어야 한다.
  * **의존 관계 주입으로 번역되는 DI 패턴은 의존성 주입에 생성자 메소드를 사용**한다.
  * 생성자를 통해 의존성을 주입받으므로, 의존성이 추가되더라도 테스트 코드를 수정하도록 강제할 수 있다.
* 이러한 DI 패턴은 편리하지만, **모든 의존성 주입을 위한 생성자를 여기저기에서 작성해야하는 불편함도 수반**된다.
  * 상술한 문제를 해결하기 위해 IoC 컨테이너 패턴을 적용할 수 있으며, 소프트웨어는 IoC 컨테이너의 설정에 따라 의존성을 해결하고 인스턴스를 생성한다.
* IoC 컨테이너의 설정 역시 서비스 로케이터와 마찬가지로 시작 스크립트 등을 이용한다.

## 결론
```
> 의존 관계는 발생을 막을 수 없지만, 그 방향은 개발자가 절대적으로 제어할 수 있으므로 두려워할 필요가 없다.
```
* 의존 관계는 소프트웨어를 작성하는 과정에서 자연스럽게 발생하지만, 잘 제어하지 못하는 경우 수정이 어려울 정도로 유연하지 못한 소프트웨어가 되기 쉽다.
* 소프트웨어는 사용자가 처한 환경의 변화에 맞추어 지속적으로 변경되고, 사용자에게 도움을 줄 수 있어야 한다.
  * 즉, '소프트'해야 한다.
* **최대한 도메인을 중심으로 하여 주요 로직이 기술적인 세부사항과 의존 관계를 갖지 않게 하면서 소프트웨어의 유연성을 확보할 수 있어야 한다**.