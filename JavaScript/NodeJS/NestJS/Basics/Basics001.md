# Basics
## 2023-01-02 Mon

### NestJS란?
```
> NestJS는 효율적이고 확장성 있는 Node.js 기반의 서버 애플리케이션을 구축하기 위한 웹 프레임워크이다.
```
* **NestJS는 내부적으로 Express를 사용**하며, 선택적으로 Fastify를 사용하도록 구성할 수도 있다.
  * 즉, NestJS는 Node.js의 프레임워크를 감싸는 추상화 계층 역할을 수행한다.
* **NestJS는 여러 유용한 라이브러리가 존재함에도 아키텍쳐적인 주요 문제를 해결하지 못한 Node.js의 아쉬운 점을 개선하기 위해 설계**되었다.
  * 예를 들어 Express는 테스트나 TS를 적용하기 위해 여러 기능을 별도로 구성해주어야하는 측면이 존재한다. 
  * 이로 인해 NestJS를 사용하는 팀은 느슨하게 결합되어 확장성이 있으면서도 테스트 용이성과 유지보수성이 높은 애플리케이션 아키텍쳐를 즉시 적용할 수 있다.

### NestJS 시작하기
* NestJS는 `npm i -g @nestjs/cli` 명령어로 설치할 수 있다.
* NestJS를 설치한 경우, `nest new [프로젝트명]` 명령어로 새로운 프로젝트를 생성할 수 있다.

## 2023-01-03 Tue
### NestJS 기본 프로젝트 구조
* 상술한 바와 같이 `nest new ./`와 같은 명령어로 생성한 새 NestJS 프로젝트는 기본적으로 다음과 같은 구조를 갖는다.
  1. eslintrc.js: Javascript의 정적 검사 도구인 eslint를 설정하기 위한 파일이다.
  2. prettierrc: 프로젝트에서 코드의 일관성을 유지하는 코드 포맷터를 위한 파일이다. 
  3. nest-cli.json: NestJS 프로젝트 자체를 위한 설정을 위해 존재하는 파일이다.
  4. tsconfig.json: TS 컴파일 방식을 정의하기 위한 파일이다.
  5. tsconfig.build.json: tsconfig.json과 유사하지만, build를 위해 필요한 설정들을 정의한다.
  6. package.json: 일반적인 Node.js 프로젝트에서 사용되는 package.json과 역할이 같다.
  7. src/main.ts: **애플리케이션을 생성하고 실행하기 위한 파일**이다.
  8. src/app.module.ts: 
* 특히 main.ts 상단에 `import { AppModule } from './app.module';`와 같은 형태로 정의된 AppModule은 애플리케이션의 가장 큰 루트 모듈이다.
  * 이렇듯 **main.ts는 NestJS 프로젝트의 앱 모듈을 생성하고 실행하는 프로젝트의 시작점 역할을 수행**한다.
  * 이는 일반적인 Node.js 프로젝트에서 server.js 또는 index.js가 프로젝트 시작점인 엔트리포인트 역할을 맡는 것과 유사하다.

### 간단한 프로젝트 구조
* 일반적인 CRUD 기능의 경우, 다음과 같이 프로젝트를 구성할 수 있다.
  1. 애플리케이션 모듈
  2. 기능 모듈
  3. 인증 모듈
* 이는 애플리케이션의 어떤 기능을 위해 사용자 인증이 필요한 경우의 예시이며, 각 모듈은 다시 컨트롤러와 서비스 및 리포지토리 등으로 구성될 수 있다.

### NestJS 프로젝트 부트스트랩
* `nest new ./`와 같은 명령어로 실행된 기본 프로젝트의 경우, `npm run start:dev`와 같은 명령에서 다음과 같은 흐름으로 애플리케이션이 시작된다.
  1. main.ts의 bootstrap에서 NestFactory에 의해 AppModule 생성을 시도한다.
  2. AppModule이 생성되는 과정에서, AppController와 AppService가 등록된다.
  3. 필요한 컨트롤러와 서비스들이 등록되면 애플리케이션이 생성되어 main.ts의 bootstrap에 app 변수로 할당된다.
  4. app.listen 으로 인해 애플리케이션이 실행된다.

### NestJS 프로젝트 요청 흐름
* NestJS 애플리케이션이 실행된 경우, 기본 포트인 3000번을 통해 애플리케이션에 요청할 수 있으며 이는 다음과 같은 흐름으로 처리된다.
  1. 브라우저와 같은 클라이언트가 NestJS 애플리케이션에 요청을 전송한다.
  2. 전달된 요청에 대응되는 적절한 메소드는 컨트롤러에 의해 선택되며, 이러한 요청은 다시 서비스로 포워딩된다.
  3. 서비스는 전달된 요청을 적절히 처리한 후, 응답을 반환한다.
* 이 때, **이러한 요청과 응답 흐름은 app에서부터 시작하여 적절한 router로 포워딩된 후 처리되는 Express의 일반적인 흐름과 크게 다르지 않다**.

