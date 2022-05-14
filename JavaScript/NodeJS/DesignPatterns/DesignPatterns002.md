# DesignPatterns
## 2022-05-13 Fri

## 모듈 시스템
```
> 모듈은 주요 애플리케이션들을 구조화하기 위한 부품이다.
```
* 모듈을 사용할 경우, 애플리케이션은 다음과 같은 이점을 가질 수 있게 된다.
  1. **코드베이스를 개별적으로 개발 및 테스트 가능한 작은 유닛들로 분할**할 수 있다.
  2. **의도적으로 노출시키지 않은 모든 함수들과 변수들을 비공개로 유지하여 정보에 대한 은닉성을 강화**시킬 수 있다.
* 하나의 모듈 시스템을 갖는 다른 언어들과 달리, **Node.js는 다음과 같은 두 가지 모듈 시스템을 사용**한다.
  1. CommonJS
  2. ECMAScript modules

### 모듈의 필요성
* **모듈과 모듈 시스템은 별개의 것**이며, 다음과 같은 차이를 갖는다.
  1. 모듈 시스템이란, 문법이자 프로젝트에서 모듈을 정의하고 사용할 수 있도록 하는 도구이다.
  2. 모듈이란, 소프트웨어를 구성하는 실제 유닛을 말한다.
* 좋은 모듈 시스템은 소프트웨어 작성 과정에서 다음과 같은 몇 가지 요구사항을 만족시키는 데에 도움을 준다.
  1. 코드베이스를 나누어 여러 파일로 분할하는 방법을 제시한다.
     * 이를 통해 코드를 구조적으로 관리할 수 있다.
     * 덕분에 **독립적인 기능 조각을 개발 및 테스트할 수 있으며, 모듈 별 가독성은 높아진다**.
  2. 다른 프로젝트에서도 코드를 재사용할 수 있도록 한다.
     * 모듈은 다른 프로젝트에서도 유용할만한 특성을 구현할 수 있게 한다.
     * **모듈 형태로 구조화된 기능은 다른 프로젝트에서도 쉽게 재사용**할 수 있다.
  3. 은닉성을 제공한다.
     * 모듈을 통해 복잡한 구현은 외부로부터 숨기고, 명확한 책임을 갖는 간단한 인터페이스만 노출할 수 있다.
     * 대부분의 **모듈 시스템은 모듈 클라이언트가 사용할 수 있는 공개 인터페이스를 노출시키며, 세부 사항은 선택적으로 숨길 수 있도록 지원**한다.
  4. 종속성을 관리한다.
     * **모듈 시스템은 모듈 클라이언트가 접근하는 모듈의 실행에 필요한 종속성을 쉽게 임포트할 수 있도록 돕는다**.

### JS 모듈 시스템의 시작
```
> JS는 모듈 시스템이 없이도 관리에 어려움이 없는 상태가 길었기 때문에 내장 모듈 시스템이 존재하지 않았다.
```
* JS는 다른 언어와 달리 다음과 같은 이유에서 내장된 모듈 시스템이 오랫동안 존재하지 않았다.
  1. 브라우저 관점에서, 코드는 여러 파일로 분할될 수 있다.
  2. 단지 script 태그를 활용하는 것만으로도 분할된 코드베이스를 임포트할 수 있다.
  3. 웹사이트가 복잡하지 않았으므로, 몇 년간은 이러한 접근만으로도 웹 사이트를 빌드하는 것이 가능했다.
* 그러나 **웹 애플리케이션이 점차 복잡해지고, 프레임워크들이 생태계를 점유해나가며 점차 JS 프로젝트에 효율적으로 사용될 모듈 시스템의 필요성이 대두**되었다.
  * 이로 인해 JS 커뮤니티에서도 프로젝트에서 효율적으로 사용될 모듈 시스템을 정의하고자 하는 여러 시도가 나타나기 시작했다.

### Node.js 모듈 시스템
* Node.js가 처음 만들어진 시점에서, Node.js는 운영체제의 파일 시스템에 직접 접근하는 JS를 위한 서버 런타임으로 구상되었다.
  * 때문에 모듈 관리에 있어서 다른 방법을 도입해볼 수 있는 기회가 있었다.
* Node.js가 모듈을 관리하기 위해 도입한 방법은 다음과 같은 특징을 갖는다.
  1. script 태그와 URL을 통한 리소스 접근에 의존하지 않는다.
  2. **오직 로컬 파일 시스템의 JS 파일들에만 의존**한다.
