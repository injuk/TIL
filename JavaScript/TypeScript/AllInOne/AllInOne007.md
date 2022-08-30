# AllInOne
## 2022-08-29 Mon
### 유틸리티 타입이란?
* **유틸리티 타입은 TS에 의해 사전 정의된 타입이며, 필요할 경우 개발자가 가져다 사용할 수 있는 특수한 타입들에 해당**한다.
  * 또한 라이브러리 타입과 마찬가지로, 유틸리티 타입 역시 개발자가 필요한 경우 유사한 타입을 흉내낼 수 있어야 한다. 
* 유틸리티 타입은 기존에 정의해둔 타입 또는 인터페이스를 재사용하고자 하지만, 몇몇 속성이 제거되는 등 완전히 같지는 않은 경우에 사용할 수 있다.
  * 즉, **굳이 새로운 타이핑을 하고싶지 않은 경우에 유용하게 활용**할 수 있다.

### Partial 타입이란?
* Partial 타입은 기존의 인터페이스에 정의된 속성 몇몇을 누락한 새로운 타입을 적용하고자하는 경우에 다음과 같이 사용할 수 있다.
  * 이 때, Partial 타입은 `Partial<타입>` 형태로 사용하며 기존 인터페이스에 포함된 모든 속성을 옵셔널한 상태로 만든다.
  * 즉, 아래의 코드는 Partial 타입을 적용하는 것으로 인해 마치 `{ type?: string; name?: string, age?: number }` 처럼 취급된다.
```
interface Animal {
    type: string;
    name: string;
    age: number;
}
const bird: Animal = { type: 'bird', name: 'bird!', age: 3 };
console.log(bird);

// 새로운 타입을 정의할 필요 없이, Partial 유틸리티 타입을 활용한다.
const withoutName: Partial<Animal> = { type: 'bird', age: 0 };
console.log(withoutName);
```
* 그러나 **Partial 타입은 모든 속성을 옵셔널한 상태로 강제 전환하므로, 사실상 빈 객체마저 허용하게 되어 썩 좋은 타입이라고는 볼 수 없다**.
  * 이러한 단점으로 인해 Partial 타입은 사용을 지양하는 것이 바람직하다.

### Partial 타입 직접 정의하기
* Partial 타입은 다음과 같은 역할을 수행할 수 있어야 한다.
  1. 어떠한 객체가 들어오든, 해당 객체의 모든 키 값을 순회한다.
  2. 모든 키에 대응되는 값은 타입을 유지하며, 이 때 옵셔널할 수 있도록 변경한다.
* 모든 객체를 사용할 수 있다는 점에서 미루어 보아 제네릭을 통한 일반화가 적용되어야 하며, 예를 들어 다음과 같이 작성할 수 있다.
  * 이 때, **`[key in keyof T]`과 같은 맵드 타입을 적용하여 어떠한 객체가 전달되든지 객체의 키를 사용하도록 구현**할 수 있다.
  * 맵드 타입은 객체 또는 인터페이스에 대해 적용할 수 있으며, 어떠한 경우든 맵드 타입 대상의 키를 전부 순회할 수 있다.
```
type MyPartial<T> = {
    [key in keyof T]?: T[key];
};
interface Animal {
    type: string;
    name: string;
    age: number;
}

const bird: Animal = { type: 'bird', name: 'bird!', age: 3 };
console.log(bird);
const withoutName: MyPartial<Animal> = { type: 'bird', age: 0 };
console.log(withoutName);
```

### Pick 타입이란?
* 빈 객체마저 받아들일 수 있는 Partial 타입의 단점으로 인해서, 대체제로서 사용할 수 있는 유틸리티 타입은 Pick과 Omit이 있다.
  * 이 때, Pick은 `Pick<MyPick, 'name' | 'age'>`와 같은 형태로 사용하며 Omit은 `Omit<MyOmit, 'name'>` 형태로 사용한다.
  * 이 때 **Pick은 명시된 타입만을 가져오는 새로운 타입으로 취급하는 반면, Omit은 명시된 타입만을 제거한 새로운 타입으로 취급**한다.
* Pick의 경우를 예로 들어, 아래와 같이 작성된 Pick 타입은 원본인 Animal 인터페이스로부터 name과 age 속성만을 갖는 새로운 타입으로 취급된다.
  * 때문에 모든 속성을 강제로 옵셔널로 변경하던 Partial보다는 더 안정적으로 사용할 수 있다는 장점이 존재한다.
```
interface Animal {
    type: string;
    name: string;
    age: number;
}

const bird: Animal = { type: 'bird', name: 'bird!', age: 3 };
console.log(bird);
const withoutName: Pick<Animal, 'name' | 'age'> = { name: 'bird', age: 3 };
console.log(withoutName);
```

