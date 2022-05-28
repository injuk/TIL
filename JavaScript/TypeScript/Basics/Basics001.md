# Basics
## 2022-05-28 Sat

## TypeScript란?
```
> TypeScript is typed JavaScript. 
```
* TypeScript란, JS에 타입을 추가한 또 하나의 언어이다.
  * JS의 확장된 버전이므로, JS의 Superset이라고도 이야기할 수 있다.
* **TS는 브라우저가 온전히 이해할 수 없으므로, TS로 작성된 애플리케이션을 JS로 변환하는 컴파일 과정이 필요**하다.

### Why TypeScript?
* 일반적으로, **코드 상에서는 알 수 없는 데이터 구조에 대한 접근 결과는 항상 런타임에서만 확인할 수 있다**.
  * 예를 들어 임의의 object 내부에 포함된 프로퍼티에 대한 접근이 잘못된 경우, 이는 항상 브라우저 또는 Node.js 애플리케이션의 실행 후에만 알 수 있다.
* GET 요청 등 REST API 호출의 결과로 반환된 데이터 구조에 대한 정보 역시 런타임에서만 확인이 가능하다.
* 이러한 문제를 해결하기 위해 다음과 같은 Jsdoc 문서를 활용해볼 수 있다.
  * 이 경우, 런타임에서 구조를 확인할 수 있었던 기존 방식과 달리 IDE 차원에서 자동 완성 기능 등을 제공받을 수 있게 된다.
```
// User object의 데이터 구조를 정의한다.
/**
 * @typedef {object} User
 * @property {string} name
 * @property {number} age
 */

// 함수가 반환하는 데이터 구조를 명시한다.
/**
 * @returns {User} user
 */
function createUser() {
  // 해당 함수는 Jsondoc에 의해 반환 형에 대한 힌트가 주어지므로, 아래와 같은 데이터 구조를 작성할 때 IDE 차원에서 자동 완성 기능을 지원받을 수 있다.
  return {
    name: 'injuk',
    // age: '3', // 해당 부분을 사용할 경우 IDE 차원에서 타입에 대한 경고 알림을 줄 수 있다.
    age: 3,
  };
}
```
* 또한 JS는 타입을 강제할 수 없으므로, 다음과 같은 코드에서 의도하지 않는 결과가 반환될 수 있다.
  * 이러한 경우에도 Jsdoc을 통해 간접적으로 타입이 부여된 경우의 이점을 누릴 수 있다.
```
console.log(
    (function add10(number) {
      return number + 10;
    })('20')
);
// 실행 결과: 2010

// Jsdoc을 활용한 간접적 타입 부여
/**
 * 곱셈 함수
 * @param {number} num1 first operand
 * @param {number} num2 second operand
 * @returns {number} result
 */
function multiply(num1, num2) {
  return num1 + num2;
}
```
* 이로 미루어, JS에 타입이 존재하는 경우의 장점은 다음과 같이 정리해볼 수 있다.
  * **반드시 런타임에서만 확인 가능하던 오류 상황을 미연에 방지할 수 있으므로, 더 안정적인 애플리케이션의 작성이 가능**해진다.

### 그럼 Jsdoc만 사용해도 되지 않을까?
* 상술한 바와 같이 JS에 타입을 부여하는 경우에 얻어지는 이점을 누리기 위해 Jsdoc 등의 도구를 사용할 수 있으나, 이는 다음과 같은 단점이 존재한다.
  1. 강제성이 떨어진다.
  2. Jsdoc을 모두 작성해야하므로 코드의 양이 너무 길어진다.
  3. Jsdoc은 주석과 유사한 특징을 가지므로, 잘 관리되지 않은 경우에 쓰레기 정보가 될 가능성이 높다.

## 2022-05-29 Sun
### TypeScript 시작하기
* TS 파일은 .ts 확장자를 갖는다.
* 생성한 .ts 파일에는 일반적인 JS 문법에 따라 코드를 작성할 수도 있지만, 다음과 같은 타입 명시도 가능하다.
```
function sum(a: number, b: number): number {
    return a + b;
}

console.log(
    sum(10, 20)    
);

```
* **.ts 파일의 내용은 브라우저가 그대로 이해할 수는 없으므로, 브라우저가 이해할 수 있는 JS 파일로 변환하기 위한 컴파일 과정이 필수적**이다.
* .ts 파일을 JS 파일로 컴파일하기 위해서 npm을 활용하여 typescript 패키지를 전역으로 설치해야 한다.
  * 이를 통해 TS 컴파일을 위한 명령어인 tsc를 사용할 수 있게 된다.
```
> npm i -g typescript 
```
* 설치된 tsc 명령어를 활용하여 index.ts를 index.js로 컴파일하는 명령어는 다음과 같다.
  * 명령 실행 성공시 아무런 메시지가 노출되지 않으며, 인자로 전달한 .ts 확장자 파일과 동일한 이름을 갖는 .js 확장자 파일이 생성되는 것을 확인할 수 있다. 
```
> tsc index.ts
```

### TS 설정 파일
* tsc 명령어를 활용하여 컴파일을 수행할 경우 여러 부가적인 옵션을 줄 수 있으며, 이러한 옵션들은 일반적으로 tsconfig.json 파일에 명시된다.
  * **해당 파일에는 컴파일 과정에서 TS가 프로젝트를 어떻게 이해해야하는지에 대한 내용을 명시**한다.
```
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es5",
    "sourceMap": true,
    "allowJs": true,  // 프로젝트 내에서 JS 작성을 허용한다.
    "checkJs": true, // @ts-check와 같이 JS에서 TS 기능을 녹여낸다.
    "noImplicitAny": true // 암시적인 Any 형을 허용하지 않으며, true인 경우 any는 암시적인 추론이 아닌 명시된 경우에만 허용된다.
  },
  "exclude": [
    "node_modules"
  ]
}
```
* **tsconfig.json이 프로젝트 내에 작성된 경우, 프로젝트 내에서 수행한 tsc 명령어는 대상 파일을 컴파일하는 과정에서 해당 설정이 적용**된다.
* **tsconfig.json에 명시 가능한 모든 옵션을 외울 필요는 없으며, 필요할 때마다 TS 공식 문서의 tsconfig 레퍼런스를 참고하는 것이 바람직**하다.