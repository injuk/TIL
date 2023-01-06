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

## 2023-01-05 Thu
### NestJS 프로바이더란?
* **프로바이더는 NestJS의 기본 개념이며, 대부분의 서비스 / 리포지토리 / 팩토리 / 헬퍼 등의 클래스는 프로바이더로 취급**될 수 있다.
* 이 때, **프로바이더의 주요한 아이디어는 종속성으로서 주입될 수 있다는 점**에 있다.
  * 때문에 **객체 간의 다양한 관계와 인스턴스 간의 연결은 NestJS 런타임에게 위임**될 수 있다.
* 예를 들어 컨트롤러가 요청에 응답하기 위한 모든 로직을 포함하는 것보다는 이를 별도의 여러 서비스로 추출하되, 이들 의존성을 컨트롤러에게 주입할 수 있다.

### 서비스란?
* 서비스는 NestJS 뿐만 아닌 소프트웨어 개발 분야에서 통용되는 개념으로, 임의의 요청을 받은 컨트롤러가 처리해야 할 작업을 대부분 처리한다.
  * 예를 들어 컨트롤러가 요청을 처리하는 과정이 복잡한 경우, 이를 별도의 서비스에서 처리하도록 할 수 있다.
* NestJS의 경우, 서비스는 `@Injectable()` 데코레이터를 명시하여 모듈에 제공된다.
  * 또한, 서비스 인스턴스는 애플리케이션 전체에서 사용될 수 있다는 특징이 존재한다.
* 바꿔 말해 서비스 인스턴스는 정의만 했다고 해서 컨트롤러에서 사용할 수 있는 것은 아니며, 컨트롤러가 의존성을 주입받을 수 있도록 코드를 작성할 필요가 있다.
  * 이 때, 이는 Java Spring의 생성자 주입 방식과 유사한 형태의 코드로 작성된다.

### 프로바이더 사용 시 주의사항
```
> 프로바이더는 반드시 NestJS 런타임에 등록되어야 사용이 가능하다.
```
* **프로바이더의 등록은 모듈에서 명시하며, `@Module()` 데코레이터의 인자로 `providers: [프로바이더명]`와 같은 형태로 프로바이더 정보를 등록**한다.
  * 이 때, **`providers` 에는 해당 모듈에서 사용하고자 하는 프로바이더 목록을 전달**한다.

### 서비스 생성하기
* 서비스 역시 nest cli를 활용하여 생성하며, 명령어 형식은 컨트롤러와 유사하게 `nest g service [서비스명] --no-spec` 와 같이 입력한다.
  * 이 떄, 명령어 입력 결과로 모듈에 자동으로 서비스가 import되는 식으로 코드가 수정된다는 점 역시 동일하다.
* 이렇게 생성된 파일에는 `@Injectable()` 데코레이터가 명시되어 있으며, NestJS 런타임은 이를 통해 다른 컴포넌트가 해당 서비스를 사용할 수 있도록 한다.
  * 때문에 프로바이더는 다른 컨트롤러 등 NestJS 프로젝트 전역에서 사용될 수 있다.
```typescript
import { Injectable } from '@nestjs/common';

@Injectable() // 해당 데코레이터를 활용하여 NestJS는 프로바이더를 다른 모듈에 주입한다.
export class BoardsService {}
```

### 의존성 주입하기
* **NestJS의 경우, 의존성 주입은 다음과 같이 생성자 안에서 이루어진다**.
  * 이 때, private 접근 제어자는 TS의 기능을 활용한 것으로 Vanilla JS에서는 불가능하다.
  * 또한 **`this.some = some`와 같은 코드가 없는 이유는 생성자 매개변수에 명시된 private은 암묵적으로 인자를 프로퍼티로 선언하기 때문**이다.
```typescript
import { Controller } from '@nestjs/common';
import { BoardsService } from './boards.service';

@Controller('boards')
export class BoardsController {
    // NestJS 런타임의 DI는 생성자 주입 방식을 사용한다. 
    constructor(private boardsService: BoardsService) {}
}
```
* 이는 생성자에 private 접근 제어자를 명시한 인자는 암묵적으로 클래스 프로퍼티로 취급되는 것을 이용한 예시이며, 이를 사용하지 않는 경우 코드는 다음과 같다.
```typescript
@Controller('boards')
export class BoardsController {
  private boardsService: BoardsService; // TS 에서는 생성자 등에서 프로퍼티를 사용하기 전에 반드시 명시해두어야 한다.
  constructor(boardsService: BoardsService) {
    this.boardsService = boardsService;
  }
}
```

