# DesignPattern
## 2022-02-03 Thu

## Builder Pattern
### 기본 개념
* 빌더 패턴은 복잡한 오브젝트의 생성 과정을 간단하게 바꾸어주는 패턴이다.
  * 예를 들어 하나의 오브젝트의 생성에 여러 인수가 필요한 경우가 빌더 패턴을 사용하기 좋은 예시이다.

### 코드 예시
* inner Builder 클래스를 정의하며, Builder 내부에서 새로운 대상 오브젝트를 만들어 활용하는 방식 
```
public class BuilderMain {

    public static void main(String[] args) {
        BuilderTarget fromConstructor = new BuilderTarget();
        System.out.println(fromConstructor);

        BuilderTarget withBuilder = BuilderTarget.builder()
                .isStudent(true)
                .age(3)
                .name("Hellooo")
                .build();

        System.out.println(withBuilder);
    }
}

class BuilderTarget {
    String name;
    int age;
    boolean isStudent;

    @Override
    public String toString() {
        return "name: " + name + ", age: " + age + ", isStudent: " + isStudent;
    }

    public static Builder builder() {
        return new Builder();
    }

    public static class Builder {
        BuilderTarget target = new BuilderTarget();

        public Builder name(String name) {
            target.name = name;
            return this;
        }
        public Builder age(int age) {
            target.age = age;
            return this;
        }
        public Builder isStudent(boolean isStudent) {
            target.isStudent = isStudent;
            return this;
        }
        public BuilderTarget build() {
            return target;
        }
    }
}
```

### 코드 예시 2
* Builder 자체에 동명의 인스턴스 변수를 갖고, 이를 생성자에 넘기도록 활용하는 방식
```
public class BuilderMain {

    public static void main(String[] args) {
        BuilderTarget builderTarget = BuilderTarget
            .builder()
            .name("hello")
            .age(3)
            .isStudent(true)
            .build();
        System.out.println(builderTarget);
    }
}

class BuilderTarget {
    private final String name;
    private final int age;
    private final boolean isStudent;

    @Override
    public String toString() {
        return "name: " + name + ", age: " + age + ", isStudent: " + isStudent;
    }

    private BuilderTarget(Builder builder) {
        name = builder.name;
        age = builder.age;
        isStudent = builder.isStudent;
    }
    
    public static Builder builder() {
        return new Builder();
    }

    public static class Builder {
        private String name;
        private int age;
        private boolean isStudent;

        public Builder name(String name) {
            this.name = name;
            return this;
        }
        public Builder age(int age) {
            this.age = age;
            return this;
        }
        public Builder isStudent(boolean isStudent) {
            this.isStudent = isStudent;
            return this;
        }
        public BuilderTarget build() {
            return new BuilderTarget(this);
        }
    }
}
```