# Basics
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