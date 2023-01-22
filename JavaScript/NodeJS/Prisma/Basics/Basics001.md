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
* 