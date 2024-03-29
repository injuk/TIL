# Basics
## 2022-09-10 Sat

## 트랜잭션과 잠금
```
> 트랜잭션과 잠금은 서로 유사해보이지만, 잠금은 동시성을 제어하기 위한 기능인 반면 트랜잭션은 데이터의 정합성을 보장하기 위한 기능이라는 점이 다르다.
```
* **트랜잭션은 작업의 완전성을 보장하기 위해 존재**한다.
  * 즉, **논리적인 작업 세트를 모두 완벽하게 완료하거나 그렇지 못한 경우에는 원복하여 작업의 일부만이 적용되는 Partial Update를 미연에 방지**한다.
* 반면 **잠금은 여러 커넥션으로부터 동시에 동일한 레코드 또는 테이블과 같은 자원을 요청하는 경우, 이를 제어하기 위해 존재**한다.
  * 잠금이 없다면 단일 자원을 여러 커넥션에서 동시에 변경하게 되므로, 해당 레코드의 값을 예측할 수 없게 된다.
  * 그러나 **잠금 개념이 도입됨으로써 순서대로 한 시점에는 단 하나의 커넥션만 대상 자원을 변경할 수 있게 된다**.
* 나아가 **트랜잭션 격리 수준이란, 단일 트랜잭션 내에서 또는 여러 트랜잭션 간의 작업 내용을 어떻게 공유하고 차단할지 결정하는 레벨**을 의미한다.

### MySQL과 트랜잭션
```
> 트랜잭션은 반드시 여러 변경 작업을 수행하는 쿼리가 조합된 경우에만 의미가 있는 것은 아니다.
> 대신, 트랜잭션은 논리적인 작업 세트 내부에 쿼리가 몇 개가 있든 간에 해당 작업 세트 결과의 원자성을 보장하는 데에 의의가 있다.
```
* 몇몇 개발자는 트랜잭션 자체를 골치 아픈 기능으로 생각하곤 하며, 트랜잭션이 선택적인 MySQL 서버 환경에서는 특히 더 그렇다. 
  * 그러나 **트랜잭션은 애플리케이션 개발에서 반드시 고민해야 할 문제를 줄여주는 필수적인 DBMS의 기능**이다.
* 예를 들어, PK가 3인 컬럼이 이미 존재하는 테이블에 대해 각각 PK가 1, 2, 3인 레코드를 추가한다면 스토리지 엔진의 종류에 따라 결과는 다음과 같다.
  1. MyISAM, MEMORY: 트랜잭션이 없으므로, 1과 2의 추가에는 성공하나 3의 추가에 실패하여 결과는 `1, 2, 3` 세 개의 레코드를 남긴 채 쿼리를 종료한다.
  2. InnoDB: 트랜잭션을 지원하므로, 마찬가지로 3의 추가에서만 실패하지만 트랜잭션 처리를 위해 1과 2의 추가를 롤백하여 `3` 하나의 레코드만 남게 된다.
* 이렇듯 트랜잭션을 지원하지 않는 MyISAM 스토리지 엔진 환경에서는 부분 업데이트가 발생하기 쉽기 때문에 그만큼 테이블 데이터의 정합성을 맞추기 어렵다.
  * 부분 업데이트 현상이 발생한 경우, 실패한 쿼리로 인해 남은 불필요한 레코드를 다시 확인하여 삭제하는 재처리 작업이 필요할 수 있다.
  * 이러한 작업은 **단일 쿼리에서는 복잡하지 않을 수 있으나, 둘 이상의 쿼리가 실행되는 경우라면 재처리 작업 자체가 고되질 수 밖에 없다**.
  * **반면, InnoDB의 경우 쿼리 내용 중 일부라도 오류가 발생하면 상태를 되돌려야 한다는 트랜잭션의 원칙을 준수하므로 원자성이 보장**된다.
* **트랜잭션이 지원되지 않는 환경에서 작성된 코드는 이러한 재처리 로직을 추가하게 되어 코드의 퀄리티와 유지보수성이 크게 떨어지게 된다**.
  * 그렇다고 해서 재처리 로직을 추가하지 않는다면 쿼리 실행 실패 시 부분 업데이트의 결과로 쓰레기 데이터가 테이블에 남게 된다.
* 그러나 **트랜잭션을 지원하는 환경에서 작성한 코드는 이러한 재처리 로직을 작성할 필요 없이도 훨씬 간단한 코드로 완벽한 구현이 가능**하다.

## 2022-09-11 Sun
### 트랜잭션 주의사항
```
> 트랜잭션 역시 DBMS의 커넥션과 마찬가지로 반드시 필요한 최소한의 코드에만 적용하는 것이 바람직하다.
```
* 애플리케이션 코드 상에서 트랜잭션의 범위는 최소화되어야 하며, 불필요하게 광범위하게 적용된 트랜잭션은 좋지 않은 영향을 미칠 수 있다.
* **일반적으로 데이터베이스 커넥션은 개수가 제한적이나, 애플리케이션의 일부가 커넥션을 소유하는 시간이 길어질수록 가용한 여유 커넥션의 개수는 줄게 된다**.
  * 이러한 **상황이 악화될 경우, 애플리케이션의 일부가 커넥션을 가져가기 위해 대기해야하는 상황이 발생하기 쉽다**.
* 트랜잭션 범위 안에서 불필요한 네트워크 IO는 포함되지 않아야 한다.
  * 예를 들어, 트랜잭션 안에서 메일 등을 송신하여 사용자에게 알리는 방식의 구현은 지양해야 한다.
  * 이러한 실수로 인해 DBMS 서버가 예상치 못한 높은 부하 상태에 빠지는 등 위험 상황이 발생하기 쉽다.
  * **외부 IO는 트랜잭션 내부로부터 반드시 제거해야 하며, 외부 서비스가 가용하지 못한 경우 애플리케이션 뿐만 아니라 DBMS까지 위험해질 수 있다**.
* **반면, 단순한 데이터 확인 및 조회 쿼리의 경우 굳이 트랜잭션에 포함할 필요는 없다**.
* 이렇듯 **불필요한 작업을 많이 포함하는 광범위한 트랜잭션을 생성하기보다는 작업 단위 별로 성격에 따라 별도의 트랜잭션으로 분리하는 것이 바람직**하다.

