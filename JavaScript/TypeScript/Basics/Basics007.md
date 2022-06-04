# Basics
## 2022-06-03 Fri

### Promise와 제네릭
* **순차대로 실행되는 동기적인 코드에 대해서는 TS 자체적인 타입 추론이 가능하며, 개발자는 이러한 기능을 지원**받을 수 있다.
* 반면, **Promise 등을 반환하는 비동기 함수의 경우 일반적으로 Promise<unknown> 타입을 반환하는 것으로 추론**된다. 
  * 이는 프로미스 자체가 내부적으로 제네릭을 활용하여 정의되기 때문이다.
  * 때문에 비동기 처리를 통해 돌려받을 반환형을 반드시 명시해야 프로미스를 제대로 사용할 수 있다.
* **프로미스의 제네릭 타입 변수에 전달할 타입은 프로미스가 resolve를 통해 결정되어 반환하는 값의 타입을 명시**한다.

### TS와 클래스 생성자
* **클래스의 생성자는 기본적으로 반환형을 명시하지 않는다**.

## 2022-06-04 Sat
### 타입 추론
* TS는 많은 경우에 명시되지 않은 타입을 추론하도록 동작한다.
* TS는 `let num = 3;`과 같은 코드에서 타입을 명시하지 않더라도 number로 추론할 수 있다.
  * 이렇듯 **변수를 선언하거나 초기화하는 시점에 타입이 추론되며, 이러한 방식은 가장 자주 사용되는 타입 추론 방식**이다.
  * 이외에도 **변수나 속성, 인자의 기본 값, 함수의 반환값 등을 설정할 때 타입 추론이 발생**한다.
* 이렇듯 **코드를 작성했을 때, 타입 스크립트가 코드의 타입이 무엇인지 판단해나가는 과정이 타입 추론**이다.
* 함수의 인자에 기본 값을 다음과 같이 적용한 경우, 함수의 인자와 반환값 각각의 타입 역시 추론된다.
  * 이렇게 **추론된 타입은 명시적인 타입 정의와 유사하게 동작**한다.
```
function getSomething(something = 3) { // 기본 값에 의해 number를 받아 number를 반환하는 것으로 추론된다.
    // return 'ingnoh' + something; // 해당 라인을 주석 해제하여 적용하는 경우, 반환형은 string으로 추론된다.
    return something;
}
// getSomething('a'); // 함수에 명시적인 타입 정의는 없었지만, number로 추론되었으므로 다른 타입을 전달할 수 없다.
```

### 인터페이스와 제네릭을 활용하는 타입 추론
* 아래와 같이 제네릭을 정의한 인터페이스의 경우, 인터페이스를 적용한 변수에 굳이 속성 값을 채워넣지 않더라도 TS는 각 타입을 추론할 수 있다.
```
interface Hello<T> {
    value: T;
    title: string;
}

let hello: Hello<boolean> = {
    value: , // 이러한 미완성 코드는 당연히 컴파일되지 않지만,
    title: , // TS는 각 타입을 상술한 Hello 인터페이스에 의해 추론할 수 있다.
}
```
* **인터페이스에 제네릭을 정의한 경우, 제네릭의 값을 TS 내부적으로 추론하여 변수의 타입까지 추론을 통해 보장**받을 수 있다.
  * 해당 방식 역시 변수의 값을 정의할 때, 값을 통해 타입을 추론해나가는 방식과 일맥상통한다.

### 복잡한 구조에서의 추론
* 제네릭이 적용된 인터페이스를 확장하는 인터페이스의 경우에도 마찬가지로 같은 방식으로 타입 추론이 가능하다.
  * **아래의 코드에서, complicatedHello 변수의 타입에 전달된 타입 변수는 ComplicatedHello와 Hello 각각에도 전달**된다.
  * 이 경우 string 타입이므로, ComplicatedHello와 Hello 모두의 타입 변수는 string으로 적용된다.
  * 제네릭 타입을 string 대신 number로 수정할 경우, 변동 사항이 적용되는 Hello<T>.value와 ComplicatedHello<T>.something은 에러가 발생한다. 
```
interface Hello<T> {
    value: T;
    title: string;
}
interface ComplicatedHello<T> extends Hello<T>{
    something: T;
    bye: string;
}

let complicatedHello: ComplicatedHello<string> = {
    value: 'value',
    title: 'title',
    something: 'something',
    bye: 'bye',
}
```

### Best Common Type 추론 방식
* 상술한 내용은 변수 하나에 명시적인 값을 초기화하므로, 쉽게 타입 추론이 가능하다.
* 반면 배열과 같이 여러 타입이 포함될 수 있는 객체의 경우, TS는 각 타입을 포함하는 유니온 타입으로 추론한다.
  * 즉, **배열에 포함된 값의 타입들의 교집합이 될 수 있는 타입을 유니온으로 정의해나가는 방식**이다.
  * 이렇듯 **Best Common Type이란 임의의 타입을 가리키는 용어가 아니며, 오히려 TS가 코드가 어떤 타입인지 찾아나가는 알고리즘을 의미**한다.
```
let arr = [1, '2', true]; // (number | string | boolean)[] 타입으로 추론된다.
```
* 결국 **모든 타입을 아우를 수 있는 가장 쉬운 방식을 택하며, 이는 곧 모든 타입에 대한 유니온 타입으로 추론하는 것**이다.

