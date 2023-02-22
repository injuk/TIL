# Basics
## 2023-01-28 Sat
### Prisma와 Atomic 연산
* **여러 사용자로부터 같은 레코드에 대해 수정을 요청하는 등, 동시성이 중요한 상황에서는 다음과 같이 Atomic 연산을 활용**할 수 있다.
    * 이 때, 아래의 예시는 Prisma에 의해 제공되는 Atomic 연산 중 덧셈을 의미하는 `increment` 속성을 사용한 경우에 해당한다.
```typescript
async patchUser(payload) {
  const userInfo = await this.prisma.userInfo.update({
    where: {
      userId: Number(payload.userId),
    },
    data: {
      weight: {
        // update 대상 레코드에 대해 적절한 값을 더해준다.
        increment: Number(payload.weight),
      },
    },
    include: {
      user: true,
    },
  });

  return userInfo;
}
```
* **Atomic 연산은 연산이 실행되는 도중에 다른 연산으로부터 영향을 받지 않도록 보장**하며, 크게 다음과 같은 종류의 연산으로 분류된다.
    1. increment: 인자로 전달된 값을 Atomic하게 더한다.
    2. decrement: 인자로 전달된 값을 Atomic하게 뺀다.
    3. multiply: 인자로 전달된 값을 Atomic하게 곱한다.
    4. divide: 인자로 전달된 값을 Atomic하게 나눈다.
    5. set: 인자로 전달된 값을 Atomic하게 설정한다.

### Prisma의 multi schema
* PG의 경우 데이터베이스 차원에서 여러 스키마를 허용하며, 용도에 따라 여러 스키마를 사용하는 점을 권장하므로 Prisma 역시 이를 지원한다.
* 그러나 Prisma의 multi schema 기능은 아직 완성되지 않은 preview 기능이므로, 다음과 같이 `schema.prisma`를 수정해야만 사용이 가능하다.
```
generator client {
  provider = "prisma-client-js"
  previewFeatures = ["multiSchema"]
}
```
* 또한, **multi schema 기능을 활용하는 경우 datasource에는 반드시 다음과 같이 `schemas`를 배열 형태로 명시**해두어야 한다.
    * 나아가 **각 모델 또는 enum 역시 `@@schema("스키마명")` 키워드를 다음과 같이 반드시 명시**한다.
```
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["multiSchema"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  schemas  = ["study", "not_study", "maybe_study"]
}

enum MY_FIRST_PROVIDER {
  KAKAO
  NAVER
  GOOGLE
  ETC

  // enum에도 스키마 정보를 명시한다.
  @@schema("study")
}

model User {
  userId   Int               @id @default(autoincrement()) @map("USER_ID")
  provider MY_FIRST_PROVIDER @default(ETC) @map("PROVIDER")

  // enum을 사용하는 provider 필드를 활용하여 UQ 인덱스를 정의한다.
  @@unique(name: "UQ_PROVIDER_EMAIL", fields: [provider, email])
  @@map("USER")
  
  // 각 모델 역시 어떠한 스키마에 테이블을 생성할지 명시해준다.
  @@schema("study")
}

model Post {
  postId   Int    @id @default(autoincrement()) @map("POST_ID")

  @@map("POST")
  // 각 모델 역시 어떠한 스키마에 테이블을 생성할지 명시해준다.
  @@schema("study")
}

model UserInfo {
  userId Int  @id @map("USER_ID")

  @@map("USER_INFO")
  // 각 모델 역시 어떠한 스키마에 테이블을 생성할지 명시해준다.
  @@schema("study")
}

model TableForNotStudySchema {
  id Int @id @map("not_study_id")

  // 각 모델 역시 어떠한 스키마에 테이블을 생성할지 명시해준다.
  @@schema("not_study")
}

model TableForMaybeStudySchema {
  id Int @id @map("maybe_study_id")

  // 각 모델 역시 어떠한 스키마에 테이블을 생성할지 명시해준다.
  @@schema("maybe_study")
}
```

### findMany API를 활용한 페이지네이션 구현하기
* `count()` API와 `findMany()` API를 활용할 경우, 다음과 같이 간단하게 페이지네이션을 구현할 수 있다.
  * 이 때, `count()` API는 조건에 맞는 레코드의 개수를 반환한다.
  * `findMany()` API 역시 조건에 맞는 레코드 목록을 반환하는 점에서는 유사하며, take나 skip 및 orderBy 등의 속성 역시 사용이 가능하다.
