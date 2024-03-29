# HeadFirst
## 2022-10-25 Tue

### new 연산자 살펴보기
```
> 특정한 구현을 바탕으로 코드를 작성하지 않는 것이 바람직하지만, new 연산자를 사용하는 것은 곧 임의의 구현에 의존하는 것과 같다.
> 객체의 인스턴스를 생성하는 작업이 항상 new 연산자와 함께 공개되어야 하는 것은 아니며, 이는 오히려 단단한 결합에 의한 문제를 발생시킬 수 있다. 
```
* **new 연산자를 사용하면 구체 클래스의 인스턴스가 생성되며, 이는 당연하게도 인터페이스가 아닌 구현에 의존하는 코드**를 만들게 된다.
  * 앞서 다루었듯 **구체 클래스에 의존하는 코드는 코드의 수정 가능성이 높고, 유연성을 떨어트리기 쉽다**.
* 즉, `Animal animal = new Bird();`라는 코드는 다음과 같은 의미를 갖는다.
  1. 변수 타입을 Animal 인터페이스로 선언하여 다형성으로 인한 코드의 유현성을 꾀한다.
  2. **그럼에도 `new Bird();`라는 코드에 의해 구체 클래스인 Bird에 의존**하게 된다.
* 나아가 Animal 인터페이스를 구현하는 클래스가 여럿인 경우, 객체 생성을 위해 if - else if - else 분기를 사용하는 코드를 작성할 수도 있다.
  * 이 때 인스턴스 타입은 if 분기의 조건에 따라 결정되므로, 코드에 수정이 필요한 경우 항상 코드를 다시 확인해야하는 아쉬움이 있다.
  * 때문에 **이러한 유형의 코드는 수정 사항을 반영하기 어렵고, 더불어 휴먼 에러의 발생 가능성 또한 높다**.

### 다형성 돌아보기
```
> 인터페이스에 맞춘 코딩은 애플리케이션에서 발생할 여러 변경 사항에 대응할 수 있게 한다.
```
* new 연산자는 Java의 기초 중의 기초이며, 애플리케이션 개발에서 단 한 번도 사용하지 않는 것은 불가능하다.
  * **이는 즉 new 연산자 자체에는 문제가 없음을 시사하며, 실제로 주의하여 다루어야할 문제는 애플리케이션에 가해질 변화에 해당**한다.
* **인터페이스를 기반으로 작성된 코드는 어떠한 클래스든 인터페이스만 구현하면 사용이 가능하므로, 여러 변화에 대응이 가능한 유연성을 부여**한다.
  * 또한, 이러한 유용한 특징을 다형성이라고 지칭할 수 있다. 
* 반면, **구체 클래스를 많이 사용할수록 새로운 구체 클래스가 추가될 때마다 반드시 기존 코드를 수정해야 하므로 없었던 문제가 발생**시키기 쉽다.
  * **이는 즉 변경에 닫혀 있는 코드를 의미하므로, 기능을 확장하기 위해서는 변경을 캡슐화하여 확장에 열린 구조를 만들 수 있어야 한다**.

### 팩토리 패턴이 필요한 코드
* 예를 들어, 임의의 타입 정보에 따라 다른 인스턴스를 생성해야할 수 있다.
  * 이 경우 **각 인스턴스는 동일한 인터페이스를 구현하지만, 수 많은 if - else if - else 분기로 new 연산자로 각각 다른 인스턴스를 생성**한다.
* 또한, **이렇듯 if - else if - else 분기로 객체를 생성하는 코드는 변경에 닫혀 있지 않을 가능성이 높다**.
  * 때문에 객체의 종류가 추가되거나 줄어드는 등 변경이 발생한 경우, 반드시 해당 소스 코드를 수정해야만 한다.
* **이러한 코드에서는 인스턴스를 생성할 구체 클래스를 선택하는 부분이 변경 가능성이 높으며, 후속 코드는 다형성을 활용하므로 변경 가능성이 높지 않다**.
* 즉, **이 경우에 객체 생성과 관련된 코드는 변경 가능성이 높아 캡슐화되어야 하므로 별도의 객체에 책임을 할당하는 것이 바람직**하다. 
  * 이 경우, 해당 객체는 오로지 객체를 만드는 일에만 집중하게 되어 기존 코드는 해당 객체의 클라이언트 코드가 된다.
  * 이 때, **이렇듯 객체 생성만을 전담하여 처리하는 객체를 팩토리 객체라고 지칭**한다.
  * 이로 인해 기존 코드에서는 더 이상 객체 생성에 대해 고민할 필요가 없으며, 팩토리로부터 인스턴스를 반환받아 다형성을 활용하여 유연하게 동작할 수 있다.
  * 상술한 예시에서, if - else if - else 분기를 갖던 기존 코드를 새로 정의한 팩토리 객체로 그대로 옮기기만 하면 가장 간단한 팩토리를 정의할 수 있다.
* **객체 생성 코드를 캡슐화한 팩토리 패턴은 일견 코드의 위치만 다른 객체로 넘긴 것처럼 보이지만, 여러 클라이언트가 객체를 생성하는 경우 큰 이점**이 있다.
  * 예를 들어, 해당 객체를 생성하는 클라이언트 코드가 매우 많은 경우 변경 사항을 반영하기 위해 여기 저기에 퍼진 객체 생성 코드를 수정할 필요가 없다.
