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