```typescript
async getPostsWithPagination(page, take) {
  const [count, posts] = await Promise.all([
    this.prisma.post.count(),
    this.prisma.post.findMany({
      // take 속성은 몇 개의 레코드를 조회할지를 의미한다.
      take,
      
      // skip 속성은 몇 개의 레코드를 건너 뛸지를 의미하며, 일반적인 offset의 기능을 수행한다.
      skip: take * (page - 1),
      
      // orderBy는 어떠한 컬럼을 기준으로 결과를 정렬할지를 의미하며, 기본적으로 오름차순 정렬을 적용한다.
      orderBy: {
        postId: 'desc',
      },
    }),
  ]);

  return {
    currentPage: page,
    totalPage: Math.ceil(count / take),
    posts,
  };
}
```

### Prisma ORM에서의 조인 연산
* 읽기 연산에서 조인은 일반적으로 편리하나, 불필요한 데이터가 늘어나 리소스의 낭비를 늘린다는 단점이 수반된다.
  * 반면, Prisma의 경우 ORM 차원에서 쿼리 결과를 정돈하여 사용자가 보기 쉬운 결과를 반환하며, 중복되는 정보는 제거하여 리소스 낭비를 방지한다.
  * 이렇듯 **Prisma ORM에서 조인하는 경우에는 가상 컬럼을 활용하여 중복성을 제거**한다.

## 2023-01-29 Sun
### select 속성 활용하기
* 조회 결과의 일부만을 반환하고자 하는 경우, 다음과 같이 `select` 속성을 활용할 수 있다.
  * 이 때, **조회하고자 하는 각 컬럼을 true로 명시하는 반면 false는 인식조차하지 않음에 주의**한다.
  * 즉, **조회하지 않고자 하는 컬럼은 false로 두는 것이 아니라 애초에 명시하지 않아야 한다**.
```typescript
getUserWithPost() {
  return this.prisma.user.findUnique({
    where: {
      userId: 1,
    },
    // 사용자 레코드의 userId, provider, email 컬럼 정보만을 조회한다.
    select: {
      userId: true,
      provider: true,
      email: true,
    },
  });
}
```
* 반면, **`select` 속성을 사용하면서 `include` 속성을 활용하여 조인을 하고자 하는 경우에 두 속성은 함께 명시할 수 없다는 특징이 존재**한다.
  * 때문에 이러한 경우에는 **`select` 속성에 조인하고자 하는 가상 컬럼을 true로 명시해야 하며, Prisma는 가상 컬럼을 암시적으로 조인하여 반환**한다.
```typescript
getUserWithPost() {
  return this.prisma.user.findUnique({
    where: {
      userId: 1,
    },
    select: {
      userId: true,
      provider: true,
      email: true,
      
      // 가상 컬럼을 select한 경우, Prisma는 내부적으로 조인 연산을 적용한다. 
      posts: true,
    },
  });
}
```

### Prisma 중첩 쿼리 활용하기
* 상술한 바와 같이 조인 연산에는 중복되는 데이터가 포함될 수 있으며, 이러한 종류의 데이터는 Prisma의 중첩 쿼리를 활용하여 해결할 수 있다.
  * 이 때, **중첩 쿼리는 `select` 속성에 명시된 가상 컬럼에 대해 다시 `select` 속성을 명시하는 형식**을 갖는다.
```typescript
getUserWithPost() {
  return this.prisma.user.findUnique({
    where: {
      userId: 1,
    },
    select: {
      userId: true,
      provider: true,
      email: true,
      posts: {
        // 가상 컬럼으로부터 다시 select 속성을 활용하여 정확히 원하는 컬럼의 정보만을 조회할 수 있다.
        select: {
          postId: true,
          content: true,
        },
      },
    },
  });
}
```
* 이러한 **중첩 쿼리는 단지 한 단계만이 가능한 것은 아니며, 다음과 같이 추가적인 단계를 명시하는 것으로 조인을 더 유발시킬 수도 있다**.
```typescript
  getUserWithPost() {
    return this.prisma.user.findUnique({
      where: {
        userId: 1,
      },
      select: {
        userId: true,
        provider: true,
        email: true,
        posts: {
          select: {
            postId: true,
            content: true,
            
            // post가 갖는 가상 컬럼인 author를 활용하여 다시 조인을 유발시킨다.
            author: {
              select: {
                userId: true,
                name: true,
                provider: true,
              },
            },
          },
        },
      },
    });
  }
```
* 이 때, **중첩 쿼리를 사용하는 경우에는  이미 조회한 정보를 하위 단계에서 다시 명시하는 순환 참조가 발생하지 않도록 주의를 기울이는 것이 바람직**하다.

