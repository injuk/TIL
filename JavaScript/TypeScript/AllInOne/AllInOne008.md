# AllInOne
## 2022-08-30 Tue
### Awaited 타입
* **Awaited 타입은 다음과 같은 구조를 갖고, Promise의 배열을 제네릭 타입으로 받아들인다**.
```
type Awaited<T> =
    T extends null | undefined ? T :
        T extends object & { then(onfulfilled: infer F): any } ?
            F extends ((value: infer V, ...args: any) => any) ?
                Awaited<V> :
                never :
        T;
```
* Awaited는 다음과 같은 흐름으로 동작한다.
  1. T가 null 또는 undefined인 경우, 해당 타입을 그대로 반환한다.
  2. null 또는 undefined가 아닌 경우, 객체이면서 then을 갖는지 확인한다.
     * 이 때, **`{ then(onfulfilled: infer F): any }`는 Promise의 모양을 의미**한다.
     * **또한 해당 문장에서는 Duck typing을 적용하므로, 전달되는 객체는 반드시 Promise일 필요가 없이 단지 thenable이기만 하면 된다**.
     * **때문에 Promise 객체가 아닌 thenable 객체를 Awaited에 전달하더라도 여전히 타입 추론은 정상적으로 수행**된다.
  3. 갖는 경우, Promise의 콜백 함수에 전달되어야 할 매개 변수의 타입을 infer 키워드로 추론한다.
  4. 추론에 성공한 경우, 추론된 타입을 통해 `Awaited<V>`를 재귀호출한다.
* 이러한 **복잡한 과정을 통해 여러 Promise의 타입 역시 TS에 의해 추론될 수 있다**.
* Awaited는 여러 번 중첩되는 복잡한 Promise 타입에서 유용하게 사용될 수 있다.
  * 예를 들어, 다음의 타입은 number로 추론된다.
  * **Awaited는 Promise 타입이 복잡하게 중첩되어 있을 경우, 이를 풀어서 최종적으로 Promise가 반환할 값으로 추론하는 역할을 수행**한다.
```
type IAmNumber = Awaited<Promise<Promise<Promise<number>>>>;
const num: IAmNumber = 3;
```
* 이렇듯 **Awaited 타입은 재귀적으로 중첩 Promise를 풀어나가 최종적으로 반환될 타입을 미리 추론하도록 하는데 그 의의가 있다**.
  * 이 과정에서 삼항연산자를 활용하는 컨디셔널 타입과 infer 키워드의 조합은 매우 즁요하다.
  * 특히, **infer 키워드의 경우 단순한 추론 뿐만 아니라 제네릭 타입 안에서 사용할 수 있는 새로운 타입 변수를 만들어내는 주요한 역할을 맡을 수 있다**.