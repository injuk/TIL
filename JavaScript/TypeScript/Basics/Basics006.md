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