### findUnique와 findUniqueOrThrow API
* Prisma에서 검색을 위해 활용할 수 있는 API 중 `findUnique()` API는 유일한 값을 기반으로 레코드를 조회하기 위해 사용된다.
  * 때문에 해당 **API에 전달할 where 속성에는 PK 또는 UQ 제약 조건 등 DB 차원에서도 유일성이 보장되는 컬럼 정보만을 전달**할 수 있다.
* 그러나 `findUnique()` API는 존재하지 않는 컬럼에 대해 조회할 경우 빈 값을 의미하는 null을 반환하며, 로직으로 이를 검증하는 것이 필수적이게 된다.
```typescript
async getUserWithPost(userId) {
  const user = await this.prisma.user.findUnique({
    where: {
      userId,
    },
  });

  // 이렇듯 레코드의 존재 여부를 애플리케이션 로직으로 검증해주어야 한다.
  if (!user)
    throw new HttpException(`user(${userId}) not found`, HttpStatus.NOT_FOUND);

  return user;
}
```
* Prisma는 이러한 상황에 사용할 수 있도록 `findUniqueOrThrow()` API를 제공하며, 다음과 같이 간단하게 코드에 적용할 수 있다.
  * 이 경우, 검색에 실패한다면 해당 API를 호출한 코드 상에서 즉시 예외가 발생하게 된다.
```typescript
async getUserWithPost(userId) {
  try {
    const user = await this.prisma.user.findUniqueOrThrow({
      where: {
        userId,
      },
    });
    return user;
  } catch (e) {
    if (e instanceof NotFoundError) {
      throw new HttpException(`user(${userId}) not found`, HttpStatus.NOT_FOUND);
    }
    
    throw e;
  }
}
```
* **그러나 실무에서는 이러한 방법보다는 상술한 `findUnique()` API를 호출하되 결과를 코드로 검증하는 방식이 선호**되곤 한다.

### findUnique API를 활용한 복합키 검색
* 상술한 바와 같이 `findUnique()` API는 유일성이 보장되는 컬럼에 대해서만 조회할 수 있으며, 이는 UQ 제약 조건에도 해당이 된다.
* 이렇듯 UQ 제약 조건을 활용하여 검색하는 경우, 다음과 같이 `schema.prisma`에 명시했던 UQ 제약 조건의 이름을 명시한다.
```typescript
async getUserWithPost(userId) {
  const user = await this.prisma.user.findUniqueOrThrow({
    where: {
      // UQ_PROVIDER_EMAIL은 schema.prisma에 명시된 UQ 제약 조건의 이름이며, 객체 형태로 해당 UQ 제약 조건을 구성하는 컬럼을 명시한다. 
      UQ_PROVIDER_EMAIL: {
        provider: 'KAKAO',
        email: 'ingnoh1001',
      },
    },
  });
  if (!user)
    throw new HttpException(
            `user(${userId}) not found`,
            HttpStatus.NOT_FOUND,
    );

  return user;
}
```

## 2023-01-30 Mon
### where 속성과 다양한 연산자
* 상술한 내용중 where 속성을 명시하여 조건을 전달하는 것은 내부적으로는 다음과 같이 equals 연산이 생략된 것과 같다.
```typescript
async getUser() {
  const users = await this.prisma.user.findMany({
    where: {
      // 아래는 name: 'Elnora'를 작성한 것과 동일한 결과를 보인다.
      name: {
        equals: 'Elnora',
      },
    },
  });

  return users;
}
```
* equals와 반대되는 동작은 not 연산자가 있으며, 이 역시도 다음과 같이 간단한 사용이 가능하다.
```typescript
async getUser() {
  const users = await this.prisma.user.findMany({
    where: {
      name: {
        not: 'Elnora',
      },
    },
    // 너무 많은 검색 결과가 존재할 것으로 보이는 경우, take 속성을 통해 조회 개수를 지정할 수 있다.
    take: 5,
  });

  return users;
}
```
* 또한, 이러한 검색 조건의 전달은 다음과 같이 중첩 쿼리에서도 적용이 가능하다.
  * 이 경우, in 또는 startsWith 등 where 속성이 제공하는 다른 연산자들도 사용이 가능하다.
