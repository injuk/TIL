# Pattern
## 2022-02-02 Wed

## Template Method Pattern
* 템플릿 메소드 패턴은 고차원의 중복을 제거하기 위해 자주 사용되는 기법이다.
* 템플릿 메소드 패턴이란, 로직이 유사한 두 메소드의 공통 부분은 추상 클래스에서 구현하고, 각 메소드를 추상 클래스를 상속받는 클래스에서 구현하는 기법이다.
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