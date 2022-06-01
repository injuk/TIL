# Basics
## 2022-06-01 Wed

### JS 프로토타입
* 배열, 객체, 숫자, 문자열 등에 제공되는 API는 각 타입 별 프로토타입으로부터 상속된다.
  * 크롬의 경우, 프로토타입은 __proto__으로 확인할 수 있다.
* **JS의 경우, 프로토타입이 있는 타입은 생성시 프로토타입의 멤버를 모두 자동적으로 상속**받는다.
* 특히, 배열이나 객체 등을 생성했을 때 자동으로 상속받는 API들은 다음과 같이 부를 수 있다.
  1. Built in JS API
  2. 또는 JS Native API
* **프로토타입은 단순히 객체의 정보를 확장하는 것 뿐만 아니라, 새로운 타입을 생성했을 때 프로토타입에 정의된 정보를 쉽게 활용할 수 있게 한다**.

### 프로토타입과 클래스
* ES6 이전의 문법 상에서도 프로토타입을 활용하여 상속을 구현할 수 있었다.
* 그러나 객체지향에 익숙한 개발자들이 더 쉽게 JS에 적응할 수 있도록, ES6에서는 문법 설탕에 해당하는 클래스 기반의 문법이 도입되었다.
  * 실제로 **JS는 내부적으로 여전히 프로토타입을 기반으로 상속하는 언어이며, 클래스 문법이 도입되었다고 해서 내부적인 동작이 변경되지는 않았다**.
  * 때문에 **바벨 등과 같은 도구로 트랜스파일하는 경우, ES6의 클래스 문법은 함수 기반으로 변환**된다.
* 상술한 이유에서 아래의 두 구현 방식은 실제로는 동일하게 동작한다.
```
function Bird(name, age) {
    this.name = name;
    this.age = age;
}

class Cat {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
}
```

### TS에서의 클래스
* TS에서의 클래스는 멤버 변수에 대한 정보를 모두 클래스 내부에 명시해야 하며, 생성자에 전달되는 인자에도 타입을 정의할 수 있다.
* 또한 멤버 변수의 접근 제어자를 정의할 수도 있다.
  * 이 때, TS의 클래스에 명시할 수 있는 접근 제어자는 private, protected, public 세 종류이다.
* **값을 할당한 후 변경할 수 없도록 하고 싶은 멤버에는 readonly 키워드를 명시**할 수 있다.
```
export class Person {
    private name: string;
    public age: number;
    readonly alias: string;

    constructor(name:string, age: number, alias: string) {
        this.name = name;
        this.age = age;
        this.alias = alias;
    }
}
```