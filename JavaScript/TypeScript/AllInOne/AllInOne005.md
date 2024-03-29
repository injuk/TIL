# AllInOne
## 2022-08-25 Thu
### 옵셔널 타입
* **옵셔널 타입은 `?` 키워드로 표시하며, 이는 없을 수도 있는 옵셔널한 값을 나타내기 위해 사용**한다.
```
// skills 필드는 값이 없을 수도 있는 옵셔널한 값이다.
type Some = { name: string; skills?: string[] };
const some1: Some = { name: 'ingnoh' }; // skills가 없더라도 문제가 발생하지 않는다.
const some2: Some = { name: 'ingnoh', skills: [ 'JS', 'TS' ] };
// 그러나 여전히 타입에 명시되지 않은 속성은 사용할 수 없다.
// const some3: Some = { name: 'ingnoh', skills: [ 'JS', 'TS' ], age: 3 }; 
```
* 또한, **옵셔널 타입은 함수의 매개 변수에도 적용할 수 있다**.
```
function doSomething(paramA: string, paramB?: string, paramC?: number) {
    console.log(paramA);
    console.log(paramB);
    console.log(paramC);
}

doSomething('hello');
doSomething('hello', 'world');
doSomething('hello', 'world', 3);
// 여전히 함수에 정의된 매개 변수 개수를 넘을 수는 없다.
// doSomething('hello', 'world', 3, undefined);
```

### TS에서의 제네릭의 활용
* 문자열 또는 숫자를 받아 문자끼리 또는 숫자끼리 더하는 함수를 작성하는 경우, 결과를 다음과 같을 수 있다.
```
type NumOrString = number | string;
function plus(a: NumOrString, b: NumOrString): NumOrString {
  // ...생략
}
```
* 그러나 해당 함수는 예상치 못하게 다음과 같이 사용될 가능성을 배제할 수 없다.
```
console.log(plus(10, 20));
console.log(plus(10, '20')); // 의도치 않은 사용 방법이다.
console.log(plus('10', '20'));
console.log(plus('10', 20)); // 의도치 않은 사용 방법이다.
```
* 이를 모두 충족시키기 위해서는 함수는 다음과 같이 지저분하고, 가독성이 떨어지게 된다.
  * 또는 다른 이름의 함수를 나누어 작성해야 한다.
```
type NumOrString = number | string;
function plus(a: NumOrString, b: NumOrString): NumOrString {
    if(typeof a === 'number') {
        if(typeof b === 'number') {
            return a + b;
        } else {
            return a + Number(b);
        }
    } else {
        if(typeof b === 'string') {
            return a + b;
        } else {
            return a + b.toString();
        }
    }
}
```
* 제네릭은 이러한 경우를 일반화하기 위해 도입된 개념으로, 다음과 같이 적용하여 코드를 깔끔하게 정리할 수 있다.
* 이 때, **제네릭이란 함수에서 사용할 타입을 지금 당장은 알 수 없더라도 이를 마치 변수처럼 일반화하여 나중에 구체화하고 싶을 때 적용할 수 있는 개념**이다.
* 제네릭 함수는 다음과 같이 작성하며, 무슨 타입인지는 아직 알 수 없으나 같은 타입인 경우에 동일한 문자로 나타낸다.
  * **제네릭은 함수 선언 당시에는 타입을 알 수 없으나, 함수가 호출되는 시점에 그 타입이 결정된다**.
  * 이 때, 문자의 종류는 하술한 T가 아니더라도 상관이 없다.
```
// 이로 인해 plus 함수에는 반드시 타입이 같은 매개 변수만 전달될 수 있다.
function plus<T>(a: T, b: T): T {
    return a;
}
```
* 나아가 화살표 함수에 제네릭을 적용하는 경우, 다음과 같이 작성할 수 있다.
```
const merge = <T>(x: T, y: T): { x: T, y: T } => ({ x, y });
```

### 제네릭에 제한을 두기
* 그러나 상술한 코드는 제네릭인 T에 아무런 제한이 없으므로, 숫자 또는 문자열 끼리만 연산이 가능하도록 하고자 하는 초기의 목적과 부합하지 않는다.
  * 즉, 타입이 같기만 하면 함수에 두 개의 boolean 값이 전달될 수도 있다.
  * **이러한 경우를 미연에 방지하기 위해, `<T extens 타입>`과 같은 형태로 T의 범위를 제한**할 수 있다.
  * **이 경우, T에는 명시된 타입의 부분 집합인 타입만이 적용**될 수 있다.