```typescript
async getUser() {
  const users = await this.prisma.user.findMany({
    where: {
      // 사용자와 1:1 관계를 갖는 사용자 정보를 가상 컬럼으로 참조하되, 조건에 맞는 레코드만을 조회한다.
      userInfo: {
        address: 'Pleasanton',
        // address: { in: ['Pleasanton'] },
        // address: { startsWith: 'P' },
      },
    },
    take: 5,
    include: {
      userInfo: true,
    },
  });

  return users;
}
```
* 이 때, 검색 조건에 명시된 컬럼이 숫자형인 경우 다음과 같이 이상 / 초과 / 이하 / 미만을 검증할 수 있다.
  * 이 경우, 적절히 혼합이 가능한 연산자를 둘 이상 명시할 경우 범위에 맞는 값만을 조회할 수 있다.
```typescript
async getUser() {
  const users = await this.prisma.user.findMany({
    where: {
      userInfo: {
        weight: {
          // 아래와 같이 작성할 경우, 80 초과 90 미만인 값만을 조회한다.
          gt: 80,
          lt: 90,
          // gte: 80,
          // lte: 80,
        },
      },
    },
    take: 5,
    include: {
      userInfo: true,
    },
  });

  return users;
}
```
* 반면, **여러 조건에 대해 AND 또는 OR 조건을 적용하고 싶은 경우에는 다음과 같이 배열 형태로 조회 조건을 나열**할 수 있다.
```typescript
async getUser() {
  const users = await this.prisma.user.findMany({
    where: {
      AND: [
        { userInfo: { weight: { lt: 80 } } },
        { userInfo: { address: { contains: 'e' } } },
      ],
      // OR: [
      //   { userInfo: { weight: { lt: 80 } } },
      //   { userInfo: { address: { contains: 'e' } } },
      // ],
    },
    take: 5,
    include: {
      userInfo: true,
    },
  });

  return users;
}
```

### $transaction API
* 트랜잭션은 여러 개의 쿼리를 논리적으로 하나의 단위로 묶어주며, Prisma는 `$transaction()` API를 통해 해당 기능을 제공한다.
* 이 때, `$transaction()` API를 사용하기 위한 구성 요소는 크게 다음과 같이 분류된다.
  1. 첫 번째 인자: 배열 형태로 전달되며, 실행할 개별 쿼리들을 의미한다.
  2. 두 번째 인자: 옵셔널한 값이며, 트랜잭션에 관련된 다음과 같은 옵션을 전달한다.
    1. maxWait: 해당 트랜잭션이 완료될 때까지 몇 초 간 대기할 것인지 명시한다.
    2. timeout: 트랜잭션 실행 시간에 대한 제한을 몇 초로 설정할 것인지 명시한다.
    3. isolationLevel: **트랜잭션이 실행되는 과정에서 적용될 격리 레벨을 명시**한다.
* **상술한 바에 따라 `$transaction()` API는 다음과 같은 형태로 사용하며, 트랜잭션에 포함된 각 쿼리 별 결과가 배열 형태로 반환**된다.
```typescript
async createPost() {
  // 이렇듯 각 쿼리는 $transaction() API에 첫 번째 인자로 전달될 배열에 나열한다.
  const result = await this.prisma.$transaction([
    this.prisma.user.findMany({
      where: {
        userId: 1,
      },
    }),
    this.prisma.user.findMany({
      where: {
        userId: 2,
      },
    }),
    this.prisma.user.create({
      data: {
        name: faker.name.firstName(),
        email: faker.datatype.uuid(),
        profile: faker.lorem.paragraph(),
      },
    }),
  ]);

  // 결과인 result는 배열이며, 각 쿼리 별 실행 결과가 포함된다.
  return result;
}
```
* 또한, 이렇게 정의된 쿼리는 다음과 같은 로그를 기반으로 동일한 트랜잭션 내에서 실행되었음을 확인할 수 있다.
```shell
{
  args: { where: { userId: 1 } },
  dataPath: [],
  // 아래 속성이 트랜잭션 내에서 실행되었음을 의미한다.
  runInTransaction: true,
  action: 'findMany',
  model: 'User'
}
// ...생략
prisma:query SELECT 1
// 각 쿼리는 동일한 BEGIN - COMMIT 블록에 위치하게 된다.
prisma:query BEGIN
prisma:query SELECT "study"."USER"."USER_ID", "study"."USER"."NAME", "study"."USER"."PROVIDER", "study"."USER"."EMAIL", "study"."USER"."PROFILE" FROM "study"."USER" WHERE "study"."USER"."USER_ID" = $1 OFFSET $2
prisma:query SELECT "study"."USER"."USER_ID", "study"."USER"."NAME", "study"."USER"."PROVIDER", "study"."USER"."EMAIL", "study"."USER"."PROFILE" FROM "study"."USER" WHERE "study"."USER"."USER_ID" = $1 OFFSET $2
prisma:query INSERT INTO "study"."USER" ("NAME","PROVIDER","EMAIL","PROFILE") VALUES ($1,$2,$3,$4) RETURNING "study"."USER"."USER_ID"
prisma:query SELECT "study"."USER"."USER_ID", "study"."USER"."NAME", "study"."USER"."PROVIDER", "study"."USER"."EMAIL", "study"."USER"."PROFILE" FROM "study"."USER" WHERE "study"."USER"."USER_ID" = $1 LIMIT $2 OFFSET $3
prisma:query COMMIT
```
* 반면, Prisma 4.7 버전 이전에서의 **`$transaction()` API는 첫 쿼리에서 조회한 결과를 토대로 두 번쨰 쿼리를 실행할 수 없었다**.
  * 즉, `$transaction()` API에서 각 쿼리는 서로 독립적으로 실행된다.