* 이렇듯 **간단한 팩토리 객체를 도입하는 경우, 기존 클라이언트 코드는 생성자로 팩토리 객체를 외부로부터 전달 받아 자신의 멤버 변수로 저장**할 수 있다.
  * 이 때, 기존의 if - else if - else 분기를 사용하던 코드 대신 팩토리로부터 객체를 생성하여 다형성을 활용하도록 코드를 수정할 수 있다.
  * 이 경우, 코드는 더 이상 new 연산자를 사용하지 않으므로 구체 클래스의 인스턴스에 의존하지 않게 된다.

### 팩토리 패턴과 정적 팩토리 메소드
* 정적 팩토리 메소드는 간단한 팩토리를 static 메소드로 정의하는 기법이며, 이는 객체 생성을 위해 굳이 new 연산자를 사용하지 않아도 되도록 하는 이점이 있다.
  * 그러나 **정적 팩토리 메소드는 static 메소드이므로, 서브클래스를 정의할 때 객체 생성 메소드의 행동을 변경할 수 없다는 단점이 존재**한다.

## 2022-10-26 Wed
### 간단한 팩토리
```
> 간단한 팩토리라는 용어는 디자인 패턴이라기보다는 프로그래밍에서 자주 사용되는 관용구에 가깝다.
```
* 간단한 팩토리에서 생성하는 인스턴스가 다음의 조건을 만족할 때, 간단한 팩토리로서 클라이언트에게 인스턴스를 반환할 수 있다.
  1. 팩토리에서 생성하는 모든 인스턴스는 같은 인터페이스를 구현한다.
  2. 팩토리에서 생성하는 인스턴스는 팩토리에 정의된 구상 클래스로부터 생성된다.
* 즉, **간단한 팩토리는 동일한 인터페이스를 구현하는 구상 클래스에 대한 정보를 애플리케이션에서 유일하게 직접 참조**한다.

### 다양한 팩토리 정의하기
* 애플리케이션 요구사항에 따라 팩토리가 다양한 종류로 정의되지만, 클라이언트 코드가 팩토리로부터 얻은 인스턴스를 사용하는 방식은 같을 수 있다.
  * 이 경우, **중복으로 인한 휴먼 에러를 방지하기 위해 인스턴스 생성부터 클라이언트 코드에서 작업하는 내용까지 모두 하나의 프레임워크로 만들 필요**가 있다.
  * 또한, **간단한 팩토리를 제거하고 새로운 팩토리를 정의하는 이 과정에서 유연성을 잃지 않기 위한 팩토리 메소드와 같은 새로운 방법이 필요**하다.

### 팩토리 메소드 적용하기
* 팩토리로부터 반환받은 인스턴스의 구체 클래스는 동적으로 결정되지만, 작업 자체는 동일한 경우에는 팩토리 메소드를 활용할 수 있다.
  * 이 경우, 상술한 간단한 팩토리에서 별도의 팩토리 객체로 추출했던 로직을 다시 원본 객체로 되돌린다.
  * 대신, **원본 클래스를 추상 클래스로 변경한 후 추상 팩토리 메소드를 하나 정의한 후에 팩토리를 사용하던 객체 생성 코드를 추상 메소드 호출로 변경**한다.
  * 이 경우, **팩토리 메소드의 구현은 해당 추상 클래스를 확장하는 서브클래스에서 결정**하게 된다.
* **이러한 방식은 각 팩토리로부터 생성한 인스턴스의 종류가 유동적으로 변경될 수 있으나, 해당 인스턴스가 동일한 인터페이스를 구현하는 경우에 적절**하다.
* 즉, 원본 클래스는 이제 다음과 같은 두 메소드를 통해 프레임워크의 형태를 띄게 되며, 프레임워크의 역할에 충실하면서도 여러 구체 인스턴스에 대응할 수 있다.
  1. 추상 팩토리 메소드로서, 해당 클래스를 확장하는 서브클래스에서 인스턴스를 생성하는 방법을 반드시 구현해야 한다.
  2. 인스턴스를 생성하여 해당 인스턴스의 메시지를 호출하던 메소드의 경우, 자신의 추상 메소드를 호출하여 인스턴스를 생성한 후 작업을 요청한다.
* **해당 방식에서, 이제 구체 클래스의 인스턴스를 만드는 작업은 하나의 팩토리 객체가 전부 처리하는 대신 일련의 서브클래스에서 처리하는 방식으로 변경**된다.
  * 즉, 이제 구체적인 객체의 타입은 추상 클래스를 확장하는 서브클래스에서 결정되며 원본 추상 클래스는 구체적인 객체의 타입을 전혀 알 수 없다.
  * **대신 객체의 생성은 팩토리 메소드가 처리하며, 원본 추상 클래스에서는 팩토리 메소드가 반환하는 객체가 특정 인터페이스를 구현한다는 사실에 집중**한다.

### 팩토리 메소드의 특징 I
* **팩토리 메소드는 객체 생성에 대한 구체적인 내용을 서브클래스에 캡슐화하며, 슈퍼클래스의 클라이언트 코드와 서브클래스의 객체 생성 코드를 분리**할 수 있다.
* 팩토리 메소드는 일반적으로 `abstract Animal createAnimal(String type);`와 같은 형식으로 정의된다.
  * 이는 **팩토리 메소드를 추상 메소드로 선언하여, 서브클래스에서 객체의 생성을 책임지도록 하기 위함**이다.
  * 또한, 팩토리 메소드는 `String type`과 같은 매개변수를 통해 실제로 생성될 객체의 종류를 선택하기 위한 정보를 간접적으로 제공할 수 있다.
