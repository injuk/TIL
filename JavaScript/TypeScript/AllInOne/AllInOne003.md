# AllInOne
## 2022-08-22 Mon
### 타입 별칭과 인터페이스의 확장
* 예를 들어, **타입 별칭을 마치 상속과 유사하게 확장해나가 사용할 수 있으려면 다음과 같은 코드를 작성**해볼 수 있다.
```
const buddy: Bird = { exist: true, move: true, fly: true };
```
* 반면, **인터페이스는 애시당초 `extends` 키워드를 제공하며 다음과 같이 확장을 표현**할 수 있다.
  * 이 때, `extends` 키워드는 상속보다는 확장의 의미로 받아들이는 것이 바람직하다.
```
interface IThing { exist: true; }
interface IAnimal extends IThing { move: true; }
interface IBird extends IAnimal { fly: true; }

const buddyImpl: IBird = { exist: true, move: true, fly: true };
```
* **두 개념 사이의 차이는 크지 않으며, 타입 별칭은 간단하게 사용이 가능한 반면 인터페이스는 명시적으로 확장 구조를 표현할 수 있다는 장점**이 있다.
  * 때문에 간단하게 타이핑하는 경우에는 타입 별칭을 활용하고, OOP를 통해 개발을 진행하고자하는 경우에는 인터페이스를 선택할 수 있다.
* **심지어 타입 별칭이 인터페이스를 확장하거나, 그 역도 가능하므로 둘 사이의 명확한 차이를 두지 않는 것이 바람직**하다.
  * **오히려 인터페이스의 표현력과 타입 별칭의 표현력 차이에 중점을 두어야 하며, 기능적 차이에 집중할 필요는 없다**.
  * **실무에서는 객체 타입을 명시하기 위해서 일반적으로 인터페이스를 사용하며, 이는 타입 별칭보다 상대적으로 확장이 용이한 인터페이스의 특징에서 기인**한다. 
```
// 타입 별칭이 인터페이스를 확장할 수 있다.
type 이게되나 = IBird & { available: true };
const 진짜되나: 이게되나 = { exist: true, move: true, fly: true, available: true };

// 인터페이스 역시 타입 별칭을 확장할 수 있다.
interface 이것도되나 extends Bird { available: true; }
const 진짜이것도되나: 이것도되나 = { exist: true, move: true, fly: true, available: true };
```

### 타입 별칭과 인터페이스의 차이점
* **인터페이스는 동일한 이름으로 여러 번 정의할 수 있으며, 이 경우 각각의 인터페이스는 모두 하나로 합쳐진다**.
  * 반면, **타입 별칭은 동명의 타입 별칭을 여러 번 정의하는 것이 근본적으로 불가능**하다.
  * 이 때, `interface Man { mbn: true; }`와 같은 인터페이스를 여러 줄에 거쳐 여러 번 정의할 수도 있으나 반드시 동일한 타입이어야만 한다.
```
interface Man { mbn: true; }
interface Man { mcn: true; }
interface Man { mdn: false; }

// 세 번에 걸쳐 정의된 Man 인터페이스는 각 인터페이스에서 정의된 모든 속성을 포함한다.
const man: Man = { mbn: true, mcn: true, mdn: false };
```
* 이러한 **인터페이스의 특징을 활용하여 서드 파티 라이브러리의 코드에 자신이 원하는 동작을 추가하는 식으로 확장하는 응용 역시 가능**해진다.