### 트랜잭션 주의사항 - 정리
* **애플리케이션 코드 상에서, 코드가 트랜잭션을 활설화하고 있는 범위는 반드시 최소화**되어야 한다.
  * 이는 **코드 상에서 데이터베이스 커넥션을 갖는 범위는 반드시 최소화되어야 하는 것과 상통**한다.
* **외부 서비스와 상호작용하는 등의 IO 코드는 설령 그 코드 수가 한 두 줄 정도로 짧더라도 반드시 트랜잭션에서 제외**해야 한다.

### MySQL 서버 상의 잠금 분류
* MySQL에서 사용되는 잠금은 크게 다음과 같은 레벨로 분류할 수 있다.
  1. MySQL 엔진 레벨의 잠금
  2. 스토리지 엔진 레벨의 잠금
* 이 때, **MySQL 엔진은 MySQL 서버 상에서 스토리지 엔진을 제외하는 나머지 부분을 의미**한다. 
  * 이러한 **MySQL 엔진 레벨의 잠금은 모든 스토리지 엔진에 영향**을 미친다.
  * **반면, 스토리지 엔진 레벨의 잠금은 스토리지 엔진 간에는 상호적으로 영향을 주지 못한다**.
* 특히, MySQL 엔진에서는 다음과 같은 잠금 기능을 제공한다.
  1. 테이블 락: **테이블 데이터 동기화를 위해 사용**된다.
  2. 메타데이터 락: **테이블의 구조 자체를 잠그기 위해 사용**된다.
  3. 네임드 락: 사용자의 필요에 따라 사용할 수 있도록 제공되는 잠금 기능에 해당한다.

### 글로벌 락
* **글로벌 락은 `FLUSH TABLES WITH READ LOCK` 명령으로 획득할 수 있으며, MySQL이 제공하는 잠금 종류 중 가장 범위가 크다**.
  * 해당 명령은 실행과 동시에 MySQL 서버 상의 모든 테이블을 닫고 잠근다.
  * 해당 명령이 실행되기 전에 테이블 또는 레코드에 쓰기 잠금을 거는 SQL이 실행된 경우, 해당 명령은 읽기 잠금을 걸기 위해 앞선 SQL의 완료되기까지 대기한다.
* 일단 하나의 세션에서 글로벌 락을 획득했다면, 다른 세션에서는 SELECT 문을 제외한 대부분의 DDL과 DML을 실행할 수 없게 된다.
  * **정확히는 실행은 가능하나, 글로벌 락이 해제될 때까지 해당 문장은 대기 상태로 남게 된다**.
* **글로벌 락은 MySQL 서버 전체에 영향을 주며, 작업 대상 테이블이나 데이터베이스가 다르더라도 동일하게 영향을 준다**.
* 주로 여러 데이터베이스에 존재하는 MyISAM 또는 MEMORY 테이블에 대해 `mysqldump`로 일관된 백업을 받아야하는 경우에 사용한다.
* **글로벌 락은 MySQL 서버의 모든 테이블에 큰 영향을 주므로, 웹 서비스용으로 사용되는 MySQL 서버에는 가급적 사용하지 않는 것이 바람직**하다.

### 백업 락
* MySQL 서버가 업그레이드되면서 InnoDB 스토리지 엔진이 일반화되었으며, 이는 트랜잭션을 지원하므로 일관된 상태 보장을 위해 모든 변경을 멈출 필요는 없다.
* 무엇보다도 MySQL 8.0부터 InnoDB 엔진이 기본 스토리지 엔진으로 선정되었으므로, 조금 더 가벼운 글로벌 락의 필요성이 대두되었다.
  * 때문에 **MySQL 8.0 버전부터는 여러 백업 툴의 안정적인 실행을 위한 백업 락 개념이 도입**되었다.
* **임의의 세션에서 백업 락을 획득한 경우, 모든 세션에서 다음과 같은 변경이 불가능**하게 된다.
  1. 데이터베이스 또는 테이블 등의 모든 객체 생성 및 변경, 또는 삭제
  2. REPAIR TABLE과 OPTIMIZE TABLE 등의 명령
  3. 사용자 관리 및 비밀번호의 변경
* 반면, **백업 락의 경우 일반적인 테이블의 데이터 변경은 허용**한다.
* 백업 락은 긴 백업 과정이 단지 DDL 명령 하나만으로 실패하는 경우를 방지하기 위해 도입되었다.
  * 때문에 백업 락을 사용하더라도 복제는 정상적으로 진행되나, 백업의 실패를 막기 위해서 DDL 명령이 실행되는 경우 복제를 일시 중지하는 역할을 맡는다.

### 테이블 락
* 테이블 락은 개별 테이블 단위로 설정되는 잠금이며, 명시적 또는 암시적인 방식으로 임의의 테이블 락을 획득할 수 있다.
  * 예를 들어 **명시적으로는 `LOCK TABLES 테이블_이름 [ READ | WRITE ]` 명령으로 획득**할 수 있다.
  * 반면, `UNLOCK TABLES` 명령으로 획득한 잠금을 반납할 수 있다.
* 테이블 락은 MyISAM 뿐만 아니라 InnoDB 엔진을 사용하는 테이블에서도 동일한 설정이 가능하다.
* **명시적인 테이블 락 역시 글로벌 락과 동일하게 온라인 작업에 큰 영향을 주므로, 특수한 상황이 아니라면 애플리케이션에서 사용할 필요가 거의 없다**.
* **암시적인 테이블 락은 MyISAM 또는 MEMORY 테이블에서 데이터를 변경하는 쿼리를 실행한 경우에 발생**한다.
  * MySQL 서버는 데이터가 변경되는 테이블에 대해 잠금을 설정하고 나서 데이터를 변경하고, 그 이후에 즉시 잠금을 해제한다.
  * 이렇듯 **암시적인 테이블 락은 쿼리가 실행되는 과정에서 자동으로 획득되거나, 쿼리가 완료된 후에 반환**된다.
* 반면, **InnoDB의 경우 스토리지 엔진 차원에서 레코드 기반 잠금을 제공하므로 단순한 데이터 변경 쿼리로 인해 암시적인 테이블 락이 영향을 주지는 않는다**.
  * 정확히는 **InnoDB에서도 테이블 락이 설정되나, 대부분의 DML 쿼리에서는 무시**된다.
  * 그러나 **스키마를 변경하는 DDL 쿼리의 경우에는 테이블 락이 영향을 미친다**.

