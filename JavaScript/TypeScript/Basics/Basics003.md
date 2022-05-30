# Basics
## 2022-05-30 Mon

### any 타입과 void 타입
* any 타입은 string, number, Array, 나아가 다른 수 많은 타입들을 총칭하는 타입이다.
  * 때문에 any는 실행 시점에 타입을 구분하여 할당하는 JS의 본질과 유사한 타입이다.
  * **JS 프로젝트에 TS를 입혀나가는 초기 과정인 경우, 우선 모든 타입을 any로 설정해둔 후에 차근차근 구체화해나가는 방식이 권장**된다.
  * **tsconfig.json의 noImplicitAny는 최소한 any 타입을 명시하는 것을 권장하며, 추론되는 암시적인 any 타입을 에러로 판단**한다.
* **void 타입은 함수의 반환값이 없음을 명시적으로 지정하기 위한 타입**이다.

### type과 interface
* 동물 배열에 대한 CRUD 코드를 작성한 경우는 다음과 같다.
  * 이 때, 동물을 나타내는 객체는 이름과 나이, 별명을 갖는다.
```
// service.ts
const animals: { name: string; age: number; alias: string }[] = [];

function createAnimal(animal: { name: string; age: number; alias: string }): void {
    animals.push(animal);
}

function getAnimal(index: number): { name: string; age: number; alias: string } {
    return animals[index];
}

function listAnimals(): { name: string; age: number; alias: string }[] {
    return animals.slice(0);
}

function deleteAnimal(index: number): void {
    animals.splice(index, 1);
}
```
* 이 경우 동물 타입은 { name: string; age: number; alias: string } 형태로 명시되었으나, 다음과 같은 단점으로 인해 생산성을 크게 떨어트린다.
  * 가독성이 떨어진다: animal 객체를 다루는 모든 곳에 상술한 코드를 작성해야 한다.
  * 수정이 어렵다: animal 객체에 새로운 속성이 추가되거나, 기존 속성이 제거된 경우 모든 코드를 수정해야 한다.
* 이 경우, type 또는 interface를 활용하여 다음과 같이 수정할 수 있다.
```
// type Animal = { name: string; age: number; alias: string };
interface Animal { name: string; age: number; alias: string };
type Animals = Animal[]

const animals: Animals = [];

function createAnimal(animal: Animal): void {
    animals.push(animal);
}

function getAnimal(index: number): Animal {
    return animals[index];
}

function listAnimals(): Animals {
    return animals.slice(0);
}

function deleteAnimal(index: number): void {
    animals.splice(index, 1);
}
```

### 인터페이스란?
* TS에서의 인터페이스는 일반적으로 다음과 같이 새로운 타입의 형태로 정의된다.
  * 아래의 예시는 변수에 인터페이스를 적용한 경우에 해당한다.
```
interface Animal {
    name: string;
    age: number;
    alias: string;
};

const bird: Animal = {
    name: 'buddy',
    age: 3,
    alias: 'twit',
    // hello: 'world', // 인터페이스에 정의되지 않은 속성은 추가할 수 없다. 
};
```
* **인터페이스는 반복되는 타입을 하나의 유형으로 정의하고, 이를 재사용할 수 있게 해주는 이점**이 있다.
  * 이로 인해 **가독성이 높아질뿐더러, 인터페이스 자체를 명시함으로써 코드에 규칙을 강제**할 수 있다.
* 반면, **인터페이스는 변수 뿐만 아니라 함수 역시 특정한 규약을 준수하도록 다음과 같이 강제할 수 있다**.
  * 예를 들어, 아래의 코드는 showAnimal 함수의 인자로 Animal 타입만을 전달할 수 있다.
  * **함수의 인자에 인터페이스를 정의한 경우, TS는 함수 호출 시 해당 인자가 파라미터에 정의한 인터페이스의 규약을 준수하는지 확인**한다.
  * 이렇듯 JS 코드 상에 엄격한 규약을 강제하는 것은 TS의 주요한 장점 중 하나이다.
```
function showAnimal(animal: Animal): void {
    console.log(animal);
}
```
* 함수의 인자에 타입을 정의하는 방식은 TS에서 매우 빈번하게 사용되며, 프론트엔드의 경우 API 호출 시 응답에 대한 데이터 구조를 정의하는 것이 대표적이다.

