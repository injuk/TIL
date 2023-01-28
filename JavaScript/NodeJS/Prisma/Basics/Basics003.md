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