## 2022-09-12 Mon
### 네임드 락
* **네임드 락은 `GET_LOCK()` 함수를 활용하여 임의의 문자열에 대한 잠금을 획득하거나 반납**할 수 있다.
  * 이렇듯 네임드 락은 잠금 대상이 테이블이나 레코드, 또는 AUTO_INCREMENT와 같은 데이터베이스 객체가 아니다.
  * 대신 **단순히 사용자가 인자로 전달한 문자열에 대해 잠금을 획득하거나 반납하기 위해 사용**할 수 있다.
  * 이로 인해 네임드 락은 자주 사용되는 개념은 아니다.
* 예를 들어 1 대의 데이터베이스 서버에 여러 대의 웹 서버가 접근하는 경우, 어떠한 정보를 동기화해야할 필요성이 발생할 수 있다.
  * 이렇듯 **여러 클라이언트에 대한 상호 동기화를 처리해야하는 경우에 네임드 락을 활용하면 문제를 쉽게 해결**할 수 있다.
* 네임드 락과 관련하여 사용할 수 있는 MySQL 내장 함수는 다음과 같다.
  1. GET_LOCK(문자열, 대기시간): 첫 번째 인자로 전달한 문자열에 대해 두 번째 인자로 전달한 대기시간 초만큼 잠금을 획득하여 대기한다.
  2. IS_FREE_LOCK(문자열): 인자로 전달한 문자열에 대해 잠금이 설정되어 있는지 확인한다.
  3. RELEASE_LOCK(문자열): 인자로 전달한 문자열에 대해 잠금을 해제한다.
* 이 때, **상술한 내장 함수는 모두 정상적으로 잠금을 획득하거나 해제한 경우에는 1을 반환하며 그렇지 못한 경우에는 NULL 또는 0을 반환**한다.
* MySQL 8.0부터는 네임드 락과 관련하여 다음과 같은 기능이 신규 지원된다.
  1. 서로 다른 여러 문자열에 대해 네임드 락을 여러 번 중첩해서 사용할 수 있게 되었다.
     * 예를 들어, lock_1에 대한 작업과 lock_2에 대한 작업을 실행해야하는 경우를 가정한다.
     * 이 경우, 우선 GET_LOCK('lock_1', 10)을 실행한 후 lock_1에 대한 작업을 처리한다.
     * 그 후에 GET_LOCK('lock_2', 10)을 실행한 후 lock_2에 대한 작업을 처리한다.
     * 여기까지 현재 세션에서는 lock_1과 lock_2에 대한 잠금이 중첩되어 획득되었으며, 작업을 완료하기 전에 두 잠금을 모두 해제할 필요가 있다.
  2. 현재 세션에서 획득한 네임드 락을 `RELEASE_ALL_LOCKS()` 함수를 통해 한 번에 모두 해제할 수 있게 되었다.

### 네임드 락의 용도
* **네임드 락은 여러 레코드에 대해 복잡한 조건으로 레코드를 변경해야 하는 트랜잭션에 유용하게 사용**될 수 있다.
* 예를 들어, 배치 애플리케이션과 같이 한 번에 여러 레코드를 변경하는 쿼리는 자주 데드락을 발생시킨다.
  * 이 경우 **각 애플리케이션의 실행 시간을 분산하거나 코드를 수정하여 데드락을 최소화할 수는 있지만, 이 방식은 간단하지도 않고 완전하지도 않다**.
  * **대신 동일한 데이터를 변경하거나 참조해야만 하는 애플리케이션끼리 분류한 후, 네임드 락을 통해 쿼리를 실행하도록 하면 문제를 간단히 해결**할 수 있다.

### 메타데이터 락
* **메타데이터 락의 경우, 테이블 또는 뷰 등의 데이터베이스 객체에 할당된 이름이나 구조를 변경하는 경우에 획득하는 잠금**이다.
* 이는 **명시적으로 획득하거나 해제할 수 있는 개념은 아니며, 대신 `RENAME` 과 같이 테이블의 이름을 변경하는 경우에 자동으로 획득하는 잠금 유형**이다.
  * **`RENAME TABLE` 명령의 경우, 원본 이름과 변경될 이름 둘에 대해 모두 한 번에 잠금을 설정**한다.

### InnoDB 스토리지 엔진 상의 잠금
* **InnoDB 스토리지 엔진은 스토리지 엔진 내부에 자체적인 레코드 기반의 잠금 방식을 갖고 있다**.
  * 이는 **MySQL에서 제공하는 잠금과는 별개의 개념에 해당하며, 이로 인해 MyISAM과 비교하여 훨씬 뛰어난 동시성 처리를 제공**할 수 있다.
  * 그러나 이원화된 잠금 처리로 인해 InnoDB 스토리지 엔진 자체적으로 사용하는 잠금에 대한 정보는 MySQL 명령만으로 접근하기가 까다롭기도 하다.
  * 다행히도 최근 버전의 MySQL에서는 InnoDB 상에서 사용하는 트랜잭션과 잠금, 나아가 잠금 대기 중인 트랜잭션 목록을 조회할 수 있는 방법이 추가되었다.
* 상술했듯 **InnoDB 스토리지 엔진은 레코드 기반의 잠금을 제공하며, 이러한 잠금 정보는 상당히 작은 공간만으로 관리**된다.
  * 때문에 **레코드 락이 페이지 락으로, 나아가 테이블 락으로 확장되는 락 에스컬레이션과 같은 상황은 발생하지 않는다**.
* 또한, 다른 상용 DBMS들과는 달리 InnoDB 엔진에서는 레코드 락 뿐만 아니라 레코드 간의 간격을 잠그는 갭 락이라는 개념이 존재한다.

### 레코드 락
* **말 그대로 레코드 자체만을 잠그는 것을 레코드 락이라는 용어로 표현하며, 이는 다른 상용 DBMS의 레코드 락과 동일한 역할을 수행**한다.
  * 이 때, **InnoDB에서는 레코드 자체가 아닌 인덱스 상의 레코드를 잠근다는 점에서 주요한 차이가 있으며, 두 방식은 매우 큰 차이를 만들어낼 수 있다**.
  * 설령 **인덱스가 하나도 존재하지 않는 테이블의 경우에도 내부적으로 자체 생성된 클러스터 인덱스를 활용하여 잠금이 설정**된다.