```
// 이로 인해 plus 함수에는 string 또는 number 타입만이 전달될 수 있다.
function plus<T extends string | number>(a: T, b: T): T {
    return a;
}
```
* 또한, **제네릭 타입은 여럿을 명시할 수 있으며 각각 다른 제한을 두는 것 역시 가능**하다.
```
function complicatedGeneric<K extends string, V extends number>(key: K, value: V) {}
```

## 2022-08-26 Fri
### 기본값의 타이핑
* JS에서, 함수의 매개 변수는 `const plus = (a = 1, b = 0) => a + b;`와 같은 형태로 기본 값을 설정할 수 있다.
* TS 역시 함수의 기본 값에 `const plus = (a: number = 1, b: number = 0): number => a + b;`와 같이 타이핑을 적용할 수 있다.
  * 이러한 형태의 기본 값 타이핑은 변수의 타입이 조금만 복잡해져도 분석이 어려워질 수 있다.
  * 그러나 **중요한 것은 언제나 TS 코드가 JS 코드로 변경된다는 것이며, 헷갈리는 경우 모든 타이핑 정보를 제거하고 분석을 진행**할 수도 있다.
  * 이렇듯 함수의 매개 변수에 기본 값이 설정되고, 객체 또는 복합 타입으로 타이핑되는 경우 타이핑 자체가 언뜻 어려워보이기 쉽다.

### 제네릭 기본 값
* 제네릭 역시 기본 값을 적용할 수 있으며, TS가 타입을 명확히 추론하지 못하는 경우에는 기본 타입을 적용한다.
  * 즉, **TS가 명확히 타입을 추론했다면 제네릭은 기본 값이 아닌 추론된 타입으로 적용**된다.
* 반면, **기본 값과는 관계가 없으나 `<T extends unknown>` 형태의 제네릭 제한 역시 가능**하다.
```
const merge = <T = unknown>(x: T, y: T): { x: T, y: T } => ({ x, y });
// 이렇게 사용할 수도 있다!
// const merge = <T extends unknown>(x: T, y: T): { x: T, y: T } => ({ x, y });
console.log(merge(1, 2));
```
* 이 때, **상술한 제네릭 기본 값 또는 unknown 제한의 경우 JSX 문법을 사용하는 React 기반 개발자들을 위한 편의 기능에 해당**한다.

### 인터페이스, 타입 별칭, 클래스의 제네릭
* 인터페이스와 타입 별칭에도 제네릭을 적용할 수 있으며, 다음과 같은 형태로 작성한다.
```
interface IGeneric<T> {
    some: T;
}
const numSomething: IGeneric<number> = { some: 3 };
const strSomething: IGeneric<string> = { some: '3' };

type GenericType<T> = { some: T };
const genericNumType: GenericType<number> = { some: 3 };
const genericStrType: GenericType<string> = { some: '3' };
```
* 클래스 역시 마찬가지이며, 인터페이스 또는 타입 별칭과 마찬가지로 모두 이름 뒤에 제네릭 타입을 작성한다.
  * 이 때, **다른 객체지향 언어와 마찬가지로 new 연산자에서 사용하는 인스턴스 생성자에는 굳이 제네릭 타입을 명시하지 않아도 무방**하다.
```
class GenericClass<T> {
    some: T;

    constructor(some: T) {
        this.some = some;
    }
}
const numInstance: GenericClass<number> = new GenericClass(3);
console.log(numInstance);
// 아래와 같이 new 연산자에서 사용하는 인스턴스 생성자에 제네릭 타입을 명시할 수도 있다.
const strInstance: GenericClass<string> = new GenericClass<string>('3');
console.log(strInstance);
```
* 이렇듯 **제네릭은 코드 작성 시점에는 타입을 알 수 없으나, 코드를 실행하는 시점에서는 타입이 무엇인지 알 수 있다는 점을 활용**한다.
  * **TS는 제네릭 타입이 적용된 대상 중 하나라도 타입이 추론되는 시점에 모든 제네릭 타입을 추론된 타입으로 대체하는 식으로 동작**한다.
  * 또한, 제네릭 타입 자리에 `<number>`와 같이 개발자가 직접 타입을 명시하는 것은 상술한 타입 추론 과정 없이 모든 제네릭 타입을 대체한다.
  * 이렇듯 직접 제네릭 타입 변수를 타이핑하는 상황은 TS 자체가 타입을 정확히 추론해내지 못할 때 발생할 수 있다.

### 혼동하기 쉬운 제네릭과 타입 변환
* `something<number>`와 같은 형태로 작성하는 것은 제네릭 타입 변수를 타이핑하는 것에 해당한다.
* 반면, `<number>something`은 `as` 키워드를 사용하는 것과 유사한 강제 변환에 해당한다.