* 그러나 최근 버전의 경우, 다음과 같이 async 함수를 전달하는 것으로 해당 기능을 간단히 구현할 수 있게 되었다.
  * 이 경우, **async 함수의 인자로는 `this.prisma`가 전달되므로 반드시 다음과 같이 `this.prisma`를 직접 참조하지 않도록 코드를 작성**한다.
```typescript
async createPost(payload: Prisma.PostUncheckedCreateInput) {
  // 아래와 같이 트랜잭션 내에서 원하는 로직을 마음 껏 작성하며, 각 쿼리는 모두 동일한 트랜잭션에 위치하게 된다.
  const result = await this.prisma.$transaction(async (transactionCtx) => {
    // 이 경우, transactionCtx는 this.prisma가 된다.
    const user = await transactionCtx.user.findUnique({
      where: { userId: 1 },
    });

    if (!user) throw new NotFoundException(`user(${1})`);

    return transactionCtx.post.findMany({
      where: { authorId: user.userId },
    });
  });

  return result;
}
```
* **상술한 트랜젹션 방식을 Interactive transaction이라고 지칭하며, 이 경우 result 변수에는 트랜잭션 내에서 마지막으로 반환한 값이 참조**된다.

### Prisma 동작 원리
* Prisma를 활용하여 쿼리를 시도하는 경우, 요청은 다음과 같은 흐름에 의해 처리된다.
  1. 사용자의 요청인 쿼리는 백엔드 애플리케이션으로부터 시작하여 Prisma ORM 그 자체이기도 한 Prisma client에게 전달된다.
  2. Prisma client는 해당 요청을 Prisma 내부에 위치한 엔진인 Prisma engine에 전달한다.
  3. **Prisma engine은 요청된 쿼리에 대한 유효성을 검증하여 올바른 쿼리인지 확인한 후, 필요한 매핑 작업을 수행**한다.
  4. 3.의 과정에서 쿼리는 DB가 이해할 수 있는 형태로 매핑되므로, Prisma engine은 실제 DB에 이를 요청한 후 결과를 반환받는다.
  5. Prisma engine은 DB로부터 반환 받은 결과값을 다시 Prisma client에게 반환한다.
  6. Prisma client는 전달 받은 결과값을 다시 백엔드 애플리케이션에 반환한다.
* 이 때, Prisma 동작의 핵심인 Prisma engine 및 실제 DB 단에서 소요된 시간은 PrismaService의 미들웨어에서 다음과 같이 측정할 수 있다.
  * 이러한 **동작 원리와 흐름을 적절히 활용할 경우, 각 쿼리와 트랜잭션 별 소요 시간을 파악하여 성능을 튜닝하는 데에 도움**을 받을 수 있다.
```typescript
async onModuleInit() {
  await this.$connect();

  this.$use(async (params, next) => {
    // 아래에서 선언하는 시간은 Prisma client가 요청을 Prisma engine에 전달하기 직전까지의 시간이다. 
    const beforeQuery = Date.now();
    
    // 아래에서 Prisma engine이 실제로 DB와 상호작용한 후, 결과를 변수 result에 반환한다.
    const result = await next(params);

    // 아래에서 선언하는 시간은 Prisma client가 요청을 Prisma engine로부터 반환받기 직전까지의 시간이다.
    const afterQuery = Date.now();

    console.log(`query execution time in ms: ${afterQuery - beforeQuery}`);

    return result;
  });
}
```