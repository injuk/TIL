# DesignPatterns
## 2022-05-16 Mon

### ECMAScript 모듈
* ESM으로도 알려진 ECMAScript 모듈은 ECMAScript 2015 명세의 일부분으로 도입되었다.
  * **ESM은 서로 다른 실행 환경에서도 적절하게 동작하는 공식 모듈 시스템을 JS에 부여**한다.
  * ESM 명세는 CommonJS나 AMD와 같은 기존 모듈 시스템의 장점을 계승하고자 하였으며, 문법은 간단하고 짜임새를 갖춘다.
  * 또한, **ESM은 비동기적으로 모듈을 로드할 수 있는 방법과 순환 종속성에 대한 지원을 제공**한다.
* ESM은 static 방식을 택한다는 점에서 CommonJS와는 큰 차이점을 갖는다.
  * 이로 인해 임포트는 모든 모듈의 최상위 레벨과 제어 흐름 구문의 바깥쪽에 정의된다.
  * 또한, **모듈의 이름을 코드 상에서 동적으로 결정할 수 없으므로 임포트 대상 모듈은 반드시 상수 문자열로 표현**해야 한다.
* 이러한 ESM의 특징은 불필요한 제약으로 보일 수도 있지만, 정적 임포트를 지원하는 것으로 다음과 같은 시나리오에 대응할 수 있게 된다.
  1. 종속성 트리의 정적 분석을 가능하게 한다. 
  2. 이로 인해 tree shaking과 같이 사용하지 않는 코드 제거 등의 코드 최적화를 지원할 수 있다.

### Node.js에서 ESM 사용하기
* **Node.js는 기본적으로 모든 js 확장자 파일이 CommonJS 문법을 사용한다고 판단**한다.
  * 때문에 js 확장자 파일에 곧장 ESM 문법을 도입하면 인터프리터는 에러를 발생시킨다.
* Node.js 인터프리터가 CommonJS 모듈 대신 ESM을 받아들일 수 있도록 하려면 다음과 같은 방식을 적용할 수 있다.
  1. 모듈 파일의 확장자를 .mjs로 정의한다.
  2. **모듈과 가장 근접한 package.json의 type 필드에 module을 기재**한다.

### named exports, named imports
* **CommonJS에서 exports와 module.exports를 제공하는 것과 달리, ESM에서는 export 키워드만을 사용하여 모듈의 기능을 내보낸다**.
* **ESM에서는 기본적으로 모든 것이 private이며, export된 개체들만 공개되어 다른 모듈로부터 접근이 가능**해진다.
```
// moduleA.js
export function hello() {
  console.log('hello world');
}

export const NAME = 'injuk';

export const info = { age: 3, isMale: true };

export class Injuk {
  constructor(name) {
    this.name = name;
  }
  sayHello() {
    console.log(`my name is ${this.name}`);
  }
}
```
* 다른 모듈로부터 임의의 개체를 임포트하고자 하는 경우에는 import 키워드를 활용한다.
  * 해당 **문법은 유연하며, 하나 이상의 개체를 한 번에 임포트하거나 다른 네임스페이스를 할당할 수도 있다**.
  * 아래 예시에서 moduleA.js의 모든 멤버를 임포트하며 moduleA 변수에 할당한다.
  * **import * 구문은 네임스페이스 임포트에 해당하며, 해당 모듈의 모든 멤버를 임포트하는 동시에 as 구문에 정의된 네임스페이스를 할당**한다.
```
// main.js
import * as moduleA from './moduleA.js'

console.log(moduleA);
console.log(moduleA.hello());
```
* **CommonJS와 달리 ESM에서는 반드시 모듈의 파일 확장자를 구체적으로 명시**해야 한다.
* 규모가 큰 모듈을 임포트하는 과정에서 모듈의 모든 기능을 임포트하지 않고, 하나 또는 몇 개의 객체만 임포트하고 싶은 경우에는 다음과 같이 작성한다.
```
import { hello, NAME } from './moduleA.js';

hello();
console.log(NAME);
```
* 그러나 해당 임포트 구문은 임포트되는 대상이 현재의 스코프로 임포트되므로 이름의 충돌 가능성이 존재한다.
  * 동일한 이름의 변수가 존재하는 경우, 인터프리터는 문법 에러를 반환하며 동작을 멈추게 된다.
  * 이러한 **이슈를 해결하기 위해 as 구문을 활용하여 임포트되는 개체의 이름을 변경**해줄 수 있다.
```
import { hello, NAME as name } from './moduleA.js';

const NAME = 'monkey';

hello();
console.log(NAME);
console.log(name);
```
* **as 키워드를 활용하는 방식은 서로 다른 모듈에서 동일한 이름을 갖는 개체의 임포트로 인해 충돌이 발생한 경우에 유용**하다.
  * 이로 인해 모듈 클라이언트는 모듈의 원래 이름을 바꿀 것인지 고려할 필요가 없다.

### default export, default import
* CommonJS에서 자주 사용되는 특성은 이름이 없는 임의의 개체를 module.exports에 할당하여 익스포트하는 것이다.
  * 이는 모듈 개발자에게 단일 책임 원칙을 권장하며, 깔끔한 하나의 인터페이스만을 공개한다는 점에서 매우 편리한 기능이다.
