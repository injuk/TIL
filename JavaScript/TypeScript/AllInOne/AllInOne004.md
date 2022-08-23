# AllInOne
## 2022-08-24 Wed
### 타입 가드
* 다음과 같이 유니온 타입을 받아들일 수 있는 함수의 경우, 다짜고짜 하나의 타입에서만 사용 가능한 프로퍼티에 접근할 수는 없다.
  * 이는 **모든 가능성을 고려하는 TS의 특징에서 기인하는 것으로, param이 number인 경우 정상 동작하지 않기 때문**이다.
```
function test(param: number | string) {
    return param.split(' '); // 에러, number.split은 불가능하다!
}
```
* 이를 정상 동작시키기 위해서는 TS에게 param의 속성이 어떻게 결정되는지 정확히 명시할 필요가 있으며, 다음과 같은 방식을 고려할 수 있다.
  1. `(param as string).split(' ')` 사용하기: 가장 쉬운 방법이나, 휴먼 에러가 발생하기 쉬워 위험성 역시 높다.
     * 사실상 **unknown으로 추론되었거나 잘 못 추론된 변수를 명확히 타이핑하려는 경우가 아니라면, `as`는 사용을 지양하는 것이 바람직**하다.
  2. 타입 가드 활용하기
* **타입 가드란, 다음과 같이 분기 처리를 적절히 활용하여 임의의 코드 상에서의 타입을 명확히 하는 방식을** 의미한다. 
```
function test(param: number | string) {
    if(typeof param === 'string') {
        // param의 타입이 문자열인 경우, 안심하고 String.prototype의 프로퍼티에 접근할 수 있다.
        return param.split(' ');
    } else {
        // 해당 함수는 유니온 타입을 매개 변수로 받으므로, param의 타입이 문자열이 아니라면 반드시 숫자가 된다.
        return param.toFixed(2);
    }
}
```
* **상술한 코드는 `typeof` 키워드를 활용하여 원시 타입을 분류하는 타입 가드이며, 타입 가드 기법은 여러 방식이 존재**한다.
  1. 원시 타입의 경우, `typeof` 키워드를 활용하여 분류하기
  2. 유니온 타입에 배열이 포함된 경우, `Array.isArray()`를 활용하여 분류하기
  3. 임의의 클래스의 인스턴스를 분류하는 경우, `instanceof` 키워드를 활용하여 분류하기 
     * 이 경우, **A 클래스 또는 B 클래스를 매개 변수로 받아들일 수 있는 함수는 `param: A | B` 형태로 클래스의 이름을 통해 유니온 타입을 작성**한다.
  4. 상수로 타이핑된 속성을 갖는 객체를 의미하는 타입 별칭의 경우, 각 객체가 유일하게 갖는 프로퍼티의 값을 확인하여 분류하기
* 특히 마지막 방법의 경우, 다음과 같은 타입 가드를 의미한다.
```
// 아래의 세 타입 별칭은 공통된 속성인 type을 갖지만, 각각 상수로 타이핑되어 유일한 값을 갖는다. 
type Ab = { type: 'ab', name: string };
type Cd = { type: 'cd', age: number };
type Ef = { type: 'ef', isBird: boolean };

function process(param: Ab | Cd | Ef) {
    if(param.type === 'ab') {
        console.log(param.name);
    } else if(param.type === 'cd') {
        console.log(param.age);
    } else {
        console.log(param.isBird);
    }
}
```
* 이렇듯 **여러 객체 간의 타입 검사가 필요한 경우, 가능한 한 각각의 객체를 구분할 수 있도록 객체의 타입을 설계**할 수 있어야 한다.
  * 예를 들어, 상술한 세 타입 별칭은 상수로 타이핑된 type 속성 외에도 서로 다른 속성인 name, age, isBird를 갖는다.
  * **이 점을 활용하여 `in` 연산자를 활용하는 다음과 같은 방식의 타입 가드를 적용할 수도 있다**.
  * `in` 연산자 역시 유효한 JS의 문법에 해당하며, TS 상에서는 타입 검사를 위해 `in` 연산자를 자주 사용하게 된다.
```
type Ab = { type: 'ab', name: string };
type Cd = { type: 'cd', age: number };
type Ef = { type: 'ef', isBird: boolean };

function process(param: Ab | Cd | Ef) {
    // in 연산자를 활용하여 객체 내부에 임의의 속성이 존재하는지를 확인할 수 있다.
    if('name' in param) {
        console.log(param.name);
    } else if('age' in param) {
        console.log(param.age);
    } else {
        console.log(param.isBird);
    }
}
```

### 값을 활용한 객체 타입 검사와 in 연산자를 활용한 객체 타입 검사
* **둘 모두 객체들 간의 타입을 구분하기 위해 사용하는 방식이지만, 일반적으로 더 자주 사용되는 것은 값을 활용한 객체 타입 검사**이다.
* 이 때, **중요한 것은 여러 객체를 타이핑하는 경우, TS의 추론을 지원하기 위해 type과 같은 타입 추론용 속성을 추가하는 습관을 들이는 것**이다.
  * 예를 들어, 상술한 예시와 같이 type 속성을 추가하되 각 타입을 구분할 수 있도록 서로 다른 값을 할당한다.
  * 이러한 습관은 객체에 태깅, 또는 라벨링한다는 표현으로 지칭할 수도 있다.
* 반면, **사전에 타입 추론용 속성을 태깅해두지 않은 경우 각 객체의 차이를 찾아 `in` 연산자를 활용하여 구분하는 타입 가드를 적용**한다.