# AllInOne
## 2022-08-29 Mon
### 유틸리티 타입이란?
* **유틸리티 타입은 TS에 의해 사전 정의된 타입이며, 필요할 경우 개발자가 가져다 사용할 수 있는 특수한 타입들에 해당**한다.
  * 또한 라이브러리 타입과 마찬가지로, 유틸리티 타입 역시 개발자가 필요한 경우 유사한 타입을 흉내낼 수 있어야 한다. 
* 유틸리티 타입은 기존에 정의해둔 타입 또는 인터페이스를 재사용하고자 하지만, 몇몇 속성이 제거되는 등 완전히 같지는 않은 경우에 사용할 수 있다.
  * 즉, **굳이 새로운 타이핑을 하고싶지 않은 경우에 유용하게 활용**할 수 있다.

### Partial 타입
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