* 때문에 **Node.js는 상술한 특징을 준수하여 브라우저가 아닌 환경에서 JS 모듈 시스템을 제공할 수 있도록 고안된 CommonJS의 명세를 구현**하게 되었다.
  * CommonJS는 시작과 더불어 Node.js에서의 주 모듈 시스템이 되었으며, Webpack과 같은 모듈 번들러 덕분에 브라우저 환경에서도 유명세를 탔다.

### ES6의 등장
```
> 지속 가능한 모듈을 작성하려면 CommonJS보다 ESM을 사용하는 것이 바람직하다.
```
* 2015년에 발표된 ECMAScript 6는 ECMAScript Modules라고 불리우는 표준 모듈 시스템을 위한 공식 제안을 포함하였다.
  * ESM은 JS 생태계에 큰 혁신을 불러왔으며, 특히 모듈 관리 관점에서 브라우저와 서버의 차이점을 연결하기 위해 노력하였다.
  * 그러나 ES6는 문법과 의미론적 관점에서 ESM을 위한 명세만을 정의하였고, 실제 구현을 제공하지는 않았다.
  * 이로 인해 여러 브라우저 회사들과 마찬가지로 Node.js 커뮤니티 역시 ESM 명세를 구현하기 위해 많은 시간을 소요하게 되었다. 
  * 그 결과, **Node.js는 13.2 버전부터 ESM을 안정적으로 지원**할 수 있게 되었다.
* **현재로서는 브라우저와 서버 환경 모두에서 ESM이 JS 모듈을 관리하는 실질적인 방법으로 사용된다**.
  * 그러나 **여전히 많은 Node.js 프로젝트들이 CommonJS에 의존하고 있으므로, ESM이 완전히 지배적인 표준이 되기까지는 더 많은 시간이 필요**하다.

## 2022-05-14 Sat
## 모듈 시스템과 패턴
* JS의 주요 문제점 중 하나는 네임스페이스가 없다는 점이며, 모든 스크립트는 전역 범위에서 실행된다.
* 이로 인해 내부 애플리케이션 코드 또는 종속성 라이브러리가 기능을 노출할 때마다 스코프를 오염시킬 가능성이 발생한다.
  * 예를 들어, 종속성 라이브러리가 선언한 변수의 이름이 외부에 선언된 변수의 이름과 같은 경우 애플리케이션은 오동작할 수 있다.
* 이는 매우 위험한 상황이므로, 전역 범위에 의존하는 것은 언제나 위험한 작업이다.
* 이러한 문제를 해결하기 위한 보편적인 기법에는 노출식 모듈 패턴이 있다.

### 노출식 모듈 패턴
```
const revealingModule = (() => {
    const name = 'injuk';
    const sayHello = () => { console.log('Hello!'); };
    
    return {
        publicName: name, 
        publicSayHello: sayHello,
    };
})();

console.log(revealingModule);
console.log(revealingModule.name, revealingModule.sayHello);
console.log(revealingModule.publicName, revealingModule.publicSayHello);
```
* 해당 패턴은 즉시 실행 함수를 사용하며, 함수 내부에 private한 범위를 만든 후 공개할 부분만 반환한다.
* **JS는 함수 내부에서 선언한 변수를 외부 범위에서 접근할 수 없으므로, 노출식 모듈 패턴은 외부에 공개할 정보만을 return 구문으로 명시**한다.
  * 상술한 코드의 경우, return 구문을 통해 외부로 반환된 객체의 속성만을 사용할 수 있게 된다.
* **노출식 모듈 패턴은 비공개 정보는 은닉하고 공개될 API만을 내보낼 수 있으며, CommonJS 모듈 시스템은 이 패턴을 기반으로 한 아이디어에서 시작**한다.

### CommonJS 명세
* CommonJS는 Node.js의 첫 번째 내장 모듈 시스템이다. 
* Node.js의 CommonJS는 실제 CommonJS의 명세를 고려하며 덧붙인 추가적인 자체 확장 기능과 함께 구현되었다.
* CommonJS 명세 중 주요한 개념은 크게 다음과 같이 요약할 수 있다.
  1. require 구문은 로컬 파일 시스템으로부터 모듈을 임포트하는 데에 사용된다.
  2. exports와 module.exports는 특별한 변수로서 현재 모듈에서 공개된 기능들을 내보내기 위해 사용된다.

### CommonJS 모듈 로더 구현하기
```
const fs = require('fs');

function loadModule(filename, module, require) {
  const wrappedSrc =
    `(function (module, exports, require) {
        ${fs.readFileSync(filename, 'utf8')}
      })(module, module.exports, require)`
  ;
  eval(wrappedSrc);
}
```
* 상술한 코드는 다음과 같은 특징을 갖는다.
  1. 모듈의 내용을 읽어들인 후 private 범위로 감싸 평가한다.
  2. 이 때, module, exports, require 변수들은 모듈에 전달된다.
  3. 래핑 함수의 exports 인자는 module.exports의 내용으로 초기화된다.