* **팩토리 메소드는 임의의 객체를 알아서 선택하여 반환하며, 일반적으로 해당 객체를 사용하는 클라이언트 코드는 슈퍼클래스에 정의된 메소드에 위치**한다.
  * 이 경우, **팩토리 메소드는 슈퍼클래스의 메소드에 위치한 클라이언트 코드가 객체의 구체적인 타입을 전혀 알 수 없도록 캡슐화하는 역할을 수행**한다.

### 팩토리 메소드를 사용하는 프레임워크 방식의 장점
```
> 해당 방식의 경우, 팩토리로부터 반환받은 인스턴스에 실제 작업을 요청하는 메소드는 런타임에서 어떤 타입의 인스턴스가 작업을 처리할지 전혀 알 수 없다.
> 대신, 해당 메소드는 팩토리 메소드로부터 임의 타입의 인스턴스가 반환된다는 사실과 해당 인스턴스에 어떤 작업을 요청할 수 있다는 사실만을 알게 된다.
> 이로 인해 팩토리 메소드를 갖는 클래스와 팩토리 메소드가 반환하는 인스턴스의 클래스는 완전히 분리되어 유연성을 보장할 수 있게 된다.
```
* 이제 추상 클래스로 변경된 원본 클래스는 추상 팩토리 메소드로부터 반환받은 인스턴스에 메시지를 요청하기 위한 메소드만이 정의된다. 
  * 때문에 **해당 클래스는 서브클래스가 정의되기 전까지는 실제로 어떤 서브클래스가 어떤 인스턴스를 생성하여 해당 메소드를 실행할지 알 수가 없게 된다**.
  * 즉, **런타임에서 어떤 구체 클래스가 상술한 메소드를 실제로 처리할지 알 방법이 없으므로 원본 클래스와 팩토리에서 사용하는 클래스는 완전히 분리**된다.
* 이는 **결국 실제로 작업을 처리할 객체는 구현될 서브클래스의 종류에 따라 결정되는 셈**이 된다.
  * 또한, 생성된 인스턴스에 작업을 요청하는 메소드 입장에서는 서브클래스의 종류를 알 수 없으므로 마치 인스턴스의 타입을 서브클래스가 결정하는 것처럼 보인다.

## 2022-10-27 Thu
### 팩토리 메소드의 특징 II
* **모든 팩토리 패턴은 객체 생성을 캡슐화하며, 특히 팩토리 메소드 패턴은 서브클래스에서 어떤 구체 클래스를 만들지 결정하는 것으로 객체 생성을 캡슐화**한다.
* 이 때, 팩토리 메소드 패턴은 크게 다음과 같은 구성 요소로 분류할 수 있다.
  1. 추상 생산자 클래스: **추후에 서브클래스에서 제품을 생산하기 위해 구현하는 추상 팩토리 메소드를 정의**한다.
  2. 추상 제품 클래스: **팩토리 패턴의 주 목적은 인스턴스를 생성하는 것이며, 이 때 각 인스턴스는 제품 클래스를 확장하거나 구현**한다.
* 이 때, 추상 생산자 클래스에는 추상 제품 클래스에 의존하는 클라이언트 코드가 존재할 수도 있다.
  * 그러나 **구체 제품을 반환하는 팩토리 메소드는 항상 추상 생산자 클래스의 서브클래스에서 정의되므로, 어떤 구체 제품 클래스가 만들어질지는 알 수 없다**.
* **추상 생산자 클래스를 확장하는 구체 생산자 클래스는 추상 생산자 클래스에서 추상 메소드로 정의된 팩토리 메소드를 자신만의 방식으로 구현**한다.
* 또한 **모든 팩토리는 제품을 생산하므로, 구체 제품 클래스는 추상 팩토리 메소드가 생성하는 추상 제품 클래스를 확장하거나 구현**한다.
* 이 때, **구체 생산자 클래스 별로 많은 구체 제품 클래스를 생성할 수 있으므로 각 생산자 및 생산자에 대응되는 제품은 병렬 계층 구조로 볼 수 있다**.
* **병렬 클래스 계층 구조에서, 생산자와 제품은 모두 추상 클래스로 시작하며 각 클래스를 확장하는 구체 클래스들을 갖는 동일한 구조를 보인다**.
  * 이렇듯 두 클래스 모두 구체적인 구현은 구체 클래스가 책임지는 구조를 가지며, 이 때 생산자 클래스의 팩토리 메소드는 구체 제품을 만드는 방식을 캡슐화한다.
  * 다시 말해 **팩토리 메소드는 구체 제품 클래스를 생산하는 방법 자체를 캡슐화하는 데에 있어 가장 핵심적인 역할을 수행**한다.

### 팩토리 메소드 패턴이란?
```
> 팩토리 메소드 패턴에서는 객체 생성을 위해 필요한 인터페이스를 따로 만들어 둔다.
> 이후, 어떤 구체 클래스의 인스턴스를 생성할지는 해당 인터페이스를 구현하는 서브클래스에서 결정한다.
> 이렇듯 팩토리 메소드 패턴을 사용하는 경우, 어떤 구체 클래스의 인스턴스를 생성할지 결정하는 것은 서브클래스가 된다.
```
* **팩토리 메소드 패턴을 통해 구체 클래스를 생성하는 작업을 캡슐화**할 수 있다.
* 팩토리 메소드 패턴에서, 생산자 클래스는 다음과 같은 특징을 갖는다.
  1. 팩토리 메소드에 의해 생산된 구체 제품 클래스로 필요한 작업을 처리하는 별도의 메소드를 갖는다.
  2. 그러나 **팩토리 메소드는 추상 메소드로 정의되며, 이를 구현하고 구체 제품 인스턴스를 생산하는 작업은 오로지 서브클래스에서만 가능**하다.
