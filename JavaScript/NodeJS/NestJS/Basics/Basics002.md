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

### 엔티티 정의하기
* **관계형 데이터베이스의 테이블 엔티티 각각은 TypeORM에서 같은 클래스의 객체로 취급되므로, 이를 엔티티 클래스로 정의하여 테이블을 자동 생성할 수 있다**.
  * 쉽게 말해 TypeORM 상에서는 엔티티로 정의한 클래스가 관계형 데이터베이스의 테이블로 변환된다.
* 이 떄, 엔티티를 정의하기 위해 자주 사용되는 데코레이터들은 크게 다음과 같다.
  1. @Entity(): 해당 클래스가 엔티티임을 명시하기 위해 사용되며, `@Entity()` 데코레이터가 명시된 클래스는 관계형 데이터베이스의 테이블로 변환된다.
  2. @PrimaryGeneratedColumn(): 컬럼에 명시되는 데코레이터로서 해당 컬럼이 테이블의 PK이자, AI가 설정됨을 의미한다.
  3. @Column(): 역시 컬럼에 명시되는 데코레이터이며, 해당 프로퍼티가 일반적인 테이블 컬럼임을 의미한다.
* 상술한 내용을 토대로 간단한 엔티티 클래스를 다음과 같이 정의할 수 있다.
  * 이러한 엔티티 클래스를 정의한 후, NestJS 애플리케이션을 실행하면 해당 엔티티 클래스에 정의된 내용에 맞는 테이블이 pg에 자동으로 생성되게 된다.
```typescript
import { BaseEntity, Column, Entity, PrimaryGeneratedColumn } from 'typeorm';
import { BoardStatus } from './board.model';

@Entity()
export class Board extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  description: string;

  @Column()
  status: BoardStatus;
}
```
* 이 때, **엔티티 클래스의 이름은 관계형 데이터베이스에 생성되는 테이블의 이름을 결정짓기 때문에 반드시 적절한 이름을 명명하는 것이 바람직**하다.
  * **기본적으로 `lower_snake_case`를 채택하므로, 카멜 케이스로 작성된 클래스 이름은 자동으로 적절히 변환되어 적용**된다.
  * 또한, pg의 경우 별도의 설정이 없다면 해당 엔티티는 `public` 스키마에 생성된다.

### 리포지토리 정의하기
* 리포지토리는 데이터베이스 관련 작업을 담당하며, 이렇듯 서비스 클래스로부터 데이터베이스 관련 작업을 분리하는 것을 리포지토리 패턴이라고도 한다.
  * 때문에 리포지토리는 엔티티 객체와 함께 동작하며, 엔티티의 조회나 수정 및 생성과 삭제 등을 담당한다.
* TypeORM의 경우 간단한 리포지토리는 다음과 같이 정의할 수 있으며, EntityRepository와 Repository를 import하여 사용한다.
  * 리포지토리 클래스가 `Repository`를 상속하는 것으로 엔티티를 조회하거나 삽입 및 삭제 등의 작업을 수행할 수 있게 된다.
  * 반면, `@EntityRepository()` 데코레이터는 클래스를 사용자 정의 리포지토리로 명시하기 위해 사용한다.
  * 아래 코드의 경우, `@EntityRepository(Board)`와 같은 데코레이터를 명시하는 것으로 해당 리포지토리 클래스가 Board 엔티티를 관리함을 선언한다. 
```typescript
import { EntityRepository, Repository } from 'typeorm';
import { Board } from './board.entity';

@EntityRepository(Board)
export class BoardRepository extends Repository<Board> {
  
}
```
* 또한, **이렇게 생성한 리포지토리 모듈을 다른 서비스에서 import하여 사용할 수 있도록 도메인 모듈을 다음과 같이 수정**해야 한다.
```typescript
@Module({
  imports: [TypeOrmModule.forFeature([BoardRepository])],
  controllers: [BoardsController],
  providers: [BoardsService],
})
export class BoardsModule {}
```