# Basics
## 2023-01-07 Sat

### TypeORM이란?
* TypeORM은 TS로 작성된 객체 관계형 매퍼 라이브러리이며, MySQL이나 PostgreSQL 등 여러 관계형 데이터베이스를 지원한다.
* TypeORM 역시 일반적인 ORM을 사용하는 이유와 마찬가지로 객체와 관계형 데이터베이스의 데이터를 연결하기 위해 사용하며, 개발시 생산성과 유지보수성을 높인다.
  * 즉, TypeORM 역시 객체지향과 관계형 데이터베이스 사이의 패러다임 불일치로 인한 문제를 해결하기 위해 사용된다.

### TypeORM의 특징
* TypeORM은 모델을 기반으로 데이터베이스 테이블을 자동으로 생성하며, 데이터베이스에 객체를 손쉽게 삽입하거나 조회 및 삭제할 수 있도록 한다.
* TypeORM은 또한 테이블 간의 매핑 관계를 관리하며, 간단한 CLI 기능을 제공한다.
* TypeORM은 사용법이 간단하며, 다른 모듈과도 쉽게 통합된다는 장점 역시 존재한다.

### TypeORM을 애플리케이션에 적용하기
* NestJS 프로젝트에서 PostgreSQL(이하 pg)을 사용하는 경우, 다음과 같은 세 모듈을 우선 설치한다.
  1. pg: pg를 사용하기 위한 모듈이다.
  2. typeorm: TypeORM을 사용하기 위한 모듈이다.
  3. @nestjs/typeorm: NestJS에서 TypeORM을 사용할 수 있도록 연동하는 역할을 수행한다.
* 세 모듈을 프로젝트에 설치했다면, 다음과 같은 TypeORM 설정 파일을 작성한다.
  * 이 중 **`synchronize` 속성을 특히 주의해야 하며, true인 경우에는 애플리케이션 실행 시점에 엔티티가 변경되었다면 테이블을 DROP한 후 재생성**한다.
```typescript
import { TypeOrmModuleOptions } from '@nestjs/typeorm';

export const typeOrmConfig: TypeOrmModuleOptions = {
  type: 'postgres',
  host: '127.0.0.1',
  port: 5432,
  username: 'postgres',
  password: '비밀번호',
  database: '나의 데이터베이스',
  // 엔티티를 활용하여 데이터베이스 테이블을 생성하게 되므로, 엔티티 파일의 위치를 명시해준다.
  entities: [__dirname + '/../**/*.entity.{js,ts}'],
  // synchronize 옵션은 데이터 유실을 방지하기 위해 운영 환경에서는 반드시 false로 지정해두어야 한다.
  synchronize: true,
};
```
* 이제 이렇게 **작성한 TypeORM 설정을 애플리케이션에 연결하기 위해 최상위 모듈인 `app.module.ts`에서 다음과 같이 작성하여 import**한다.
  * 이 때, **`TypeOrmModule.forRoot()`는 모든 서브 모듈에 설정이 적용될 수 있도록 한다**.
```typescript
import { Module } from '@nestjs/common';
import { BoardsModule } from './boards/boards.module';
import { TypeOrmModule } from '@nestjs/typeorm';
import { typeOrmConfig } from './configs/typeorm.config';

@Module({
  imports: [BoardsModule, TypeOrmModule.forRoot(typeOrmConfig)],
})
export class AppModule {}
```