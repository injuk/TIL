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

## 2022-08-23 Tue
### 큰 타입과 작은 타입 구분하기
* **타입은 일종의 집합으로 볼 수 있으며, 이에 미루어 보았을 때 `type First = string | number`가 `type Second = string` 보다 큰 타입**이다.
  * **유사한 논리로 any 타입은 전체 집합이며, never 타입은 공집합으로 이해**할 수 있다.
* 이러한 **집합론적 관점에서, 작은 타입은 항상 큰 타입에 대입할 수 있지만 그 역은 불가능**하다.

### 더 작은 객체 타입
* 언뜻 다음과 같은 예시에서, 더 많은 속성을 갖는 Third가 가장 큰 타입처럼 보일 수 있다.
  * 그러나 **TS에서, 객체 타입 간의 크기 비교는 항상 더 상세한 정보를 갖는 객체가 작은 타입이 된다는 점을 기억**해야 한다.
  * 따라서, **가장 상세한 정보를 갖는 타입 별칭인 Third가 가장 작은 타입**이 된다.
```
type First = { name: string; };
type Second = { age: number; };
type Third = { name: string; age: number; };
```
* **언제나 좁은 타입은 더 구체적인 타입에 해당하며, 더 큰 타입에 대입이 가능**하다.

### 잉여 속성 검사
* 다음과 같은 코드에서, 더 작은 타입을 더 큰 타입에 대입하지만 대입에 실패하는 것을 확인할 수 있다.
  * 이는 객체 리터럴을 그대로 사용하는 경우에 잉여 속성 검사가 수행되기 때문으로, 이를 우회하기 위해서는 객체 리터럴을 별도의 변수로 우선 초기화할 수 있다.
  * 이는 **타입의 넓고 좁음을 검사하는 데에 더해 잉여 속성을 추가로 검사하기 때문에 발생하는 에러**로 볼 수 있다.
```
type First = { name: string; };
type Second = { age: number; };
type Third = First & Second; // Third 타입은 { name: 'ingnoh', age: 3, isMale: true } 보다 분명히 큰 타입에 해당한다. 

const enriched = { name: 'ingnoh', age: 3, isMale: true };
const third: Third = enriched;
// 아래의 코드와 같이 객체 리터럴을 직접 초기화하는 경우, 잉여 속성 검사의 대상이 되어 사용이 불가능해진다.
// const third: Third = { name: 'ingnoh', age: 3, isMale: true };
```