## 2022-08-27 Sat
### 기능과 타입은 필요할 때 추가하기
* **시간이 지남에 따라 프로젝트가 성숙해지고, 애플리케이션이 고도화되가는 과정에서 자연스럽게 새로운 기능과 적절한 타이핑을 추가하는 것이 바람직**하다.
  * 예를 들어, 너무 이른 시기에 당장 필요 없는 기능을 설계하고 타이핑하는 데에 시간을 쏟을 필요는 없다.
* **처음부터 모든 것을 예상하여 타이핑을 완벽히 완료하기는 매우 어려운 일이며, TS 개발 초기에는 단순히 코드에서 오류가 발생하지 않는 정도를 목표**한다.
  * 이후, 기능이 추가되 감에 따라 코드 및 타이핑을 추가한다.

### TS 타이핑 연습하기
* TS로 **타이핑을 능숙하게 할 수 있도록 배워나가는 효율적인 방법 중 하나는 기존의 JS 코드를 보고 TS 코드로 타이핑해보는 것**이다.
  * 또는, **여러 라이브러리에서 이미 작성되어 있는 TS 코드를 분석하는 것도 큰 도움**이 된다.
  * 예를 들어, 아래의 코드는 `Array.prototype.forEach`와 `Array.prototype.map`을 직접 타이핑해본 결과이다.
```
interface MyArr<T> {
    forEach(cb: (e: T) => void): void;
    map<U>(cb: (e: T) => U): U[];
}
```
* 이렇듯 **TS를 사용하는 개발자는 필요할 때 JS 코드를 TS 코드로 타이핑할 수도 있어야 하고, 다른 개발자가 작성한 TS 코드를 파악할 수도 있어야 한다**.
  * 상술한 바와 같이 이를 위해 다른 개발자의 TS 코드를 분석하고, 사용 예시로부터 자신이 직접 타이핑해보는 양방향 연습을 지속적으로 수행하는 것이 바람직하다.
  * **결과적으로는 원하는 타입을 보고 이에 맞는 적절한 코드를 작성하거나, 또는 원하는 코드를 기반으로 타입을 작성할 수 있어야 한다**.

### filter 메소드 분석하기
* 언뜻 배열의 filter 메소드는 다음과 같이 타이핑될 것으로 예상할 수 있다.
  * 이는 filter 메소드의 결과는 원본 배열로부터 임의의 요소가 걸러진 배열이기 때문에 그렇다.
```
interface MyArr<T> {
  filter<S>(predicate: (e: T) => e is S): S[];
}
```
* 그러나 이 경우, 전혀 관계가 없는 S와 T에 대해 반환 값이 `e is S`와 같은 커스텀 타입 가드를 사용하게 되므로 형식을 할당할 수 없다는 오류가 발생한다.
  * **TSC 입장에서는 전혀 무관한 제네릭 타입 변수 T와 S에 대해 갑자기 T는 S이다, 라고 선언된 꼴**과 같다. 
* **이를 가능케하기 위해서는 S와 T 사이에 어떠한 관계가 존재해야 하며, 정확히는 T라는 타입이 S라는 타입으로 타입 가드에 의해 좁혀질 수 있어야 한다**.
  * 타입 가드의 역할은 큰 타입을 작은 타입으로 좁히는 것이므로, **S가 T의 부분집합이어야만 타입 가드를 활용한 내로잉이 가능할 것**이다.
* 때문에 **S와 T 사이의 포함 관계를 `<S extends T>`와 같이 코드에 명시**할 수 있다.
```
interface MyArr<T> {
  // S는 T의 부분집합이며, T에 포함된다는 사실을 아래와 같은 형식으로 TS에게 알릴 수 있다.
  filter<S extends T>(predicate: (e: T) => e is S): S[];
}
```
* 이를 **실제 코드에 적용하는 경우에도 JS와 같은 일반 predicate 형식이 아닌, 반드시 형식 조건자(= 커스텀 타입 가드) 형태의 함수를 명시**해야 한다.
  * 이 경우 가독성을 위해 predicate 용 함수를 외부로 추출하거나 인라인시킬 수 있다.
  * **그러나 이렇듯 복잡한 타입이 반드시 명시되어야 하는 TS의 특징으로 인해 가독성은 두 방식 다 좋지 못하다**.
```
const tempArr: MyArr<string | number | boolean> = [1, '2', true];
const filtered2 = tempArr.filter((e): e is string => e === '2');
```
* 상술한 바와 같이 이미 존재하는 라이브러리 코드를 상상하며 타이핑을 직접 해보는 것은 큰 의미가 있으며, 그만큼 난이도가 높은 작업이기도 하다.
  * **이는 `filter(): void;`처럼 간단한 타입으로부터 시작하여 점점 고도화하고, 제네릭을 활용하는 일반화를 적용하는 식으로 연습하는 것이 바람직**하다.