### Pick 타입 직접 정의하기
* Pick 타입은 다음과 같은 역할을 수행할 수 있어야 한다.
  1. 원본 인터페이스를 첫 번째 타입 변수로 받아들일 수 있어야 한다.
  2. 이 중 가져올 속성의 이름만을 갖는 유니온 타입을 두 번째 타입 변수로 받아들일 수 있어야 한다.
  3. 두 타입 변수 간에 관계가 주어져야 한다.
* 상술한 조건을 토대로 Pick 타입은 다음과 같이 정의할 수 있다.
```
type MyPick<T, K extends keyof T> = {
    [key in K]: T[key];
};
```
* 이렇듯 **제네릭 타입을 정의하는 경우, 반드시 `K extends keyof T`와 같은 제네릭 제한 조건에 우선 집중하는 것이 바람직**하다.

### Exclude 타입이란?
* **Exclude 타입을 적용하는 경우, 제네릭의 첫 번째 타입 변수의 속성으로부터 두 번째 타입 변수의 속성을 제거**한다.
  * 아래의 예시에서, Thing 타입은 'some'이 제거되어 'thing'만을 갖게 된다.
```
type Something = 'some' | 'thing';
type Thing = Exclude<Something, 'some'>;
const thing: Thing = 'thing';
```

### 복합 유틸리티 타입이란?
* 유틸리티 타입은 조합되어 새로운 유틸리티 타입을 구현할 수 있으며, 예를 들어 Omit은 다음과 같이 Pick과 Exclude를 조합하여 구현된다. 
  * 이 때, **두 번째 제네릭 변수로 아무런 값이나 전달되는 것을 방지하기 위해 `K extends keyof any`와 같이 키만을 받아들일 수 있도록 제한**한다.
```
type MyOmit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>
```
* **`K extends keyof any`와 같은 문장은 `string | number | symbol`로 추론되며, 이는 JS 상에서 객체의 키로 사용될 수 있는 모든 타입이다**.

### 유틸리티 타입을 활용하여 학습하기
* TS 상에 사전 정의된 여러 유틸리티 타입들은 제네릭과 복합 유틸리티 타입을 적절히 활용하므로, 해당 코드를 분석하는 것만으로도 큰 도움을 받을 수 있다.
  * 특히, **`lib.es5.d.ts`를 분석하는 과정에서 TS 상에서 활용할 수 있는 여러 제네릭의 활용법을 익힐 수 있다**.

### Exclude, Extract와 삼항연산자
* 두 유틸리티 타입은 다음과 같이 삼항연산자를 통해 구현되며, 반환 값이 서로 반대임을 확인할 수 있다.
  * 이 때, 두 유틸리티 타입은 모두 임의의 키 값을 선택하기 위해 사용된다.
```
type Exclude<T, U> = T extends U ? never : T;
type Extract<T, U> = T extends U ? T : never;

type MyKeys = 'name' | 'age' | 'type';
type Excluded = Exclude<MyKeys, 'name'>; // Excluded는 'age' | 'type' 으로 추론된다.
type Extracted = Extract<MyKeys, 'name'>; // Extracted는 'name'으로 추론된다.
```
* 상술한 두 유틸리티 타입은 우선 MyKeys에 정의된 name, age, type을 각각 체크한다.
  * 이 때, 각각의 name, age, type이 두 번째 제네릭 타입으로 전달된 name을 확장하는지 확인한다.
  * **결과는 삼항연산자로 체크하되, 결과에 따라 해당 값을 남기는 경우 제네릭 타입 변수 T를 명시하고 버리는 경우에는 never를 명시**한다.
* 이렇듯 **TS에서는 새로운 타입을 정의할 때 타입 정의문에 삼항연산자를 사용할 수 있다**.

## 2022-08-30 Tue
### Required 타입이란?
* 임의의 인터페이스가 모든 타입을 옵셔널하게 갖는 경우, 때때로 반드시 모든 타입을 전달받아야 하는 경우가 있을 수 있다.
  * 예를 들어, **모든 타입을 옵셔널로 갖는 인터페이스로부터 옵셔널 연산자를 제거하되 새로운 타입을 정의하고 싶지는 않은 경우에 활용**할 수 있다.
  * Required 타입은 정확히 이 경우에 사용하며, 이를 위해 다음과 같은 코드를 작성할 수 있다.
```
const requiredAnimal: Required<OptionalAnimal> = { name: 'hello', age: 3, type: 'bird' };
```

