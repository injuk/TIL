# AllInOne
## 2022-08-24 Wed
### 타입 가드
* 다음과 같이 유니온 타입을 받아들일 수 있는 함수의 경우, 다짜고짜 하나의 타입에서만 사용 가능한 프로퍼티에 접근할 수는 없다.
  * 이는 **모든 가능성을 고려하는 TS의 특징에서 기인하는 것으로, param이 number인 경우 정상 동작하지 않기 때문**이다.
```
function test(param: number | string) {
    return param.split(' '); // 에러, number.split은 불가능하다!
}
```
* 이를 정상 동작시키기 위해서는 TS에게 param의 속성이 어떻게 결정되는지 정확히 명시할 필요가 있으며, 다음과 같은 방식을 고려할 수 있다.
  1. `(param as string).split(' ')` 사용하기: 가장 쉬운 방법이나, 휴먼 에러가 발생하기 쉬워 위험성 역시 높다.
     * 사실상 **unknown으로 추론되었거나 잘 못 추론된 변수를 명확히 타이핑하려는 경우가 아니라면, `as`는 사용을 지양하는 것이 바람직**하다.
  2. 타입 가드 활용하기
* **타입 가드란, 다음과 같이 분기 처리를 적절히 활용하여 임의의 코드 상에서의 타입을 명확히 하는 방식을** 의미한다. 
```
function test(param: number | string) {
    if(typeof param === 'string') {
        // param의 타입이 문자열인 경우, 안심하고 String.prototype의 프로퍼티에 접근할 수 있다.
        return param.split(' ');
    } else {
        // 해당 함수는 유니온 타입을 매개 변수로 받으므로, param의 타입이 문자열이 아니라면 반드시 숫자가 된다.
        return param.toFixed(2);
    }
}
```
* **상술한 코드는 `typeof` 키워드를 활용하여 원시 타입을 분류하는 타입 가드이며, 타입 가드 기법은 여러 방식이 존재**한다.
  1. 원시 타입의 경우, `typeof` 키워드를 활용하여 분류하기
  2. 유니온 타입에 배열이 포함된 경우, `Array.isArray()`를 활용하여 분류하기
  3. 임의의 클래스의 인스턴스를 분류하는 경우, `instanceof` 키워드를 활용하여 분류하기 
     * 이 경우, **A 클래스 또는 B 클래스를 매개 변수로 받아들일 수 있는 함수는 `param: A | B` 형태로 클래스의 이름을 통해 유니온 타입을 작성**한다.
  4. 상수로 타이핑된 속성을 갖는 객체를 의미하는 타입 별칭의 경우, 각 객체가 유일하게 갖는 프로퍼티의 값을 확인하여 분류하기
* 특히 마지막 방법의 경우, 다음과 같은 타입 가드를 의미한다.
```
// 아래의 세 타입 별칭은 공통된 속성인 type을 갖지만, 각각 상수로 타이핑되어 유일한 값을 갖는다. 
type Ab = { type: 'ab', name: string };
type Cd = { type: 'cd', age: number };
type Ef = { type: 'ef', isBird: boolean };

function process(param: Ab | Cd | Ef) {
    if(param.type === 'ab') {
        console.log(param.name);
    } else if(param.type === 'cd') {
        console.log(param.age);
    } else {
        console.log(param.isBird);
    }
}
```
* 이렇듯 **여러 객체 간의 타입 검사가 필요한 경우, 가능한 한 각각의 객체를 구분할 수 있도록 객체의 타입을 설계**할 수 있어야 한다.
  * 예를 들어, 상술한 세 타입 별칭은 상수로 타이핑된 type 속성 외에도 서로 다른 속성인 name, age, isBird를 갖는다.
  * **이 점을 활용하여 `in` 연산자를 활용하는 다음과 같은 방식의 타입 가드를 적용할 수도 있다**.
  * `in` 연산자 역시 유효한 JS의 문법에 해당하며, TS 상에서는 타입 검사를 위해 `in` 연산자를 자주 사용하게 된다.
```
type Ab = { type: 'ab', name: string };
type Cd = { type: 'cd', age: number };
type Ef = { type: 'ef', isBird: boolean };

function process(param: Ab | Cd | Ef) {
    // in 연산자를 활용하여 객체 내부에 임의의 속성이 존재하는지를 확인할 수 있다.
    if('name' in param) {
        console.log(param.name);
    } else if('age' in param) {
        console.log(param.age);
    } else {
        console.log(param.isBird);
    }
}
```