### 인터페이스로 함수의 구조를 정의하기
* **인터페이스는 객체의 데이터 구조 뿐만 아니라 함수의 인자와 반환형과 같은 구조를 정의하는 데에도 다음과 같이 사용**될 수 있다.
  * 이러한 방식은 직접 공용 라이브러리를 설계하거나, 팀원 간의 협업 시 공통으로 사용될 함수에 대한 규약을 정의하기 위해 사용될 수 있다.
```
interface Add {
    (number1: number, number2: number): number
}
let add: Add;
add = function(a: number, b: number): number {
    return a + b;
}
```

### 인터페이스와 인덱싱
* 기존의 객체 인터페이스는 정해진 속성 명과, 그에 대응되는 값의 타입을 정의하였다.
* TS에서는 이 뿐만 아니라 **배열의 인덱싱 방식에 대해서도 인터페이스를 정의**할 수 있다.
  * 즉, **인덱싱 방식을 정의한 인터페이스는 배열의 각 요소에 대한 접근 법인 인덱스의 타입과 요소 타입을 정의하기 위해 사용**한다.
  * 아래 코드의 경우, array는 AnimalArray이므로 인덱싱은 number로 하지만 실제 배열 값은 Animal 형태로만 초기화할 수 있다. 
```
interface AnimalArray {
    [index: number]: Animal;
}

const array: AnimalArray = [];
array[0] = bird; // 각 인덱스에 대한 접근은 number 타입을 활용한다.
// array[0] = 1; // Animal 타입이 아닌 값으로는 초기화할 수 없다.
```

### 인터페이스와 딕셔너리
* 상술한 **인덱싱 방식을 정의하는 인터페이스는 다음과 같은 딕셔너리 패턴의 구현에 유용하게 사용**될 수 있다. 
* 인덱싱을 활용하는 딕셔너리는 다음과 같은 규칙을 준수하여 작성한다.
  1. **속성 명은 injuk와 같이 임의로 정의해도 무방하지만, 속성 타입은 string, number, symbol 형태만 적용 가능**하다.
  2. 속성에 대응되는 값의 타입을 정의한다.
```
interface AnimalDictionary {
    [injuk: string]: Animal
}
const aminal: AnimalDictionary = {
    ingnoh: bird,
    // injuk: { name: 'hello' },
};
// aminal['injuk'] = 1;
```
* **딕셔너리 패턴을 적절히 사용한 경우, TS 자체의 타입 추론의 이점을 쉽게 누릴 수 있다**.
  * 예를 들어 상술한 코드의 경우, Object.keys(aminal).forEach(value => console.log(value));와 같은 예시를 들 수 있다.
  * 이 경우, **TS에 의해 value 변수는 자동적으로 인터페이스를 기반으로 string으로 추론**된다.
* 이렇듯 **인터페이스를 활용한 딕셔너리 패턴을 통해 객체의 인덱스 또는 키 기반 접근에 있어서 엄격한 규약 준수를 강제할 수 있다**. 
  * 상술한 예시에서, 주석 처리된 코드는 딕셔너리 패턴을 구현한 인터페이스에 정의된 타입이 아니기 때문에 컴파일 오류를 발생시키게 된다.
  * 이렇듯 **인터페이스를 활용하게 되면 타입 안정성을 손쉽게 구현**할 수 있다.

### 인터페이스 확장하기
* **인터페이스의 확장은 OOP에서의 상속 또는 JS에서의 prototype과 유사한 개념이며, 기존의 인터페이스를 상속 받아 기능을 확장할 수 있는 개념**이다.
* **아래의 코드는 Bird 인터페이스가 기존의 Animal에 정의된 name, age, alias를 확장하기 위해 extends 키워드를 활용하여 인터페이스를 상속**받는다.
  * Bird의 속성이 color 뿐이라고 해서 color만을 정의할 수는 없다.
  * **인터페이스 Bird는 Animal을 상속받았으므로, 인터페이스 확장 규칙에 따라 반드시 Animal에 정의된 속성까지 모두 정의**해야 한다.
```
interface Bird extends Animal {
    color: string;
}

const buddy: Bird = { // name, age, alias는 Bird에 명시되지 않았으나, Animal를 통해 상속된다.
    name: 'twitter',
    age: 10,
    alias: 'twit',
    color: 'blue',
};
```