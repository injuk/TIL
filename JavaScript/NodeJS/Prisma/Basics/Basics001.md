# Basics
## 2023-01-22 Sun
### ORM과 추상화 
* 현대적인 애플리케이션이 데이터베이스와 상호작용함에 따라, 데이터베이스와의 연결을 위해 `Connector` 개념이 필요하게 되었다.
* 그러나 **Connector 역시 구현이 필요한 것으로, 이러한 기술이 성숙해가는 과정에서 커넥터의 표준인 ODBC가 등장**하게 된다.
  * 예를 들어, Java 진영의 대표적인 커넥터는 JDBC가 된다.
* 그러나 커넥터만으로는 데이터베이스와의 상호 작용에 번거로운 점이 존재하기에 그 이후로는 `xBatis`로 대두되는 Query Builder가 사용되었다.
* **ORM은 Query Builder 이후에 등장한 개념으로, 이렇듯 데이터베이스와의 상호작용은 Connector로부터 ORM까지 추상화**되어 왔다.

### 패러다임의 불일치
* 객체지향 패러다임을 적용하는 백엔드 애플리케이션의 경우, 유지보수성과 확장성을 위해 객체를 기반으로 한 설계 구조를 갖게 된다.
  * 반면, 데이터베이스의 경우 엔티티를 기반으로 테이블을 설계하는 구조를 갖게 된다.
* 이렇듯 두 개념은 패러다임 차원에서 본질적으로 다른 부분이 존재하며, 때문에 애플리케이션 개발 과정에서 패러다임의 불일치로 인한 어려움이 발생할 수 밖에 없다.
* 때문에 **ORM 개념이 도입되었으며, 이는 객체지향 애플리케이션과 데이터베이스 사이에서 두 개념 간의 패러다임의 불일치를 상당수 해소하는 역할을 수행**한다.

### 객체와 테이블의 매핑
* 애플리케이션과 ORM, 데이터베이스가 연결되는 관계에는 크게 다음과 같은 개념들이 존재한다.
  1. 클래스: 임의의 특징을 설명하기 위한 필드를 갖는다.
  2. 엔티티: 클래스의 필드를 표현하기 위한 속성(attribute)을 갖는다.
  3. 테이블: 엔티티의 속성은 테이블의 컬럼으로 표현된다.
* 또한, 클래스로부터 생성된 객체로부터 시작되는 다음과 같은 관계 역시 존재한다.
  1. 객체: 클래스에 new 연산자 등을 적용하여 생성된다.
  2. 튜플: 객체는 ORM의 튜플에 매핑된다.
  3. row: 튜플은 데이터베이스에 실제 데이터로서 저장되는 row로 매핑된다.
* 상술한 바와 같이, **서로 대응되는 개념을 연결시키는 일련의 과정을 매핑이라는 용어로 표현되며 이는 ORM이 전담하는 영역**이기도 하다.

## 2023-01-23 Mon
### 포워드 엔지니어링과 백워드 엔지니어링
* 뷰 계층에서 취합된 데이터는 논리 계층으로 넘어가며 ERD로 매핑되고, ERD는 다시 데이터베이스의 구조 설계로 이어진다.
  * 이 때, 상술한 바와 같은 애플리케이션 - ORM - 데이터베이스 구조에서 ERD 정보는 데이터베이스에 주입되어야 한다.
  * 반면, 이러한 ERD의 정보는 비즈니스 로직에도 지대한 영향을 주므로 애플리케이션에도 주입이 될 필요가 있다.
* 결국 ERD와 같은 정보는 애플리케이션과 데이터베이스, 나아가 ORM까지 모두가 알아야하는 정보에 해당하므로 이를 관리하는 방법론이 다음과 같이 나뉘게 된다.
  1. 포워드 엔지니어링: **ORM 설계를 활용하여 ERD 정보를 데이터베이스에 주입하는 방법론**이다.
  2. 백워드 엔지니어링: **데이터베이스 상의 DDL을 활용하여 ERD 구조를 설계한 후, 이를 ORM이 pull 해가는 방법론**으로 리버스 엔지니어링이라고도 한다.

### Prisma 설치하기
* Prisma는 package.json의 종속성 형태로 추가되므로, `npm i prisma` 명령어를 통해 간단히 설치할 수 있다.
* 그 후, `prisma init` 명령어를 입력하면 다음과 같이 진행되며 프로젝트에 Prisma가 초기 구성된다.
  * 이 때, `prisma/schema.prisma` 파일도 함께 생성된다.