* 즉, 팩토리 메소드 패턴에서는 어떤 구체 클래스의 인스턴스를 생산할지 서브클래스가 결정한다.
  * 이 때, **결정이라는 표현의 핵심적인 의미는 `생산자 클래스가 실제 생산될 제품을 전혀 알 수 없는 상태로 정의되는 것`에 있다**.
  * 때문에 실제로는 코드 상에서 사용할 서브클래스인 구체 생산자 클래스에 따라 생산되는 구체 제품 클래스가 결정된다.
* 반면, **서브클래스로부터 생산되는 모든 구체 제품 클래스는 반드시 모두 같은 인터페이스를 구현**해야 한다.
  * 이러한 **특징을 반드시 준수해야만 구체 제품 인스턴스를 사용하는 클래스에서 구체 클래스가 아닌 인터페이스에 의존할 수 있게 된다**.
* 이로 인해 구체 제품 클래스는 구체 생산자 클래스에 의해 생성되며, 실제로 구체 제품 클래스와 결합되는 클래스 역시 구체 생산자 클래스 뿐이 된다.

### 팩토리 메소드 패턴의 특징 III
* **팩토리 메소드 패턴은 구체 생산자 클래스가 단 하나 뿐인 경우에도 객체의 생성과 사용을 분리할 수 있으므로 충분히 유용**하다.
* 팩토리 메소드와, 이를 포함하는 생산자 클래스는 간단한 구체 제품 클래스를 사용하는 경우에 한해 추상으로 선언하지 않아도 무방하다.
  * 예를 들어, 간단한 구체 제품 클래스를 생성하는 경우에는 생산자 클래스의 서브클래스를 사용하지 않고 직접 생성할 수 있다.
* 팩토리 메소드 패턴에서, 구체 생산자 클래스는 때로는 하나의 구체 제품 클래스만 반환하는 형태로 구현될 수도 있다.
  * 즉, 구체 생산자 클래스가 반드시 여러 구체 제품 클래스를 다루어야하는 것은 아니다.
* **팩토리 메소드가 매개 변수를 통해 구체 제품 클래스를 여럿 생성하는 경우, 오타로 인한 런타임 오류가 발생할 가능성에 주의**해야 한다. 
  * 이러한 형식 안정성을 더 보장하기 위한 기법으로는 매개변수용 객체를 정의하거나, 정적 상수 또는 enum을 활용하는 방법을 고려할 수 있다.

### 간단한 팩토리와 팩토리 메소드 패턴의 비교
```
> 간단한 팩토리와 팩토리 메소드 패턴은 비슷하지만, 구체 제품 객체를 생성하기 위해 다른 방법을 사용한다.
```
* 간단한 팩토리의 경우, 다음과 같은 특징을 갖는다.
  1. 팩토리는 클라이언트 클래스의 멤버 변수로 포함되는 별도의 객체이다.
  2. 기본적으로 간단한 팩토리는 일회용 객체에 해당한다.
  3. **간단한 팩토리는 객체 생성을 캡슐화할 수 있으나, 구체 제품 클래스를 마음대로 변경할 수 없으므로 유연하지 않다**.
* 반면 팩토리 메소드 패턴의 경우, 다음과 같이 간단한 팩토리에 대비되는 특징을 갖는다.
  1. 팩토리는 추상 생산자 클래스에 인터페이스로 정의되며, 이를 서브클래스인 구체 생산자 클래스에서 확장한다.
  2. 팩토리 메소드 패턴은 재사용성이 높은 프레임워크에 해당한다.
  3. **팩토리 메소드 패턴은 생성될 구체 제품 클래스를 서브클래스에서 결정하므로 유연성이 높다**.

### 팩토리 메소드 패턴의 장점
```
> new 연산자를 활용한 구체 클래스의 인스턴스화는 기본적으로 변하기 쉬우며, 이를 코드 상에 흩어놓을수록 코드의 유연성과 유지보수성은 떨어진다.
```
* **변할 수 있는 부분은 반드시 캡슐화되어야 하는 반면, new 연산자를 사용하여 구체 클래스의 객체를 생성하는 코드 역시 변하기 쉬운 축**에 속한다.
  * 때문에 팩토리 패턴을 사용할 경우, 이렇듯 변하기 쉬운 인스턴스 생성 코드를 쉽게 캡슐화할 수 있다.
* 팩토리 패턴을 적용할 경우, 크게 다음과 같은 이점을 누릴 수 있다.
  1. 객체 생성 코드를 전부 하나의 객체 또는 메소드에 모아 중복을 제거할 수 있다.
  2. 객체 생성 코드가 여러 곳에 흩어지지 않고 코드의 한 지점에 집중되므로, 유지보수성이 높아진다.
  3. **팩토리를 통해 구체 클래스의 인스턴스를 생성할 경우, 인터페이스를 바탕으로 개발이 가능하므로 유연성과 확장성이 높은 애플리케이션을 개발**할 수 있다.
* **팩토리 패턴이 코드 상에서 new 연산자를 완전 제거하는 것은 아니며, Java의 예로 들어 new 연산자 없이 애플리케이션을 개발하는 것은 불가능에 가깝다**.
  * **팩토리 패턴은 대신 변화하기 쉬운 객체 생성 코드를 코드의 한 지점에 모아 체계적으로 관리하는 데에 의의가 있으며, 유연성과 유지보수성을 높인다**.

