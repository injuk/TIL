# Java
## 2022-01-11 Tue

### 제네릭
* 제네릭은,
    1. 다양한 타입의 인스턴스를 다루는 메소드나,
    2. 다양한 타입의 인스턴스를 다루는 컬렉션 클래스에,
* 컴파일 시 타입체크를 해주는 기능이다.
  * 쉽게 말해, 어떤 클래스나 메소드가 다룰 타입을 미리 명시하는 것으로 번거로운 형변환을 줄여준다.
  * 덕분에 타입체크와 형변환 코드를 생략할 수 있어 코드가 간결해진다.
* 인스턴스의 타입을 컴파일 시점에 체크하므로 인스턴스의 타입 안정성을 높이고, 형변환의 번거로움이 줄어든다.
    1. 타입 안정성을 높인다: 원하지 않은 타입의 인스턴스가 저장되는 것을 막는다.
    2. 형변환의 번거로움이 줄어든다: 인스턴스를 꺼내올 때 다른 타입으로 잘못 형변환되는 상황을 줄여준다.
* ArrayList와 같은 컬렉션 클래스는 다양한 타입의 인스턴스를 담을 수 있지만, 일반적으로는 하나의 타입만을 다룬다.
  * 이 때, 꺼낼 때마다 인스턴스의 타입을 체크하는 것은 불편하다.
  * 또한, 꺼낼 때마다 형변환을 해주는 것도 불편하다.
  * 심지어 원치 않는 타입의 인스턴스가 저장되는 것을 막을 방법이 없다.
  * 제네릭은 이러한 문제점을 해결해준다!

### 클래스의 제네릭
* 클래스에 선언하는 제네릭은 다음과 같다.
```
class Temp<T> {
    T temp;
    void setTemp(T temp) {
        this.temp = temp;
    } 
}
```
* 이 때, <T>는 타입 변수라고 하며, Type의 머릿 글자를 따온 것으로 이해할 수 있다.
  * T가 아닌 다른 글자를 사용하여도 무방하다.
  * 오히려 항상 T를 고집하는 것보다는 시멘틱을 고려한 문자 선정이 바람직하다.
* 타입 변수가 여럿인 경우, <K, V>와 같이 쉼표로 구분한다.
* 타입 변수는 작성한 글자에 관계 없이 **임의의 참조형 타입**을 의미한다.
* JDK 1.5에서 제네릭이 출시되기 전에는 다양한 타입을 다루는 클래스는 모두 Object 타입을 사용하였다.
  * 그러나 **이러한 방식은 번거로이 형변환 작업을 해주어야 하는 문제점**이 있다.
  * 제네릭을 사용할 경우, 타입 변수를 지정하는 것으로 원하는 참조형 타입을 사용할 수 있다.
* 제네릭 클래스인 Temp 클래스의 객체를 생성하는 경우, 다음과 같이 Temp에서 사용할 실제 타입을 명시해준다.
```
Temp<Integer> temp = new Temp<Integer>();
temp.setTemp(1); // 오토 박싱
```
* JDK 1.5 이전에 작성된 코드와의 호환성 유지를 위해, 제네릭 클래스여도 타입 변수를 생략한 인스턴스 생성이 허용된다.
  * **이 경우, 타입 변수 T는 Object로 간주**된다.
```
Temp temp = new Temp();
temp.setTemp(1);
```
* 제네릭 클래스의 인스턴스를 생성할 때, 인스턴스 별로 다른 타입을 지정할 수 있다. 애초에 제네릭은 인스턴스 별로 다르게 동작시키기 위해 고안되었다.
* 그러나 모든 인스턴스에 대해 동일하게 동작해야하는 static 멤버에는 타입 변수를 적용할 수 없다.
  * **타입 변수는 인스턴스 변수로 간주**된다!
  * static 멤버는 타입 변수에 지정된 타입과 관계 없이 동일해야 하기 때문이다.
* 제네릭 타입의 배열은 생성할 수 없다.
  * new 연산자는 컴파일 시점에 타입 변수의 값을 정확히 알아야 한다.
  * 그러나 컴파일 시점에 타입 변수의 값을 정확히 알 수 없는 경우가 있으므로, new를 통한 생성이 필요한 배열에는 제네릭 타입을 적용할 수 없다.

### 제네릭 클래스의 사용
* 상술했듯, 제네릭 클래스는 다음과 같은 형태로 사용한다.
```
public class GenericTest {
    public static void main(String[] args) {
        Temp<String> temp = new Temp<String>();
        temp.setTemp("Hello");
        System.out.println(temp.getTemp());
    }
}

class Temp<T> {
    T temp;

    public T getTemp() {
        return this.temp;
    }

    public void setTemp(T temp) {
        this.temp = temp;
    }
}
```
* Temp<String> temp = new Temp<String>();에서, 참조 변수에 매개변수화된 타입과 생성자에 매개변수화된 타입은 같아야 한다.
  * 참조 변수에 매개변수화된 타입이 생성자에 매개변수화된 타입의 부모여도 불가능하다.
  * 반면 참조 변수의 타입(이 경우, Temp)이 생성자에서 호출된 클래스의 부모인 경우는 허용한다. 다형성을 기억하자.