### 값을 활용한 객체 타입 검사와 in 연산자를 활용한 객체 타입 검사
* **둘 모두 객체들 간의 타입을 구분하기 위해 사용하는 방식이지만, 일반적으로 더 자주 사용되는 것은 값을 활용한 객체 타입 검사**이다.
* 이 때, **중요한 것은 여러 객체를 타이핑하는 경우, TS의 추론을 지원하기 위해 type과 같은 타입 추론용 속성을 추가하는 습관을 들이는 것**이다.
  * 예를 들어, 상술한 예시와 같이 type 속성을 추가하되 각 타입을 구분할 수 있도록 서로 다른 값을 할당한다.
  * 이러한 습관은 객체에 태깅, 또는 라벨링한다는 표현으로 지칭할 수도 있다.
* 반면, **사전에 타입 추론용 속성을 태깅해두지 않은 경우 각 객체의 차이를 찾아 `in` 연산자를 활용하여 구분하는 타입 가드를 적용**한다.

### 커스텀 타입 가드
* **함수의 반환 형에 `is` 연산자를 활용하여 타입을 좁히기 위한 전용 함수로서의 타입 가드를 다음과 같이 작성**할 수 있다.
  * 이는 타입을 구분해주기 위한 사용자 정의 함수에 해당한다.
```
function isAb(param: Ab | Cd | Ef): param is Ab {
    return !!(param as Ab).name;
}

function customTypeGuard(param: Ab | Cd | Ef) {
    if(isAb(param)) {
        console.log(param.name);
    }
    if('age' in param) {
        console.log(param.age);
    } 
    if('isBird' in param) {
        console.log(param.isBird);
    }
}
```
* 이렇듯 **커스텀 타입 가드는 타입 판별을 위한 방법을 함수 내에 직접 명시해야 하며, 반환형에 반드시 `is` 연산자를 작성**한다.
  * 바꿔 말해, **반환형에 `is` 연산자가 포함되어 있는 함수는 모두 커스텀 타입 가드에 해당**한다.
  * 이렇게 작성된 **커스텀 타입 가드 함수는 유니온 타입을 사용하는 함수 내에서 타입을 좁히기 위해 if 문과 함께 활용**할 수 있다.
* 기본적으로는 typeof, instanceof, in, Array.isArray와 같은 간단한 방식을 사용하되, 이러한 방식만으로는 부족한 경우에 커스텀 타입 가드를 정의한다.
  * 반면, `is` 연산자를 활용하지 않으면 타입 추론이 불가능한 경우 역시 발생할 수 있다.
  * 예를 들어, **TS가 타입을 적절히 추론하지 못하는 경우 `is` 연산자를 활용하는 커스텀 타입 가드를 작성하여 정확한 타입을 추론하도록 지원**할 수 있다.

### readonly 키워드 사용하기
* **readonly 키워드는 다음과 같이 인터페이스에 할당할 수 있으며, 해당 키워드가 명시된 속성은 쓰기 작업이 불가능한 읽기 전용**이 된다.
  * 이는 읽기 전용 속성 개념을 도입하여 휴먼 에러에 의해 값이 바뀌는 것을 방지한다.
```
interface ValueObject {
    readonly prop: 'immutable!';
    name: string;
}

const valueObj: ValueObject = {
    prop: 'immutable!',
    name: 'hello',
};

valueObj.name = 'world';
// valueObj.prop = 'mutable!' // readonly로 설정된 속성은 그 값을 수정할 수 없다.
```

### indexed signature란?
* 예를 들어, 다음과 같이 **객체가 불특정 다수의 속성을 받아들일 수 있도록 하고 싶은 경우**가 있다.
  * 이는 즉 객체에 명확한 형태가 없으며, 애플리케이션 로직에 따라 속성이 충분히 많아질 수 있는 경우를 말한다.
```
interface BigObject {
    propA: string;
    propB: string;
    propC: string;
    propD: string;
    propE: string;
    propF: string;
    // ...생략
}
```
* **인덱스드 시그니쳐는 정확히 이러한 상황에 사용할 수 있으며, `[key: string]`과 같은 형태로 작성**한다.
  * 예를 들어 `[key: string]: string`은 객체에 어떠한 문자열 키를 받아들여도 무방하지만, 이에 대응되는 값 역시 문자열이어야 함을 의미한다. 
```
interface BigObject {
    [key: string]: string;
}
const bigObj: BigObject = {
    propA: 'value',
    propB: 'value',
    propC: 'value',
    propD: 'value',
    // ...생략
};
```

