# Basics
## 2023-01-31 Tue
### Prisma 실전팁
* `findFirst()`와 `findUnique()` 두 API 모두 하나의 결과만을 반환한다는 공통점을 갖는다.
    * 그러나 `findFirst()` API가 조건에 맞는 레코드 중 첫 번째를 반환한다면 `findUnique()` API는 정말 유일한 조건을 기반으로 레코드를 반환한다.
    * 때문에 **`findUnique()` API는 where 속성에 전달하는 검색 조건 자체가 PK 또는 UQ와 같이 유일성을 보장해야 한다**.
* 조회 시 **select 속성에 원하는 컬럼을 모두 명시하는 것은 반복적인 중복 코드가 될 수 있으므로, 별도의 모듈로 추출하여 사용하는 것이 일반적**이다.
* **include 속성은 select 속성과 함께 사용할 수 없으므로, 대신 select 내부에 가상 컬럼을 명시하여 조인을 유발**시킬 수 있다.
    * 이 경우, 가상 컬럼에 대해서 select 속성을 중첩 작성할 수 있다.
    * 이렇듯 Prisma의 조인 방법은 두 가지가 있으며, include 속성은 조인만 가능한 반면 select 속성은 원하는 컬럼만을 조회하면서도 조인을 수행할 수 있다.
* `createMany()` API의 경우, FK를 활용하는 가상 컬럼을 활용할 수 없다는 단점이 존재한다.

### Prisma 집계 API 사용하기
* `aggregate()` API는 개수나 합계 또는 최대값 / 최소값 및 평균 등을 손쉽게 구할 수 있도록 도우며, 다음과 같이 간단하게 사용이 가능하다.
```typescript
getAggregates() {
  return this.prisma.user.aggregate({
    // user의 레코드 수를 산출한다.
    _count: true,
    
    // userId의 평균 값을 산출한다.
    _avg: {
      userId: true,
    },
    
    // userId의 합계를 산출한다.
    _sum: {
      userId: true,
    },

    // 각각 userId의 최대값과 최소값을 산출한다.
    _max: {
      userId: true,
    },
    _min: {
      userId: true,
    },
  });
}
```

### distinct를 활용한 결과값 중복 제거
* `findMany()` API 등을 활용한 레코드 조회시, 결과로부터 특정 컬럼에 대해 중복을 제거하는 distinct 속성은 다음과 같이 사용할 수 있다.
```typescript
getWithDistinct() {
  return this.prisma.user.findMany({
    orderBy: {
      name: 'asc',
    },
    where: {},
    
    // 이 경우, name 컬럼으로부터 중복을 제거한다.
    distinct: ['name'],
  });
}
```
* 또한, **distinct 속성은 배열 형태로 전달되므로 당연히 여러 컬럼에 대한 중복 제거도 수행이 가능**하다.
  * 이 경우, **distinct 속성에 전달된 모든 컬럼을 쌍으로 취급하여 중복되는 조합만을 결과에서 제거**하게 된다.

### Prisma groupBy API를 활용하여 결과 그룹화하기
* 결과 레코드 목록을 특정한 컬럼을 기준으로 그룹화하고자 하는 경우, 다음과 같이 `groupBy()` API 를 활용하여 간단하게 구현할 수 있다.
  * 또한 일반적으로 **중복만을 제거하는 distinct와 달리, groupBy 연산은 집계 함수와 조합하는 등 더 고차원적인 연신을 적용**할 수 있다.
  * 이 과정에서 **having 절 역시 적용이 가능하며, `groupBy()` API 호출 시 속성으로 전달하여 조건에 맞는 그룹만을 선택**할 수 있다.
```typescript
groupBy() {
  return this.prisma.user.groupBy({
    // 이 경우, name 컬럼을 기준으로 그룹화한다.
    by: ['name'],
    
    // 또한, 각 그룹 별로 집계 연산을 적용할 수도 있다.
    _count: {
      _all: true,
    },
    
    // 또는 그룹화된 레코드 집합의 임의의 컬럼에 대해서만 집계 함수를 적용할 수도 있다.
    _sum: {
      userId: true,
    },

    // having 속성을 명시하는 것으로 사용자 Id의 합계가 1만보다 작은 그룹만을 조회할 수 있다.
    having: {
      userId: {
        _sum: {
          lt: 10_000,
        },
      },
    },
  });
}
```

