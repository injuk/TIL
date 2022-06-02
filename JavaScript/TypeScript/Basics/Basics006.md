# Basics
## 2022-06-02 Thu

### 제네릭이란?
```
> 제네릭은 함수의 타입을 우선은 비워두고, 호출한 시점에 타입을 정의하는 개념이다.
```
* 제네릭은 Java 등 강타입 언어에서 재사용성이 높은 모듈을 만들기 위해 자주 사용되는 특징이다.
  * 특히, **하나의 타입만을 명시하지 않고 여러 타입에 대해 동작할 수 있는 모듈을 작성할 때 유용**하다.
  * 제네릭을 활용한 함수는 함수에서 사용되는 타입을 마치 인자처럼 받아들여 유연하게 동작할 수 있게 된다.

### 제네릭의 문법
* 일반적으로 TS의 함수는 인자와 반환형에 타입을 명시하며, 그렇지 않은 경우 any 타입으로 적용된다.
```
function logAny(param: any) {
    console.log(param);
    return param;
}

function logString(param: string): string {
    console.log(param);
    return param;
}

const maybeNumber = logAny(1);
const maybeString = logAny('1');
const maybeBoolean = logAny(true);

// logString(1); // 인자에 string을 정의하였으므로 number 타입을 매개변수로 사용할 수 없다.
const definitelyString = logString('1');
```
* 상술한 두 함수는 사용에 문제는 없지만, 장기적인 관점에서 다음과 같은 문제를 수반한다.
  1. any 타입의 함수는 TS 컴파일러의 이점을 누릴 수 없다.
  2. **string 타입의 함수는 유연하지 못하며, 추후 다른 타입을 지원해야할 경우 새로운 함수를 추가하거나 기존 함수를 모두 수정**해야 한다.
* 다음과 같이 제네릭을 도입한 함수를 정의할 경우, 상술한 문제점에서 자유로워질 수 있게 된다.
  * **제네릭이 도입된 함수는 함수 내부적으로 사용될 타입을 호출 시 전달할 매개 변수의 타입을 기반으로 호출 시점에 결정**한다. 
```
function genericLog<T>(param: T): T {
    console.log(param);
    return param;
}
const definitelyString = genericLog<string>('hello');
console.log(`definitelyString: ${definitelyString}`);

const definitelyNumber = genericLog(1); // 제네릭의 타입은 추론되므로 생략해도 무방하다.
console.log(`definitelyNumber: ${definitelyNumber}`);

const definitelyBoolean = genericLog<boolean>(true);
console.log(`definitelyBoolean: ${definitelyBoolean}`);

type Cat = {
    name: string; age: number;
}
const cat: Cat = { name: 'meow', age: 3 };
const definitelyCat = genericLog<Cat>(cat);
console.log(`definitelyCat: ${JSON.stringify(definitelyCat)}`);
```
* 제네릭 함수의 문법은 다음과 같은 의미를 갖는다.
```
// 함수에 정의된 타입 T를 각각 인자와 반환형의 타입에도 적용하겠다.
function genericLog<T>(param: T): T {
    console.log(param);
    return param;
}
// 여기서 호출하는 generigLog는 <>에 전달한 string 타입이므로, 인자와 반환형 역시 string임의 명시한다.
const definitelyString = genericLog<string>('hello');
```

### 기존 방식과 제네릭의 차이점
* 제네릭을 적용하지 않는 경우, TS에서의 함수는 인자와 반환형을 any로 정의하거나 타입 별로 다른 함수를 정의해야 한다.
  * any로 정의한 함수는 TS의 이점을 누릴 수 없는 방식이다.
  * **타입 별로 다른 함수를 여럿 정의하는 것은 코드의 중복과 관리 지점을 늘리며, 수정 사항의 반영을 어렵게 하는 방식**이다.
* 반면, 제네릭 없이 이러한 문제를 해결하기 위해서는 유니온 타입의 도입을 고려해볼 수도 있다. 
```
function unionLog(param: string | number | boolean): string | number | boolean {
    console.log(param);
    return param;
}
const maybeString = unionLog('1');
const maybeNumber = unionLog(1);
const maybeBoolean = unionLog(true);
```
* 그러나 유니온 방식을 적용한 함수 내부에서는 유니온에 정의된 타입의 공통 API만을 활용할 수 있게되는 단점이 존재한다.
  * 또한, **상술한 코드에서의 함수 반환값들 역시 유니온 타입으로 추론**된다.
* 때문에 유니온 타입을 적용하는 방식은 아래와 같은 코드에서 주석을 해제할 경우 컴파일이 불가능해지는 단점이 존재한다.
```
function unionLog(param: string | number | boolean): string | number | boolean {
    console.log(param);
    return param;
}
const maybeString = unionLog('1');
// maybeString.trim(); // string이 반환되었지만, 유니온 타입이므로 string 타입의 API를 활용할 수 없다.
```
* 즉, **유니온 타입 방식은 함수 인자의 유연성만을 해결할 수 있으며 반환형에 대한 지원이 불가능**하다.

### 제네릭 도입의 이점
* 함수에 제네릭을 도입하는 경우, 상술한 모든 문제점이 다음과 같이 해결된다.
  1. 함수의 타입과 반환형에 타입을 지정할 수 있는 TS의 이점을 누릴 수 있다.
  2. 타입 별 함수를 중복 정의할 필요가 없으므로 유지보수성 역시 높아진다.
  3. 함수에 전달되는 인자는 물론 반환되는 값에 대한 추론도 적절히 처리되므로, 타입에 맞는 API를 활용할 수 있다.