* InnoDB에서 세컨더리 인덱스를 활용하는 변경은 넥스트 키 락 또는 갭 락을 사용한다.
  * 반면, **PK 또는 UQ 인덱스에 의한 변경 작업의 경우 갭은 잠그지 않고 레코드 자체에 대해서만 락을 설정**한다.

### 갭 락
* **갭 락은 레코드 자체가 아닌 레코드와 인접한 레코드 사이의 간격만을 잠그는 것을 의미하며, 다른 DBMS와의 주요한 차이점**이기도 하다. 
* **갭 락은 레코드와 레코드 사이의 간격에 새로운 레코드가 INSERT되는 것을 제어하는 역할을 수행**한다.
  * 갭 락은 그 자체로서 사용되기 보다는 넥스트 키 락의 일부로서 자주 사용된다.

### 넥스트 키 락
* **레코드 락과 갭 락을 합친 형태의 잠금을 넥스트 키 락이라는 용어로 지칭**한다.
* 갭 락과 넥스트 키 락은 바이너리 로그에 기록되는 쿼리가 레플리카 서버의 결과가 소스에서 실행된 결과와 동일한 결과를 생성하도록 보장하는 것이 주 목적이다.
  * 그러나 넥스트 키 락과 갭 락으로 인해 데드락이 발생하거나, 다른 트랜잭션을 대기하게 만드는 일이 자주 발생한다.
* **가능하다면 바이너리 로그 포맷을 STATEMENT가 아닌 ROW 형태로 바꾸어 갭 락과 넥스트 키 락을 줄이는 것이 바람직**하다.
  * MySQL 5.7과 8.0 버전에서는 ROW 형태의 바이너리 로그에 대한 안전성이 많이 높아졌으며, 이로 인해 STATEMENT 바이너리 로그의 단점을 많이 해결한다.
  * 때문에 MySQL 8.0 버전부터는 ROW 형태의 바이너리 로그가 기본 설정으로 변경되었다.

### AUTO_INCREMENT 락
* MySQL의 경우, 자동으로 증가하는 숫자 값을 알아내기 위해 `AUTO_INCREMENT`라는 컬럼 속성을 제공한다.
  * 해당 속성이 적용된 테이블에 동시에 여러 레코드가 INSERT되는 경우에도 저장되는 각 레코드는 중복되지 않고 순차대로 증가하는 일련 번호를 가져야 한다.
  * **InnoDB에서는 이를 위해 내부적으로 AUTO_INCREMENT 락이라고 하는 테이블 수준의 잠금 기능을 사용**한다.
* **AUTO_INCREMENT 락은 INSERT 또는 REPLACE 쿼리와 같이 새로운 레코드를 저장하는 쿼리에서만 필요**하다.
  * 따라서 UPDATE 또는 DELETE 쿼리에서는 해당 잠금이 설정되지 않는다.
* **InnoDB에서 사용하는 레코드 락, 넥스트 키 락과 같은 다른 잠금과는 달리 AUTO_INCREMENT 락은 트랜잭션과 관계 없이 획득되고 반환**된다.
  * 예를 들어, **INSERT 또는 REPLACE 쿼리에서 AUTO_INCREMENT 값을 가져오는 순간에만 잠금이 획득된 후 즉시 해제**된다.
* **AUTO_INCREMENT 락은 테이블 당 단 하나만 존재**한다.
  * 때문에 두 개의 INSERT 쿼리가 동시에 실행된 경우, 둘 중 하나의 쿼리가 우선 AUTO_INCREMENT 락을 획득하게 된다.
  * 이 때, **잠금을 획득하지 못한 다른 쿼리는 AUTO_INCREMENT 락이 해제되는 것을 대기**해야만 한다.
* **AUTO_INCREMENT 속성이 설정된 컬럼에 명시적으로 값을 설정하는 경우에도 마찬가지로 AUTO_INCREMENT 락을 걸게 된다**.
* **AUTO_INCREMENT 락을 명시적으로 획득하거나 해제하는 방법은 존재하지 않는다**.
  * 그러나 해당 잠금은 아주 짧은 시간 동안만 걸렸다가 해제되므로 큰 문제가 되지 않는다.
* **AUTO_INCREMENT 값이 한 번 증가하면 절대 줄어들지 않는 이유는 AUTO_INCREMENT 잠금 자체를 최소화하기 위함**이다.
  * 때문에 **INSERT 쿼리가 실패하더라도 한 번 증가된 AUTO_INCREMENT 값은 다시 줄지 않고 그대로 남게 된다**.

### MySQL 5.1 이후의 AI 락
* AI 락에 대한 상술한 설명은 모두 MySQL 5.0 버전 이전에 대한 설명이며, 5.1 버전 이후로는 다음과 같은 시스템 변수를 활용하여 동작을 변경할 수 있다.
  1. `innodb_autoinc_lock_mode=0`: MySQL 5.0 버전과 동일한 잠금 방식이며, 모든 INSERT 문장에 대해 AI 락을 사용한다.
  2. `innodb_autoinc_lock_mode=1`: 경우에 따라 기존의 AI 락 또는 훨씬 가볍고 빠른 래치(뮤텍스) 방식을 통해 처리한다.
  3. `innodb_autoinc_lock_mode=2`: 절대 AI 락을 사용하지 않고, 경량화된 래치 방식을 사용한다.

### innodb_autoinc_lock_mode=1 설정
* 단순히 한 건 또는 여러 건의 레코드를 INSERT 하는 쿼리에서, MySQL 서버가 레코드 수를 정확히 예측할 수 있는 경우에는 래치를 사용한다.
  * **개선된 래치 방식은 AI 락보다 훨씬 짧은 시간에만 잠금을 걸고, 필요한 AI 값을 가져온 즉시 잠금을 해제**한다.
* 그러나 **레코드 건수를 정확히 예측할 수 없는 경우에는 5.0 버전과 같은 AI 락을 사용하며, 이 경우에는 다른 커넥션의 INSERT 쿼리는 대기**된다.
* **해당 방식에서의 대량 INSERT의 경우 InnoDB는 여러 AI 값을 한 번에 할당받아 INSERT되는 레코드 각각에 사용**한다.
  * 그러나 한 번에 할당받은 AI 값을 전부 사용하지 못한 경우, 사용되지 못한 AI 값은 모두 폐기된다.
  * 따라서 **대량 INSERT 문이 실행된 이후에 INSERT되는 레코드의 AI 값은 연속하지 않고 누락된 값이 발생할 수도 있다**.
  * 이로 인해 **해당 설정에서는 최소한 하나의 INSERT 문장으로 생성된 레코드는 반드시 연속된 AI 값을 갖게 된다**.
  * 이러한 이유에서 해당 설정을 연속 모드라고도 지칭한다.

