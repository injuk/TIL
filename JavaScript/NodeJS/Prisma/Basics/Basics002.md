# Basics
## 2023-01-25 Wed
### Prisma 옵션 설정하기
* Prisma의 옵션을 설정하는 방법은 크게 다음과 같은 세 가지 방식으로 분류될 수 있다.
    1. .env 파일의 `DATABASE_URL`에 `/my_first_prisma?schema=study` 형태의 쿼리 스트링으로 전달하는 방식
    2. ORM 상에서 동작하는 객체인 `PrismaClient`를 활용하여 옵션을 설정하기
* 이 때, 두 번째 방식의 경우 `PrismaService`가 `PrismaClient`를 상속하므로 `super()`를 활용하여 다음과 같이 옵션을 명시하게 된다.
```typescript
@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  constructor() {
    // 아래 라인에서, super 키워드는 PrismaClient를 의미한다.
    super({
      log: [{emit: 'stdout', level: 'query'}],
      errorFormat: 'pretty',
    });
  }
  // ...생략
}
```

### NestJS 미들웨어 활용하기
* 미들웨어는 Express에서 유용하게 사용되는 개념이듯, Express를 래핑하는 NestJS 역시 미들웨어의 활용을 적극적으로 권장한다.
    * 이 때, **Prisma 역시 쿼리 결과의 포맷을 수정하거나 입력과 출력을 로깅하는 등의 용도로 미들웨어를 사용**할 수 있다.
* Prisma에서 미들웨어는 `PrismaClient`로부터 상속받은 `this.$use()` 메소드를 통해 호출하며, 다음과 같은 인자를 받는 콜백을 전달한다.
    1. params: 백엔드 애플리케이션으로부터 유발되는 쿼리 관련 정보들을 의미한다.
    2. next: **ORM에 의해 데이터베이스와 상호작용한 후, 그 결과를 반환받기 위해 사용하는 함수를 의미**한다.
```typescript
async onModuleInit() {
  await this.$connect();

  // Prisma의 경우, 아래와 같이 $use() 메소드를 호출하여 미들웨어를 사용하게 된다.
  this.$use((params, next) => {
    console.log('this sentence is logged from onModuleInit middleware!');
    console.log(params);

    // PrismaClient는 아래와 같이 params를 활용하여 데이터베이스를 호출하고, 그 결과를 반환받기 위해 next() 함수를 사용한다.
    return next(params);
  });
}
```

### NestJS에서 Prisma 활용하기
* 임의의 서비스 로직에서, 앞서 정의한 `PrismaService`를 활용하려면 우선 다음과 같이 의존성을 주입받을 필요가 있다.
```typescript
@Injectable()
export class AppService {
  constructor(private readonly prisma: PrismaService) {}
}
```
* 그 이후에는 `PrismaService`를 자유롭게 사용하되, 반드시 `prisma.모델명.쿼리(객체)`와 같은 형태를 지켜 호출한다.
    1. prisma: 상술한 생성자에서 주입받은 `PrismaService`를 참조하는 변수를 의미한다.
    2. 모델명: **`schema.prisma`에 정의한 테이블명을 의미하며, 명시한 테이블명 형식과는 무관하게 camelCase로 변환된다는 점에 주의**한다.
    3. 쿼리: **ORM에 요청할 실제 동작을 의미하며, 쿼리를 실행하기 위한 정보는 객체 형태로 전달**한다.
* 이러한 내용을 토대로 임의의 정보를 생성하는 `faker` 라이브러리를 활용하여 새로운 사용자 정보를 생성하는 코드의 예시는 다음과 같다.
```typescript
async createUser() {
  // 테이블명은 User로 명시하였으나, 모델명은 자연스레 camelCase로 치환되므로 user로 참조해야 한다.
  const user = await this.prisma.user.create({
    data: {
      name: faker.name.firstName(),
      email: faker.datatype.uuid(),
      profile: faker.lorem.sentence(),
    },
  });

  this.logger.debug(
    `this.prisma.user.create result: ${JSON.stringify(user)}`,
  );

  return user;
}
```

