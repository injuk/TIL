# AllInOne
## 2022-08-21 Sun
### JS로 변환했을 때 사라지는 개념들 파악하기
* type, type alias, interface, generic, as, enum, declare 등의 개념은 온전히 TS만의 것으로, JS로 변환된 코드에서는 모두 제거된다.
  * 또한, 간혹 타이핑을 위해 함수의 시그니쳐 부분만 존재하는 코드 역시 존재할 수 있으며 이 역시 JS로 변환되는 경우에 제거된다.
  * 때문에 **상술한 종류와 같이 JS 코드에서 제거되는 개념들 없이도 코드가 동작하도록 개발**해야 한다.
* 이렇듯 TS를 활용하여 개발하는 경우, JS 코드로 변환되었을 때 제거되는 TS만의 개념을 정확히 이해하고 있어야 한다.

### never 타입
* TS에서 `const temp = [];`와 같이 빈 배열을 선언한 경우, 타입은 자동으로 `never[]` 타입으로 추론된다.
  * never 타입은 일반적인 타입을 받아들이지 않으므로, 해당 빈 배열은 더 이상 어떠한 요소도 추가할 수 없게 된다.
  * 이러한 상황을 방지하기 위해서는 **반드시 `const temp: string[] = [];`의 형태로 직접 타이핑**해야 한다.

### 타입 단언
* `const my = document.querySelector('#my');`와 같은 코드에서, my 변수는 `Element | null`로 추론된다.
  * 이는 해당 함수가 요소가 없는 경우에 null이 반환되는 것을 대비하여 타이핑되어 있기 때문이다.
  * 그러나 **개발자가 해당 요소가 반드시 존재함을 확신할 수 있는 경우, 이를 다음과 같이 단언**할 수 있다.
```
// ! 기호를 붙여 반드시 Element가 존재함을 단언하므로, my는 Element로 추론된다.
const my = document.querySelector('#my')!;
```
* **`!` 기호를 활용하는 단언은 해당 함수의 결과가 절대 null 또는 undefined일 수 없음을 개발자가 책임지고 보증하기 위해 사용**된다.
* 그러나 **세상에 절대라는 것은 존재할 수 없으므로, 많은 경우에 `!` 기호를 활용하는 단언보다는 다음과 같은 if 문의 활용이 권장**된다.
  * 이 때, if 블록 안에서 변수 my는 자동으로 Element로 추론된다.
```
const my = document.querySelector('#my');
if(my) my.innerHTML = 'exist!'; // if 블록 안에서 변수 my는 자동으로 Element로 추론된다.
```

### 래퍼 타입
* `new String()`과 같은 코드의 사용을 지원하기 위해, TS 역시 래퍼 타입을 지원한다.
  * 그러나 **문자열 표현을 위해 `String` 클래스가 잘 사용되지 않듯, 해당 타입 역시 사용을 지양하는 것이 바람직**하다.
```
const str = 'from string';
const Str = new String('from String');
```

### 템플릿 리터럴 타입 활용
* 백틱을 활용하는 템플릿 리터럴 역시 다음과 같이 타이핑을 활용할 수 있다.
  * 이 때, SayHello 타입은 두 종류의 문자열을 지원하는 타입으로서 동작하며 이외의 값을 초기화할 수는 없게 된다.
  * 이 경우, **SayHello 타입으로 지정된 변수에 문자열을 입력하는 과정에서 아래와 같은 두 예시에 해당하는 자동 완성 기능을 제공**받을 수 있다.
```
type Name = 'ingnoh' | 'injuk';
type SayHello = `hello, ${Name}`;

const hello1: SayHello = 'hello, ingnoh';
const hello2: SayHello = 'hello, injuk';
```
* SayHello 타입에서 볼 수 있듯, Name 자리는 타입이 적용된 것이므로 이외에도 string, number, boolean 등을 입력할 수도 있다.
  * 예를 들어, `hello, ${boolean}`의 경우 ingnoh와 injuk 대신 true 또는 false 라는 문자열만 명시할 수 있게 된다.

### 함수 매개 변수에서의 rest 연산자 타이핑
* `...` 키워드를 활용하는 rest 연산자 역시 배열 또는 튜플로 타이핑이 가능하다.
  * 이 경우, **명시된 타입을 만족하지 않는 매개 변수를 활용한 함수 호출은 불가능**하다. 
```
function getRest(...rest: number[]) {
    console.log(rest);
}

function getTuple(...tuple: [number, number, number, boolean]) {
    console.log(tuple);
}

getRest(1, 2, 3, 4);
getTuple(1, 2, 3, false);
// getTuple(1, 2, 3, 0); // 이러한 형태의 호출은 불가능하다.
```