### mapped type이란?
* **객체에 사용 가능한 키를 몇 종류로 명확히 한정짓고자하는 경우, 다음과 같이 타입 별칭과 맵드 타입을 활용**할 수 있다. 
  * 그러나 **이 경우, 객체는 맵드 타입에 명시된 모든 키를 반드시 포함**해야 한다.
```
type Animals = 'dog' | 'cat' | 'bird';
const animals: { [ key in Animals ]: string } = {
    dog: 'dog', // Animals에 명시된 dog, cat, bird를 모두 키로 작성해야 한다.
    cat: 'cat',
    bird: 'bird',
};
console.log(animals);
```
* 이 때, **인터페이스에서는 `|`연산자를 사용할 수 없으므로 상술한 코드와 같은 유형의 맵드 타입을 정의하는 경우에는 반드시 타입 별칭을 사용**해야 한다.
  * 비단 **맵드 타입 뿐만 아니라, 파이프 연산자 또는 앰퍼샌드 연산자를 활용해야하는 경우에는 반드시 타입 별칭을 사용**해야 한다.
* **TS에서 가장 중요한 것은 가능한 한 정확한 타이핑을 적용하는 것이므로, TS는 이렇듯 상세한 타이핑 기법을 여럿 제공**한다.

## 2022-08-25 Thu
### 클래스 활용하기 
* 기본적으로 TS에서의 클래스는 다음과 같은 형태로 작성할 수 있다.
  * 생성자에는 기본 값을 정의할 수 있으며, 이 경우 생성자에 기본 값을 전달하지 않아도 무방하다.
```
class Cat {
    name: string;
    age: number;

    constructor(name: string, age: number = 3) {
        this.name = name;
        this.age = age;
    }
}
const cat = new Cat('cat!');
console.log(cat);
```

### 인스턴스 타입과 클래스 타입
* **클래스 역시 타입 별칭으로 사용될 수 있으나 이는 일반적으로 해당 클래스의 인스턴스**를 가리킨다.
  * **반면, `typeof` 키워드와 클래스의 이름을 혼용할 경우 이는 해당 클래스 자체**를 가리킨다.
```
class Template {}
// Template 자체는 해당 클래스의 인스턴스를 가리키는 타입으로 사용된다.
const t: Template = new Template();
// typeof Template은 해당 클래스 자체를 가라키는 타입으로 사용된다.
const typeofT: typeof Template = Template;
```

### 클래스의 private 제어자
* JS의 경우, `#name`과 같이 #을 붙여 해당 속성이 private함을 나타낸다.
  * 반면, **TS의 경우 `private` 키워드 자체를 사용하여 해당 속성이 private함을 나타낸다**.
```
class VeryPrivate {
    #jsPrivate = 'jsPrivate';
    private tsPrivate = 'tsPrivate';
}
```
* 이 때, **두 방식 중에서는 TS 방식의 접근 제어 방식이 권장**된다.
  * 이는 **TS가 private 외에도 protected 등의 접근 제어자를 제공하지만, JS의 `#name`과 같은 private 변수는 이를 구분할 수 없기 때문**이다.
  * 당연하게도 TS의 private 변수는 JS에서는 public하게 노출되며, 별도로 `#` 키워드를 할당하지는 않는다.

### TS에서의 접근 제어자
* **TS 역시 다른 객체지향 언어들과 마찬가지로 private, protected, public 접근 제어자를 제공하며, 기본으로 적용되는 것은 public에 해당**한다.
  * 때문에 public한 필드는 굳이 public 키워드를 명시할 필요가 없다.
  * **이는 어디까지나 TS의 기능이지 JS의 기능이 아니므로, 한 번 JS 코드로 변환된 후에는 모두 public하게 접근이 가능**해진다.
* public한 필드는 외부에서 자유로이 접근이 가능한 반면, private한 변수는 외부로부터의 접근이 불가능하여 클래스 내부에서만 사용이 가능하다.
  * 반면, protected 제어자는 클래스 내부와 해당 클래스를 상속 받은 자식 클래스에서만 접근이 가능하다.
* 또한, 각각의 접근 제어자는 `readonly` 키워드와 조합하여 읽기 전용 필드로 사용할 수도 있다.
```
class IHaveProtectedProps {
    // 부모 클래스에서만 변수를 정의한다.
    protected name = 'ingnoh';
    // 접근 제어자와 readonly 키워드를 조합하여 사용할 수도 있다.
    private readonly age = 3;
    public readonly skills = [ 'skillA', 'skillB' ];
}
class IAmChildClass extends IHaveProtectedProps {
    constructor() {
        super();
        // 아무런 변수를 정의하지 않았으나, 부모의 protected 필드에는 접근이 가능하다.
        console.log(this.name);
        
        // 부모의 private 변수에는 접근할 수 없다.
        // console.log(this.age);
        
        // 부모의 public 변수에는 접근할 수 있다.
        console.log(this.skills);
    }
}
const child: IAmChildClass = new IAmChildClass();
```

