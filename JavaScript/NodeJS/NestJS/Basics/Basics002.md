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

## 2023-01-08 Sun
### 리포지토리를 서비스에 주입하기
* 컨트롤러 클래스가 서비스 클래스를 사용하듯, 서비스 클래스 역시 리포지토리 클래스를 사용하기 위해 다음과 같이 의존성을 주입한다.
  * 그러나 **리포지토리 클래스는 프로바이더가 아니므로 컨트롤러가 서비스 클래스를 주입 받던 예시와 달리 `@InjectRepository()` 데코레이터를 사용**한다.
  * 이렇듯 해당 데코레이터를 통해 서비스에서 원하는 리포지토리를 사용함을 명시할 수 있다.
```typescript
@Injectable()
export class BoardsService {
  constructor(
    @InjectRepository(BoardRepository) private boardRepository: BoardRepository,
  ) {}
}
```

### 리포지토리에서 데이터베이스 테이블을 조회하기
* TypeORM을 사용하는 리포지토리는 TypeORM이 제공하는 메소드를 활용하며, 상세 조회를 예로 들었을 떄 사용되는 메소드는 `findOneBy()`가 된다.
* 이 떄, 서비스에서 다음과 같이 작성해두는 것만으로도 무방하며 별도로 리포지토리 클래스를 수정할 필요는 없다.
```typescript
@Injectable()
export class BoardsService {
  constructor(
    @InjectRepository(BoardRepository) private boardRepository: BoardRepository,
  ) {}

  async getBoardById(id: number): Promise<Board> {
    const board = await this.boardRepository.findOne({ where: { id } });
    // 또는 아래와 같은 방식도 가능하다!
    // const board = await this.boardRepository.findOneBy({ id });
    if (!board) throw new NotFoundException(`there is no board with id ${id}`);

    return Promise.resolve(board);
  }
}
```

### 리포지토리에서 신규 리소스를 생성하기
* 리소스 생성 로직 역시 서비스와 컨트롤러에서 모두 작업되어야 하며, 이 때 리소스 생성 로직을 다음과 같이 간단하게 정의할 수 있다.
```typescript
async createBoard(dto: CreateBoardDto): Promise<Board> {
  const { title, description } = dto;

  const board = this.boardRepository.create({
    title,
    description,
    status: BoardStatus.PUBLIC,
  });
  await this.boardRepository.save(board);

  return board;
}
```
* 이 때, 하나의 메소드만으로 조회가 가능했던 상술한 조회 로직과 달리 리소스 생성에는 다음과 같은 두 메소드가 사용된다.
  1. create(): 해당 리포지토리에서 관리하는 엔티티의 인스턴스를 new 연산자로 생성하는 것과 같다.  
  2. save(): 주어진 단일 엔티티 또는 배열 형태로 정의된 엔티티 목록을 저장하되, 엔티티의 존재 여부에 따라 `createOrUpdate` 형태로 동작한다.
* 특히 **`save()` 메소드의 경우 단일 트랜잭션에서 엔티티의 생성 또는 업데이트를 수행**한다.
* 또한, 컨트롤러 역시 기존에 작성해둔 유효성 검증 데코레이터를 활용할 수 있도록 `@UsePipes()` 데코레이터를 다음과 같이 명시한다.
  * 이를 통해 **`createBoard` 핸들러에 전달되는 request body인 `CreateBoardDto`의 모든 프로퍼티는 `ValidationPipe`에 의해 검증**된다.
```typescript
@Post('/')
@UsePipes(ValidationPipe)
createBoard(@Body() dto: CreateBoardDto): Promise<Board> {
  return this.boardService.createBoard(dto);
}
```
* 그러나 리포지토리 패턴에서 데이터베이스와 관련된 작업은 리포지토리에서 작업하는 것이 바람직하므로, 상술한 서비스 로직의 위치를 다음과 같이 옮겨준다.
  * 이 경우, 리소스 생성과 관련된 컨트롤러 및 서비스의 메소드는 단순히 리포지토리의 메소드를 호출하기 위한 포워딩 역할만을 수행하게 된다.
```typescript
export class BoardRepository extends Repository<Board> {
  async createBoard(dto: CreateBoardDto): Promise<Board> {
    const board = new Board();
    board.title = dto.title;
    board.description = dto.description;
    board.status = BoardStatus.PUBLIC;

    // 또는 아래와 같이 작성해도 무방하다.
    // const board = this.create({
    //   title: dto.title,
    //   description: dto.description,
    //   status: BoardStatus.PUBLIC,
    // });

    await this.save(board);

    return board;
  }
}
```

### 리포지토리에서 리소스 삭제하기
* TypeORM의 경우, 리포지토리에서 제공하는 리소스 삭제 관련 메소드는 크게 다음과 같이 분류된다.
  1. remove(): **반드시 존재하는 리소스만을 제거해야 하며, 그렇지 않은 경우 404 에러가 발생**한다.
  2. delete(): 리소스가 존재하는 경우에만 제거하며, 그렇지 않은 경우에는 아무런 영향을 주지 않는다.
* 이러한 특징으로 인해 `remove()` 메소드는 일반적으로 리소스 조회 후 제거를 시도하게 되며, 때문에 DB IO를 최소한 두 번 사용하게 된다는 특징이 있다.
* 예를 들어 리소스 삭제를 위해서는 서비스 클래스에 다음과 같이 작성할 수 있다.
```typescript
async deleteBoardById(id: number): Promise<void> {
  const result = await this.boardRepository.delete(id);
  console.log(result);

  if (!result.affected)
    throw new NotFoundException(`there is no board with id ${id}`);
}
```
* 또한, 해당 메소드를 컨트롤러에서 호출하면서 StatusCode를 204로 변경하는 핸들러 메소드의 예시는 다음과 같다.
  * 나아가 path variable로 전달되는 id는 반드시 정수형이어야 하므로, `@Param()` 데코레이터에 유효성 검증을 위해 `ParseIntPipe`를 명시할 수 있다.
```typescript
@Delete('/:id')
@HttpCode(204)
deleteBoardById(@Param('id', ParseIntPipe) id: number): Promise<void> {
  return this.boardService.deleteBoardById(id);
}
```

### 리소스 수정하기
* 데이터베이스 상의 리소스를 수정하는 경우의 작업 흐름은 크게 다음과 같다.
  1. `find()` 등의 메소드를 활용하여 데이터베이스 상의 리소스를 조회한다.
  2. 조회한 리소스를 임의로 수정한다.
  3. `save()` 메소드를 활용하여 수정된 리소스를 저장한다.
* 상술한 흐름에 따라, 서비스 클래스에서 구현한 수정 메소드를 예로 들 경우 코드는 다음과 같다.
```typescript
async updateBoard(id: number, dto: UpdateBoardDto): Promise<Board> {
  const board = await this.getBoardById(id);

  const { title, description, status } = dto;
  if (title) board.title = title;
  if (description) board.description = description;
  if (status) board.status = status;

  await this.boardRepository.save(board);

  return board;
}
```

### 리포지토리에서 리소스 목록 조회하기
* 목록 조회는 단순히 리포지토리의 `find()` 메소드를 활용하여 다음과 같이 간단하게 구현할 수 있다.
```typescript
getAllBoards(): Promise<Board[]> {
  return this.boardRepository.find();
}
```
* 반면, 찾아낸 리소스 목록과 개수를 함꼐 반환하고자 하는 경우에는 다음과 같이 구현할 수도 있다.
```typescript
async getAllBoards() {
  const [results, count] = await this.boardRepository.findAndCount();
  return { results, count };
}
```