```shell
[my-first-prisma-orm] prisma init

✔ Your Prisma schema was created at prisma/schema.prisma
  You can now open it in your favorite editor.

warn You already have a .gitignore file. Don't forget to add `.env` in it to not commit any private information.

Next steps:
1. Set the DATABASE_URL in the .env file to point to your existing database. If your database has no tables yet, read https://pris.ly/d/getting-started
2. Set the provider of the datasource block in schema.prisma to match your database: postgresql, mysql, sqlite, sqlserver, mongodb or cockroachdb.
3. Run prisma db pull to turn your database schema into a Prisma schema.
4. Run prisma generate to generate the Prisma Client. You can then start querying your database.

More information in our documentation:
https://pris.ly/d/getting-started
    
[my-first-prisma-orm]
```

### 애플리케이션과 ORM, 데이터베이스 연결하기
* 세 가지 요소가 서로 연결되기 위해서는 우선 데이터베이스의 정보를 ORM이 알아야 하며, 애플리케이션은 ORM을 통해서 데이터베이스에 연결되어야 한다.
  * 때문에 데이터베이스 인프라가 준비되면 해당 데이터베이스에 대한 연결 정보를 ORM에게 전달할 필요가 있다.
* NestJS와 Prisma의 예로 들어, 세 요소 간의 연결은 다음과 같은 요소들이 담당하게 된다.
  1. 애플리케이션과 ORM 간의 연결: NestJS 코드 상에 정의한다.
  2. ORM과 데이터베이스 간의 연결: **`prisma.schema` 파일에 명시**한다.

### prisma.schema
* `prisma.schema` 파일에는 데이터베이스 연결과 관련된 모든 정보를 명시하며, 이로 인해 다음과 같은 장단점이 존재할 수 밖에 없다.
  1. 파일 하나로 모든 것을 관리할 수 있으므로 편리하다.
  2. 그러나 해당 파일이 노출된 경우 큰 문제로 번지기 쉽다.
* 반면, `prisma init` 명령어를 통해 생성된 기본적인 `prisma.schema` 파일은 아래와 같은 형태를 가지며 각 구성 요소는 다음과 같은 의미를 갖는다.
  1. generator: 데이터 모델을 기반으로 어떤 클라이언트가 생성될지 명시한다.
  2. datasource: 데이터베이스 자체를 의미한다.
     1. provider: 연결 대상 데이터베이스의 엔진을 의미한다.
     2. url: 데이터베이스 호스트 주소를 의미하며, 하드코딩보다는 `env("환경 변수명")` 형태로 환경 변수를 활용하는 것이 바람직하다.
```
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```
* 백엔드 애플리케이션에서 정의한 객체는 데이터베이스가 곧장 이해할 수 없으므로 ORM을 통해 데이터베이스의 row로 변환되며, 이 때 client가 사용된다.
  * 즉, **`generator client`는 ORM 상에서 동작하며 백엔드 상의 객체를 데이터베이스로 매핑하는 Mapper 역할을 수행**한다.

### Prisma Service 정의하기
* NestJS 프로젝트에서 Prisma를 사용하기 위해, 다음과 같은 PrismaService를 정의한다.
  * 이 경우, `npm i @prisma/client` 명령어를 우선 입력하여 필요한 종속성을 준비해두어야 한다.
```typescript
import { Injectable, OnModuleInit, INestApplication } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect();
  }

  async enableShutdownHooks(app: INestApplication) {
    this.$on('beforeExit', async () => {
      await app.close();
    });
  }
}
```
* 이는 NestJS의 생명주기 이벤트와 관련이 있으며, 부트스트랩 이후의 단계에 해당하는 OnModuleInit 시점에 데이터베이스에 연결을 시도하는 점을 알 수 있다.
  * 반면, `enableShutdownHooks` 메소드에서는 애플리케이션이 종료되는 이벤트에 맞추어 데이터베이스와의 연결 종료를 시도하게 된다.

