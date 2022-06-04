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