* JDK 1.7부터는 **생성자 측에 매개변수화된 타입을 생략할 수 있게** 되었다.
  * 이는 참조 변수에 매개변수화된 타입으로 미루어 생성자에 매개변수화될 타입을 추정할 수 있기 때문이다. 예를 들어, 다음과 같다.
```
// 아래의 생성자 측에 매개변수화될 타입은 참조 변수의 <String>과 일치할 것을 추정할 수 있으므로 생략될 수 있다.
Temp<String> temp = new Temp<>();
// Temp<String> temp = new Temp<String>(); // 위 문장과 동일하다. 
```
* 제네릭 클래스에서 **매개변수화된 타입과 다른 타입의 인스턴스는 추가할 수 없다**.
* 제네릭 클래스에서 **매개변수화된 타입의 자식 클래스 인스턴스는 추가할 수 있다**.
  * 다형성에서 부모 타입의 참조 변수로 자식 타입의 인스턴스를 가리킬 수 있음을 기억하자.
  * **제네릭 역시 매개변수화된 타입이 부모 클래스라면, 자식 타입의 인스턴스를 가리킬 수 있다**.
```
public class GenericTest {
    public static void main(String[] args) {
        // 참조 변수 측에 매개변수화된 타입인 Parent로 미루어 생성자 측에 매개변수화될 타입을 생략할 수 있다.
        List<Parent> list = new ArrayList<>(); 
        // 매개변수화된 타입인 Parent의 자식 클래스 인스턴스는 사용할 수 있다.
        list.add(new Child());

        Iterator<Parent> iterator = list.iterator();
        while(iterator.hasNext())
            iterator.next().sayHello(); // 자식 클래스의 메소드가 호출된다.
//            iterator.next().sayBye(); // 부모 클래스를 매개변수화한 제네릭 클래스이므로, 자식 클래스의 메소드는 사용할 수 없다.
            
    }
}

class Parent {
    void sayHello() {
        System.out.println("Helloooo?");
    }
}

class Child extends Parent {
    void sayHello() {
        System.out.println("No!");
    }
    void sayBye() {
        System.out.println("Ok, bye.");
    }
}
```

### 제네릭 클래스의 타입 변수 제한
* 제네릭을 활용하면 클래스가 다룰 수 있는 타입을 제한할 수 있음을 확인했다.
  * 예를 들어, <T>를 명시한 제네릭 클래스는 하나의 타입만을 다룰 수 있을 것이다.
* 그러나 여전히 제네릭 클래스는 모든 클래스를 받아들일 수 있는 아쉬움이 있다.
```
public class GenericTest {
    public static void main(String[] args) {
        Temp<String> str = new Temp<>("hello");
        System.out.println(str.temp.getClass()); // class java.lang.String

        Temp<Integer> integer = new Temp<>(123);
        System.out.println(integer.temp.getClass()); // class java.lang.Integer

        Temp<Object> obj = new Temp<>(new Object());
        System.out.println(obj.temp.getClass()); // class java.lang.Object
    }
}
class Temp<T> {
    T temp;

    Temp(T temp) {
        this.temp = temp;
    }
}
```
* 제네릭 타입 자체에 extends 키워드를 활용하면 제네릭 클래스가 **특정한 타입의 자식 클래스들만 사용할 수 있도록 제한**할 수 있다.
  * 예를 들어, <T extends [부모 클래스 이름]>과 같은 형식이다.
  * 이 경우 제네릭 클래스는 여전히 하나의 타입만 다룰 수 있지만, 특정한 클래스의 자식 클래스 인스턴스만 사용할 수 있도록 한 번 더 제한하게 된다.
```
public class GenericTest {
    public static void main(String[] args) {
        GenericTester<Parent> test1 = new GenericTester<>(); // Parent 자체도 허용한다.
        GenericTester<ChildA> test2 = new GenericTester<>();
        GenericTester<ChildB> test3 = new GenericTester<>();
        GenericTester<ChildC> test4 = new GenericTester<>();
//        GenericTester<NotChild> test5 = new GenericTester<>(); // NotChild 클래스는 Parent의 자식 클래스가 아니므로, 사용할 수 없다.
    }
}

class GenericTester<T extends Parent> { } 
class Parent { }
class ChildA extends Parent { }
class ChildB extends Parent { }
class ChildC extends Parent { }
class NotChild { }
```
* 위 예제의 경우, <>를 생략한 GenericTester test6 = new GenericTester();를 만들면 타입 변수는 Parent가 되는 듯 하다.
* 클래스가 아닌 특정한 인터페이스로 제한하고 싶은 경우에도 extends 키워드를 사용한다.
  * **implements 키워드가 아님에 주의**하자.