### Prisma Module 정의하기
* 상술한 Prisma Service를 애플리케이션 전역에서 활용할 수 있도록 다음과 같이 PrismaModule을 정의한다.
```typescript
import { Global, Module } from "@nestjs/common";
import { PrismaService } from './prisma.service';

@Global() // PrismaModule은 애플리케이션 전역에서 사용될 것으로 보여지므로, @Global() 데코레이터를 명시한다.
@Module({
  providers: [PrismaService],
  exports: [PrismaService], // PrismaService를 다른 모듈에서도 사용할 수 있도록 export한다.
})
export class PrismaModule {}
```

## 2023-01-24 Tue
### Prisma push 활용하기
* git과 마찬가지로, ORM이 데이터베이스에 데이터를 밀어 넣는 포워드 엔지니어링은 `push`라는 용어로 지칭한다.
  * 반면, 반대의 동작은 자연스레 `pull`이라는 용어로 지칭된다.
* 이를 활용하여 `prisma/schema.prisma`에 다음과 같이 새로운 엔티티를 명시한 후 `prisma db push` 명령어를 입력할 수 있다.
  * 해당 명령어를 실행할 경우, Prisma cli는 기본적으로 현재 위치로부터 `./prisma/schema.prisma` 파일을 탐색하므로 명령어 실행 위치에 주의한다.
```shell
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  // autoincrement에는 @기호를 명시하지 않음에 주의한다.
  id Int @id @default(autoincrement())
}
```
* 해당 명령어가 정상적으로 실행되면 다음과 같은 메시지가 노출되며 엔티티가 데이터베이스 테이블로 동기화되는 것을 확인할 수 있다.
```shell
[my-first-prisma-orm] prisma db push
Environment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Datasource "db": PostgreSQL database "my_first_prisma", schema "study" at "localhost:35432"

🚀  Your database is now in sync with your Prisma schema. Done in 326ms

✔ Generated Prisma Client (4.9.0 | library) to ./node_modules/@prisma/client in 51ms

[my-first-prisma-orm]
```

### generator란?
```
> Mapper는 ORM의 많은 기능 중 백엔드 애플리케이션 상의 객체를 데이터베이스에 전달하는 역할만을 수행하며, 코드 상에서는 Prisma Client 형태로 구현된다. 
```
* Mapper는 백엔드 애플리케이션 - ORM - 데이터베이스로 이어지는 관계에서, ORM 상에서 애플리케이션의 객체를 데이터베이스와 연결하는 역할을 맡는다.
* 이 떄, **이러한 Mapper를 Prisma에서는 generator라고 지칭**하며 `generate` 명령어를 통해 다음과 같이 생성할 수 있다.
```shell
[my-first-prisma-orm] prisma generate
Environment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma

✔ Generated Prisma Client (4.9.0 | library) to ./node_modules/@prisma/client in 1.81s
You can now start using Prisma Client in your code. Reference: https://pris.ly/d/client
```
import { PrismaClient } from '@prisma/client'
const prisma = new PrismaClient()
```
[my-first-prisma-orm] 
```
* 또한, **`prisma generate` 명령어를 활용하여 생성된 클라이언트는 `node_modules/.prisma/client` 상에 자동 생성된 코드의 형태로 존재**한다.
* 이렇듯 기본적으로는 `prisma db push` 명령어만으로도 데이터베이스와 Prisma 스키마가 동기화될 수 있다.
  * 그러나 **가능하다면 `prisma generate` 명령어를 이어서 입력하는 습관을 들여 두 개념 간의 동기화를 한 번 더 확인하는 것이 바람직**하다.

### Prisma pull 활용하기
* 상술한 바와 같이 데이터베이스로부터 `prisma/schema.prisma`로 동기화하는 것은 pull 동작에 해당하며, 다음과 같이 사용할 수 있다.
  * 이 때, 반드시 PG에 새로운 테이블을 생성한 후에 다음의 명령어를 입력한다.
