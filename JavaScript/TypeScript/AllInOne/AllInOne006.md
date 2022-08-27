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