### 튜플의 한계
* **튜플로 타이핑된 배열은 길이가 고정되므로, 튜플의 길이를 넘는 인덱스로의 접근은 불가능**하다.
  * 그러나 튜플에 명시된 타입과 동일한 값을 사용하는 경우, push를 활용하여 튜플의 길이를 증가시킬 수는 있다.
```
const tuple: [number, string, boolean, number] = [1, 'hello', false, 2];
// console.log(tuple[5]); // 불가능하다.
tuple.push(3); // number, string, boolean 타입 중 하나라도 만족하는 값은 push할 수 있다.
tuple.push('bye');
tuple.push(true);
// console.log(tuple[5]); // push하며 길이는 증가하였으나, 여전히 인덱스를 활용한 접근은 불가능하다.
```

## 2022-08-22 Mon
### 객체와 enum 
* enum은 다음과 같이 사용하며, **그 용법 측면에서 객체로 대체가 가능한 면이 있다**.
```
enum Numbers {
    ONE,
    TWO,
    THREE = 'three'
}

const NumberEnum = {
    ONE: 0,
    TWO: 1,
    THREE: 'three',
};
```
* 이 경우, 객체와 enum은 각각 다음과 같은 차이점이 존재한다.
  1. 객체는 JS의 개념이므로, JS 코드로 변환되더라도 남는다.
  2. enum은 TS의 개념이므로, JS 코드로 변환된 경우 남지 않는다.
* 이에 따라, 둘 중 어느 것을 사용할지는 다음과 같은 흐름으로 결정할 수 있다.
  1. 변수 또는 값을 그룹화한 어떤 개념이 JS 코드로 변환된 후에도 표현되어야 하는 경우, 객체를 사용한다.
  2. JS 코드로 변환되었을 때는 남지 않아도 무방한 경우, enum을 사용한다.
  3. **남겨야할지 아닐지 판단이 서질 않는 경우, 가능한 한 객체를 사용**한다.
* 정의된 enum 역시 다른 타입과 마찬가지로 변수, 매개 변수 등에 명시하여 사용할 수 있다.

### as const 활용하기
* 상술한 NumberEnum 객체의 경우, `const NumberEnum: {ONE: number, TWO: number, THREE: string}` 형태로 추론된다.
* 그러나 TS는 가능한 한 정확한 타입 추론이 가능해야하므로, 이를 위해 `as const` 키워드를 다음과 같이 작성할 수 있다.
  * 이 경우, NumberEnum은 `const NumberEnum: {readonly ONE: 0, readonly TWO: 1, readonly THREE: "three"}`의 형태로 추론된다.
  * 이렇듯 **`as const` 키워드를 명시할 경우, 객체의 각 값을 상수로서 사용하겠다는 의미**를 갖는다.
```
const NumberEnum = {
    ONE: 0,
    TWO: 1,
    THREE: 'three',
} as const; // 초기화시 as const 키워드를 명시한다.
```
* **이는 또한 TS는 가능한 한 좁게 타이핑해야 하며, TSC가 적절한 타입을 추론하지 못한 경우 개발자가 명시적으로 타이핑해야 한다는 원칙에 따른 결과**이다.

### keyof와 typeof 조합하기
* 임의의 객체로부터 타입 별칭을 명시하고 싶은 경우, 아래와 같은 코드를 작성할 수 있다.
  * 이는 temp **객체에 추론된 타입을 typeof 키워드를 통해 가져온 후 새로운 타입 별칭인 Types에 정의**한다. 
```
const temp = { name: 'ingnoh', age: 3 }; 
type Types = typeof temp;
```
* 만약 **temp 객체의 키만을 모아 새로운 타입 별칭을 정의하고 싶은 경우, 다음과 같이 keyof 연산자를 조합**할 수 있다.
```
const temp = { name: 'ingnoh', age: 3 } as const;
type Keys = keyof typeof temp;
const nameKey: Keys = 'name';
const ageKey: Keys = 'age';
// const addressKey: Keys = 'address'; // temp 객체에 존재하지 않는 키를 사용할 수는 없다.
```
* 만약 `keyof temp`와 같은 형태로 명시하는 경우, temp는 객체의 값이지 타입이 아니므로 원하는대로 사용할 수 없다.
  * 때문에 **temp 객체가 갖는 타입 정보를 가져오기 위해 typeof를 조합하여 `keyof typeof temp` 의 형태로 코드를 작성**하게 된다.