### innodb_autoinc_lock_mode=2 설정
* **해당 설정 모드에서 InnoDB는 절대 기존의 AI 락을 걸지 않는 대신, 경량화된 래치 방식을 사용**한다.
* 그러나 **해당 방식은 연속 모드와 달리 하나의 INSERT 문장으로 실행되어 INSERT 된 레코드라고 하더라도 연속된 AI 값을 보장하지는 않는다**.
  * **대신 해당 설정에서의 AI 기능은 반드시 유일한 값이 생성된다는 것만을 보장**한다.
  * 때문에 해당 설정을 인터리빙 모드라고도 지칭한다.
* **해당 설정 모드에서는 대량의 INSERT 문이 실행되는 도중에도 다른 커넥션으로부터 INSERT를 수행할 수 있으므로 동시 처리 성능이 높아진다**.
* STATEMENT 형의 바이너리 로그를 사용하는 복제의 경우, 레플리카 서버와 소스 서버의 AI 값이 달라질 수도 있기 때문에 반드시 주의를 기울여야 한다.
  * 그러나 MySQL 8.0부터는 바이너리 로그 포맷의 기본 값이 ROW이므로, 해당 설정 모드를 기본으로 사용한다.
  * 그 **이전 버전 또는 MySQL 8.0 이지만 바이너리 로그 포맷이 STATEMENT 형태인 경우, 연속 모드의 사용이 권장**된다.

## 2022-09-13 Tue
### InnoDB의 잠금과 인덱스
```
> InnoDB 스토리지 엔진에서는 잠금과 인덱스가 매우 중요한 연관 관계를 갖는다.
```
* 상술한 바와 같이, **InnoDB에서의 레코드 락은 임의의 레코드를 잠그는 것이 아니라 인덱스 자체를 잠그는 방식으로 처리**된다.
  * 즉, **변경 대상 레코드를 찾기 위해 검색했던 모든 인덱스의 레코드를 모두 잠금 처리**해야 한다.

### 인덱스 잠금의 예시
* 예를 들어 employees 테이블에 약 30만 건의 데이터가 존재하며, 다음과 같은 추가 조건이 주어졌다고 가정한다.
  1. 테이블의 first_name 컬럼에 대해서만 인덱스가 적용되었다.
  2. 전체 레코드 중 first_name이 Georgi인 사원은 253명이 존재한다.
  3. 전체 레코드 중 first_name이 Georgi이고 last_name이 Klassen인 사원은 단 1명만 존재한다.
  4. 이 때, `UPDATE employees SET hire_date=NOW() WHERE first_name='Georgi' AND last_name='Klassen';` 쿼리를 실행한다.
* 상술한 조건에 의해 **UPDATE 문으로 수정되는 레코드는 단 1건만 존재하지만, 이를 위해 253개의 레코드를 모두 잠그게 된다**.
  * **253은 first_name이 Georgi인 레코드의 수를 말하며, UPDATE 문을 처리하기 위해 사용한 인덱스에서 스캔한 모든 레코드를 잠그는 것을 의미**한다.
* 때문에 UPDATE 문을 처리하기 위해 적절한 인덱스가 준비되어 있지 않은 경우 각 클라이언트 간의 동시성은 크게 떨어지게 된다.
  * 이는 **관련된 레코드를 모두 잠그기 때문이며, 한 세션에서 UPDATE 문을 요청했다면 다른 클라이언트는 해당 테이블을 업데이트하지 못하고 대기**해야 한다.
  * 당연히 이러한 MySQL의 특징을 잘 모르고 개발을 진행하게 되는 경우 MySQL 서버를 제대로 다룰 수 없다. 
* UPDATE 문을 요청한 **대상 테이블에 인덱스가 전혀 존재하지 않는다면 내부적으로 자동 생성되는 클러스터 인덱스를 활용하여 레코드를 잠그게 된다**.
  * 이는 **최악의 경우에 해당하며, 테이블을 풀 스캔하며 UPDATE 작업을 처리하기 위해 30만 건의 모든 레코드를 잠그게 된다**.
  * 이러한 이유에서 MySQL 서버에서 InnoDB 스토리지 엔진을 사용하는 경우에는 인덱스 설계가 더더욱 중요하다.

### 레코드 수준의 잠금 확인과 해제
* InnoDB 스토리지 엔진을 사용하는 테이블에 걸린 레코드 수준의 잠금은 다음과 같은 이유에서 테이블 수준의 작업보다 복잡하다.
  1. 테이블 잠금의 경우, 잠금 대상이 테이블 자체이므로 문제의 원인은 쉽게 발견되고 해결된다.
  2. **레코드 잠금의 경우, 잠금 대상이 테이블의 레코드 각각이므로 잠긴 레코드가 자주 사용되지 않는다면 오랜 시간 잠긴 상태로 방치될 가능성이 존재**한다.
* 예전 버전의 MySQL 서버에서는 레코드 잠금에 대한 메타 정보(딕셔너리 테이블)를 전혀 제공하지 않았기에 이러한 정보를 확인하기는 더더욱 어려웠다.
  * 그러나 **MySQL 5.1 버전부터는 레코드 잠금과 잠금 대기에 대한 조회가 쿼리 한 줄만으로 가능하도록 변경**되었다.
  * **이를 통해 잠금 또는 잠금 대기 정보를 확인하고, 강제로 잠금을 해제하기 위해 KILL 명령을 통해 MySQL 서버의 프로세스를 강제 종료**시킬 수 있다.

### 레코드 변경 작업 중 발생한 잠금 경합에 대한 레코드 수준 잠금 확인하기
* **각 트랜잭션이 어떤 잠금을 대기하고, 대기하고 있는 잠금을 어떤 트랜잭션이 갖는지 조회할 수 있는 메타 데이터는 버전에 따라 달리 제공**된다.
* MySQL 5.1 버전부터는 다음과 같은 메타 데이터를 확인할 수 있다.
  1. DB: `information_schema`
  2. 테이블: `INNODB_TRX`, `INNODB_LOCKS`, `INNODOB_LOCK_WAITS`
