# Basics
## 2023-01-13 Fri
### 엔티티 간 관계 설정
* 사용자와 게시물을 예로 들어, 사용자는 여러 게시물을 작성할 수 있는 동시에 게시물을 소유하는 1:N 관계를 맺는다.
  * 또한, **이러한 엔티티 간의 관계를 코드 상에서 표현하기 위해서는 각 엔티티에 서로를 가리키는 필드를 삽입할 필요**가 있다.
  * 이 때 사용자의 관점에서 게시물과의 관계는 `OneToMany`이며, 게시물의 관점에서는 `ManyToOne`이 된다.
* 이러한 사용자와 게시물의 관계를 각 엔티티에 새로운 필드를 추가하여 표현할 경우, 코드는 다음과 같이 정의할 수 있다.
```typescript
// 사용자 엔티티
@Entity()
@Unique(['username'])
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column()
  password: string;

  // 사용자의 입장에서 게시물과의 관계는 1:N, 즉 OneToMany이다.
  @OneToMany((type) => Board, (board) => board.user, { eager: true })
  boards: Board[];
}

// 게시물 엔티티
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

  // 게시물의 입장에서 사용자와의 관계는 N:1, 즉 ManyToOne이다.
  @ManyToOne((type) => User, (user) => user.boards, { eager: false })
  user: User;
}
```
* 이 때, 관계 데코레이터에 명시된 각 인자는 다음과 같은 의미를 갖는다.
  1. type: **해당 엔티티가 관계를 맺는 상대 엔티티의 타입 정보를 명시**한다.
  2. inverseSide: 관계를 맺는 **상대 엔티티로부터 자신에게 어떠한 경로로 접근해야하는지 명시**한다. 
  3. options.eager: **true로 설정한 경우, 해당 엔티티를 조회할 때 반대 편 엔티티를 함께 조회**한다.
* 이 경우, 엔티티 형태가 수정된 것이므로 사용자의 요청에 대해 컨트롤러로부터 리포지토리까지 흐름에 따라 사용자 정보가 함께 전달되도록 반드시 수정되어야 한다.
* 또한, 목록 조회시 임의의 사용자가 작성한 게시물만 나타나도록 리포지토리를 수정하는 경우 검색 조건을 다음과 같이 명시한다.
```typescript
async getAllBoards(user: User) {
  const [results, count] = await this.boardRepository.findAndCount({
    where: {
      user: {
        id: user.id,
      },
    },
  });
  return { results, count };
}
```
* 상술한 코드와 같은 맥락에서, 자신의 게시물만 삭제 할 수 있도록 코드를 수정하려면 다음과 같이 작성할 수 있다.
```typescript
// board.repository.ts
deleteBoardById(id: number, user: User) {
  return this.delete({ id, user: { id: user.id } });
}
```

### QueryBuilder 사용하기
* 반면, 최대한 raw 쿼리로 직접 작업하고 싶은 개발자들은 `QueryBuilder`를 활용하여 다음과 같이 목록 조회 코드를 작성할 수도 있다.
```typescript
async getAllBoards(user: User) {
  // 쿼리 빌더를 사용한 경우
  const [results, count] = await this.boardRepository
          .createQueryBuilder('board') // board 게시판에 대해 접근한다.
          .where('board.userId = :userId', { userId: user.id }) // where 절의 조건을 입력한다.
          .getManyAndCount();

  return { results, count };
}
```
* 그러나 **가능한 한 대부분의 작업은 ORM을 사용하되, 복잡한 경우에 한해서만 쿼리 빌더는 사용하는 것이 바람직**하다.

## 2023-01-14 Sat
### 로그의 종류와 로그 레벨
* 로그는 애플리케이션에서 문제가 발생했을 경우, 이에 대한 정보를 얻기 위해 매우 유용하게 사용될 수 있다.
* 이 때, 로그의 종류는 일반적으로 다음과 같이 분류될 수 있다.
  1. Log: 중요한 정보이며, 범용적인 로깅을 의미한다.
  2. Error: 치명적이거나 파괴적이며, 아직 처리되지도 않은 문제를 의미한다.
  3. Warning: 치명적이거나 파괴적이지는 않지만, 아직 처리되지 않은 문제를 의미한다.
  4. Debug: 개발자 용 로그이며, 오류 발생시 로직의 정보를 얻는 데에 유용한 정보를 의미한다.
  5. Verbose: 운영자 용 로그이며, 애플리케이션의 동작에 대한 통찰을 제공하는 정보를 의미한다.
* 또한, **상술한 로그들은 레벨 개념을 도입하여 다음과 같이 애플리케이션의 환경에 따라 출력할 정보를 선택하는 것이 일반적**이다.
  1. 개발 환경: Log, Error, Warning, Debug, Verbose 
  2. 스테이징 환경: Log, Error, Warning
  3. 운영 환경: Log, Error

### NestJS 애플리케이션 로그
* 일반적으로 개발 과정의 중간 중간에 로그를 삽입하는 경우가 많으며, Express 애플리케이션의 경우 Winston 라이브러리가 주로 사용된다.
  * 반면, NestJS 애플리케이션에서는 별도의 빌트인 모듈인 `logger` 클래스를 활용한다.
* 예를 들어 게시물 컨트롤러에서 사용자가 모든 게시물을 조회하고자 한 경우, 이러한 정보는 `logger.verbose()`를 활용하여 다음과 같이 구현할 수 있다.
```typescript
@Controller('boards')
@UseGuards(AuthGuard())
export class BoardsController {
  // logger를 컨트롤러의 private 프로퍼티로 정의하되, 로그의 출력 위치를 구분하기 위한 문자열을 생성자에 전달한다.
  private logger = new Logger('BoardsController');

  constructor(private boardService: BoardsService) {}
  
  @Get()
  getAllBoards(@GetUser() user: User) {
    this.logger.verbose(`User ${user.username} trying to get all boards`);
    return this.boardService.getAllBoards(user);
  }
}
```
* 이 경우, 사용자가 자신의 모든 게시물을 조회하고자 할 때 `logger.verbose()`는 콘솔에 다음과 같은 로그를 출력한다.
```shell
[Nest] 2041  - 01/14/2023, 4:00:55 PM VERBOSE [BoardsController] User injuk1 trying to get all boards
```