### 인터페이스의 구현
* **TS의 경우, 인터페이스 개념이 존재하지 않는 JS와 달리 `implements` 키워드를 활용하여 클래스가 인터페이스를 구현**하도록 할 수 있다.
  * 이 경우, 구현 대상 인터페이스가 갖는 모든 속성을 해당 클래스가 반드시 갖도록 코드를 작성해야 한다.
```
interface Dev {
    name: string;
    skills: string[];
}
class Developer implements Dev {
    name: string;
    skills: string[];

    constructor(name: string, skills: string[] = []) {
        this.name = name;
        this.skills = skills;
    }
}
const junior: Developer = new Developer('ingnoh', [ 'JS', 'TS' ]);
console.log(junior);
```
* JS의 개념이 아닌 인터페이스는 코드 변환시 제거되지만, 그럼에도 개발 단게에서 클래스가 인터페이스를 적절히 준수하는지 컴파일 시점에 확인할 수 있다.
  * 즉, **클래스의 형태를 인터페이스로 통제할 수 있으므로 보다 객체지향적인 개발이 가능**해진다.

### 실무에서의 인터페이스
* 인터페이스로 가능한 것은 사실 인터페이스 없이 클래스 내부적으로 모두 해결할 수 있으므로, 적어도 TS의 세계에서는 굳이 인터페이스를 사용하지 않아도 무방하다.
* 그러나 **OOP 개발자로서 객체지향의 원칙 중 추상을 강조하는 의존성 역전 원칙을 준수하고자 하는 경우, 인터페이스를 적극적으로 활용**할 수 있다.
  * 다만 **JS에서는 인터페이스가 제거되는 반면, 클래스는 남는다는 점에서 큰 차이점이 존재**한다.
* **클래스는 순수한 JS 코드 상에서도 적절히 활용이 가능하므로, JS에 반드시 코드가 남겨져야 하는 경우에는 클래스를 사용하는 것이 바람직**하다.

### 추상 클래스 정의하기
* **TS는 다른 객체지향 언어와 유사한 추상 클래스 개념을 지원하며, 자식 클래스가 이를 상속하여 추상 메소드를 구현하는 식으로 사용**할 수 있다.
  * 추상 클래스는 인스턴스화할 수 없으며, 반드시 추상 클래스를 상속받는 자식 클래스가 구체 클래스로 구현된 후에야 사용이 가능하다.
    * 물론 추상 메소드 외에도 이미 구현된 구체 메소드를 가질 수도 있다.
  * 이 때, 추상 메소드는 `private` 접근 제어자를 제외한 모든 접근 제어자를 사용할 수 있다.
  * 또한, **추상 메소드를 구현하는 경우 추상 메소드보다 더 허용적인 접근 제어자를 적용할 수 있다**.
* **추상 클래스 역시 다른 클래스와 마찬가지로 추상 클래스를 상속받아 기능을 확장할 수 있다**.
```
abstract class IAmAbstract {
    // private 접근 제어자는 사용할 수 없다.
    // private abstract hello(): void;
    
    // protected 또는 public 접근 제어자를 활용할 수 있다.
    protected abstract hello(): void;
    // public abstract hello(): void;
}
// 추상 클래스는 인스턴스화할 수 없다.
// const abstract: IAmAbstract = new IAmAbstract();

// 추상 클래스는 추상 클래스를 상속받아 기능을 확장할 수 있다.
abstract class MeToAbstract extends IAmAbstract {
    public abstract bye(): void;
    
    // 추상 클래스는 구체 메소드를 가질 수 있다.
    protected info() {
        console.log('info!');
    }
}

class IAmConcrete extends MeToAbstract {
    // 추상 클래스에 적용된 protected 접근 제어자보다 허용적인 public 접근 제어자는 사용할 수 있다.
    public hello(): void {
        console.log('hello!');
    }

    public bye(): void {
        this.info();
        console.log('bye!');
    }
}
const concrete: IAmConcrete = new IAmConcrete();
concrete.hello();
concrete.bye();
```

### 인터페이스, 추상 클래스, 접근 제어자
* **TS는 객체지향 언어가 제공하는 인터페이스와 추상 클래스, 그리고 접근 제어자 기능을 제공하므로 순수 JS의 경우보다 OOP를 쉽게 진행**할 수 있다.