### 모델 정의하기
* 비즈니스 로직에서 사용되는 임의의 개념을 사용하기 전에, 우선 해당 개념이 어떠한 형태인지 정의하는 것이 바람직하다.
  * 이러한 형태를 모델이라고 지칭하며, TS에서는 모델을 정의하기 위해 인터페이스나 클래스를 활용할 수 있다.
  * 이 때, 인터페이스는 모델이 갖는 정보에 대한 타입만을 체크하는 반면 클래스는 인스턴스화까지 가능하다는 차이점이 존재한다.
* 이렇듯 인터페이스 또는 클래스를 활용하여 모델을 정의하는 것은 많은 에러를 컴파일 시점에 미리 잡을 수 있게 하며, 코드의 가독성 역시 높여주는 효과가 있다.

### 컨트롤러 핸들러에서 데이터를 수신하기
* POST 등의 메소드로 컨트롤러를 호출하는 경우, 핸들러에서 사용자가 전달한 요청 본문을 처리해야하는 경우가 있을 수 있다.
  * Express의 경우, `req.body`와 같은 형식으로 사용자의 요청 본문을 수신할 수 있다.
  * NestJS의 경우, **핸들러에서 요청 본문을 수신하기 위해서는 핸들러 메소드의 매개 변수에 `@Body()` 데코레이터를 다음과 같이 명시**할 수 있다.
```typescript
@Post('/')
createBoard(@Body() dto: BoardDto): Board {
  return this.boardsService.createBoard(dto.title, dto.description);
}
```
* 또는 아래와 같이 `@Body([타입명])` 데코레이터를 명시하는 것으로 원하는 프로퍼티만을 사용할 수도 있다.
```
@Post('/v2')
createBoardV2(@Body('title') title: string, @Body('description') description: string): Board {
  return this.boardsService.createBoard(title, description);
}
```

## 2023-01-06 Fri
### DTO란?
```
> 계층형 구조를 채택하는 아키텍쳐에서 DTO는 계층 간의 데이터 교환을 위해 존재한다.
```
* DTO는 일반적으로 데이터베이스로부터 얻은 데이터를 서비스 또는 컨트롤러에게 전송하기 위해 사용하는 객체이며, 역시 인터페이스 또는 클래스로 정의할 수 있다.
  * 그러나 **NestJS의 경우, 인터페이스 DTO보다는 클래스 형태의 DTO를 권장**한다.
  * 이는 클래스가 인터페이스와는 달리 런타임에서도 사용되기 때문이며, 이로 인해 NestJS가 지원하는 파이프 등의 기능을 유용하게 활용할 수 있다.
* 데이터를 교환하기 위해 DTO를 정의하는 것은 크게 다음과 같은 장점을 갖는다.
  1. 데이터 유효성을 검증하는 과정이 더 효율적이다.
  2. 타입으로서 정의되어 코드의 안정성을 높여준다.
  3. 데이터의 형태가 변경되는 경우 DTO를 사용하는 방식이 변경에 더 유연하게 대응할 수 있다.

### 간단한 DTO
* DTO는 다음과 같은 클래스를 활용하여 간단하게 정의할 수 있으며, 필요한 컨트롤러 또는 서비스에서 이를 import하여 사용한다.
```typescript
export class CreateBoardDto {
  public title: string;
  public description: string;
}
```

### 컨트롤러에서 Path variable 수신하기
* 간단한 상세 조회 또는 삭제 API 등, Path를 통해 id와 같은 식별자를 전달받아야 하는 경우에는 다음과 같이 `@Param([Path명])` 데코레이터를 사용한다.
```typescript
@Get('/:id')
getBoard(@Param('id') id: string): Board {
  return this.boardsService.getBoard(id);
}
```

### NestJS의 파이프
* NestJS의 **파이프 역시 `@Injectable()` 데코레이터가 명시된 클래스이며, 주로 데이터의 형태 변형 또는 유효성 검증을 위해 사용**한다.
* **파이프는 컨트롤러 핸들러에 의해 처리되는 인수에 대해 동작하며, 핸들러 메소드가 동작하기 전에 우선적으로 호출**된다.
* 예를 들어 사용자의 리소스 생성 요청은 기본적으로 컨트롤러의 적절한 핸들러에게 바로 포워딩되지만, 파이프가 있는 경우 마치 프록시처럼 요청을 가로챈다.
  * 이후 파이프가 정상적으로 동작하는 데에 성공했다면 그 때 컨트롤러의 핸들러에게 처리한 데이터를 전달하게 된다.
  * 반면, 파이프의 동작 과정에서 유효성 검증이나 데이터 변환에 실패한 경우에는 에러를 발생시킨다.