* 반면, temp 객체의 값에 정의된 타입만 접근하고자 하는 경우에는 다음과 같이 코드를 작성할 수 있다.
  * 이 때, **`typeof temp[keyof typeof temp]` 형태로 객체의 값에 할당된 타입 정보에 접근**할 수 있게 된다.
```
const temp = { name: 'ingnoh', age: 3 } as const;
type Keys = keyof typeof temp;
const nameKey: Keys = 'name';
const ageKey: Keys = 'age';

type Values = typeof temp[keyof typeof temp];
const nameValue: Values = 'ingnoh';
const ageValue: Values = 3;
```
* 이는 다음과 같은 흐름으로 인해 타입 별칭인 Values가 추론되기 때문이다.
  1. `keyof typeof temp`로 인해 temp 객체가 갖는 모든 키 정보를 알 수 있게 된다.
  2. **`temp[keyof typeof temp]`를 통해 temp 객체의 프로퍼티에 직접 접근하는 경우, 이는 타입이 아닌 값에 접근하는 것**과 같다.
  3. 때문에 각 값의 타입 정보에 접근하기 위해 다시 typeof 키워드를 조합하여 `typeof temp[keyof typeof temp]` 형태로 코드를 작성해야 한다.
* 상술한 코드에서, **`as const` 키워드를 객체에 명시하지 않은 경우에는 타입 별칭 Values의 타입이 `string | number`로 추론**된다.

### 타입 별칭과 인터페이스의 비교
* 두 개념은 각각 새로운 타입을 정의하여 타이핑에 활용할 수 있다는 공통점이 있다.
  * 이 때, **타입 별칭은 간단하게 타이핑을 명시하는 경우에 활용**한다.
  * 반면, **인터페이스의 경우 해당 타입을 활용한 OOP를 진행하고자 하는 경우에 고려**할 수 있다.

### 유니온 타입 활용하기
* **유니온 타입은 `a: string | number`와 같은 형태로 정의할 수 있으며, 이를 통해 둘 이상의 타입을 받아들일 수 있는 타이핑이 가능**해진다.
  * 반면, **유니온 타입은 유연성을 얻는 대신 정확한 타입 추론 역시 어려워진다는 단점 역시 존재**한다.
  * TS는 기본적으로 모든 가능성을 고려하는 식으로 동작하므로, 상술한 타입의 경우 매개 변수 a 역시 string이거나 number인 두 경우 모두를 고려하게 된다.
* 이러한 **복합 타입으로 인해 TS에서 초기 타이핑의 중요성은 더더욱 높아지며, 개발 초기에 잘 못 정의한 타입으로 인해 이후의 모든 타이핑이 꼬일 수도** 있다.

### 인터섹션 타입 활용하기
* **인터섹션 타입은 `a: string & number`와 같은 형식으로 정의할 수 있으며, 이 때 매개 변수 a는 string임과 동시에 number일 수 있어야 한다**.
  * 상술한 예시에서, string임과 동시에 number인 타입은 존재할 수 없으므로 이는 never 타입으로 추론된다.
* 인터섹션 타입은 원시 타입보다는 객체에서 유용하게 활용될 수 있다.
  * 예를 들어, 아래의 **예시에서 새로이 정의된 WhoAmI 타입 별칭은 두 객체의 모든 특징을 가져야만 한다**. 
```
type WhoAmI = { name: 'ingnoh', age: 3 } & { alias: 'injuk' };
const me: WhoAmI = { name: 'ingnoh', age: 3, alias: 'injuk' };
```

### 객체를 활용한 유니온과 인터섹션 타입의 비교
* **객체를 활용하는 유니온 타입이 정의된 변수의 경우, 유니온 타입에 명시된 모든 객체의 공통된 속성에만 접근이 가능**하다. 
  * 예를 들어 상술한 예시가 인터섹션이 아닌 유니온 타입의 경우, 공통된 속성이 없으므로 변수 me로부터는 어떠한 속성에도 접근이 불가능하다. 
* **객체를 활용하는 인터섹션 타입이 정의된 변수의 경우, 인터섹션 타입에 명시된 모든 객체의 속성 각각에 접근이 가능**하다.
  * 예를 들어 상술한 예시의 경우 변수 me의 name, age, alias 속성 모두에 접근이 가능하다.
* 이렇듯 유니온 타입의 경우 타이핑에 명시된 타입 중 하나의 형태만을 만족시켜도 무방하지만, 인터섹션 타입의 경우 모든 형태를 아우를 수 있어야 한다.