## 2023-02-01 Wed
### connect를 활용하여 FK 참조 관계 추가하기
* FK로 참조되는 어떤 레코드에 대해 FK를 참조하는 새로운 레코드를 생성하는 경우, 다음과 같이 connect 속성을 활용하여 연결할 수도 있다.
  * 즉, **connect를 활용할 경우 FK로 참조되는 레코드를 굳이 생성하지 않더라도 기존재하는 레코드에 새로운 자식 레코드를 추가**할 수 있다.
  * 또한 이 경우, **FK 관계를 활용하는 connect 속성의 특성 상 해당 속성이 참조하는 컬럼은 반드시 유일성이 보장되어야 한다**.
```typescript
connectToUser() {
  return this.prisma.post.create({
    data: {
      content: faker.lorem.paragraph(),
      author: {
        connect: {
          userId: 1,
        },
      },
    },
  });
}
```

### 복합 키 조회시 주의사항
* 아래와 같은 모델이 `schema.prisma`에 정의되어 있다고 할 때, 해당 모델에 대해 `findUnique()` API를 호출한다면 특수한 검색 조건을 사용해야 한다.
```
model Cart {
  user_id Int
  book_id Int
  
  @@id([user_id, book_id])
}
```
* 상술한 모델을 예로 들어 where 속성에 적용해야 하는 특수한 검색 조건은 `user_id_book_id`의 형태와 같다.
  * 즉, **해당 검색 조건은 단순히 복합 키를 구성하는 컬럼들을 `_`로 이어 붙인 lower_snake_case 를 적용한 것**과 같다.
```typescript
getWithComposite(userId, bookId) {
  return this.prisma.user.findUnique({
    where: {
      user_id_book_id: {
        user_id: userId,
        book_id: bookId,
      }
    },
  });
}
```
* 이는 **where 속성에 검색 조건으로 복합키의 이름을 명시하기 위함이며, 특히 Prisma는 복합 키의 이름을 상술한 규칙과 같이 정의**한다.

### $queryRaw API를 활용한 로우 쿼리의 실행
* Prisma의 경우, raw query를 실행하기 위해 제공되는 API는 크게 다음과 같이 분류된다.
  1. `$queryRaw`: raw query를 실행하기 위한 API이다.
  2. `$queryRawUnsafe`: 마찬가지로 raw query를 실행할 수 있으나, sql 관련 공격을 당할 수 있는 API에 해당한다.
* 상술한 이유에서 일반적으로는 `$queryRaw` API가 선호되며, 다음과 같이 간단하게 사용할 수 있다.
  * 이 경우, raw query는 일반적인 sql 문법에 맞추어 작성한다.
```typescript
getByRawQuery() {
  return this.prisma.$queryRaw`select * from "study"."USER" where "USER_ID" = 1;`;
}
```
* 또는 PrismaClient에 포함되어 있는 `Prisma.sql`을 활용하여 다음과 같이 작성할 수도 있다.
  * 이러한 두 방식은 내부적으로 큰 차이가 없으며, 어디까지나 개발자 자신의 취향에 맞추어 `$queryRaw` API를 활용하는 것이 바람직하다.
```typescript
getByRawQuery() {
  return this.prisma.$queryRaw(
    Prisma.sql`select * from "study"."USER" where "USER_ID" = 1;`,
  );
}
```
* 또한, 이렇듯 raw query를 활용하여 DB와 상호작용 하는 방식은 크게 다음과 같은 장점이 존재한다.
  1. **Prisma ORM이 제공하는 기능을 활용했을 때보다 상대적으로 좋은 성능**을 보여준다.
  2. **Prisma ORM이 제공할 수 없는 복잡한 쿼리를 사용할 수 있도록 한다**.