### 인터페이스에 제네릭을 적용하기
* 객체의 인터페이스 역시 동일한 구조를 갖지만, 몇몇 속성의 타입만이 다른 경우가 발생할 수 있다.
  * 인터페이스를 여럿 만들어서 유니온 타입으로 묶어줄 수도 있지만, 이러한 방식은 상술한 이유에서 보았듯이 문제의 소지가 많다.
* 제네릭을 도입하는 경우, 매번 타입 정의를 인터페이스로 정의하는 것을 반복할 필요 없이 어떠한 타입이든 제네릭 인터페이스를 통해 유연하게 대처할 수 있다.
* 인터페이스 제네릭은 다음과 같이 정의할 수 있으며, 이러한 방식은 다음과 같은 의미를 갖는다.
  * **제네릭 인터페이스는 타입의 선언 시점에 타입 변수 T를 추가적으로 전달하고, 이에 따라 인터페이스의 타입을 유연하게 바꿀 수 있게 된다**.
```
interface Temp<T> {
    value: T; 
    selected: boolean;
}

// 인터페이스에 타입 변수 T 역할을 수행할 임의의 타입을 반드시 명시해야 한다.
const temp: Temp<string> = {
    value: 'hello',
    selected: false,
}
```

### 제네릭과 타입 제한하기
* 제네릭을 사용하고 있는 경우, 제네릭을 더 엄격하게 사용하거나 더 많은 옵션을 주고싶을 수 있다.
  * 타입 제한은 이러한 경우에 가장 쉽고도 유용하게 사용할 수 있는 이론이다.
* 또한 제네릭은 타입에 대한 힌트를 줌으로써 추론에 사용될 힌트를 더 많이 제공할 수 있다.
  * 또는, **타입 추론에 사용될 힌트를 더 주기 위해 제네릭을 활용할 수도 있다는 뜻으로 이해해도 무방**하다.
  * **아래의 코드를 예시로 들어, 제네릭 타입이 배열임을 함수 내부에서 알 수 있으므로 배열의 속성과 API를 활용해도 컴파일이 가능**하게 된다.
  * 이는 즉 제네릭의 타입 힌트가 컴파일러의 타입 추론에 도움을 주고 있는 셈이 된다.
```
function log<T>(text: T[]): T[] { // 2. 이러한 문제를 해결하기 위해, 타입 변수 T의 배열 타입을 사용함으로써 인자가 배열이라는 힌트를 줄 수 있다.
    console.log(text.length); // 1. 타입 변수를 T로만 사용하는 경우, T는 무슨 타입이든지 될 수 있으므로 length를 사용할 수 없다.
    return text;
}
```

## 2022-06-03 Fri
### 정의된 타입으로 타입을 제한하기
* 상술한 방식은 함수 내부에서 length 속성을 사용하기 위해 제네릭을 배열로 처리하였다.
* 그러나 실제로는 별도의 인터페이스를 정의하고, 이를 활용하는 다음과 같은 방식으로 문제를 해결할 수도 있다.
```
interface LengthType {
    length: number;
}
function logLength<T extends LengthType>(text: T): T {
    console.log(text.length);
    return text;
}

logLength('hello');
logLength([]);
// logLength(123); // number 타입은 length 속성을 갖지 않으므로, 인자로 전달할 수 없다.
```
* <T extends LengthType>의 경우, **logLength 함수에서 사용하는 타입의 범위를 제한하는 효과**를 갖는다.
  * 이를 통해 **T는 LengthType을 확장하는 하위 타입이 되므로, 언제나 length 속성을 갖는 타입으로 한정**된다.
  * 이렇듯 새로이 정의된 타입을 통해 제네릭에 포함될 수 있는 속성, 또는 타입들을 구분할 수 있게 된다.
* **해당 방식은 기본적으로 모든 타입을 전달받을 수 있는 타입 변수가 반드시 특정한 속성을 갖도록 제한하는 역할을 수행**한다.
* 이렇듯 **extends 키워드는 클래스와 인터페이스를 다룰 때, 이미 정의되어 있는 상위 타입을 확장하기 위해 사용**한다.

### keyof를 활용한 타입 제한
* keyof를 활용하는 타입 제한 방식은 <T extends keyof 인터페이스명>의 형식을 띈다.
  * 이는 **명시된 인터페이스의 키들 중에 하나가 타입 변수 T가 된다는 의미**를 갖는다.
  * 결과, 해당 제네릭을 활용하는 함수의 인자로는 명시된 인터페이스의 속성 중 단 하나만 포함될 수 있다.
  * 마치 extends 키워드를 활용하여 제네릭에 타입을 제한한 것처럼, 해당 방식은 타입의 범위를 더더욱 좁혀준다.
```
interface ShoppingItem {
    name: string;
    price: number;
    stock: number;
}

function getShoppingItemOption<T extends keyof ShoppingItem>(item: T): T {

    return item;
}
getShoppingItemOption('name'); // 정말 인터페이스의 '키 값'들 중 하나만 들어갈 수 있다.
getShoppingItemOption('price');
getShoppingItemOption('stock');
```
* 이렇듯 **keyof 예약어는 인터페이스 등 이미 정의되어 있는 타입의 키 값들만 포함할 수 있도록 타입을 제한하는 역할을 수행**한다.
* **extends와 keyof 등의 예약어는 타입 변수에 전달될 수 있는 타입을 제한하는 것으로 더 안전하게 제네릭을 사용할 수 있도록 지원**한다.