## 2023-01-04 Wed
### NestJS의 모듈 개념
* 일반적인 NestJS 프로젝트는 애플리케이션을 감싸는 AppModule 안에 도메인 별 모듈을 정의하는 식으로 구성된다.
  * 다시 말해 **NestJS를 기반으로 하는 애플리케이션은 루트 모듈로 인해 최소한 하나 이상의 모듈을 갖고, 루트 모듈은 NestJS의 시작점 역할을 수행**한다. 
  * 이 때, 모듈은 `@Module()` 데코레이터를 표기한 클래스의 형태로 작성할 수 있다.
  * 이렇듯 **NestJS는 여러 데코레이터를 통해 애플리케이션 구조를 구성하는 데에 필요한 여러 메타데이터를 관리**한다.
* **모듈은 밀접한 관련이 있는 기능 단위로 정의할 수 있으며, 같은 기능에 속하는 구성 요소는 전부 하나의 모듈로 묶을 수 있다**.
* **모듈은 기본적으로 싱글톤이 보장되며, 때문에 여러 모듈 간에 동일한 인스턴스를 손쉽게 공유**할 수 있다.
  * 때문에 공통적인 기능은 하나의 모듈로 묶어 여러 다른 모듈에서 이를 사용하는 것으로 재사용성을 높일 수 있다.

### 필요한 모듈 생성하기
* NestJS의 경우, 대부분의 생성 과정은 nest cli 명령어를 활용한다.
  * 예를 들어, **`nest g module [모듈명]` 과 같은 명령어는 모듈명에 해당하는 신규 모듈을 generate 한다는 의미**를 갖는다.
  * 해당 명령어를 통해 생성된 모듈은 AppModule에도 추가해주어야 하지만, nest cli를 활용한 경우 이 역시 자동으로 import된다.
```shell
[my-first-nest] nest g module boards
CREATE src/boards/boards.module.ts (83 bytes)
UPDATE src/app.module.ts (163 bytes)
[my-first-nest] 
```
```typescript
import { Module } from '@nestjs/common';
import { BoardsModule } from './boards/boards.module'; // import 문은 자동으로 삽입된다.

@Module({
  imports: [BoardsModule], // imports 배열의 내용은 자동으로 삽입된다.
})
export class AppModule {}
```

### NestJS 컨트롤러란?
* 다른 프레임워크의 컨트롤러와 마찬가지로, **NestJS의 컨트롤러 역시 요청을 처리하고 적절한 응답을 반환하는 역할을 수행**한다.
  * 이 때, NestJS의 컨트롤러는 `@Controller(경로)` 데코레이터를 명시하는 것으로 정의할 수 있다.
  * 또한 경로를 인자로 전달하여 해당 컨트롤러가 처리할 리소스를 명시할 수 있다.
* 각 **컨트롤러는 `@Get()`과 같이 HTTP 메소드를 데코레이터의 이름으로 명시하는 단순한 메소드인 핸들러**를 갖는다.

### 필요한 컨트롤러 생성하기
* 상술한 바와 같이 NestJS 프로젝트는 많은 부분을 nest cli로 생성할 수 있으며, 컨트롤러 역시 예외가 아니다.
  * 예를 들어, `nest g controller [컨트롤러명] --no-spec` 명령어는 컨트롤러명에 해당하는 신규 컨트롤러를 생성하되 테스트 코드는 생성하지 않는다.
  * 일반적으로 NestJS는 컨트롤러의 테스트 코드를 함께 생성해주나, `--no-spec` 옵션을 명시하는 경우 테스트 코드는 생성하지 않는다.
* nest cli를 활용하여 컨트롤러를 생성한 경우, 다음과 같은 터미널 메시지가 출력되는 것을 확인할 수 있다.
```shell
[my-first-nest] nest g controller boards --no-spec 
CREATE src/boards/boards.controller.ts (101 bytes) # 새로운 컨트롤러가 생성된다.
UPDATE src/boards/boards.module.ts (174 bytes) # 기존 모듈 코드에는 컨트롤러를 import 하는 코드가 자동으로 삽입된다.
[my-first-nest] 
```
```typescript
import { Module } from '@nestjs/common';
import { BoardsController } from './boards.controller'; // import 문은 자동으로 삽입된다.

@Module({
  controllers: [BoardsController], // controllers 배열의 내용은 자동으로 삽입된다.
})
export class BoardsModule {}
```

### nest cli를 통해 컨트롤러가 생성되는 과정
* `nest g controller boards --no-spec`와 같은 명령어를 입력한 경우, 컨트롤러는 다음과 같은 과정을 거쳐 생성된다.
  1. nest cli는 우선 boards라는 이름의 폴더가 있는지 확인한다.
  2. 폴더가 존재하는 경우 해당 폴더를 사용하되, 없는 경우에는 새로 생성한다.
  3. 2.에서 결정된 폴더 안에 controller 파일을 생성하고, `--no-spec` 옵션이 명시되지 않았다면 테스트 코드도 함께 생성한다.
  4. 2.에서 결정된 폴더 안에 module 파일이 존재하는지 확인한다.
  5. **module 파일이 존재한다면 이를 업데이트하여 controller 정보를 import하며, 존재하지 않는다면 AppModule에 import**한다.