```shell
[my-first-prisma-orm] prisma db pull
Prisma schema loaded from prisma/schema.prisma
Environment variables loaded from .env
Datasource "db": PostgreSQL database "my_first_prisma", schema "study" at "localhost:35432"

✔ Introspected 3 models and wrote them into prisma/schema.prisma in 180ms

┌────────────────────────────────────────────────────────────────────────┐
│                                                                        │
│  Upgrading from Prisma 1 to Prisma 2+?                                 │
│                                                                        │
│  The database you introspected could belong to a Prisma 1 project.     │
│                                                                        │
│  Please run the following command to upgrade to Prisma 2+:             │
│  npx prisma-upgrade [path-to-prisma-yml] [path-to-schema-prisma]       │
│                                                                        │
│  Note: `prisma.yml` and `schema.prisma` paths are optional.            │
│                                                                        │
│  Learn more about the upgrade process in the docs:                     │
│  https://pris.ly/d/upgrading-to-prisma2                                │
│                                                                        │
│  Once you upgraded your database schema to Prisma 2+,                  │
│  continue with the instructions below.                                 │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
      
Run prisma generate to generate Prisma Client.

[my-first-prisma-orm] 
```
* 이 때, **`prisma generate`를 암시적으로 실행하는 push 명령과는 달리 pull 명령에서는 반드시 Prisma Client를 별도로 갱신**할 필요가 있다.
  * 이러한 과정을 통해서만 ORM 상에 위치한 Mapper가 비로소 데이터베이스와 동기화될 수 있다.
* 이러한 특징으로 인해 하나의 마스터 서버와 다수의 워커 서버로 구성되는 백엔드 애플리케이션은 데이터베이스 변경 사항을 일반적으로 다음과 같이 처리하게 된다.
  1. 마스터 서버의 `schema.prisma`를 수정한 후, 이를 `prisma db push` 명령어로 데이터베이스에 반영한다.
  2. 모든 워커 서버는 `prisma db pull` 명령어를 통해 수정사항을 동기화한 후, `prisma generate` 명령어로 이를 코드에 반영한다.

### Prisma migrate 활용하기
* ORM, 즉 Prisma로부터 데이터베이스에 동기화하는 방법은 push 명령이 외에도 migrate 명령이 존재하며, 둘은 다음과 같은 장단점을 갖는다.
  1. push: 버전 관리 없이 데이터를 동기화하며, 대부분의 경우 오류가 발생하지 않아 사용이 쉽다.
  2. migrate: 마치 git과 같은 **버전 관리 개념을 지원하므로, 버전을 활용한 롤백이 쉬운 반면 파일 관리가 굉장히 중요하다**.
* 이 때, migrate 명령을 최초 실행하는 경우 버전 관리를 위해 우선 기존의 데이터를 모두 제거하므로 신중히 고민한 후에 진행하는 것이 바람직하다.
  * 또한 **migrate 명령은 push 명령과 마찬가지로 `prisma generate` 명령을 암시적으로 포함**한다.
```shell
[my-first-prisma-orm] prisma migrate dev
Environment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Datasource "db": PostgreSQL database "my_first_prisma", schema "study" at "localhost:35432"

Drift detected: Your database schema is not in sync with your migration history.

The following is a summary of the differences between the expected database schema given your migrations files, and the actual schema of the database.

It should be understood as the set of changes to get from the expected schema to the actual schema.

If you are running this the first time on an existing database, please make sure to read this documentation page:
https://www.prisma.io/docs/guides/database/developing-with-prisma-migrate/troubleshooting-development

[+] Added tables
  - Info
  - User
  - hello_table

✔ We need to reset the "study" schema at "localhost:35432"
Do you want to continue? All data will be lost. … yes # 이렇듯 모든 데이터가 제거될 수 있다는 점을 경고한다.

✔ Enter a name for the new migration: … v1 # migration의 이름을 명시할 경우, Prisma migrate 과정이 다음과 같이 진행된다.
Applying migration `20230123083433_v1`

The following migration(s) have been created and applied from new schema changes:

migrations/
  └─ 20230123083433_v1/
    └─ migration.sql

Your database is now in sync with your schema.

✔ Generated Prisma Client (4.9.0 | library) to ./node_modules/@prisma/client in 62ms


[my-first-prisma-orm]
```
* Prisma의 **migrate 명령이 완료될 경우, 데이터베이스에는 자동으로 `_prisma_migrations`라는 테이블이 생성**되게 된다.
* 상술한 절차에 따라 **최초의 migrate 명령을 실행했다면 버전 관리 준비가 완료되었므로, 두 번째의 migrate 명령부터는 변경 사항이 증분 형태로 관리**된다.
* 당연히 **migrate 명령은 큰 위험을 동반하며, 트러블 슈팅 또한 어려우므로 일반적으로는 개발 환경이 아니라면 사용이 권장되지 않는다**.