* 모듈의 내용을 읽어들일 때 비동기 방식이 아닌 동기 방식의 readFileSync가 사용되었다.
  * 파일 시스템의 동기 방식을 사용하는 것은 일반적으로 권장되지 않지만, **CommonJS의 모듈을 로드하는 것은 동기 방식에 해당**한다.
  * 때문에 **CommonJS에서는 여러 모듈의 임포트에 적절한 순서를 지키는 것이 중요**하다.

### require 함수 구현하기
```
function require(moduleName) {
  console.log(`Require invoked for module: ${moduleName}`);
  // 1.
  const id = require.resolve(moduleName);
  
  // 2.
  if(require.cache[id])
    return require.cache[id].exports;
  
  // 3.
  const module = {
    exports: {},
    id,
  };
  
  // 4.
  require.cache[id] = module;
  
  // 5.
  loadModule(id, module, require);
  
  // 6.
  return module.exports;
}
require.cache = {};
require.resolve = (moduleName) => {
  // 모듈 이름을 기반으로 id로 사용될 모듈의 전체 경로를 찾아낸다.
}
```
* 상술한 코드는 다음과 같은 특징을 갖는다.
  1. 우선 모듈의 전체 경로를 resolve 하고, 이를 id 변수로 초기화한다.
     * 모듈의 전체 경로 해석은 실제 알고리즘을 구현하는 require.resolve에 위임된다.
  2. 모듈이 이미 로드되어 있는 경우, 캐시된 모듈을 즉시 반환한다.
  3. 모듈이 아직 로드되어 있지 않은 경우, 최초 로드를 위한 환경인 모듈 메타데이터를 설정한다.
     * exports 속성은 빈 객체 리터럴로 정의된다.
     * **module 객체는 불러올 모듈의 코드에서 public API를 익스포트하는 데에 사용**된다.
  4. **최초 로드 과정을 마친 후에는 모듈 객체를 캐시**한다.
  5. **모듈 소스 코드는 파일에서 직접 읽어들이며, 이를 위해 앞서 생성한 module 객체와 require 함수의 참조를 전달**한다. 
     * 이렇게 **전달된 module 객체는 모듈의 코드가 실행되는 과정에서 module.exports 객체를 조작하기 위해 사용**된다.
     * module.exports 객체를 조작하거나 대체하는 과정을 통해 모듈은 public API를 내보낼 수 있다.
  6. 모듈이 내보낸 public API를 의미하는 module.exports의 내용은 클라이언트에게 반환된다.

### 모듈 정의 해보기
```
// 각 모듈은 또 다른 모듈 종속성을 로드할 수도 있다.
const anotherDependency = require('./anotherModule');

// private 멤버
function log() { console.log('helloooo'); }

// 공개적으로 사용되기 위해 익스포트되는 API
module.exports.run = () => { log(); };
```
* **module.exports 변수에 할당되지 않는 이상 모듈 내부의 모든 내용은 비공개**된다.
* **앞선 require 함수의 동작 방식에 따라, 모듈을 로드할 때 변수의 내용은 캐시된 후에 리턴**된다.

### module.exports와 exports
* **변수 exports는 module.exports의 초기 값에 대한 참조에 불과**하다.
  * 상술한 require 함수의 내용을 토대로, exports는 본질적으로 빈 객체 리터럴이다.
* 때문에 exports가 참조하는 객체에는 새로운 속성을 추가할 수 있다.
```
// exports = { hello: 'world' } 를 반환
exports.hello = 'world';
```
* exports 변수 자체에 대한 재할당은 지역 변수인 exports의 내용을 변경하므로, 실제 module.exports 객체에는 영향을 주지 못한다.
  * 즉, 실제로는 module.exports의 내용을 변경하지 않고 exports라는 지역 변수 자체만을 재할당한다.
  * 때문에 다음과 같은 exports의 재할당은 잘못된 사용 예시에 속한다.
```
exports = 'world';
```
* 이로 미루어 **exports는 모듈로부터 익스포트된 속성을 포함하는 객체를 반환하기 위해 사용하는 것으로 이해**할 수 있다.
* 반면 객체가 아닌 함수나 인스턴스, 또는 문자열을 익스포트하고자 하는 경우 전달된 module.exports 자체를 재할당해야 한다.
```
// exports = 'hello world' 를 반환
module.exports = 'hello world';
```
* 결국 **모듈 내에서 사용되는 module.exports 키워드는 module이 갖는 exports 객체의 실체인 반면, exports 키워드는 실제로는 지역 변수**다.