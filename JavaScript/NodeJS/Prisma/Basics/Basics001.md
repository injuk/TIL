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