### Required 타입 직접 정의하기
* Required 타입을 직접 구현하는 경우, 앞서 사용하지 않은 `-?` 연산자가 다음과 같이 적용된다.
```
// -? 연산자가 적용되었다.
type MyRequired<T> = {
    [key in keyof T]-?: T[key];
};

const requiredAnimal: MyRequired<OptionalAnimal> = { name: 'hello', age: 3, type: 'bird' };
```
* **`-?` 연산자는 모든 옵셔널을 제거하기 위해 사용**하며, 이는 모든 속성을 필수로 변경하는 Required의 용도에 맞는다.
  * 반면, 옵셔널을 추가하는 `+?` 연산자 역시 사용이 가능하지만 이는 일반적인 `?` 연산자와 같아 사용되지 않는다.
  * **`[key in keyof T]`는 옵셔널 적용 여부까지 그대로 가져오므로 `-?` 연산자를 사용하지 않는 경우에는 옵셔널 속성을 필수값으로 변경할 수 없다**.

### Readonly 타입이란?
* **Readonly는 임의의 인터페이스에 할당된 모든 속성에 대해 readonly를 적용하여 수정을 방지하고 싶은 경우에 사용**할 수 있다.
```
const mutable: Animal = { type: 'bird', name: 'bird!', age: 3 };
mutable.name = 'bird?';

const immutable: Readonly<Animal> = { type: 'bird', name: 'bird!', age: 3 };
// 모든 속성에 대해 readonly가 적용되므로 수정할 수 없어진다.
// immutable.name = 'bird?'; 
```
* 이 때, Readonly 타입은 다음과 같이 간단하게 정의할 수 있다.
```
type MyReadOnly<T> = {
    readonly [key in keyof T]: T[key];
}

const immutable: MyReadOnly<Animal> = { type: 'bird', name: 'bird!', age: 3 };
// 모든 속성에 대해 readonly가 적용되므로 수정할 수 없어진다.
// immutable.name = 'bird?';
```
* 반면, **오히려 readonly 속성을 제거하는 새로운 타입으로서 취급하고자 하는 경우에는 역시 `-readonly`를 명시**할 수 있다.
```
interface ImmutableAnimal {
    readonly type?: string;
    readonly name?: string;
    readonly age?: number;
}
const immutableAnimal: ImmutableAnimal = { type: 'bird', name: 'bird!', age: 3 };
// readonly 속성은 수정할 수 없다.
// immutableAnimal.name = 'immutable';

type MyMutableAnimal<T> = {
    -readonly [key in keyof T]: T[key];
};
const mutableAnimal: MyMutableAnimal<ImmutableAnimal> = { type: 'bird', name: 'bird!', age: 3 };
// 모든 readonly를 제거하였으므로 수정이 가능하다.
mutableAnimal.name = 'mutable!';
```
* 이렇듯 **`-`와 같은 modifier는 이미 존재하는 readonly 또는 옵셔널을 제거하기 위해 유용하게 사용**할 수 있다.

### Record 타입이란?
* 일반적으로, 무엇이든 받아들일 수 있는 객체 타입은 다음과 같이 정의된다.
```
interface Hello {
    [key: string]: number;
}
const hello: Hello = { a: 1, b: 2, c: 3, d: 4 };
```
* 반면, **이는 Record 유틸리티 타입을 통해 완벽히 같은 기능을 구현**할 수 있다.
```
const recordedHello: Record<string, number> = { a: 1, b: 2, c: 3, d: 4 };
```

### Record 타입 직접 정의하기
* Record 타입은 다음과 같은 방식으로 정의할 수 있다.
  * **이 경우에도 첫 번째 제네릭 타입 변수에는 객체의 키로 사용될 수 있는 값만이 포함되어야 하므로 `K extends keyof any`를 명시**한다.
```
type MyRecord<K extends keyof any, V> = {
    [key in K]: V;
};

const recordedHello: MyRecord<string, number> = { a: 1, b: 2, c: 3, d: 4 };
```

### NonNullable 타입이란?
* `string | null | undefined | number`와 같이 **null과 undefined를 포함하는 유니온 타입은 NonNullable 타입을 통해 이를 제거**할 수 있다.
```
type Nullable = string | null | undefined | number;
// Nullable 타입은 null과 undefined를 포함하는 유니온 타입이므로, 이를 할당할 수 있다.
const iAmNull: Nullable = null;
const iAmUndefined: Nullable = undefined;

type NotNull = NonNullable<Nullable>;
// 반면, NonNullable이 명시된 타입은 null과 undefined 타입을 할당할 수 없게 된다.
// const iAmNotNull: NotNull = null;
// const iAmNotUndefined: NotNull = undefined;
const iAmNumber: NotNull = 42;
```

### NonNullable 타입 직접 정의하기
* **NonNullable 타입은 앞선 Extract 또는 Exclude 타입과 같이 삼항연산자를 활용하여 다음과 같이 구현**할 수 있다.
```
type MyNonNullable<T> = T extends null | undefined ? never : T;

type MyNotNull = MyNonNullable<Nullable>;
const iAmNotNull2: MyNotNull = null;
const iAmNotUndefined2: MyNotNull = undefined;
const iAmNumber2: MyNotNull = 42;
```
* 이렇듯 **유틸리티 타입에는 오로지 키에만 적용되는 타입들이 있는 반면, 객체 자체에만 적용되는 타입들도 존재하므로 이를 구분할 수 있어야 한다**.
  * 예를 들어, **상술한 NonNullable과 Extract 및 Exclude는 키에만 적용되는 유틸리티 타입에 해당**한다.

