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