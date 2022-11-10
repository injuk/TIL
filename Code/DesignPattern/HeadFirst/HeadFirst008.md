# HeadFirst
## 2022-11-10 Thu

### 알고리즘의 캡슐화
* 다른 디자인 패턴이 객체 생성이나 메소드 호출, 또는 복잡한 인터페이스를 캡슐화하듯 알고리즘을 캡슐화할 수도 있다.
  * 예를 들어, **서브클래스에서 필요할 때마다 알고리즘을 가져다 사용할 수 있도록 캡슐화**할 수 있다.

### 템플릿 메소드 패턴의 필요성
* 예를 들어 다음과 같은 두 클래스가 존재한다고 가정한다.
```java
class TempA {
    public void doSomething() {
        preProcess();
        doTaskA();
        postProcess();
        doTaskB();
    }
    // 아래 메소드들은 본문이 구현되어 있다고 가정한다.
    public void preProcess() {}
    public void doTaskA() {}
    public void postProcess() {}
    public void doTaskB() {}
}

class TempB {
  public void doSomething() {
    preProcess();
    doTaskC();
    postProcess();
    doTaskD();
  }
  // 아래 메소드들은 본문이 구현되어 있다고 가정한다.
  public void preProcess() {}
  public void doTaskC() {}
  public void postProcess() {}
  public void doTaskD() {}
}
```
* 이 경우, 전체적으로 유사한 동작을 수행하지만 세세한 부분이 다르므로 공통되는 코드를 추상화하여 다음과 같이 수정할 수 있다.
```java
class TempBase {
    abstract void doSomething();
    // 아래 메소드들은 본문이 구현되어 있다고 가정한다.
    public void preProcess() {}
    public void postProcess() {}
}
class TempA extends TempBase {
    @Override public void doSomething() {
        preProcess();
        doTaskA();
        postProcess();
        doTaskB();
    }
    // 아래 메소드들은 본문이 구현되어 있다고 가정한다.
    public void doTaskA() {}
    public void doTaskB() {}
}

class TempB extends TempBase {
  @Override public void doSomething() {
    preProcess();
    doTaskC();
    postProcess();
    doTaskD();
  }
  // 아래 메소드들은 본문이 구현되어 있다고 가정한다.
  public void doTaskC() {}
  public void doTaskD() {}
}
```
* 그러나 상술한 방식의 코드는 다음과 같은 아쉬운 점이 존재한다.
  1. 여전히 `doSomething()` 메소드가 중복된다.
  2. `doSomething()` 메소드가 호출하는 일련의 흐름이 사실상 유사하다.

### 템플릿 메소드 패턴 적용하기
* 상술한 코드에서, doTaskA - doTaskC와 doTaskB - doTaskD가 크게 다르지 않은 경우에는 다음과 같이 인터페이스를 추상 클래스로 바꿀 수 있다.
```java
abstract class TempBase {
    // 전체적인 작업의 흐름이 같으므로, 서브클래스가 함부로 오버라이드할 수 없도록 final로 정의한다.
    final void doSomething() {
        preProcess();
        doFirstTask();
        postProcess();
        doLastTask();
    }
    
    // 서브클래스에서 변경될 수 있는 메소드이므로 추상 메소드로 정의한다.
    abstract void doFirstTask();
    abstract void doLastTask();

    // 서브클래스에서 변경이 없는 메소드는 구체 메소드로 정의하며, 아래 메소드들은 본문이 구현되어 있다고 가정한다.
    public void preProcess(){}
    public void postProcess(){}
} 
```
* 이제 **TempA와 TempB 클래스는 추상 클래스인 TempBase를 확장하며, doFirstTask와 doLastTask를 적절하게 오버라이드**할 수 있다.
  * 이러한 수정은 **여러 클래스의 전체적인 행동 양식은 같지만, 사소한 차이가 있는 경우에 일련의 과정을 일반화한 것에 해당**한다.
  * 이러한 방법은 **일련의 과정을 구성하는 메소드 중 서브클래스마다 세부적인 동작 양식이 달라지는 경우, 해당 메소드를 구현하여 유연성**을 둘 수 있다.
* **이러한 방식을 선택할 경우, 전체적인 작업 과정과 일부 메소드의 구현은 서브클래스가 추상 부모 클래스에 의존**하게 된다.
  * 반면, 일부 메소드는 서브클래스가 부모 클래스에 의존하는 대신 직접 구현하여 처리하게 된다.
* 이러한 디자인 패턴을 템플릿 메소드 패턴이라고 하며, 각 메소드는 다음과 같은 역할을 수행한다.
  1. **doSomething 메소드는 템플릿 메소드로서, 어떠한 알고리즘의 전체적인 흐름을 의미하는 틀 역할을 수행**한다.
  2. **템플릿 메소드 내에서 알고리즘의 각 단계는 추상 클래스에 정의된 여러 메소드로 표현**된다.
  3. **템플릿 메소드를 구성하는 일부 메소드는 추상 클래스에서 구현되는 반면, 필요한 경우 abstract 키워드를 명시하여 서브클래스에서 처리**될 수도 있다.
* 이렇듯 **템플릿 메소드는 알고리즘의 각 단계를 정의하며, 서브클래스에서 알고리즘의 일부를 직접 구현할 수 있도록 유도**한다.

## 2022-11-11 Fri
### 템플릿 메소드 패턴이란?
```
> 템플릿 메소드는 일련의 알고리즘을 관리한다.
> 또한, 알고리즘을 수행하는 과정에서 상황에 따라 임의의 단계는 서브클래스에서 처리하도록 할 수도 있다.
```
* **템플릿 메소드에 정의된 알고리즘은 일반적으로 다른 누구도 변경할 수 없으며, 단지 일부 단계 또는 전체 단계를 서브클래스가 변경**할 수 있다.
* 템플릿 메소드를 사용할 경우, 다음과 같은 장점을 누릴 수 있다.
  1. 추상 부모 클래스가 전체적인 알고리즘을 독점하며, 이를 관리한다.
  2. 추상 부모 클래스 덕분에 서브클래스에서는 코드를 재사용할 수 있다.
  3. 전체적인 알고리즘이 추상 부모 클래스 한 곳에 집중되므로, 일반적으로 수정 범위가 적고 일부 구현만을 서브클래스에 의존하게 된다.