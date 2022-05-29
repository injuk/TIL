# Basics
## 2022-05-29 Sun

### TS 기본 타입 정의하기
* JS와 TS 각각에 대한 변수 선언은 다음과 같은 차이가 있다.
```
let jsString = 'hello world!';
let tsString: string = 'hello world!';

let jsNumber = 3;
let tsNumber: number = 3;

let jsBoolean = true;
let tsBoolean: boolean = true;

let jsArray = [0, 1, 2];
let tsArray: Array<number> = [0, 1, 2];
let tsArray2: number[] = [0, 1, 2]; // tsArray와 상동하다.
```
* TS에서 사용되는 튜플은 각 인덱스에 임의의 타입을 명시하는 배열이며, 다음과 같은 방식으로 선언할 수 있다.
```
let jsTuple = ['ingnoh', 3];
let tsTuple: [string, number] = ['ingnoh', 3];
```
* TS에서 사용되는 객체는 기본적으로 다음과 같이 선언할 수 있다.
```
let jsObject = {};
let tsObject: object = {};
```
* 그러나 **JS의 특성상 많은 것이 본질적으로 객체이므로 상술한 형식에 부합**한다.
    * 또한, 상술한 표기법의 경우 객체 안에 어떠한 속성이 포함되더라도 이를 제한하지 않는다.
* 객체의 정의를 더욱 엄격하게 제한하기 위해 다음과 같은 표기법을 사용할 수 있다.
```
let animal: { name: string, age: number } = {
    name: 'bird',
    age: 3,
    // alias: 'buddy', // 타입에 정의되지 않은 속성을 작성할 수 없다.
};
```

### TS 함수 정의하기
* TS 함수의 타입 명시는 크게 파라미터와 반환형으로 나뉘며, 다음과 같이 정의할 수 있다.
    * 이 때, 반환형의 경우 작성을 생략하더라도 어느 정도는 추론이 되는 경우가 있다.
    * **반환형을 명시한 경우, 반드시 해당 타입의 값을 반환하도록 코드를 작성**해야 한다.
```
function onlyParams(num1: number, num2: number) {
    return num1 + num2;
}

function onlyReturns(num1, num2): number {
    return num1 + num2; // 반환형을 명시하였으므로 해당 부분을 누락시킬 수 없다.
}

function bothOfThem(num1: number, num2: number): number {
    return num1 + num2;
}
```
* 반환형의 경우 항상 작성하지 않더라도 어느 정도는 추론이 되는 경우가 있다.

### 엄격한 함수 파라미터 검사
* JS의 경우, 다음과 같이 함수에 정의된 인자의 개수를 벗어나는 호출이 가능하다.
```
function sum(num1, num2) {
    return num1 + num2;
}

sum(1, 2); // 행복 경로이다.
sum(1); // 함수에 정의된 인자보다 적다.
sum(1, 2, 3, 4); // 함수에 정의된 인자보다 많다.
```
* 반면, **TS의 경우 TS 컴파일러가 함수의 스펙을 정확히 이해하므로 이러한 호출은 컴파일 에러를 일으키는 원인**이 된다.
```
function sum(num1: number, num2: number): number {
    return num1 + num2;
}

sum(1, 2);
// sum(1); // 주석 해제시 컴파일이 불가능하다.
// sum(1, 2, 3, 4); // 주석 해제시 컴파일이 불가능하다.
```

### 옵셔널 파라미터
* 상술한 엄격한 파라미터 검사는 애플리케이션에 안정성을 부여하는 TS의 특징이지만, 한 편으로는 JS의 유연성을 해치는 결과를 낳을 수도 있다.
* **함수에 인자를 두되, 전달 가능한 인자의 수를 유연하게 결정하고 싶은 경우에는 다음과 같이 물음표 기호를 활용한 옵셔널 파라미터를 적용**할 수 있다.
```
function doSomething(num1: number, num2?: number, num3?: number) {
}

doSomething(1, 2, 3);
doSomething(1);
doSomething(1, 2);
// doSomething(1, 2, 3, 4); // 이 경우에도 함수에 정의된 인자의 개수를 벗어나는 호출은 불가능하다.
```
* 이렇듯 함수에 정의된 인자 중 특정한 인자는 선택적으로 전달하고자 하는 경우, 물음표 기호를 붙여 옵셔널 파라미터로 정의할 수 있다.
    * 이 때, 함수에서 옵셔널 파라미터로 정의된 인자 이후의 인자는 모두 옵셔널 파라미터를 적용해야 한다.
    * 예를 들어 함수에 세 개 인자를 정의하는 경우, (required, optional, required) 순으로 정의할 수는 없다.
* **옵셔널 파라미터가 정의된 인자는 함수 호출시 명시된 타입의 값을 전달하거나, 생략할 수 있다**.