## 2022-10-28 Fri
### 객체의 의존성
* **new 연산자 등을 활용하여 구체 클래스의 인스턴스를 직접 생성하는 클라이언트 코드는 해당 구체 클래스에 강하게 결합되고, 의존**하게 된다.
* 때문에 팩토리를 사용하지 않고 자신이 사용할 모든 구체 클래스를 직접 생성하는 클라이언트 코드는 모든 구체 클래스에 직접적으로 의존하게 된다.
  * 이 경우, 각 구체 제품 클래스가 변경되면 클라이언트 클래스까지 함께 변경되어야할 가능성이 존재한다.
  * **이를 의존한다는 단어로 표현할 수 있으며, 또한 새로운 구체 제품 클래스가 추가되면 클라이언트 클래스는 더욱 많은 구체 클래스에 의존**하게 된다.

### 여섯 번째 디자인 원칙 - 의존성 역전
```
> 객체는 구체 클래스처럼 구체적인 것에 의존하지 않는 대신, 추상 클래스 또는 인터페이스와 같이 추상적인 것에 의존해야 한다.
> 또한, 의존성 역전 원칙은 고수준 모듈과 저수준 모듈 모두에 적용될 수 있다.
```
* 구체 클래스에 대한 의존성은 애플리케이션의 유연성을 떨어트리므로, 구체 클래스에 대한 의존성은 줄일수록 좋다고 볼 수 있다.
* 애플리케이션의 구성 요소는 크게 다음과 같이 분류해볼 수 있다.
  1. 고수준 구성 요소: 저수준 구성 요소에 의해 정의되는 행동을 포함하는 요소이다.
  2. 저수준 구성 요소: 고수준 구성 요소의 세부적인 행동을 의미하는 요소이다.
* 이 때, **의존성 역전 원칙은 또한 고수준 구성 요소가 저수준 구성 요소에 의존하는 대신 항상 추상에 의존해야 함을 강조**한다.

### 의존성 역전 원칙과 팩토리 패턴
* 팩토리 패턴을 전혀 사용하지 않고 클라이언트 코드에서 모든 구체 제품 클래스를 생성하는 경우, 생산자는 저수준 구성 요소인 구체 제품 클래스에 의존한다.
  * **이러한 의존성은 클라이언트 코드에서 new 연산자 등을 활용하여 구체 클래스의 객체를 직접 생성하기 때문**이다.
  * 이 경우에는 제품 클래스들의 공통 동작을 추상 제품 클래스로 추출한다고 해도 얻을 수 있는 이점이 크지 않다.
* 이러한 코드에 의존성 역전 원칙을 준수하기 위해 팩토리 메소드 패턴을 적용할 경우, 코드는 다음과 같은 특징을 갖게 된다.
  1. **추상 생산자 클래스는 추상 제품 클래스에만 의존하며, 런타임에서 어떤 구체 클래스가 반환될지 알 수도 없고 알 필요도 없다**.
  2. **구체 생산자 클래스에서 구현된 팩토리 메소드에서 생성하는 모든 구체 제품 클래스 역시 추상 제품 클래스에 의존**한다.
* 즉, **팩토리 메소드 패턴을 적용함으로써 고수준 구성 요소인 생산자 클래스와 저수준 구성 요소인 제품 클래스 모두 추상화에 의존**하게 된다.
  * 이 때, 의존 대상인 추상은 추상 제품 클래스가 된다.
  * 물론 **팩토리 메소드 패턴이 의존성 역전 원칙을 준수하는 유일한 원칙은 아니지만, 적합한 방법 중 하나임은 명확**하다.

### 의존성 역전 원칙 준수 가이드라인
* 다음과 같은 가이드를 준수할 경우, 의존성 역전 원칙을 준수하고 이를 위배하는 디자인을 지양할 수 있다.
  1. 변수에 구체 클래스의 레퍼런스를 저장하지 않도록 한다.
  2. 구체 클래스로부터 유도되는 클래스를 만들지 않도록 한다.
  3. 베이스 클래스에 이미 구현된 메소드를 오버라이드하지 않도록 한다.
* **new 연산자 등을 활용하면 구체 클래스의 레퍼런스를 사용하게 되므로, 가능한 한 팩토리 패턴을 도입하여 구체 클래스의 레퍼런스를 저장하지 않도록 한다**.
* 구체 클래스로부터 유도되는 클래스를 정의할 경우, 특정한 구체 클래스에 의존하게 될 수 밖에 없다.
  * 따라서 **구체 클래스보다는 인터페이스 또는 추상 클래스 등 추상화에 의존하는 클래스를 정의하는 것이 바람직**하다.
* 이미 구현된 메소드를 오버라이드하는 경우, 베이스 클래스는 적절히 추상화되지 않는다.
  * 때문에 **베이스 클래스에서 메소드를 정의하는 경우, 가능한 한 모든 서브클래스로부터 공유될 수 있는 것만을 정의하는 것이 바람직**하다.

### 가이드라인을 반드시 준수하기?
* 엄밀히 말해, 상술한 가이드라인을 모두 준수하는 Java 애플리케이션은 없다고 볼 수 있다.
* 즉, **상술한 가이드라인은 반드시 준수해야 할 규칙이라기보다는 개발자가 지향해야 할 것을 알려주는 것으로 이해**할 수 있다.
  * 무엇보다도 가이드라인을 명확히 이해한 후에 애플리케이션을 디자인한다면 원칙을 준수하지 않는 부분을 쉽게 찾아낼 수 있다.
  * 또한, **가이드라인을 위배하는 부분을 찾아냈다면 합리적인 이유가 있는지 확인하여 불가피한 상황에 예외를 두는 식으로 개발을 진행**할 수 있다.
  * 예를 들어 **String 클래스에 의존하는 것 역시 가이드라인을 위배하지만, 사실 거의 변하지 않을 클래스에 대해서는 의존하더라도 상관이 없다**.
