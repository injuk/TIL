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