### 타입 단언이란?
* 타입 추론에 의해 `let temp`는 any, `let num = 10`은 number로 추론된다.
  * 당연히 `let num = temp`의 경우, num은 any로 추론될 것이다.
* 그러나 다음과 같은 코드에서, TS는 b를 any로 추론하지만 개발자가 보는 관점에서는 number로 추론되기를 원할 수 있다.
```
let a;
a = true;
a = 'a';
a = 10;
let b = a; // any로 추론된다.
```
* 이러한 경우를 위해 **TS는 `let b = a as number`의 형식과 같이 타입을 단언할 수 있는 기능을 제공**한다.
  * 이 경우 a는 타입 단언에 의해 number로 추론되므로, 자연스럽게 b 역시 number로 추론된다.
  * 반면, a의 타입은 여전히 any를 유지한다.
* 결국 **타입 단언이란, 타입 추론에 있어 TS보다 개발자가 더 잘 알고 있다고 가정하고 타이핑을 강제**하는 것이다.
* 다음과 같이 DOM API에 접근하는 경우, div element가 항상 존재하리라는 보장이 없으므로 컴파일리 불가능하다.
  * 때문에 일반적으로는 if 분기를 통해 div의 존재성을 확인한다.
```
let div = document.querySelector('div');
// console.log(div.innerText); // div가 존재하지 않을 수도 있으므로, div는 null일 수도 있다. 
if(div) 
  console.log(div.innerText);
```
* 반면, **해당 DOM 요소가 반드시 존재한다는 것을 보장받을 수 있는 경우에 한해 타입 단언을 사용**할 수 있다.
```
let div = document.querySelector('div') as HTMLDivElement;
console.log(div.innerText);
```

### 타입 단언만으로는 아쉬운 경우
* 아래와 같이  유니온 타입의 모든 조건을 충족하는 값을 반환하는 함수를 작성한다.
  * 이 경우, choco 변수는 유니온 타입인 `Person | Dog`로 추론된다. 
  * 이렇듯 유니온 타입을 반환하는 함수를 작성하는 것은 실무에서도 자주 사용되는 방식이다.
```
interface Person {
    name: string;
    age: number;
}

interface Dog {
    name: string;
    alias: string;
}

function whoAreU(): Person | Dog {
    return {
        name: 'ingnoh',
        alias: 's',
        age: 3,
    }
}

let choco = whoAreU();
// console.log(choco.age); // 컴파일 에러!
```
* 그러나 **함수의 반환형이 유니온 타입이므로, 실제로는 age 속성을 포함하도록 하드코딩된 객체를 반환함에도 이에 접근할 수 없는 문제가 발생**한다.
  * **이러한 이슈는 명시된 각 타입의 공통 속성에만 접근이 가능한 유니온 타입의 특징에서 기인**한다.
* 이 경우, 원하는 age 속성에 접근하기 위해 상술했던 타입 단언을 응용할 수 있다.
  * `choco as Person`의 반환형은 Person으로 단언된 객체이다.
  * 이 경우, **원본 객체와 반환된 객체는 `===` 연산자를 통해 확인해볼 경우 동일한 객체로 취급**된다.
```
let choco = whoAreU();
if((choco as Person).age) {
    console.log((choco as Person).age);
} else {
    console.log((choco as Dog).alias);
}
```
* 그러나 **이러한 방식은 유니온 타입의 객체를 사용할 때마다 타입 `as` 키워드를 활용하는 타입 단언문을 반복적으로 작성해주어야하는 단점이 존재**한다.
  * 이러한 방식은 당연히 유지보수성과 가독성을 떨어트릴 수 밖에 없다.

### 타입 가드란?
* **타입 가드는 `is` 키워드를 활용하며, 일반적으로 `is`라는 접두사로 시작되는 함수의 형태로 작성**된다.
```
function isPerson(param: Person | Dog): param is Person {
    return (param as Person).age !== undefined;
}
```
* 상술한 코드의 경우, return 구문에 작성된 표현식이 true인 경우에 Person 타입으로 취급하겠다는 의미를 갖는다.
* 이렇듯 **타입 가드를 구현한 함수를 통과한 경우, 인자로 전달된 객체는 반환형에 명시된 Person 타입이 맞는지 검증**된다.
* 아래의 코드에서, isPerson 함수를 통해 choco의 타입이 적절하게 추론된다.
  * **if 또는 else 문 내부에서, choco 변수의 타입은 이미 추론된 상태이므로 유니온 타입의 공통 속성 외에도 필요한 속성에만 접근이 가능**해진다.
```
let choco = whoAreU();
if(isPerson(choco)) {
    console.log(choco.age);
    // console.log(choco.alias); // if 블록에서 choco는 Person 타입이므로, Dog 타입의 속성에 접근할 수 없다.
} else {
    // console.log(choco.age); // else 블록에서 choco는 Dog 타입이므로, Person 타입의 속성에 접근할 수 없다.
    console.log(choco.alias);
}
```
* 이렇듯 **타입 가드는 유니온 타입의 공통된 속성에만 접근을 허용하는 TS의 타입 추론이 더 정확해지도록 도울 수 있다**.