* 그러나 **자신이 개발하는 클래스가 변경될 가능성이 존재하는 경우, 팩토리 메소드 패턴을 도입하여 변경될 수 있는 부분은 캡슐화하는 것이 바람직**하다.

### 다양한 구성 요소를 다루는 팩토리
* 예를 들어, 생산자 클래스에서 생성할 제품 클래스가 다시 여러 구성 요소로 구성되는 경우가 있을 수 있다.
  * 이 경우, **구체 생산자 클래스마다 다른 구성 요소의 집합을 사용하여 구체 제품을 생성**할 수 있다. 
  * 때문에 이러한 요구사항을 충족시키기 위해서는 반드시 구성 요소의 집합을 처리할 방법을 고려할 수 있어야 한다.
* 이렇듯 **다양한 구성 요소를 기반으로 제품을 생성하는 경우, 각 구성 요소를 생성할 수 있는 또 다른 팩토리를 정의하는 것이 바람직**하다.
  * 또한, **이렇게 정의된 팩토리 역시 각 구성 요소의 집합을 기반으로 확장 또는 구현될 수 있는 추상 팩토리**여야 한다.
* 이러한 팩토리는 크게 다음과 같은 흐름으로 정의할 수 있다.
  1. 제품 클래스를 구성하는 구성 요소 별로 추상 구성 요소를 정의한다.
  2. 추상 구성 요소 집합을 생성할 수 있는 추상 팩토리 클래스를 정의한다.
  3. 1.의 과정에서 정의된 각 추상 구성 요소를 기반으로 구체 구성 요소 클래스를 정의한다.
  4. 구체 제품 클래스 별로 사용할 수 있는 구체 팩토리 클래스를 2.의 과정에서 생성된 추상 팩토리 클래스를 기반으로 정의한다.
  5. 구체 생산자 클래스가 4.에서 정의된 구체 팩토리 클래스를 기반으로 구체 제품 클래스를 생성할 수 있도록 수정한다.
* 즉, **해당 방식은 여러 개의 구성 요소 집합을 기반으로 생성되는 제품 클래스 집합의 생성 과정을 캡슐화할 때 유용**하다.
  * 이 경우, **각 구체 제품 클래스를 구성하는 구성 요소를 생성하기 위한 팩토리는 제품 클래스를 멤버 변수로 참조하는 구체 생산자가 전달**한다.
  * 때문에 구체 생산자 클래스의 종류에 따라 동일한 구체 제품 클래스를 생성하더라도 제품 클래스를 구성하는 구체 구성 요소는 달라질 수 있다. 

## 2022-10-29 Sat
### 추상 제품 클래스 수정하기
* 다양한 구성 요소를 사용하는 경우, 추상 제품 클래스는 추상 구성 요소들에 대한 멤버 변수를 갖는다.
* 또한, 별도의 준비용 추상 메소드를 통해 각 추상 구성 요소를 서브 클래스에서 구체 구성 요소로 결정할 수 있도록 작성할 수 있다.
  * **각 멤버 변수에 참조를 전달할 구체 구성 요소는 구성 요소를 반환하는 팩토리로부터 반환받아 참조에 저장하는 준비용 메소드를 정의하는 식으로 동작**한다.
  * 이렇게 **정의된 준비 용 메소드는 추상 메소드로 정의되며, 추상 제품 클래스의 서브클래스인 구체 제품 클래스에서 결정**한다.  

### 구체 제품 클래스 수정하기
* 해당 시나리오의 경우, 구체 제품 클래스를 구성하는 구성 요소만 변경되므로 코드는 구성 요소 팩토리로부터 가져오는 식으로 수정되어야 한다.
  * 이 경우, **각 제품 클래스는 서로 다른 구체 구성 요소를 사용할 뿐이며 구성 요소를 사용하는 방식은 동일**해야 한다. 
  * 때문에 모든 **구성 요소 조합에 따라 제품 클래스를 별도로 정의할 필요가 없으며, 대신 구성 요소를 반환하는 행동을 별도의 구성 요소 팩토리에 위임**한다.
* 이 때, **구체 구성 요소를 반환해줄 구성 요소 팩토리는 일반적으로 구체 제품 클래스의 멤버 변수로 참조되며, 생성자를 통해 런타임에 제공**된다.
  * 구체 구성 요소를 생성하는 팩토리를 기반으로 구체 제품 클래스가 완료되는 반면, 구체 제품 클래스는 생성자에 어떤 구성 요소 팩토리가 전달되는지 알 수 없다.
  * 이 역시 **추상에 의존하는 것이며, 구체 제품 클래스는 생성자에 전달된 팩토리가 구성 요소를 반환할 수 있기만 하면 세부 사항에 신경 쓰지 않는다**.
  * 즉, 해당 방식을 사용하는 모든 구체 제품 클래스는 반환 받은 구성 요소가 수신할 수 있는 메시지에만 집중하며 이를 사용하는 방법만을 명시한다.

