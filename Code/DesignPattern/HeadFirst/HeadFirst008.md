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