### $executeRaw API를 활용한 로우 쿼리의 실행
* `$queryRaw` API와 마찬가지로 `$executeRaw` API 역시 raw query를 실행하면서도 안전하지 않은 버전의 API가 존재하고, 두 가지 사용 방식이 있다.
```typescript
getByRawQuery() {
  return this.prisma
    .$executeRaw`select * from "study"."USER" where "USER_ID" = 1;`;
}
```
* 그러나 `$executeRaw` API는 raw query 로 인해 조회되거나 영향을 받은 레코드의 개수를 반환한다.
  * **이러한 특징은 쿼리의 결과를 그대로 반환하는 `$queryRaw` API와 확연히 구분**된다.

### Prisma studio 활용하기
* 터미널에서 `prisma studio` 명령어를 입력할 경우, `http://localhost:5555`를 통해 접속할 수 있는 서버가 실행된다.
* `prisma studio`를 활용할 경우 프로젝트 초기 시점에 DB Admin 페이지 역할을 수행할 수 있으며, 데이터를 추가하거나 삭제하는 등의 관리 역시 가능하다.

## 2023-02-02 Thu
### MongoDB의 철학
* MongoDB는 다음과 같은 네 가지 철학을 주축으로 한다.
  1. 확장성: MongoDB는 수직 확장보다 수평 확장을 추구한다.
  2. 고성능: MongoDB 역시 다른 DB들과 마찬가지로 좋은 성능을 추구한다.
  3. 유연성: MongoDB는 자유로운 데이터의 적재를 추구하였으며, 이로 인해 자연스럽게 NoSQL이 된 케이스이다.
  4. 완전한 기능: MongoDB는 그 철학부터가 인덱싱이나 집계, 통계 등 사용자가 필요로 할 만한 기능을 모두 탑재하는 데에 목표를 둔다.

### MongoDB 구성 요소
* MongoDB는 클러스터에서 시작하여 여러 DB로 나뉘는 구조를 가지며, 각각의 DB는 다시 여러 개의 컬렉션으로 구성될 수 있다.
  * 이 때, 쉽게 생각하면 MongoDB의 컬렉션은 RDS의 테이블과 대응된다고 볼 수 있다.
* 또한, 개 별 데이터를 row로 지칭하는 RDS와는 달리 MongoDB에서는 이를 도큐먼트라는 표현으로 지칭한다.
  * 즉, **RDS 상에서 하나의 테이블에 여러 row가 저장된다면 MongoDB에서는 하나의 컬렉션에 여러 도큐먼트가 저장**된다.
  * 따라서 MongoDB의 가장 기본적인 데이터 단위는 도큐먼트가 된다.

### MongoDB와 동적 스키마
* MongoDB에 도큐먼트를 저장하는 경우, 사실상 어떠한 제한도 없이 자유롭게 데이터를 삽입할 수 있다.
  * 예를 들어, `name`이라는 프로퍼티는 때에 따라서 문자열이거나 정수 또는 아예 존재하지 않을 수 있다.
* 이러한 MongoDB의 특징을 동적 스키마라고 하며, 동적 스키마는 MongoDB가 추구하는 유연성의 근간이 된다.

### BSON이란?
```
> BSON은 MongoDB가 채택한 도큐먼트 표준이며, 바이너리 JSON이라는 의미를 갖는다.
```
* JSON 형태의 문법은 프로그래밍 언어에 따라 딕셔너리, 객체 등 지칭 방식이 다르며 내부적인 구현 방식도 다를 수 있다.
* 때문에 MongoDB는 도큐먼트에 JSON 형태의 문서를 모두 받아들일 수 있는 구조가 필요했으며, 이에 바이트 문자열 표기법을 적용한 BSON 개념을 도입하였다.
  * **MongoDB의 경우, 도큐먼트 당 최대 16MB의 BSON 문서를 저장할 수 있으며 내부적으로는 바이트 문자열 형태로 디스크에 쓰여진다**.