```
// List는 인터페이스지만 extends 키워드를 사용한다.
class GenericTester<T extends List> { }
```
* 특정한 클래스의 자식이면서 특정한 인터페이스도 구현하는 클래스로만 제한하고 싶다면 & 기호를 활용한다.
```
public class GenericTest {
    public static void main(String[] args) {
//        GenericTester<Removable> test1 = new GenericTester<>();
//        GenericTester<Parent> test2 = new GenericTester<>();
//        GenericTester<NotChild> test3 = new GenericTester<>();
//        GenericTester<ChildA> test4 = new GenericTester<>();
//        GenericTester<ChildB> test5 = new GenericTester<>();
        GenericTester<ChildC> test6 = new GenericTester<>(); // ChildC만이 조건에 부합한다.
    }
}

class GenericTester<T extends Parent & Removable> {}

interface Removable {}
class Parent {}
class NotChild {}
class ChildA extends Parent {}
class ChildB implements Removable {}
class ChildC extends Parent implements Removable {}
```

### 제네릭과 와일드카드
* **제네릭 타입이 다른 것만으로는 오버로딩이 성립하지 않는다**.
  * 제네릭은 원칙적으로 컴파일러가 컴파일 시에만 사용하고 제거된다.
```
class WildcardTester {
    static void hello(List<Integer> list) {
        System.out.println(list);
    }
//    static void hello(List<String> list) { // 매개변수화될 타입이 다르다고 해서 오버로딩이 되지는 않는다.
//        System.out.println(list);
//    }
}
```
* 와일드카드는 이러한 상황에 사용할 수 있다. 기호 물음표(?)로 명시하며, 어떠한 타입도 될 수 있음을 의미한다.
  * 와일드카드를 적용하여 위 코드를 수정하면 다음과 같다.
```
public class GenericTest {
    public static void main(String[] args) {
        WildcardTester.hello(new ArrayList<String>());
        WildcardTester.hello(new ArrayList<Integer>());
        WildcardTester.hello(new ArrayList<Object>());
    }
}

class WildcardTester {
    static void hello(List<?> list) { // 와일드카드의 위치에 어떠한 클래스도 적용이 가능하다.
        System.out.println(list);
    }
}
```
* 그러나 위 예시는 Object 타입을 허용한 것과 다를 바가 없으므로, 와일드카드는 크게 다음의 방식을 통해 접근 가능한 클래스를 제한한다.
  1. <? extends T>: T와 그 자식 클래스들만 가능하다. (상한 제한)
  2. <? super T>: T와 그 조상 클래스들만 가능하다. (하한 제한)
  3. <?>: 제한이 없으므로, 모든 클래스가 가능하다.
    * <? extends Object>와 같은 의미를 갖는다.
  * 와일드카드는 & 기호를 사용할 수 없다.
* 주로 제네릭 클래스가 메소드의 매개 변수로 사용될 때 오버로딩을 위해 사용하는 듯 하다.

### 제네릭 메소드
* 메소드의 선언부에 제네릭 타입이 선언된 메소드이다.
  * 이 때, 타입 매개변수의 선언 위치는 반환 타입의 앞이다.
```
class Temp {
    public <T> void hello() { }
}
```
* **제네릭 클래스에 정의된 타입 매개 변수와 제네릭 메소드에 정의된 타입 매개 변수는 별개의 것**이다.
  * 같은 <T>를 작성했더라도 다른 타입 매개 변수이다.
  * 제네릭 메소드의 타입 매개 변수는 마치 지역 변수와 같은 스코프를 갖는다고 생각하면 이해하기 쉽다.
* static 멤버에는 타입 매개 변수를 사용할 수 없지만, static 메소드는 제네릭 메소드일 수 있다.
```
public class GenericTest {
    public static void main(String[] args) {
        // <T>는 String이다.
        Temp<String> tempString = new Temp<>("hello");

        // 제네릭 메소드들은 제네릭 클래스에서 String으로 매개변수화된 <T>와 관계가 없다.
        tempString.<Integer>hello(1); // 원래는 이런 형식이지만, 컴파일러가 이를 추정할 수 있으므로 생략이 가능하다.
        tempString.hello(0.01f);
        tempString.hello(true);

        // static 멤버는 타입 매개 변수를 사용할 수 없지만, static 메소드는 제네릭 메소드일 수 있다.
        Temp.bye(1);
        Temp.<Float>bye(0.01f); // 원래는 이런 형식이지만, 컴파일러가 이를 추정할 수 있으므로 생략이 가능하다.
        Temp.bye(true);
    }
}

class Temp <T>{
    T temp;
    Temp(T temp) {
        this.temp = temp;
    }

    // 이 <T>는 Temp <T>의 <T>와 관계가 없다. 별개의 값이다.
    public <T> void hello(T temp) {
        System.out.println(temp);
    }

    // 이 <T>는 Temp <T>의 <T>와 관계가 없다. 별개의 값이다.
    public static <T> void bye(T temp) {
        System.out.println(temp);
    }
}
```
* **제네릭 메소드는 제네릭 클래스가 아니어도 정의할 수 있다**.
