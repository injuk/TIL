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

### TS 타이핑 팁
* 처음부터 완벽한 타입을 정의하려는 것은 다음과 같은 문제가 있다.
  1. 모든 것을 예상하고 진행하려 하게 되므로, 개발 초기에 시간이 너무 많이 든다.
  2. **단순히 어렵다**.
* 때문에 **초기에는 우선 필요한 만큼의 타입만 지정해두고, 추후에 수정 또는 고도화할 필요가 있는 경우에 그에 맞추어 타입을 추가하는 것이 바람직**하다.

### 불필요한 형변환 연산자 as 제거하기
* 예를 들어 다음과 같이 사용자 정의 예외 인터페이스를 정의하여 사용한다고 가정하자.
  * 이 경우, catch 블록 내에서 e는 항상 unknown 타입이므로 IError 인터페이스의 속성에 접근하고자 하는 경우 항상 `(e as IError)`를 명시해야 한다.
```
interface IError {
    name: string;
    message: string;
    stack?: string;
    response?: {
        data: any;
    };
}

(async () => {
    try {
        // do something!
    } catch (e: unknown) {
        console.error((e as IError).response?.data);
        (e as IError).response?.data;
        // 형변환하지 않는 경우, 이렇게 사용할 수는 없다.
        // e.response?.data;
    }
})();
```
* 이 경우 **코드 전체가 `(e as IError)`로 도배될 가능성이 있으므로, 이를 미연에 방지하기 위해 적절한 변수를 추가하여 리팩토링**을 적용할 수 있다.
  * 이는 `const myError = e as IError;` 와 같이 작성하며, **이러한 방식을 적용하지 않는 경우의 형변환은 언제나 일회성으로 그칠 위험**이 있다.
```
(async () => {
    try {
        // do something!
    } catch (e: unknown) {
        const myError = e as IError;
        console.error(myError.response?.data);
        const data = myError.response?.data;
        console.log(data);
    }
})();
```

### as 키워드는 지양하기
* **TS에서 any 타입을 지양하는 만큼 `as` 키워드를 활용한 강제적인 형변환 역시 지양**해야 한다. 
* 반면, **타입이 unknown으로 추론되는 경우 이를 바로잡기 위한 `as` 키워드의 사용은 바람직한 사용 예시에 해당**한다.
  * 사실상, `as` 키워드는 unknown 타입에 대해 어쩔 수 없이 사용해야 한다고 외워도 무방하다.
  * 이는 `any`를 사용해도 되는 상황이 오직 메소드를 오버로딩하는 경우에만 한정되는 것과 일맥상통한다.
* any가 TS를 사용하는 이유를 무색하게 하므로 지양해야 한다면, `as` 키워드는 형변환 과정에서 휴먼 에러가 발생할 가능성이 높기 때문에 지양해야 한다.
  * 이러한 측면에서, **상술한 예시 역시 바람직한 코드로 볼 수 없으며 대신 클래스와 `instanceof` 타입 가드를 활용하도록 수정할 수 있다**.
  * 이렇듯 **실무에서는 되도록 `as` 키워드를 활용한 형변환 대신 Error 클래스를 상속하는 클래스와 타입 가드를 적용하는 것이 바람직**하다.