* 그러나 MySQL 8.0 버전부터는 `information_schema`의 정보들이 점차 deprecated 처리되고 있으며, 대신 다음과 같은 메타 데이터로 대체되고 있다.
  1. DB: `performance_schema`
  2. 테이블: `data_locks`, `data_lock_waits`
* 예를 들어 여러 UPDATE 문이 실행된 프로세스 목록을 조회하고자 하는 경우, `SHOW PROCESSLIST` 명령을 사용할 수 있다.
  * **이는 테이블의 레코드에 여러 커넥션으로부터 동시에 가해진 UPDATE 문과 같은 변경 작업으로 인해 잠금 경합이 발생한 경우에 대한 예시에 해당**한다.
  * **첫 커넥션의 UPDATE 처리에 문제가 발생하여 오랜 시간 동안 COMMIT 되지 않은 경우, 다른 커넥션들은 잠금이 해제될 때까지 대기**한다.
  * 이는 **전형적인 잠금 경합에 해당하며, 잠금 경합이 발생한 스레드에 대한 개략적인 정보는 상술한 `SHOW PROCESSLIST;` 명령을 활용**할 수 있다.
* 더 정확한 잠금 대기 순서를 확인하고자 하는 경우, performance_schema의 data_locks와 data_lock_waits 테이블을 다음과 같이 조인할 수 있다.
```
SELECT
  r.trx_id waiting_trx_id,
  r.trx_mysql_thread_id waiting_thread,
  r.trx_query waiting_query,
  b.trx_id blocking_trx_id,
  b.trx_mysql_thread_id blocking_thread,
  b.trx_query blocking_query
FROM performance_schema.data_lock_waits w
INNER JOIN information_schema.innodb_trx b
  ON b.trx_id = w.blocking_engine_transaction_id
INNER JOIN information_schema.innodb_trx r
  ON r.trx_id = w.requesting_engine_transaction_id;
```
* 해당 쿼리의 실행 결과는 다음과 같이 이해할 수 있다.
  1. waiting_thread: 현재 대기 중인 스레드 ID를 확인할 수 있다.
  2. blocking_thread: 각 스레드가 어떤 스레드의 완료를 기다리고 있는지 확인할 수 있다.
* 이를 통해 어떤 스레드에 의해 각 UPDATE 문이 실행 대기 상태에 있는지 알 수 있으며, **더 자세한 정보는 data_locks 테이블의 모든 컬럼을 확인**한다.
  * 이 때, **쿼리는 `SELECT * FROM performance_schema.data_locks`와 같이 실행**한다.
* 이러한 **메타 데이터를 통해 어떤 스레드가 잠금을 가진 상태에서 대기 중인지 확인했다면, `KILL 스레드ID` 명령을 통해 스레드를 강제 종료시킬 수 있다**.
  * 이 과정에서 **명시된 스레드 ID를 갖는 스레드가 강제 종료되며 잠금이 반환되어 나머지 대기 중인 UPDATE 명령들이 실행되고, 잠금 경합은 종료**된다.

### 트랜잭션 격리 수준
```
> 일반적인 DBMS에서는 주로 READ COMMITTED 또는 REPEATABLE READ 격리 수준을 사용한다.
```
* **여러 트랜잭션이 동시에 처리되는 경우, 특정한 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있도록 허용할지 말지를 결정할 수 있다**.
* **이를 결정하는 것이 트랜잭션의 격리 수준으로 크게 다음과 같은 네 가지로 분류되며 아래로 갈수록 격리, 즉 고립 수준은 높아**진다.
  1. READ UNCOMMITTED
  2. READ COMMITTED
  3. REPEATABLE READ
  4. SERIALIZABLE
* **격리 수준이 높아지더라도 SERIALIZABLE 수준까지 가지 않는 이상 크게 성능이 떨어지지는 않는다**. 
* DIRTY READ라고도 불리우는 READ UNCOMMITTED 수준은 일반적인 데이터베이스에서는 거의 사용되지 않는다.
* 오라클과 같은 DBMS에서는 주로 READ COMMITTED 수준을 사용한다.
* MySQL에서는 REPEATABLE READ 수준을 주로 사용한다.
* SERIALIZABLE 격리 수준의 경우 동시성이 중요한 데이터베이스에서는 거의 사용되지 않는다.

### 세 가지 부정합 문제
* **데이터베이스의 격리 수준과 함께 다뤄지는 문제 중 세 가지 부정합 문제가 존재하며, 각 문제는 격리 수준에 따라 발생**할 가능성이 있다.
  1. DIRTY READ: READ UNCOMMITTED 격리 수준에서만 발생할 수 있다.
  2. NON-REPEATABLE READ: READ UNCOMMITTED, READ COMMITTED 격리 수준에서 발생할 수 있다.
  3. PHANTOM READ: READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ 격리 수준에서 발생할 수 있다.
* 이 때, **InnoDB가 갖는 특성으로 인해 REPEATABLE READ 격리 수준에서도 PHANTOM READ 부정합 문제는 발생하지 않는다**.

### READ UNCOMMITTED 격리 수준
* **READ UNCOMMITTED 격리 수준에서는 임의의 트랜잭션에서의 변경 내용이 COMMIT 또는 ROLLBACK 여부와 관계 없이 다른 트랜잭션에서 보여진다**.
* 예를 들어, 각각 트랜잭션을 사용하는 두 사용자가 다음과 같은 시나리오로 진행한다.
  1. 사용자 1이 트랜잭션을 시작(BEGIN)한다.
  2. 사용자 1이 임의의 테이블에 INSERT 문을 통해 새로운 레코드를 추가한다.
  3. **이 시점에서 사용자 2가 SELECT 문을 통해 해당 테이블을 조회할 경우, 2.에서 추가된 레코드를 조회할 수 있다**.
  4. 사용자 1이 INSERT 결과를 COMMIT한다.
* 이렇듯 **READ UNCOMMITTED 격리 수준에서는 사용자 1이 변경한 내용을 커밋하지 않았더라도 사용자 2가 해당 레코드를 검색**할 수 있다.
  * 이 때, **사용자 1이 작업 도중 문제가 생겨 INSERT된 레코드를 ROLLBACK하더라도, 사용자 2는 이를 알 방법이 없다는 치명적인 문제가 존재**한다.