### schema.prisma의 특징
* **`schema.prisma`는 Prisma라는 ORM에 위치하지만, 동시에 ORM 뿐만 아니라 데이터베이스의 정보도 포함하는 특징**을 갖는다.
* 이 때, **`schema.prisma`의 내용 중 @ 기호가 명시된 것은 모두 데이터베이스의 정보인 반면 그렇지 않은 경우 Mapper 고유의 정보로 이해해도 무방**하다.
  * 이 중 **@ 기호가 명시된 정보는 누락되더라도 많은 경우에 Mapper가 default 값을 적용하여 동작하려는 경향**을 보인다.
* `model User {}`와 같은 엔티티의 경우, 각 요소에 따라 다음과 같은 명명이 적용된다.
  1. 애플리케이션: **엔티티의 이름은 camelCase로 변환되는 반면, 컬럼명은 엔티티에 명시된 내용이 그대로 적용**된다.
  2. ORM: `model User {}`에서 명시된대로 적용된다.
  3. 데이터베이스: 기본적으로 2.와 같은 형태로 적용되지만, `@@map("테이블명")` 키워드를 명시하여 원하는 테이블 명을 매핑할 수 있다.
* 뿐만 아니라 `schema.prisma` 파일에는 다음과 같은 특수한 목적의 키워드들을 용도에 맞게 명시할 수 있다.
  1. `@map()` 키워드는 엔티티에 정의된 컬럼에도 명시할 수 있으므로 컬럼명까지도 원하는대로 수정할 수 있다.
  2. `@id` 키워드는 PK를 의미한다.
  3. `@default()` 키워드는 컬럼의 기본 값을 명시한다.
  4. **`@db.VarChar(10)`과 같은 형태의 키워드를 사용하는 것으로 각 컬럼의 상세한 타입을 정의**할 수 있다.
  5. `@unique` 키워드로 UQ 제약 조건을 정의할 수 있다.
* 이러한 키워드를 활용하여 **작성한 `schema.prisma` 파일은 `prisma format` 명령어를 통해 가독성을 높이는 포매팅을 적용**할 수 있다.
  * 또한 **`prisma format` 명령어는 `schema.prisma`의 문법 오류도 검증하는 효과가 있으므로, push 전에 적용하는 습관을 들이는 것이 바람직**하다.

### 1:N 관계 표현하기
* `schema.prisma`에서 엔티티 간의 1:N 관계를 표현하는 예시는 다음과 같다.
```
model User {
  userId  Int    @id @default(autoincrement()) @map("USER_ID")
  name    String @default("unknown") @map("NAME") @db.VarChar(10)
  email   String @unique @map("EMAIL") @db.VarChar(50)
  profile String @map("PROFILE") @db.Text

  // 아래 컬럼은 가상 컬럼으로 취급된다.
  posts Post[] @relation("USER_HAS_MANY_POSTS")

  @@map("USER")
}

model Post {
  postId   Int    @id @default(autoincrement()) @map("POST_ID")
  content  String @map("CONTENT") @db.Text
  authorId Int    @map("AUTHOR_ID")
  
  // 아래 컬럼은 가상 컬럼으로 취급된다.
  author User @relation("USER_HAS_MANY_POSTS", fields: [authorId], references: [userId])

  @@map("POST")
}
```
* 상술한 코드에서, **가상 컬럼은 ORM에서는 존재하는 것처럼 취급되지만 데이터베이스에는 존재하지 않는 컬럼을 의미**한다.
  * 또한, 이 경우에는 가상 컬럼이 엔티티 간의 관계를 표현하는 데에 사용된 것으로 이해할 수 있다.
* 이 떄, `@relation` 키워드를 구성하는 fields와 references 속성은 각각 다음과 같은 의미를 갖는다.
  1. fields: 다른 테이블을 참조하는 FK 역할을 수행할 컬럼을 명시한다.
  2. references: FK의 실체가 관계를 맺는 상대 테이블의 어떤 컬럼인지 명시한다.
* 또한, 상술한 예시와 같이 `@relation` 키워드의 첫 번째 인자로서 각 엔티티 간의 관계를 구분하기 위한 별칭을 전달하는 것이 권장된다.
  * 이러한 관계 별칭은 누락되어도 무방하지만, 되도록이면 명시적으로 작성하는 것이 바람직하다.
  * 이렇게 **관계 별칭을 누락한 경우, 관계 별칭은 전혀 사용되지 않는 것이 아닌 Mapper에 의해 결정된 값으로 적용**된다.