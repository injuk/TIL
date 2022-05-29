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