### DIRTY READ 부정합
```
> 더티 리드란 임의의 트랜잭션에서 처리 중인 작업이 COMMIT 또는 ROLLBACK 등을 통해 완료되지 않았는데도 다른 트랜잭션에서 조회할 수 있는 현상을 말한다.
```
* READ UNCOMMITTED 격리 수준은 더티 리드를 허용하며, 이로 인해 데이터가 나타났다가 사라지는 괴 현상이 발생하기 쉽다.
  * 당연히 **더티 리드 현상은 애플리케이션 개발자와 사용자를 혼란스럽게 만들 수 있는 요인 중 하나**이다.
* 이러한 이유에서 더티 리드를 유발하는 READ UNCOMMITTED는 RDBMS 표준에서는 트랜잭션 격리 수준으로 인정조차 하지 않을 정도의 격리 수준이다.
  * 이는 **READ UNCOMMITTED 격리 수준이 그만큼 정합성에 많은 문제를 일으키는 것으로 이해**할 수 있다.
  * 따라서 MySQL을 사용하는 경우, 최소한 READ COMMITTED 이상의 격리 수준을 사용하는 것이 바람직하다.

## 2022-09-14 Wed
### READ COMMITTED 격리 수준
```
> READ COMMITTED 격리 수준은 오라클 DBMS의 기본 격리 수준이며, 온라인 서비스에서 가장 많이 선택되는 격리 수준이기도 하다.
```
* **READ COMMITTED 격리 수준에서는 임의의 트랜잭션에서 데이터를 변경했더라도 COMMIT까지 완료된 데이터만 다른 트랜잭션에서 조회가 가능**하다.
  * 때문에 완료되지 않은 작업을 다른 트랜잭션에서 조회할 수 있는 더티 리드 부정합은 애초에 발생하지 않는다.
* READ COMMITTED 격리 수준에서, 임의의 트랜잭션이 수행하는 UPDATE 쿼리는 다른 트랜잭션에서 다음과 같은 흐름으로 조회된다.
  1. 사용자 1이 트랜잭션을 시작한다.
  2. 사용자 1이 임의의 레코드에 대한 UPDATE 쿼리를 통해 컬럼 값을 수정을 시도한다.
  3. **사용자 1이 변경한 새로운 값은 테이블에 즉시 기록되고, 이전 값은 언두 영역으로 백업**된다.
  4. **사용자 1이 COMMIT하기 전에 사용자 2가 해당 레코드를 조회할 경우, 테이블이 아닌 언두 영역에 존재하는 변경 전 레코드가 조회**된다.
  5. 사용자 1이 변경 내용을 COMMIT한다.
  6. 사용자 2가 다시 해당 레코드를 조회할 경우, 이제는 변경된 레코드의 값을 조회할 수 있다.
* **이렇듯 READ COMMITTED 격리 수준에서는 임의의 트랜잭션으로부터 발생한 변경 사항이 COMMIT되기 전까지 다른 트랜잭션에서 이를 조회할 수 없다**.

### NON-REPEATABLE READ 부정합
* 상술한 **READ COMMITTED 격리 수준에서 더티 리드는 발생하지 않지만, 대신 REPEATABLE READ가 불가능한 부정합이 발생**할 수 있다.
* 이 때, 해당 부정합은 다음과 같은 흐름으로 유발할 수 있다.
  1. 사용자 1이 트랜잭션을 시작하고, 아직 존재하지 않는 레코드에 대해 조회할 경우 결과는 아무 것도 반환되지 않는다.
  2. **사용자 1의 트랜잭션이 종료되기 전에, 사용자 2가 트랜잭션을 시작하고 기존 레코드의 내용을 1.에서 조회한 검색 조건과 일치하도록 수정**한다.
  3. **사용자 1이 다시 1.과 같은 검색 조건으로 조회할 경우, 이번에는 조회에 성공하여 검색 결과를 확인**할 수 있다. 
* 이는 **동일한 트랜잭션 내에서 발생한 SELECT와 같은 조회 쿼리는 항상 같은 결과를 반환해야한다는 REPEATABLE READ 정합성에 부합하지 않는 결과**이다.
* 이와 같은 부정합은 일반적인 웹 애플리케이션에서는 큰 문제를 일으키지 않는다.
  * 그러나 **하나의 트랜잭션 안에서 동일한 데이터를 여러 번 읽고 수정하는 작업이 금전적인 처리와 연관될 경우, 큰 문제가 발생**할 수 있다.
  * 예를 들어, 다른 트랜잭션에서 입금과 출금이 반복적으로 진행될 때 또 다른 트랜잭션에서 당일 입금액의 총합을 조회하는 경우가 있다.
  * 이 경우, 매 SELECT 쿼리가 실행될 때마다 다른 결과를 반환하게 된다.
* **언제나 중요한 것은 현재 사용 중인 트랜잭션의 격리 수준에 맞추어 실행된 SQL이 어떤 결과를 반환하는지 정확히 예측할 수 있어야한다는 점**이다.
  * 그러나 해당 부정합 현상은 이러한 전제 조건을 충족시키지 못한다.

### 트랜잭션 안에서 실행된 SELECT 문과 밖에서 실행된 SELECT 문
* 기본적으로 **READ COMMITTED 격리 수준에서는 트랜잭션의 안 또는 밖에서 실행된 SELECT 문의 차이는 거의 없다**.
* 반면, **REPEATABLE READ 격리 수준에서는 기본적으로 SELECT 문 역시 트랜잭션 범위 내에서만 동작**한다.
  * 때문에 BEGIN 등의 명령어로 트랜잭션을 시작한 상태에서 계속해서 동일한 SELECT 쿼리를 요청하더라도 항상 동일한 결과가 반환된다.
  * 이는 **다른 트랜잭션에서 아무리 레코드를 변경하고 COMMIT한다고 해도 마찬가지이며, 트랜잭션 내에서 실행된 SELECT 문은 항상 같은 결과를 반환**한다.
* 이러한 차이점은 언뜻 중요해보이지 않을 수 있으나, 이로 인해 데이터의 정합성이 깨져 발생한 애플리케이션 이슈는 디버그가 쉽지 않다.