### Parameters 타입이란?
* **해당 타입은 함수 내부에서 사용되며, 함수에 전달된 매개 변수들의 타입을 튜플로 가져오고 싶을 때 사용할 수 있다**.
  * 이 때, Parameters의 제네릭 타입 변수로는 함수의 타입을 전달받아야 하므로 반드시 `<typeof 함수이름>`과 같은 형태로 작성한다.
  * **반환받은 튜플은 인덱스를 갖고 있으므로, 서로 다른 타입 간에서는 인덱스를 활용한 타입 접근 역시 가능**하다.
```
function complicated(paramA: string, paramB: number, paramC: boolean) {}
type Params = Parameters<typeof complicated>; // [string, number, boolean]
// 타입으로 구성된 튜플의 인덱스를 통해 임의의 타입에 접근할 수 있다. 
type StringParam = Params[0]; // string
```

### Parameters 타입 직접 정의하기
* Parameters 타입은 제네릭 타입 변수에 함수만을 전달받을 수 있으므로, 다음과 같이 `(...args: any) => any`를 사용한다.
  * 이 때, **함수의 매개 변수 목록에 접근하기 위해서는 새로운 키워드인 `infer`가 사용**된다.
```
type MyParameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;
```
* 상술한 코드를 부분 별로 분석한 결과는 다음과 같다.
  1. `<T extends (...args: any) => any>`: 제네릭 타입 변수에 전달될 수 있는 타입을 함수로 제한한다.
  2. `T extends (...args: infer P)`: **함수에 전달된 매개 변수 목록을 순서대로 순회하며 각각의 타입을 추론**한다.
  3. `(...args: infer P) => any ? P : never`: **추론에 성공한 경우에는 해당 타입을 그대로 사용하고, 추론에 실패한 경우에는 결과를 버린다**.

### infer 키워드와 ReturnType 유틸리티 타입
* **infer 키워드는 오로지 extends 문장에서만 사용이 가능하며, TS에 의해 알아서 추론되도록 유도하는 키워드**이다.
  * 상술한 코드의 경우, `T extends (...args: infer A) => any`는 각각 추론 조건이 참인 경우에 추론된 타입 P를 반환하도록 한다. 
* 앞선 MyParameters의 예제와 infer 키워드를 적절히 조합하는 경우, 함수의 반환 타입만을 가져오는 유틸리티 타입을 직접 정의할 수도 있다.
  * **이 경우, 함수의 반환형은 any가 아닌 추론된 타입 `infer P`가 되어야 한다**.
  * **실제로도 이러한 역할을 수행하는 `ReturnType`유틸리티 타입이 존재**한다.
```
// 아래의 타입은 실제 ReturnType 유틸리티 타입의 구현 방식과 유사하다.
type MyReturns<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : never;

function returnsComplicated(): { a: string, b: number, c: boolean } {
    return { a: 'a', b: 1, c: false };
}
type Returns = MyReturns<typeof returnsComplicated>;
const returns: Returns = { a: 'string', b: 42, c: true };
```
* 이렇듯 **TS에서는 적절한 유틸리티 타입을 활용하여 임의의 함수로부터 매개 변수 목록의 타입, 또는 반환형 타입을 손쉽게 가져올 수 있다**.
  * 따라서 **용도에 맞는 유틸리티 타입을 익히는 것은 매우 중요하며, 이는 불필요한 타입 하드 코딩을 크게 줄여주는 결과를 낳는다**.

### ConstructorParameters 타입
* ConstructorParameters 타입은 생성자의 매개 변수 타입 목록을 튜플로 반환하며, 다음과 같이 구현된다.
```
type ConstructorParameters<T extends abstract new (...args: any) => any> = T extends abstract new (...args: infer P) => any ? P : never;
```
* 이 때, **`new (...args: any) => any`는 그 자체로 제네릭 타입 변수에 전달될 수 있는 것을 클래스 생성자로 제한**한다.
  * 또한, **클래스 생성자 타입은 `ConstructorParameters<typeof MyClass>`와 같이 typeof 키워드를 활용하여 전달**할 수 있다.

### 유틸리티 타입의 결론
* **유틸리티 타입을 통해 TS 개발의 생산성을 높일 수 있으며, 각각의 타입 구현을 이해하는 것으로 타입 및 매개 변수, 반환형 타입을 쉽게 가져올 수 있다**.
  * 또한, `infer` 키워드를 통해 매개 변수와 반환형의 타입을 추론할 수도 있다.