### 구체 생산자 클래스 수정하기
* **구체 제품 클래스에 전달되는 구체 구성 요소 팩토리의 경우, 일반적으로 구체 생산자 클래스가 전달할 수 있다**.
  * 각 구체 생산자 클래스는 적절한 종류의 구체 구성 요소 팩토리가 구체 제품 클래스에 전달될 수 있도록 보장해야 한다.
  * 이에 따라 **모든 구체 제품 클래스를 구성할 구성 요소들은 구체 생산자 클래스가 전달한 구체 구성 요소 팩토리로부터 생성되어 반환**된다.
  * 즉, 구체 구성 요소의 집합은 구체 생산자 클래스로부터 결정되므로 이를 사용하는 구체 제품 클래스 자신은 구성 요소에 대한 세부 사항을 알 수 없다.

### 추상 팩토리 패턴의 시작
* **추상 팩토리는 새로운 유형의 팩토리 패턴으로, 임의의 제품군을 생성하기 위한 인터페이스를 제공**한다.
  * 이 때, **제품군이란 각각의 구체 제품 클래스에서 사용할 모든 구체 구성 요소 집합을 의미**한다.
* **추상 팩토리 패턴은 지역이나 운영체제 및 룩앤 필 등, 환경에 따라 다른 클래스 집합을 사용하는 경우에 적절**하다.
  * 즉, 추상 팩토리 패턴을 통해 서로 다른 상황에 맞는 제품군을 생산하는 팩토리를 구현할 수 있다.
  * 이 경우 각 제품을 사용하는 클라이언트 코드와 제품을 생산하는 코드가 분리되므로, 다른 동작이 필요할 경우에는 단지 팩토리를 교체하기만 하면 된다.
* **추상 팩토리 패턴을 적절히 사용할 경우, 같은 기능을 수행하는 제품을 다른 방식으로 구현하는 구체 팩토리를 손쉽게 정의**할 수 있다.
  * 팩토리에서는 제품을 생산하는 코드를 작성하되, 동일한 인터페이스를 구현하는 다양한 종류의 팩토리를 전달할 수 있으므로 여러 방식으로 제품을 생성할 수 있다.
  * 그러나 다양한 팩토리를 전달하는 과정에서, 실제로 반환된 제품을 사용하기 위한 클라이언트 코드를 바꿀 필요는 없으므로 애플리케이션은 유연성을 얻을 수 있다.

## 2022-10-30 Sun
### 추상 팩토리 패턴의 정의
```
> 추상 팩토리 패턴은 구체 클래스에 의존하지 않으면서도 서로 연관되거나 의존적인 객체로 구성되는 제품군을 생산할 수 있는 인터페이스를 제공한다.
> 이 때, 실제 구체 클래스는 서브클래스에서 결정된다.
```
* **추상 팩토리 패턴을 활용할 경우, 제품 클래스를 사용하는 클라이언트 코드에서는 추상 인터페이스로 일련의 제품군을 생성**할 수 있다.
  * 이 경우, **클라이언트 코드와 팩토리의 객체 생성 코드는 완전히 분리되므로 실제로 어떤 구체 제품 클래스가 생성되는지는 전혀 알 필요가 없다**.
* 추상 팩토리 패턴은 크게 다음과 같은 구성 요소로 정의된다.
  1. 클라이언트 코드: **팩토리가 반환하는 제품군을 실제로 사용하는 코드에 해당하며, 추상 팩토리 클래스를 기반으로 일련의 제품군을 생성**한다.
  2. 추상 팩토리 클래스: **모든 구체 팩토리 클래스에서 구현하는 인터페이스이며, 각각의 제품을 생산하기 위한 일련의 메소드를 정의**한다. 
  3. 구체 팩토리 클래스: 서로 다른 제품 집합을 구현하며, 클라이언트 코드에서 임의의 제품을 사용하려면 구체 팩토리 클래스 중 적절한 것을 골라 사용한다.
  4. 추상 제품군: 각 구체 팩토리 클래스에서 생성할 제품군 별 인터페이스에 해당하며, **클라이언트 코드 역시 추상 제품군에 의존**한다.
  5. 구체 제품군: 각각의 구체 팩토리 클래스에서 실제로 생성하는 제품군 별 구체 클래스에 해당한다.
* **추상 팩토리 클래스는 서로 연관된 일련의 제품군을 구성하는 개별 추상 제품을 생성하는 방법을 정의하는 추상 인터페이스**이다.
  * 반면 구체 팩토리 클래스는 실제 제품군을 생성할 수 있도록 구체 제품을 생성하는 책임을 갖는다.
* **추상 팩토리 클래스를 멤버 변수로 참조하는 클라이언트 코드는 주입받은 구체 팩토리가 구현한 제품군을 실제로 사용**한다.

### 추상 팩토리 패턴과 팩토리 메소드 패턴의 관계
* 상술했듯 **추상 팩토리 클래스에서 제품군을 구성하는 각 추상 제품을 생성하는 방법을 개별 메소드로 정의하고, 이를 구체 팩토리 클래스에서 구현**한다.
  * 이러한 방식은 일견 실제 제품을 생성하는 팩토리 메소드를 서브클래스에서 결정하는 팩토리 메소드 패턴과 차이가 없어보일 수 있다.
* **추상 팩토리는 제품군을 만들기 위해 사용될 수 있는 인터페이스를 정의하기 위해 고안된 패턴이며, 인터페이스에 포함된 각 메소드는 구체 제품을 생성**한다.
  * **때문에 추상 팩토리 클래스의 서브클래스에서 각 제품을 생성하는 메소드의 구현을 제공하기 위해 팩토리 메소드 패턴이 사용되는 것은 자연스럽다**.
  * 실제로도 추상 팩토리 패턴에서 메소드가 팩토리 메소드로 구현되는 경우는 드문 일이 아니다.