* ESM에서는 default export가 유사한 기능을 수행하며, export default 키워드를 사용한다.
```
// default.js
export default (message) => {
  console.log(message);
}
```
* 해당 방식의 경우 **익스포트되는 개체는 이름이 무시되며, default라는 이름에 등록**된다.
* 익스포트된 이름은 다음과 같은 방식으로 임포트할 수 있다.
  * **default export는 이름이 없는 것으로 간주되므로, ESM의 named import와는 다른 방식으로 임포트**해야 한다.
  * 임포트와 동시에 모듈 클라이언트가 정의한 이름으로 할당되며, 클라이언트 개발자는 임의의 이름을 정의할 수 있다.
```
import log from './default.js';

log('hello');
```
* **default export 방식은 내부적으로 default라는 이름으로 익스포트되는 것과 동일하게 취급**된다.
  * 그러나 **default 키워드는 변수의 이름으로 명명할 수 없으므로, 명시적으로 default 개체를 임포트할 수는 없다**.
  * default 키워드는 객체의 속성으로는 정의가 가능하지만, 변수의 이름으로 설정할 수는 없다.
```
import * as log from './default.js';
// 아래의 방식은 default 변수를 정의할 수 없으므로 불가능하다.
// import { default } from './default.js'; 

console.log(log);
log.default('hello');
```

### mixed exports
* **ESM에서는 named export와 default export를 혼합하여 사용할 수도 있다**.
```
export default function(message) {
  console.log(message);
}

export function hello(name) {
  console.log(`hello, ${name}!`);
}
```
* default export와 named export를 임포트하는 경우, 다음과 같은 방식을 활용할 수 있다.
```
// 한 번에 import할 수 있다.
// import * as mixed from './mixed.js';
//
// console.log(mixed);
// mixed.hello('ingnoh');
// mixed.default('hello');

// 또는 default export와 named export를 따로 import할 수도 있다.
import log, { hello } from './mixed.js';

log('hello');
hello('ingnoh');
```

### default export와 named export의 차이점
* 두 방식은 다음과 같은 주목할만한 차이점을 갖는다.
  1. named export는 명확하지만, default export는 모든 면에서 상대적으로 복잡하다.
     * named export는 지정된 이름을 갖기 때문에 IDE의 지원을 받기 쉽다.
     * 반면, **default export는 클라이언트 모듈마다 서로 다른 이름을 가질 수 있기에 주어진 이름에 대해 실제로 어떠한 모듈이 적용될지 추론이 어렵다**.
  2. default export는 모듈의 가장 핵심적인 하나의 기능을 연결시키는 편리한 방법이다.
     * **사용자 관점에서, 바인딩을 위한 정확한 이름을 알 필요가 없이 해당 모듈이 확실히 제공하는 기능을 쉽게 임포트**할 수 있다.
  3. **default export는 특정한 상황에서 tree shaking을 어렵게 만든다**.
     * 예를 들어, **모듈의 기능을 객체로 포함하여 default export로 제공하는 경우**가 있다.
     * 해당 객체를 임포트한 경우, 모듈 번들러는 객체 전체가 사용되는 것으로 간주하여 실제로는 사용되지 않는 코드를 제거할 수 없다.
```
// 3.의 예시
// bad.js
export default {
  hello: (name) => {
    console.log(`hello, ${name}!`);
  },
  bye: (name) => {
    console.log(`bye ${name}...`);
  },
  log: (message) => {
    console.log(message);
  }
};

// main.js
import bad from './bad.js';

bad.hello('injuk');
```
* 따라서 **명확한 하나의 기능을 내보내는 경우에는 default export를 사용하되, 그 외에는 named export 사용을 지향하는 것이 일반적으로 바람직**하다.
  * 이는 엄격히 합의된 규칙이 아니며, Node.js의 코어 모듈과 React는 모두 혼합된 방식을 사용한다.
* 두 방식 중 자신의 모듈에 어떠한 것이 최선의 방식일지, 나아가 모듈 클라이언트에게 어떠한 개발자 경험을 제공하고 싶은지 고려하여 결정할 수 있어야 한다.

### 모듈 식별자
* 모듈 식별자는 import 구문에서 로드하고자하는 모듈의 경로를 명시하기 위해 사용되는 값이다.
* 모듈 식별자는 크게 다음과 같이 분류할 수 있다.
  1. 상대적 식별자: ./logger.js 또는 ../logger.js와 같은 형식이며, 파일의 경로에 상대 경로가 사용된다.
  2. 절대 식별자: 절대 경로가 사용되며, CommonJS의 파일 모듈과는 용례가 다르다.
  3. 노출 식별자: node_modules 폴더에서 사용이 가능하도록 패키지 매니저를 통해 설치된 모듈과 Node.js 코어 모듈을 가리킨다.
     * import * as fs from 'fs'; 와 같은 형식으로 사용된다.
  4. 심층 임포트 식별자: node_modules에 위치한 패키지의 경로를 가리키는 경우를 말한다.
     * import * as logger from 'fastify/lib/logger.js'와 같은 형식으로 사용된다.
* **브라우저 환경에서는 모듈의 URL을 명시하여 직접 임포트할 수 있으나, 이는 Node.js 런타임에서는 지원되지 않는 ESM의 기능**이다.