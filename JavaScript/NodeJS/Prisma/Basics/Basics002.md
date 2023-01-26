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

## 2023-01-26 Thu
### schema.prisma의 옵셔널 속성
* `schema.prisma`에서, ? 기호를 명시한 속성은 옵셔널한 속성으로 취급된다.
  * 이 때, **연관 관계를 위해 참조되는 컬럼에 옵셔널 기호를 적용했을 경우에는 아래와 같이 가상 컬럼 역시 옵셔널하게 설정**해주어야 한다.
```
model Post {
  postId   Int    @id @default(autoincrement()) @map("POST_ID")
  content  String @map("CONTENT") @db.Text
  
  // 아래와 같이 임의의 속성에 ? 기호를 명시하여 옵셔널한 속성으로 취급할 수 있다.
  authorId Int?    @map("AUTHOR_ID")

  // 그러나 authorId 컬럼은 가상 컬럼인 author에서 참조하므로, 이 경우에는 가상 컬럼 역시 옵셔널하게 설정해주어야 한다.
  author User? @relation("USER_HAS_MANY_POSTS", fields: [authorId], references: [userId])

  @@map("POST")
}
```

### findUnique API를 활용하여 단건 조회하기
* `PrismaService`를 활용하여 레코드를 단건 조회하는 경우, `findUnique()` API를 다음과 같이 활용할 수 있다.
  * 이 때, where 속성을 활용하여 where 조건 절을 나타낼 수 있으며 include 속성을 활용한 조인 연산 역시 가능하다.
```typescript
getUserWithPost() {
  return this.prisma.user.findUnique({
    where: {
      userId: 1,
    },
    include: {
      posts: true,
    },
  });
}
```

### delete API를 활용하여 레코드 삭제하기
* `PrismaService`를 활용하여 레코드를 삭제하는 경우, `delete()` API를 다음과 같이 활용할 수 있다.
  * 또한, 일반적으로 PathVariable 또는 QueryString으로 전달 받는 값은 문자열이므로 필요한 경우 반드시 명시적 형변환을을 적용해주어야 한다.
```typescript
deleteUserById(id: number) {
  return this.prisma.user.delete({
    where: {
      // PathVariable 또는 QueryString으로 전달되는 id 값은 문자열이므로, 이를 숫자형으로 형변환한다.
      userId: Number(id),
    },
  });
}
```
* 상술한 `delete()` API는 결과로 제거된 레코드를 반환하는 반면, `deleteMany()` API는 제거된 레코드의 개수를 반환한다.
  * 이 때, `deleteMany()` API 역시 where 속성을 활용하여 레코드를 제거하므로 API 호출 형태는 `delete()` API와 다르지 않다.
* 반면, 아래와 같이 `schema.prisma`에 명시된 결과에 따라 사용자가 제거된 경우 사용자 정보 역시 함께 제거된다.
```
model UserInfo {
  userId Int  @id @map("USER_ID")
  // onDelete: Cascasde에 의해 사용자 정보는 사용자가 제거될 때 함께 제거된다.
  user   User @relation("USER_DETAIL_INFO", fields: [userId], references: [userId], onDelete: Cascade)

  height String @map("HEIGHT") @db.Char(3)
  weight Int    @map("WEIGHT") @db.Integer

  address   String   @map("ADDRESS") @db.Text
  phone     String   @map("PHONE") @db.Char
  createdAt DateTime @default(now()) @map("CREATED_AT") @db.Timestamptz()

  @@map("USER_INFO")
}
```
* 이렇듯 `onDelete`는 참조 무결성을 위해 FK로 참조되는 레코드가 제거되었을 때 이를 참조하는 레코드를 어떻게 할지 결정하는 옵션이며, 다음과 같이 분류된다.
  1. Restrict: FK로 이미 참조되고 있는 레코드를 제거할 경우, 에러를 발생시킨다.
  2. Cascade: FK로 참조되는 레코드가 제거된 경우, 이를 참조하는 레코드도 함께 제거한다.
  3. SetNull: FK로 참조되는 레코드가 제거된 경우, 이를 참조하는 레코드의 FK는 null로 설정한다.
  4. SetDefault: FK로 참조되는 레코드가 제거된 경우, 이를 참조하는 레코드의 FK는 default 값으로 설정한다.
* 이 중 SetNull 옵션은 FK가 null을 허용해야 하므로 사용처가 제한적인 한계가 있으며, SetDefault 옵션은 반드시 default 값이 설정되어 있어야 한다.
  * 즉, SetNull 옵션은 FK와 가상 컬럼이 모두 옵셔널한 값이어야 하므로 `schema.prisma`에 ? 기호가 명시되어 있어야 한다.
* 그러나 **참조 무결성을 준수하기 위한 `onDelete` 옵션에 정답은 없으므로, 상황과 기획 의도에 맞는 적절한 옵션을 선택하는 것이 바람직**하다.

## 2023-01-27 Fri
### deleteMany API를 활용하여 여러 레코드 삭제하기
* 기본적으로 `deleteMany()` API 역시 where 속성을 활용하므로 `delete()` API와 사용 방법이 크게 다른 점은 없다.
  * 반면, 상술한 바와 같이 `deleteMany()` API는 제거된 레코드가 아닌 제거된 레코드의 개수를 반환한다는 점에서 차이가 있다.
* 반면, 쿼리 상에서는 where in 구문을 활용하여 여러 레코드를 삭제할 수 있듯 다음과 같이 `deleteMany()` API에 in 조건을 전달할 수 있다.
```typescript
deleteUsers() {
  return this.prisma.user.deleteMany({
    where: {
      userId: {
        // 아래와 같이 여러 userId를 제거하고 싶은 경우 in 속성을 활용할 수 있다.
        in: [1000062, 1000061, 1000060],
      },
    },
  });
}
```
* 또한, **`deleteMany()` API 역시 where 속성에 빈 객체가 참조될 경우 전체 레코드를 제거하므로 사용 상에 주의를 기울일 필요**가 있다.
* 그러나 `delete()` API 또는 `deleteMany()` API는 실무에서 잘 사용되지 않으며, 대신 삭제된 상태로 전이시키는 등의 조치를 취하는 것이 일반적이다.

### @updatedAt 활용하기
* Prisma의 경우, 레코드가 수정된 시간을 자동으로 관리해주는 `@updatedAt` 기능이 제공되며 이는 다음과 같이 사용할 수 있다.
```
model UserInfo {
  userId Int  @id @map("USER_ID")
  user   User @relation("USER_DETAIL_INFO", fields: [userId], references: [userId], onDelete: Cascade)

  height String @map("HEIGHT") @db.Char(3)
  weight Int    @map("WEIGHT") @db.Integer

  address   String   @map("ADDRESS") @db.Text
  phone     String   @map("PHONE") @db.Char
  createdAt DateTime @default(now()) @map("CREATED_AT") @db.Timestamptz()
  
  // 이렇게 작성하는 것으로 레코드가 수정된 시간은 자동으로 기록된다.
  updatedAt DateTime @updatedAt

  @@map("USER_INFO")
}
```