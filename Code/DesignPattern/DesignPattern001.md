# Pattern
## 2022-02-02 Wed

## Template Method Pattern
* 템플릿 메소드 패턴은 고차원의 중복을 제거하기 위해 자주 사용되는 기법이다.
* 템플릿 메소드 패턴은 어떤 메소드의 절차, 흐름을 추상 메소드로 정의하고 이를 자식 클래스에서 구현할 수 있도록 한다.
  * 아래의 예시를 상속받는 클래스는 항상 정해진 순서의 작업을 수행하지만, 작업의 내용은 자식 클래스에서 구현하도록 한다.
```
abstract class Template {
    public abstract void given();
    public abstract void when();
    public abstract void then();
    
    // 실제 동작의 내용은 구현되어야 하지만, 템플릿 메소드는 항상 같은 flow를 갖도록 강제한다.
    public void templateMethod() {
        given();
        when();
        then();
    }
}
```
* 템플릿 메소드 패턴은 로직이 유사한 두 메소드의 공통 부분은 추상 클래스에서 구현하고, 상속받는 클래스에서 추상 메소드를 구현하는 방식으로도 사용할 수 있다.
  * 이 때, 두 메소드에서 달랐던 로직을 추상 메소드로 정의하여 자식 클래스가 이를 구현하도록 유도한다.
```
class Before {
    public int num;

    public void isPositive() {
        this.num++; // 중복된다.

        if(this.num < 0)
            this.num *= 1;

        System.out.println(this.num); // 중복된다.
    }

    public void isNegative() {
        this.num++;

        if(this.num > 0)
            this.num *= -1;

        System.out.println(this.num);
    }
}

// 템플릿 메소드 패턴
// 중복되는 부분은 구체 메소드로 정의하고, 중복되지 않는 부분은 추상 메소드로 정의한다.
// 구현된 메소드들을 토대로 기존의 비즈니스 로직을 파사드 패턴의 메소드로 정의한다.
abstract class After {
    public int num;

    // 기존 비즈니스 로직의 형태
    public void is() {
        plus();
        multiply();
        print();
    }

    public void plus() {
        this.num++;
    }

    public abstract void multiply();

    public void print() {
        System.out.println(this.num);
    }
}

// 중복되지 않는 로직은 자식 클래스에서 구현한다.
class SeparatedPositive extends After {
    @Override
    public void multiply() {
        if(this.num < 0)
            this.num *= 1;
    }
}

// 중복되지 않는 로직은 자식 클래스에서 구현한다.
class SeparatedNegative extends After {
    @Override
    public void multiply() {
        if(this.num > 0)
            this.num *= -1;
    }
}
```