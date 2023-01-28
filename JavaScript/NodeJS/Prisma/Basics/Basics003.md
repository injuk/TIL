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