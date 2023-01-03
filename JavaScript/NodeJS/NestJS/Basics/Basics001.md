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