### 추상 팩토리 패턴과 팩토리 메소드 패턴의 차이
```
> 추상 팩토리 패턴은 클라이언트에서 서로 연관된 일련의 제품을 만들어야하는 경우, 즉 제품군을 만드는 경우에 활용할 수 있다.
> 팩토리 메소드 패턴은 클라이언트와 인스턴스를 생성하는 구체 클래스를 분리시키는 경우에 유용하며, 어떤 구체 클래스가 필요할지 미리 알 수 없을 때도 유용하다.
```
* **추상 팩토리 패턴은 제품군을 생성하는 추상 인터페이스를 제공하며, 각 서브클래스에서 제품군을 결정**한다.
  * 때문에 각각의 구체 팩토리 클래스마다 서로 다른 제품군을 구현할 수 있다.
* 반면 **팩토리 메소드 패턴은 하나의 제품을 생성하는데에 필요한 추상 인터페이스를 제공하며, 각 서브클래스에서 구체 클래스를 결정**한다.
  * 이로 인해 **팩토리 메소드 패턴은 심지어 어떤 구체 클래스가 필요한지 미리 알 수 없는 경우에도 유용하게 사용**할 수 있다.
* **두 패턴은 모두 객체 생성을 캡슐화하여 애플리케이션의 결합을 느슨하게 하고, 특정한 구현에 덜 의존하도록 만드는 공통점**을 갖는다.
  * 즉, **두 패턴 모두 구체적인 객체 생성 코드를 애플리케이션으로부터 분리하는 역할을 수행**한다. 
* 그러나 두 패턴은 다음과 같은 명확한 차이 역시 존재한다.
  1. **추상 팩토리 패턴은 객체 구성(composition)을 활용**한다.
  2. **팩토리 메소드 패턴은 객체 상속을 활용**한다.
* **팩토리 메소드는 우선 클래스를 확장한 후에 팩토리 메소드를 오버라이드하며, 이로 인해 클라이언트와 임의의 구체 형식을 분리**할 수 있다.
* 반면 **추상 팩토리 패턴은 제품군을 생성할 수 있는 추상 형식을 제공하지만, 팩토리 메소드와 마찬가지로 제품군을 생성하는 방법은 서브클래스에서 정의**한다.
  * 이렇듯 **추상 팩토리 패턴은 구체 팩토리 클래스로 제품 생성 메소드를 구현하는 과정에서 팩토리 메소드 패턴을 사용할 수도 있다**.
  * 추상 팩토리 패턴의 클라이언트는 제품군을 생성할 수 있는 추상 팩토리를 멤버로 참조하며, 런타임에 실제 제품군을 생성할 수 있는 구체 팩토리를 전달받는다.
  * 즉, **이로 인해 추상 팩토리 패턴은 팩토리 메소드 패턴과 마찬가지로 클라이언트 코드와 실제로 사용되는 구체 제품 클래스를 분리**할 수 있다.
* **추상 팩토리 패턴은 연관된 일련의 제품군을 하나로 묶을 수 있는 유용함을 제공하지만, 그로 인해 추상 팩토리 인터페이스가 거대해질 수 있다**.
  * 때문에 **추상 팩토리 패턴에서 새로운 제품을 추가하는 식으로 기능이 추가되는 경우, 부득이하게 인터페이스가 변경할 수 밖에 없다**.
  * 인터페이스의 변경은 모든 구체 팩토리 클래스에도 반영되어야 하므로, 이러한 수정 작업은 자칫 큰 작업으로 번질 수 있다.
* 반면 **팩토리 메소드 패턴은 일반적으로 인터페이스가 변경될 일은 없으나, 추상 팩토리 패턴처럼 제품군을 생성할 수는 없다**.

### 팩토리 패턴 - 결론
```
> 팩토리 메소드 패턴과 추상 팩토리 패턴은 모두 객체 생성 코드를 캡슐화하여 클라이언트 코드와 구체 클래스를 분리시킨다.
> 팩토리 패턴을 활용하여 클라이언트 코드와 구체 클래스가 분리할 경우, 그만큼 느슨하게 결합된 유연한 디자인을 설계할 수 있게 된다.
```
* 추상 팩토리 패턴은 구체 클래스에 의존하지 않고도 서로 연관되거나 의존적인 객체 제품군을 생성하는 인터페이스를 제공한다.
  * 또한 추상 팩토리 패턴은 객체 구성을 활용하며, 추상 팩토리의 인터페이스에서 정의한 메소드로부터 객체 생성을 구현한다.
* 팩토리 메소드 패턴은 객체를 생성하기 위해 필요한 인터페이스를 제공하며, 객체를 실제로 생성하는 작업을 서브클래스에게 위임한다.
  * 또한 팩토리 메소드 패턴은 객체 상속을 활용하며, 서브클래스에서 팩토리 메소드를 구현하여 객체를 생성한다.
* **모든 팩토리 패턴은 구체 클래스에 대한 의존성을 줄여주며, 이는 DIP를 활용하여 구체 클래스에 대한 의존 대신 추상화를 지향할 수 있는 점에서 기반**한다.
* **팩토리 패턴은 구체 클래스가 아닌 추상 클래스와 인터페이스에 의존하는 코드를 작성할 수 있도록 하는 강력한 기법**이다.
* **간단한 팩토리는 엄밀하게 따져봤을 때 디자인 패턴은 아니지만, 클라이언트와 구체 클래스를 분리하는 간단한 방법으로 활용**할 수 있다.