### 레코드 간의 관계를 활용하여 생성하기
* 사용자와 사용자 정보가 1:1 관계를 맺는다고 가정할 때, 두 정보를 하나의 트랜잭션 안에서 동시에 생성하고 싶은 경우가 있을 수 있다.
* 언뜻 이 경우, 상술한 내용을 따라 사용자 정보를 다음과 같이 삽입하면 될 것처럼 보일 수 있다.
```typescript
async createUser() {
  const user = await this.prisma.user.create({
    data: {
      name: faker.name.firstName(),
      email: faker.datatype.uuid(),
      profile: faker.lorem.sentence(),
    },
  });

  // 상술한 코드와 마찬가지로 사용자 정보를 별도로 삽입한다.
  const userInfo = await this.prisma.userInfo.create({
    data: {
      userId: user.userId,
      height: '174',
      weight: 70,
      address: faker.address.city(),
      phone: '',
    },
  });

  this.logger.debug(
    `this.prisma.user.create result: ${JSON.stringify({
      ...user,
      userInfo,
    })}`,
  );

  return {
    ...user,
    userInfo,
  };
}
```
* 그러나 상술한 코드는 불편할 뿐더러 트랜잭션에 묶이지 않는다는 단점이 존재하므로, 다음과 같이 NestQuery를 활용하는 것이 바람직하다.
    * 이 경우, **사용자 정보는 `schema.prisma`의 사용자 모델에 명시해두었던 가상 컬럼을 활용하여 삽입**한다.
    * 또한 **가상 컬럼을 명시하는 것만으로는 결과에 사용자 정보가 포함되지 않으므로, `include` 프로퍼티를 추가하여 결과를 조인한 후 반환**하도록 한다.
```typescript
async createUser() {
  const user = await this.prisma.user.create({
    data: {
      name: faker.name.firstName(),
      email: faker.datatype.uuid(),
      profile: faker.lorem.sentence(),

      // 사용자 모델에 명시된 가상 컬럼을 통해 사용자 정보를 삽입한다.
      userInfo: {
        create: {
          // NestQuery의 경우 userId를 명시해줄 필요는 없다.
          // userId: user.userId,
          height: '174',
          weight: 70,
          address: faker.address.city(),
          phone: '',
        },
      },
    },
    include: {
      userInfo: true,
    },
  });

  this.logger.debug(
    `this.prisma.user.create result: ${JSON.stringify(user)}`,
  );

  return user;
}
```

### createMany API를 활용하여 여러 레코드를 생성하기
* createBulk와 같은 동작이 필요한 경우, `PrismaService`의 `createMany()` API를 다음과 같이 활용하여 간단하게 구현할 수 있다.
    * 이 경우, **실행 결과에는 삽입된 레코드 자체가 아닌 삽입된 레코드의 개수가 반환**된다.
```typescript
async createUser() {
  const data = new Array(1000000).fill({}).map((e) => {
    e.name = faker.name.firstName();
    e.email = faker.datatype.uuid();
    e.profile = faker.lorem.sentence();
    return e;
  });
  const user: { count: number } = await this.prisma.user.createMany({
    data,
  });

  this.logger.debug(
    `this.prisma.user.create result: ${JSON.stringify(user)}`,
  );

  return user;
}
```
* 그러나 **`createMany()` API를 활용하는 경우, `create()` API의 경우와는 달리 가상 컬럼을 활용한 데이터의 삽입은 불가능**하다.

### PrismaClient를 활용하여 타이핑하기
* `import { Prisma } from '@prisma/client';`와 같은 형태로 import할 경우, `prisma generate`에 의해 자동 생성된 타입을 사용할 수 있다.
```typescript
// 사용자 정보를 생성하기 위해 필요한 타입은 Prisma.UserCreateInput 형태로 import할 수 있다.
async createUser(payload: Prisma.UserCreateInput) {
  // 사용자 정보를 의미하는 모델인 User 역시 @prisma/client로부터 import한다.
  const user: User = await this.prisma.user.create({
    data: {
      name: payload.name,
      profile: payload.profile,
    },
  });

  this.logger.debug(
          `this.prisma.user.create result: ${JSON.stringify(user)}`,
  );

  return user;
}
```