### 파이프의 종류
* 파이프를 사용하는 방법은 파이프의 종류에 따라 달라질 수 있으며, 이 때 파이프의 종류는 크게 다음과 같이 분류할 수 있다.
  1. Handler-level
  2. Parameter-level 
  3. Global-level

### Handler-level 파이프
* 핸들러 단의 파이프는 컨트롤러의 핸들러 메소드에서 `@UsePipes()` 데코레이터를 명시하는 것으로 사용할 수 있다.
  * 이 때, **핸들러 단의 파이프는 핸들러 메소드가 전달 받는 모든 파라미터에 대해 적용**된다.

### Parameter-level 파이프
* **매개 변수 단의 파이프는 컨트롤러의 핸들러 메소드가 전달 받는 매개 변수 중 특정한 값에 대해서만 적용**된다.
  * 이 때, 매개 변수 단의 파이프는 별도의 데코레이터 대신 매개 변수를 명시하는 `@Body([매개 변수명], [파이프명])` 데코레이터에 인자로 전달된다.
  * 물론 `@Param([매개 변수명], [파이프명])`과 같은 파이프 적용 역시 가능하다.

### Global-level 파이프
* **가장 넓은 범위에 대해 적용되는 글로벌 파이프는 클라이언트로부터 전달되는 모든 요청에 대해 적용되며, 최상위 영역을 의미하는 `main.ts`에 명시**된다.
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes([파이프명]); // 글로벌 파이프는 이러한 형태로 적용된다.
  await app.listen(3000);
}
bootstrap();
```

### NestJS의 빌트인 파이프
* NestJS는 다음과 같은 기본적인 빌트인 파이프를 제공하며, 각 파이프는 명시적인 이름을 통해 어떠한 동작을 수행하는지 쉽게 짐작할 수 있다.
  1. ValidationPipe
  2. ParseIntPipe
  3. ParseFloatPipe
  4. ParseBoolPipe
  5. ParseArrayPipe
  6. ParseUUIDPipe
  7. ParseEnumPipe
  8. DefaultValuePipe
  9. ParseFilePipe
* 예를 들어 ParseIntPipe의 경우, 다음과 같이 적용할 수 있다.
```typescript
@Get('/temp/:id')
getTemp(@Param('id', ParseIntPipe) id: number): number {
  return id;
}
```
* 이 때, id 위치에 숫자가 아닌 알파벳 등을 전달할 경우 다음과 같은 에러 메시지를 확인할 수 있다.
```json
{
    "statusCode": 400,
    "message": "Validation failed (numeric string is expected)",
    "error": "Bad Request"
}
```

### 파이프를 활용한 유효성 처리
* Dto 클래스를 예시로 들어 빈 값이 아니어야 하는 등의 유효성 조건을 적용하고 싶은 경우, 다음과 같은 두 라이브러리를 유용하게 사용할 수 있다.
  1. `class-validator`
  2. `class-transformer`
* 둘 모두를 npm 등의 패키지 관리자를 통해 설치한 후, Dto의 각 프로퍼티에 빈 값이 들어오지 않도록 다음과 같이 Dto를 수정할 수 있다.
```typescript
import { IsNotEmpty } from 'class-validator';

export class CreateBoardDto {
  @IsNotEmpty()
  public title: string;

  @IsNotEmpty()
  public description: string;
}
```
* 그러나 **이러한 데코레이터만으로는 원하는 유효성 검증 효과를 얻을 수 없으며, 반드시 컨트롤러에 핸들러 단 파이프를 다음과 같이 명시**해주어야 한다.
```typescript
@Post('/')
@UsePipes(ValidationPipe) // 핸들러 단의 파이프를 명시한다.
createBoard(@Body() dto: CreateBoardDto): Board {
  return this.boardsService.createBoard(dto);
}
```
* 이제 유효성 검증 조건에 위배되는 데이터를 전송할 경우, 다음과 같은 메시지가 반환되며 요청에 실패하게 된다.
```json
{
    "statusCode": 400,
    "message": [
        "title should not be empty",
        "description should not be empty"
    ],
    "error": "Bad Request"
}
```