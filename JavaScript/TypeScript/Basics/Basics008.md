# Basics
## 2022-06-04 Sat

### 타입 호환이란?
* **타입 호환이란, TS 코드 상에서 특정한 타입이 다른 타입과 호환되는지를 의미하는 용어**이다.
  * 예를 들어 TS의 구조적 타이핑의 경우, 코드 구조 관점에서 타입이 서로 호환되는지를 확인한다.
```
interface A {
    name: string;
    alias: string;
}

interface B {
    name: string;
}

let objectA: A = { name: 'a', alias: 'A' };
let objectB: B = { name: 'b' };
// objectA = objectB; // 컴파일이 불가능하다. 
objectB = objectA;
```
* 상술한 코드에서, 인터페이스 B의 모든 속성은 인터페이스 A에 포함된다.
* TS 관점에서, **인터페이스 A 타입의 변수를 인터페이스 B 타입의 변수에 할당할 수는 없지만 그 역은 가능**하다.
* 이렇듯 **타입 호환은 구조적으로 더 적은 속성을 갖는 타입의 작은 변수에 더 많은 속성을 갖는 큰 타입의 값을 할당할 수 있는 TS의 특징**을 말한다.
* 타입 호환은 인터페이스와 클래스, 타입 사이에서도 성립될 수 있다.
  * 아래의 코드를 예로 들어, **더 적은 속성을 갖는 클래스 C 타입의 변수에 더 많은 속성을 갖는 큰 타입의 값을 할당**할 수 있다.
```
class C {
    name: string;
}
type D = {
    name: string;
}

let objectC: C = new C();
objectC = objectA; // objectC가 더 작은 구조이므로, 더 큰 구조의 값을 할당할 수 있다.
let objectD: D = { name: 'd' };
objectD = objectA;
```
* 이렇듯 **TS에서의 타입 호환성의 판단은 type, interface, class와 같은 구체적인 형태가 아닌, 내부적인 속성과 타입의 정의를 기준**으로 한다.

### 함수와 제네릭에서의 타입 호환
* 함수의 경우에도 인자의 구조를 기준으로 타입 호환성을 검사할 수 있다.
  * 아래의 코드에서, addExpr보다 sumExpr의 구조가 더 큰 것으로 판단된다.
  * 이 경우에는 **더 적은 인자를 갖는 함수를 할당한 함수 표현식에 더 많은 인자를 갖는 함수 표현식의 값을 할당할 수 없다**.
```
let addExpr = function(num: number): void {
}
let sumExpr = function(num1: number, num2: number): void {
}
// addExpr = sumExpr; // addExpr가 더 작은 구조이므로, 더 큰 구조의 함수 값을 할당할 수 없다.
sumExpr = addExpr;
```
* **제네릭의 경우, 아래와 같은 두 변수는 다른 타입 변수를 전달하지만 내부적으로는 아무 속성을 포함하지 않기에 상호 호환이 가능**하다.
```
interface Nothing<T> {
}
let nothingString: Nothing<string>;
let nothingNumber: Nothing<number>;
// nothingNumber = nothingString; // 둘 중 어느 주석을 해제해도 컴파일이 가능하다.
// nothingString = nothingNumber; // 둘 중 어느 주석을 해제해도 컴파일이 가능하다.
```
* 반면 **제네릭에 임의의 제네릭 속성이 포함되어 있다면 이러한 타입 호환이 불가능**하다.
  * 아래의 예시에서, **타입 변수에 전달된 타입에 의해 somethingString과 somethingNumber 사이에 구조적인 차이점이 발생하므로 호환이 불가능**하다.
```
interface Something<T> {
    data: T;
}
let somethingString: Something<string>;
let somethingNumber: Something<number>;
// somethingString = somethingNumber; // 둘 중 어느 주석을 해제해도 컴파일이 불가능하다.
// somethingNumber = somethingString; // 둘 중 어느 주석을 해제해도 컴파일이 불가능하다.
```
## 2022-06-05 Sun
### TS의 모듈 시스템
* TS의 모듈 개념은 ES6 이후의 모듈 개념과 유사하다.
  * 즉, **TS는 JS 진영의 ES 모듈과 완전히 동일한 개념을 사용**한다!
* **각각의 모듈은 전역 변수와는 구분되는 자체적인 유효 범위를 갖고, export나 import 키워드를 활용**한다.

### JS의 모듈 시스템
* JS의 경우, <script>를 통해 다른 파일로부터 JS 파일을 로드했다고 하더라도 유효 범위는 전역으로 적용된다.
  * 즉, 파일마다 고유한 스코프를 갖는 것이 아니다.
  * 이러한 JS의 한계 때문에 웹 개발자들은 네임스페이스 모듈화 패턴 등을 활용하여 개발을 진행해왔다.
* 이러한 한계를 극복하기 위해 ES6 이후의 최신 JS 문법은 ES 모듈이라는 모듈 시스템을 갖게 되었으며, 이에 import와 export 키워드를 활용한다.
* 일반적으로 **타입은 별도의 파일에 정의하며, 해당 파일이 여러 객체를 export하는 경우에는 파일의 최 하단에 모아서 작성하는 관례**가 있다.
```
interface PhoneNumberDictionary {
  [phone: string]: {
    num: number;
  };
}

interface Contact {
  name: string;
  address: string;
  phones: PhoneNumberDictionary;
}

type Contacts = Contact[];

enum PhoneType {
  OFFICE = 'office',
  HOME = 'home',
  STUDIO = 'studio',
}

export { Contact, Contacts, PhoneType, PhoneNumberDictionary }; // 관례상 파일의 최하단에 모아 작성한다.
```