### REPEATABLE READ 격리 수준
```
> REPEATABLE READ는 InnoDB 스토리지 엔진에서 기본으로 사용되는 격리 수준이다.
> 바이너리 로그를 갖는 MySQL 서버에서는 기본적으로 최소 REPEATABLE READ 이상의 격리 수준을 사용해야 한다.
```
* 해당 격리 수준에서는 NON-REPEATABLE READ 부정합이 발생하지 않는다.
  * 이는 InnoDB 스토리지 엔진의 경우 트랜잭션이 ROLLBACK될 가능성에 대비해 변경되기 전 레코드를 언두 영역에 백업해두기 때문이다.
  * **이러한 변경 방식은 MVCC라고도 하며, 변경 전 레코드를 언두 영역에 백업하고 실제 테이블을 변경하는 식으로 동작**한다.
  * **REPEATABLE READ 격리 수준은 MVCC를 위해 언두 영역에 백업된 이전 데이터를 활용하여 동일한 트랜잭션 내에서 동일한 결과를 반환하도록 보장**한다.
* 정확히는 READ COMMITTED 역시 MVCC를 활용하여 COMMIT되기 전의 데이터를 보여준다.
  * 이 때, **REPEATABLE READ 격리 수준과의 차이는 언두 영역에 백업된 레코드의 여러 버전 중 몇 번째 이전 버전까지 사용하느냐에 있다**.

### InnoDB와 트랜잭션 번호
* 모든 **InnoDB 트랜잭션은 유일하며 증가하는 트랜잭션 번호를 가지며, 언두 영역에 백업되는 모든 레코드는 변경을 발생시킨 트랜잭션 번호를 포함**한다.
  * 이 때, **언두 영역에 백업된 데이터는 InnoDB 엔진의 판단에 따라 적시에 주기적으로 삭제**된다.
  * 그러나 **MVCC를 보장하기 위해 실행 중인 트랜잭션 중 가장 오래된 번호보다 앞선 트랜잭션 번호를 갖는 언두 영역의 데이터는 삭제할 수 없다**.
  * 정확히는 특정한 트랜잭션 번호 구간 내에서 백업된 언두 데이터가 보존되어야 한다.

### REPEATABLE READ 격리 수준의 원리
* 트랜잭션 번호가 6인 트랜잭션에 의해 삽입된 테이블에 하나의 레코드가 존재한다고 가정했을 때, REPEATABLE READ 격리 수준은 다음과 같이 동작한다.
  1. 사용자 1이 트랜잭션을 시작하며, 이 때 부여받은 트랜잭션 번호는 10번이라고 가정한다.
  2. 사용자 1이 SELECT 문을 활용하여 레코드를 조회하는 경우, 트랜잭션 번호 6에 의해 생성된 레코드 정보가 조회된다.
  3. 사용자 1이 트랜잭션을 완료하기 전에 사용자 2가 트랜잭션을 시작하고, 트랜잭션 번호 12를 부여받았다고 가정한다.
  4. 사용자 2가 트랜잭션 12에서 해당 레코드에 대한 UPDATE 문을 활용하여 수정을 시도하는 경우, 기존 레코드는 트랜잭션 번호 6을 갖고 언두 영역에 백업된다.
  5. 사용자 2가 수정한 데이터는 트랜잭션 번호 12를 갖는 상태에서 테이블에 저장된다.
  6. 사용자 1이 다시 SELECT 문을 활용하여 레코드를 조회화는 경우, 자신의 트랜잭션 번호인 10보다 작은 트랜잭션 번호를 갖는 변경 사항만을 검색한다.
  7. 6.의 원리에 의해 사용자 1은 2.의 과정에서 조회되었던 값과 동일한 결과를 조회한다.
* 이렇듯 **임의의 트랜잭션 번호를 부여받은 트랜잭션 내에서 실행되는 모든 SELECT 문은 해당 번호보다 작은 트랜잭션 번호에서 변경된 내용만을 조회**하게 된다.
  * 이 때, 단일 레코드에 대한 언두 영역의 백업은 하나 이상이 존재할 수 있다.
  * 때문에 임의의 사용자가 트랜잭션을 시작한 후 트랜잭션을 종료하지 않고 작업을 진행하게 되면 언두 영역의 데이터가 계속해서 커질 수 있다.
  * 또한, **이렇게 언두 영역에 백업된 레코드가 많아질수록 MySQL 서버의 처리 성능은 떨어질 수 있다**.

### PHANTOM READ 부정합
* REPEATABLE READ 격리 수준에서도 부정합이 발생할 수 있다.
  * 이는 **PHANTOM READ 부정합에 해당하며, `SELECT ... FOR UPDATE` 또는 `SELECT ... LOCK IN SHARE MODE` 쿼리에서 발생**할 수 있다.
* **해당 부정합은 서로 다른 트랜잭션에서 수행한 변경에 의해 레코드가 보였다 안보였다 하는 현상이며, 이는 언두 레코드를 잠글 수 없는 한계에서 기인**한다.
  * 이로 인해 **상술한 쿼리로 조회되는 레코드는 언두 영역의 변경 전 데이터를 조회하는 것이 아닌, 현재 레코드의 값을 조회**하게 된다.

### SERIALIZABLE 격리 수준
```
> InnoDB 스토리지 엔진을 주로 사용하는 경우, SERIALIZABLE 격리 수준을 굳이 사용할 필요는 없다.
```
* 해당 격리 수준은 가장 단순하면서도 엄격한 격리 수준으로, 동시 처리 성능이 다른 트랜잭션 격리 수준에 비해 가장 떨어지는 단점이 존재한다.
* **InnoDB 테이블의 경우 기본적으로 순수한 SELECT 문은 아무런 레코드를 잠그지 않고 실행**된다.
  * 이는 InnoDB 메뉴얼에서 `Non-locking Consistent Read`, 즉 잠금 없는 일관된 읽기라는 용어로 곧잘 표현된다.
* **SERIALIZABLE 격리 수준에서는 단순한 읽기 작업 역시 반드시 공유 잠금(읽기 잠금)을 획득**해야만 한다.
  * 이로 인해 다른 트랜잭션에서는 해당 레코드를 변경하지 못하게 된다.
  * 즉, **해당 격리 수준에서는 임의의 트랜잭션에서 읽고 쓰는 레코드는 다른 트랜잭션에서 절대 접근할 수 없다**.
* 해당 격리 수준에서는 일반적인 DBMS에서 발생하는 PHANTOM READ 부정합이 발생하지 않는다.
  * 그러나 **InnoDB의 경우 갭 락과 넥스트 키 락을 사용하므로, REPEATABLE READ 격리 수준에서도 해당 부정합이 발생하지 않는다**.