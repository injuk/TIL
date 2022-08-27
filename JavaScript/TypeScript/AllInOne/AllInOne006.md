# AllInOne
## 2022-08-28 Sun
### 함수의 공변성
* **함수의 공변성이란, 두 함수 타입 간에 서로 대입이 가능한지의 여부를 의미**한다.
  * TS에 적용되는 예시로서, **두 함수 중 반환형의 타입이 더 넓은 함수에 더 좁은 타입의 반환형을 갖는 함수를 대입**할 수 있다.
  * 예를 들어, 다음의 함수 func1은 함수 타입을 의미하는 Func2 형 변수에 대입이 가능하다.
```
function func1(param: number): string {
    return param.toString();
}
type Func2 = (param: number) => string | number;

// 더 넓은 타입의 반환형을 갖는 함수에 더 좁은 타입의 반환형을 갖는 함수를 대입할 수 있다.
const func2: Func2 = func1;
console.log(func2(123)); // 123
```
* **상술한 코드에서, func1과 Func2 타입의 형태는 분명히 다르지만 TS 상 Func2는 func1보다 큰 타입으로 추론**된다.
  * TS는 큰 타입에 작은 타입 변수를 대입할 수 있으므로, 상술한 코드 역시 정상 동작한다.
* **반면, 매개 변수의 경우 타입의 크기를 판단하지 않고 이러한 규칙이 거꾸로 적용**된다.
  * 즉, 아래와 같이 **매개 변수의 타입을 더 좁게 갖는 함수에 더 넓은 타입의 매개 변수를 갖는 함수를 대입**할 수 있다.
```
type Func3 = (param: number) => string;
function func4(param: number | string): string {
    return typeof param === 'string' ? param : param.toString();
}

// 더 좁은 타입의 매개 변수를 갖는 함수에 더 넓은 타입의 매개 변수를 갖는 함수를 대입할 수 있다.
const func3: Func3 = func4;
console.log(func3(123)); // 123
```
* 이렇듯 TS 상에서의 **함수 간에는 반환형이 더 넓은 타입으로 대입되고, 매개 변수는 좁은 타입으로 대입**된다.

### 타입 넓히기와 타입 좁히기
* 타입 넓히기란, TS 상의 타입 추론과 관련된 개념으로 변수의 타입을 더 넓게 추론하는 것을 의미한다.
  * 예를 들어, 아래의 코드는 상수인 경우와 변수인 경우에 타입이 다르게 추론된다.
  * 이는 **변경이 불가능한 상수와는 달리 `let` 변수는 변경 가능하므로, 변경 가능성을 고려하여 의도적으로 더 넓은 타입으로 추론하는 것에 해당**한다.
```
const constVar = 42; // 42 라는 타입으로 추론된다.
let letVar = 42; // 42 타입보다 넓은 타입인 number로 추론된다.
```
* 반면, **타입 좁히기는 TS 상에서 지원되는 여러 종류의 타입 가드를 통해 타입을 좁혀나가는 과정**을 말한다.
```
function narrowing(param: number | string): string {
  if(typeof param === 'number') {
    // 해당 블록에서 param은 number로 좁혀진 타입을 갖는다.
    return param.toString();
  } else {
    // 해당 블록에서 param은 string으로 좁혀진 타입을 갖는다.
    return param;
  }
```

### 함수 오버로딩
* JS는 그 특유의 유연함(?)으로 인해, 함수 선언 후에는 자유로운 오버로딩이 가능하다.
  * 예를 들어, 매개 변수의 타입과 개수를 아주 자유롭게 전달할 수 있다.
```
function doNothing(param) {
  return param;
}
// 위 함수는 아래와 같은 형태로 모두 호출이 가능하다.
doNothing(1);
doNothing(1, 3);
doNothing(1, 3, true);
doNothing();
```
* **TS의 경우, 인터페이스 또는 타입을 활용하여 동명의 함수가 다른 형태를 갖도록 여러 함수 타입을 정의하는 것으로 오버로딩**할 수 있다.
  * 때문에 **가장 이상적인 것은 하나의 방식으로 함수를 타이핑하는 것이지만, 부득이한 경우에는 여러 개의 함수 타입을 정의해도 무방**하다.
  * 이 때, **함수의 오버로딩에는 우선 TS에게 함수의 시그니쳐만 전달하고 함수 본문은 따로 작성함을 알리는 `decalre` 키워드를 활용**할 수 있다.
```
declare function plus(): string;
declare function plus(num1: number): number;
declare function plus(num1: number, num2: number): number;
declare function plus(num1: number, num2: number, num3: number): number;

// declare를 통해 선언된 함수에 접근하는 경우, 함수의 구현부는 다른 위치에 작성해도 무방하다.
const maybeString = plus();
let maybeNumber = plus(1);
maybeNumber = plus(1, 2);
maybeNumber = plus(1, 2, 3);
```
* 이러한 **함수 오버로딩은 `declare` 키워드 뿐만 아니라 인터페이스와 클래스 내부에도 다음과 같이 적용이 가능**하다. 
  * 이 중 **인터페이스를 활용하는 함수 오버로딩 시 함수의 이름을 작성하지 않으며, 매개 변수의 이름은 통일하지 않아도 무방**하다.
  * 또한, **오버로딩된 함수를 사용하는 경우 함수 본문 구현 시에는 `any` 타입을 사용하더라도 무방**하다.
```
// 인터페이스를 활용하는 함수 오버로딩 시 함수의 이름을 작성하지 않으며, 매개 변수의 이름은 통일하지 않아도 무방하다. 
interface IOverload {
    (num1: number, num2: number): void;
    (str1: string, str2: string): void;
}
// 구현 중인 함수 본문의 매개 변수에는 타입을 명시하지 않더라도 무방하며, 이 경우 매개 변수는 각각 number | string으로 추론된다.
const funcImpl: IOverload = (x, y) => console.log(x, y);
funcImpl(1, 2); // 1 2

class Overloaded {
    plus(num1: number, num2: number): number;
    plus(str1: string, str2: string): string;
    // 클래스 내에서 함수가 명확한 타입과 함께 오버로딩 되었으므로, 함수 본문 구현 시 any 타입을 사용하더라도 무방하다.
    plus(a: any, b: any) {
        return a + b;
    }
}

console.log(new Overloaded().plus(1, 2)); // 3
```