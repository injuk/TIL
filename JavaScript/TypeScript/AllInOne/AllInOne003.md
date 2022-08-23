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
* 타입 별칭의 경우 확장을 위해 `&`와 같은 인터섹션을 사용하지만 인터페이스는 `extends` 키워드를 활용한다.
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

### void 타입
* **함수의 반환형이 void인 경우, 해당 함수는 undefined를 제외한 어떠한 반환 값도 가질 수 없게 된다**.
  * void 형 함수는 일반적으로 return 문을 작성하지 않거나, 명시적인 `return` 만을 작성한다.

### void 함수의 세 가지 유형
* void 형을 반환하는 함수는 크게 다음과 같이 분류할 수 있다.
  1. 반환형이 void인 일반 함수
  2. 콜백과 같이 함수의 매개 변수로서 반환형이 void로 정의된 함수
  3. 인터페이스 내부에서 반환형이 void로 정의된 메소드
* 이 때, **매개 변수와 인터페이스 메소드는 반환 값이 존재하더라도 정상적으로 동작**할 수 있다.
  * 반면, 일반 함수는 상술한 바와 같이 명시적인 return 또는 undefined를 제외하고는 무엇도 반환할 수 없다.
```
function tempA(): void {
    // return 42; // 이렇게 작성할 수 없다.
    return;
}

function tempB(cb: () => void) {}
tempB(() => { return 42; }) // 이렇게 작성할 수는 있다.

interface TempC {
    tempC: () => void;
}
const tempC: TempC = {
    tempC() {
        return 42; // 이렇게 작성할 수는 있다.
    }
}

const resultC = tempC.tempC();
console.log(resultC);
```
* 이러한 **매개 변수로서의 함수와 메소드로서 정의된 타입에서 void의 의미는 반환 값을 사용하지 않겠다는 의미에 가까우며, 반환하지 않는다는 의미가 아니다**.
  * 반면, 일반 함수의 경우 반환하지 않는다는 의미를 갖는다.
  * 즉, **함수의 반환형으로 사용된 void는 상황에 따라 반환 값이 없음과 반환 값을 사용하지 않는다는 의미로 나뉠 수 있다**.
* 반면, **undefined를 반환하는 함수의 경우 함수의 종류와 관계 없이 반드시 undefined를 반환해야하며 다른 값을 반환할 수 없다**.
  * 이는 **매개 변수로 사용된 void 형 함수가 무엇을 반환하든지 상관하지 않는 것과 대비**된다.
* 상술한 코드에서, **resultC는 실제로는 42를 반환받지만 TS는 void 형 함수의 반환 값을 무시하므로 void 형으로 추론**된다.
  * 때문에 **원칙적으로 void 형 메소드 또는 매개 변수로 사용된 함수는 값을 아무 것도 반환하지 않는 것이 바람직**하다.
  * 만약 resultC를 number로 형변환하는 경우, `const resultC = tempC.tempC() as unknown as number;`와 같이 형변환할 수 있다.

### void와 undefined 형 함수의 차이
* **void 형은 undefined에 할당할 수 없으나, undefined는 void 형에 할당**할 수 있다.
  * 즉, **void는 undefined와 다르다**.

### declare 사용하기
* **함수의 경우, 함수의 선언과 본문을 다음과 같이 따로 작성할 수 있다**.
  * 이 때, 함수의 선언부와 본문 사이에는 다른 문장이 작성되지 않아야 한다.
```
function forEach(arr: number[], cb: (e: number) => undefined): void;
function forEach() {
    // 함수의 본문을 작성할 수 있다.
}
```
* 반면, 함수의 본문을 즉시 작성하고 싶지 않은 경우 `declare` 키워드를 명시하는 것으로 다음과 같이 오류를 제거해줄 수 있다.
  * 이 경우, **declare로 선언된 함수는 JS 코드로 변환할 경우에 함께 제거**된다.
```
declare function forEach(arr: number[], cb: (e: number) => undefined): void;
```
* **declare 키워드는 함수의 선언부와 구현부의 위치가 물리적으로 다른 파일로 분리되어 있어 TS에게 해당 사실을 알리고자 하는 경우에 사용**할 수 있다.
  * 즉, **declare 키워드는 외부에서 작성된 내용에 대해 타입 정보를 선언하는 경우에 사용**한다.
  * **이를 통해 외부에 위치한 JS 코드 등을 마치 TS 상에서 미리 선언해둔 것처럼 사용이 가능**해진다.

### unknown과 any의 차이점
* **단적으로, any를 사용하고자 하는 경우에는 항상 any 대신 unknown을 사용해야 한다**.
  * any와 never는 사실상 TS에 의해 추론되는 일이 없도록 코드를 작성하는 것이 바람직하다.
* **any 타입으로 타이핑된 경우, TS는 해당 변수에 대한 모든 추적 및 검증을 더 이상 수행하지 않는다**. 
  * 이는 사실 상 해당 변수에 대한 타입 검사를 포기하는 것과 같으므로, 더 이상 TS를 사용하는 의미가 없어지게 된다.
* **반면, unknown을 사용하는 경우 개발자는 반드시 해당 변수의 명확한 타입을 결정지은 후에야 이를 사용할 수 있게 된다**.
* 즉, **any는 타이핑 자체를 포기하는 반면 unknown은 지금 당장은 타입을 모르지만 추후에 타입을 결정하겠다는 뉘앙스**를 갖는다.
  * 때문에 unknown 역시 사용을 최소화하는 것이 바람직하며, 부득이하게 사용하게된 경우에는 반드시 명확한 타입 결정 후에 사용해야 한다.