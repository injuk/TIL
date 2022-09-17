# Basics
## 2022-09-14 Wed

### MySQL 서버의 구조
* **MySQL 서버는 사람으로 따지면 머리 역할을 담당하는 MySQL 엔진과 손 발 역할을 수행하는 스토리지 엔진으로 구분**할 수 있다.
  * 이 때, 스토리지 엔진은 핸들러 API를 만족하기만 한다면 누구나 직접 구현하여 MySQL 서버에 추가할 수 있다.
  * 또한 **MySQL 엔진과 스토리지 엔진을 합쳐 MySQL 또는 MySQL 서버라는 용어로 지칭**한다.
* MySQL은 일반적인 상용 RDBMS와 같이 대부분의 프로그래밍 언어로부터의 접근 방법 역시 모두 지원한다.
  * 이로 인해 C는 물론 JDBC, 파이썬 등 많은 언어를 통해 MySQL 서버에서 쿼리를 사용할 수 있다.
* MySQL 서버는 MySQL 엔진에 더해 스토리지 엔진으로서 기본적으로 InnoDB와 MyISAM을 제공한다.

### MySQL 엔진의 특징
```
> MySQL 엔진은 요청된 SQL 문장을 분석하거나 최적화하는 등 DBMS의 두뇌에 해당하는 처리를 수행한다.
```
* MySQL 엔진은 클라이언트로부터 발생하는 접속 및 쿼리 요청을 처리하는 커넥션 핸들러에 더해 다음과 같은 분류를 갖는다.
  1. SQL 파서 및 전처리기
  2. 쿼리의 최적화된 실행을 위한 SQL 옵티마이저
  3. SQL 인터페이스
  4. 캐시와 버퍼
* 또한, **MySQL은 표준 SQL인 ANSI SQL 문법을 지원하므로 표준 문법에 따라 작성된 쿼리는 다른 DBMS와 호환되어 실행**될 수 있다.

### 스토리지 엔진의 특징
```
> 스토리지 엔진은 실제 데이터를 디스크에 저장하거나, 디스크로부터 데이터를 읽어오는 역할을 담당한다.
> MySQL 서버에서 MySQL 엔진의 종류는 단 하나지만, 스토리지 엔진은 여러 종류를 동시에 사용할 수 있다.
```
* **테이블 생성시 사용할 스토리지 엔진을 명시하는 경우, 이후에 해당 테이블로부터 발생하는 모든 읽기 또는 변경 작업은 정의된 스토리지 엔진에 의해 처리**된다.
  * 예를 들어, `CREATE TABLE table_name (id INT) ENGINE=InnoDB`와 같은 형태로 명시한다.
  * 상술한 쿼리에서, 이후의 모든 INSERT / UPDATE / DELETE /SELECT 등의 작업은 InnoDB 스토리지 엔진에 의해 처리된다.
* 이 때, **각각의 스토리지 엔진은 성능 향상을 위해 키 캐시 또는 버퍼 풀과 같은 기능을 내장**한다.
  1. 키 캐시: MyISAM 스토리지 엔진에서 성능 향상을 위해 사용하는 기능이다.
  2. **버퍼 풀: InnoDB 스토리지 엔진에서 성능 향상을 위해 사용하는 기능**이다.

### Handler API
* MySQL 엔진의 쿼리 실행기에서 실제 데이터를 읽거나 쓰는 경우 각각의 스토리지 엔진에 쓰기 또는 읽기 작업을 요청해야 한다.
  * 이 때, **이러한 요청을 핸들러 요청이라고 하며 각 경우에 사용되는 API를 핸들러 API**라고 한다.
  * InnoDB 스토리지 엔진 역시 이러한 핸들러 API를 활용하여 MySQL 엔진과 데이터를 주고받는다.
* 핸들러 API를 활용하여 얼마나 많은 레코드 작업이 있었는지 확인하고자 하는 경우, 다음과 같은 명령어를 활용할 수 있다.
```
SHOW GLOBAL STATUS LIKE 'Handler%';
```

## 2022-09-15 Thu
### MySQL 서버 스레딩
* **MySQL 서버는 프로세스 기반이 아닌 스레드 기반으로 동작하며, 스레드는 다시 포그라운드 스레드와 백그라운드 스레드로 구분**할 수 있다.
* 이 때, MySQL 서버에서 실행 중인 스레드의 목록은 `performance_schema` 데이터베이스의 `threads` 테이블을 통해 확인할 수 있다.
  * 이 중 백그라운드 스레드의 개수는 MySQL 서버의 설정 내용에 따라 가변적이다.
  * 또한, 여러 스레드가 동일 작업을 병렬로 처리하는 경우 동일한 이름의 스레드가 존재할 수도 있다.
* **이러한 스레드 모델은 MySQL 커뮤니티 에디션이 갖는 전통적인 스레드 모델이며, 커넥션 별로 포그라운드 스레드를 하나씩 생성하는 1:1 관계를 유지**한다.
  * 반면, MySQL 엔터프라이즈 에디션은 스레드 풀 모델을 사용하며 하나의 스레드가 여러 개의 커넥션 요청을 처리한다.

### 포그라운드 스레드
* **포그라운드 스레드는 클라이언트 스레드라고도 하며, 최소한 MySQL 서버에 접속된 클라이언트 수만큼 존재**한다.
  * 포그라운드 스레드는 주로 각각의 클라이언트 사용자가 요청하는 쿼리를 처리한다.
* **클라이언트 사용자가 작업을 마치고 커넥션을 종료할 경우, 해당 커넥션을 담당하던 스레드는 스레드 캐시로 돌아간다**.
  * 이 때, 이미 스레드 캐시에 일정한 개수 이상의 스레드가 존재하는 경우 스레드 캐시에 넣지 않고 스레드를 종료한다.
  * 이렇듯 일정 개수의 스레드만이 스레드 캐시에 존재하게 하며, 이러한 스레드의 최대 개수는 `thread_cahce_size` 시스템 변수로 설정할 수 있다.
* **포그라운드 스레드는 기본적으로 데이터를 MySQL의 데이터 버퍼나 캐시로부터 조회**한다. 
  * 만약 **버퍼나 캐시에 필요한 데이터가 없는 경우, 직접 디스크 데이터나 인덱스 파일로부터 조회하여 작업을 처리**한다.
  * 이 때, MyISAM의 경우 디스크 쓰기 작업까지 포그라운드 스레드가 처리한다.
  * 반면, **InnoDB 경우 데이터 버퍼나 캐시까지만 포그라운드 스레드가 처리하고 버퍼로부터 디스크에 기록하는 나머지 작업은 백그라운드 스레드가 처리**한다.

### 사용자 스레드와 포그라운드 스레드
* MySQL에서 사용자 스레드와 포그라운드 스레드는 같은 의미로 사용된다.
* **클라이언트가 MySQL 서버에 접속하는 경우 MySQL 서버는 해당 클라이언트의 요청을 처리할 스레드를 생성하여 해당 클라이언트에게 할당**한다.
  * **해당 스레드는 DBMS의 앞단에서 클라이언트인 사용자와 통신하므로 포그라운드 스레드라는 용어로 지칭**한다.
  * 또한, **사용자가 요청한 작업을 처리하는 스레드이기에 사용자 스레드라고도 지칭**할 수 있게 된다.

### 백그라운드 스레드
* InnoDB의 경우, 다음과 같은 작업은 백그라운드로 처리된다.
  1. 인서트 버퍼를 병합하는 스레드
  2. 로그를 디스크로 기록하는 스레드
  3. InnoDB 버퍼 풀의 데이터를 디스크에 기록하는 스레드
  4. 데이터를 버퍼로 읽어들이는 스레드
  5. 잠금 또는 데드락을 모니터링하는 스레드
* 모두 중요하지만, 이 중 **특히 중요한 것은 로그 스레드와 버퍼의 데이터를 디스크로 써내려가는 쓰기 스레드**가 된다.
* InnoDB의 경우 데이터를 읽는 작업을 주로 클라이언트 스레드에서 처리하므로, 읽기 스레드는 굳이 많이 설정할 필요가 없다.
  * 그러나 **쓰기 스레드는 많은 작업을 백그라운드에서 처리하므로 환경에 따라 디스크를 최적으로 사용할 수 있을 정도로 충분히 설정하는 것이 바람직**하다.
* 사용자의 **요청을 처리하는 도중 데이터의 쓰기 작업은 버퍼링과 같이 지연 처리될 수 있으나, 데이터의 읽기 작업은 절대로 지연되지 않아야 한다**.
  * 이로 인해 **대부분의 상용 DBMS에서는 대부분 쓰기 작업을 버퍼링하여 일괄 처리하는 기능이 탑재되어 있으며, InnoDB 역시 마찬가지 방식으로 동작**한다.

### 메모리 할당과 구조
* **MySQL에서 사용되는 메모리는 크게 글로벌 메모리 영역과 로컬 메모리 영역으로 구분되며, 둘의 차이는 많은 스레드가 공유하는 공간인지 여부**에 따른다.
  * 특히 글로벌 메모리 영역의 모든 메모리 공간은 MySQL 서버가 시작되면서 운영체제로부터 할당된다.
  * 이 때, **각각의 운영체제 별 메모리 할당 방식은 상당히 복잡하므로 MySQL 서버가 사용하는 정확한 메모리 양을 측정하는 것은 쉽지 않다**.

### 글로벌 메모리 영역
* **일반적으로 클라이언트 스레드의 수와는 관계 없이 하나의 메모리 공간만이 할당**된다.
  * 필요한 경우 2개 이상의 메모리 공간을 할당받을 수도 있지만, 이 역시도 클라이언트 스레드의 개수와는 관계가 없다.
  * 또한, **생성된 글로벌 영역이 2개 이상이라고 하더라도 항상 모든 스레드에 의해 공유**된다.
* 이 때, 글로벌 메모리 영역은 크게 다음과 같이 분류된다.
  1. 테이블 캐시
  2. InnoDB 버퍼 풀
  3. InnoDB 어댑티브 해시 인덱스
  4. InnoDB 리두 로그 버퍼

### 로컬 메모리 영역
* 로컬 메모리 영역은 MySQL 서버 상에 존재하는 클라이언트 스레드가 쿼리를 처리하기 위해 사용하는 메모리 영역이다.
  * **클라이언트가 MySQL 서버에 접속하는 경우 MySQL 서버는 클라이언트 커넥션으로부터의 요청을 처리하기 위해 스레드를 각각 할당**한다.
  * 이러한 클라이언트 스레드가 사용하는 메모리 공간이므로 클라이언트 메모리 영역이라고도 하며, 커넥션을 세션이라고 표현하여 세션 메모리 영역이라고도 한다.
* **로컬 메모리 영역은 각각의 클라이언트 스레드 별로 독립적으로 할당되며, 절대 스레드 간에 공유되어 사용되지 않는다**.
  * 로컬 메모리 영역의 크기 역시 적절히 설정되어야 하며, 그렇지 않은 경우 MySQL 서버가 메모리 부족으로 인해 멈출 가능성이 희박하게나마 존재한다.
* 또한, **로컬 메모리 영역에는 각 쿼리의 용도 별로 필요할 경우에만 공간이 할당**된다.
  * 이는 즉, **쿼리의 용도에 따라서는 MySQL 서버가 메모리 공간을 전혀 할당하지 않을 수도 있다는 점을 시사**한다.
  * 예를 들어, 소트 버퍼 또는 조인 버퍼가 이러한 경우에 해당한다.
* **로컬 메모리 영역은 커넥션이 유지되는 동안 계속 할당된 상태로 남아 있는 공간과, 쿼리가 실행되는 순간에만 할당하고 이후에 해제되는 공간도 존재**한다.
  * 예를 들어, **커넥션 버퍼와 결과 버퍼는 커넥션이 유지되는 동안 계속해서 할당된 상태**로 남아 있는다.
  * 반면, **소트 버퍼와 조인 버퍼는 쿼리를 실행하는 순간에만 할당했다가 다시 해제하는 공간에 해당**한다.
* 이 때, 로컬 메모리 영역은 크게 다음과 같이 분류된다.
  1. 소트 버퍼
  2. 조인 버퍼
  3. 바이너리 로그 캐시
  4. 네트워크 버퍼

## 2022-09-16 Fri
### 플러그인 스토리지 엔진 모델
* MySQL은 기본적으로 많은 스토리지 엔진을 갖고 있지만, 이 조차도 세상의 모든 요구 조건을 만족시킬 수는 없다.
  * 이에 따라 개발 회사 또는 사용자는 자신의 요구사항을 충족하는 스토리지 엔진을 직접 개발하여 사용할 수도 있다.
* 또한, **MySQL 서버는 스토리지 엔진 뿐만 아니라 다양한 기능을 플러그인 형태로 지원**한다.
  * 예를 들어 인증 또는 전문 검색 파서 및 쿼리 재작성과 같은 플러그인이나, 비밀번호 검증과 커넥션 제어 등과 관련된 다양한 플러그인이 존재한다.
  * 이 역시도 MySQL 서버에서 제공하는 기능들을 확장하거나 완전히 새로운 기능을 추가하기 위해 플러그인 API를 활용하여 직접 구현할 수도 있다.

### MySQL에서 쿼리가 실행되는 과정
```
> 하나의 쿼리 작업은 여러가지 하위 작업으로 분류된다.
> 이 때, 각각의 하위 작업이 MySQL 엔진 영역에서 처리되는지 스토리지 엔진 영역에서 처리되는지 구분할 수 있어야 한다. 
```
* MySQL에서 쿼리가 실행되는 과정은 크게 다음과 같이 진행된다.
  1. SQL 파서
  2. SQL 옵티마이저
  3. SQL 실행기
  4. 데이터 읽기 또는 쓰기
* 이 때, **SQL 파서와 옵티마이저 및 실행기로 이어지는 흐름은 MySQL 엔진에서 처리**된다.
  * 반면, **디스크로부터 데이터를 조회하거나 새로운 데이터를 쓰는 작업은 스토리지 엔진의 처리 영역**이 된다.
  * 이로 미루어 보았을 때, 플러그인 형태로 새로운 스토리지 엔진을 작성한다는 것은 DBMS 전체가 아닌 일부 기능만을 수행하는 엔진을 정의한다는 의미를 갖는다.
* 반면, **실질적인 GROUP BY 또는 ORDER BY 등의 복잡한 처리는 스토리지 엔진 영역이 아닌 MySQL 엔진의 쿼리 실행기에서 처리**된다.

### MySQL 핸들러
* **일반적으로 프로그래밍 언어에서 핸들러는 어떠한 기능을 호출하기 위해 사용하는 API를 갖는 객체를 의미**한다.
  * 이 때, **MySQL에서의 핸들러는 MySQL 엔진이 스토리지 엔진을 조정하기 위해 사용하는 API를 의미**한다.
  * 이렇듯 **MySQL 엔진이 각각의 스토리지 엔진으로부터 데이터를 조회하거나 저장하도록 명령하기 위해서는 반드시 핸들러를 통해야 한다**.
* MySQL 서버의 경우, `Handler_`라는 접두사로 시작하는 상태 변수가 MySQL 엔진이 스토리지 엔진에 보낸 명령의 횟수를 의미하는 상태 변수가 된다.
* **핸들러의 존재로 인해 여러 종류의 스토리지 엔진을 사용하는 테이블에 대해 실행된 쿼리 역시 MySQL의 처리 내용은 대부분 동일**해진다.
  * 정확히는 단순히 테이블 별 스토리지 엔진에 따른 데이터의 읽기 및 쓰기 영역에 대한 차이만이 존재한다.

### MySQL 서버에서 지원하는 스토리지 엔진 확인하기
* 자신이 설치한 mysqld에서 지원되는 스토리지 엔진의 종류는 `SHOW ENGINES` 명령어를 활용하여 확인할 수 있다.
* 이 때, 표시된 정보 중 Support 컬럼은 다음과 같은 값이 표시될 수 있다.
  1. YES: mysqld에 해당 스토리지 엔진이 포함되어 있고, 활성화되어 사용 가능한 상태를 의미한다.
  2. DEFAULT: YES와 동일한 상태이며, 필수 스토리지 엔진이므로 해당 엔진이 없을 경우 MySQL이 시작되지 않을 수도 있음을 의미한다.
  3. NO: 현재의 mysqld에는 포함되지 않은 상태를 의미한다.
  4. DISABLED: mysqld에 포함되어 있으나, 파라미터에 의해 비활성화된 상태를 의미한다.

### 컴포넌트란?
* MySQL 8.0 이전의 플러그인 아키텍쳐는 다음과 같은 단점을 갖는다.
  1. 플러그인은 언제나 MySQL 서버와만 통신할 수 있으며, 플러그인 끼리는 통신이 불가능하다.
  2. 플러그인은 MySQL 서버의 변수 또는 함수를 직접 호출하는 등, 충분히 캡슐화되지 않아 안전하지 않다.
  3. 플러그인은 상호 의존 관계를 명시적으로 설정할 수 없어 초기화 과정이 어렵고 번거롭다.
* 이러한 단점을 보완하기 위해 컴포넌트라는 개념이 새로 도입되었다. 
  * MySQL 8.0부터는 많은 기능을 제공하는 컴포넌트를 `INSTALL COMPONENT 컴포넌트` 명령어로 설치할 수 있다.

### 쿼리 실행 시 처리 과정
* 사용자가 요청한 SQL은 다음과 같은 여러 기능을 거쳐 결과로 반환된다.
  1. 쿼리 파서
  2. 전처리기
  3. 옵티마이저에 의한 쿼리 변환 및 비용 최적화, 실행 계획 수립
  4. 쿼리 실행기(= 실행 엔진)
  5. 스토리지 엔진에 의한 읽기 및 쓰기(= 핸들러)

### 쿼리 파서란?
* **쿼리 파서는 사용자 요청으로 전달된 SQL 문장을 MySQL이 인식할 수 있는 최소 단위의 어휘 또는 기호인 토큰으로 분리**한다.
  * 또한, 이렇게 **분리된 토큰을 토대로 쿼리 파서는 트리 형태의 구조를 만들어내는 작업을 수행**한다.
* **쿼리 자체의 오류는 토큰을 만들어내는 쿼리 파서에서 발견되며, 사용자에게 오류 메시지 형태로 전달**된다.

### 전처리기란?
* **전처리기는 쿼리 파서가 만들어낸 파서 트리를 기반으로 쿼리 문장 자체의 구조적인 문제가 있는지 확인**한다.
  * 예를 들어 **각 토큰을 테이블 이름, 칼럼 명 또는 내장 함수와 같은 객체에 매핑하여 해당 객체의 존재 여부 및 접근 권한을 확인**한다.
  * 따라서 **실제로 존재하지 않거나 권한 상 사용할 수 없는 객체 토큰은 해당 단계에서 필터링**된다.

### 옵티마이저란?
* **옵티마이저는 DBMS의 두뇌에 해당하며, 사용자의 요청으로 전달된 쿼리 문장을 가장 저렴한 비용으로 가장 빠르게 처리하는 방법을 결정**한다.
  * 당연히 옵티마이저의 역할은 매우 중요하며, 그 영향 범위 역시 매우 넓은 축에 속한다.

### 실행 엔진이란?
* 옵티마이저와 실행 엔진, 핸들러는 각각 다음과 같은 역할로 비유될 수 있다.
  1. 옵티마이저: 경영진
  2. 실행 엔진: 중간 관리직
  3. 핸들러: 업무 별 실무자
* 예를 들어 옵티마이저가 GROUP BY를 처리하기 위해 임시 테이블을 사용하기로 결정했다고 가정할 경우, 실행 엔진은 다음과 같은 역할을 수행한다.
  1. 실행 엔진은 핸들러에게 임시 테이블의 생성을 요청한다.
  2. 실행 엔진은 핸들러에게 WHERE 절과 일치하는 레코드를 조회할 것을 요청한다.
  3. 실행 엔진은 핸들러에게 2.의 과정에서 조회한 레코드들을 1.의 과정에서 생성한 임시 테이블에 저장할 것을 요청한다.
  4. 실행 엔진은 핸들러에게 3.의 과정에서 구성된 임시 테이블로부터 필요한 방식으로 데이터를 조회할 것을 요청한다.
  5. 실행 엔진은 4.까지의 과정에 의해 구성된 최종 결과를 사용자 또는 다른 모듈로 전달한다.
* 이렇듯 **실행 엔진은 옵티마이저에 의해 수립된 계획대로 각 과정을 핸들러에게 요청하며, 전달 받은 결과를 또 다른 핸들러 요청의 입력으로 연결**한다.

### 핸들러란?
```
> 핸들러는 스토리지 엔진이다.
```
* 상술했듯, **핸들러는 MySQL 서버의 최하단에서 MySQL 실행 엔진의 요청에 따라 데이터를 디스크에 쓰거나 읽어들이는 역할을 수행**한다.
  * 즉, **핸들러란 결국 스토리지 엔진 자체를 의미**한다.
  * 예를 들어 **InnoDB 스토리지 엔진을 사용하는 테이블을 조작하는 경우에 호출되는 핸들러는 InnoDB 스토리지 엔진**이 된다.

## 2022-09-17 Sat
### 쿼리 캐시
```
> 쿼리 캐시는 읽기만 제공하는 서비스와 같이 특수한 경우가 아니라면 심각한 동시 처리 성능의 저하를 유발하는 기능이었다.
> 이로 인해 쿼리 캐시는 MySQL 8.0에 들어서면서 관련된 시스템 변수와 함께 완전히 제거되었다.  
```
* 쿼리 캐시는 SQL 실행 결과를 메모리에 캐시하고, 동일한 SQL 쿼리가 실행되면 테이블을 읽지 않고 즉시 결과를 반환하여 큰 성능 향상을 제공했다.
* 그러나 쿼리 캐시는 테이블의 데이터가 변경되는 경우, 캐싱된 결과 중 변경된 테이블과 관련된 모든 것을 반드시 삭제해야 했다.
  * 이러한 특징은 심각한 동시 처리 성능 저하를 유발하였으며, 대부분의 경우에 수 많은 버그의 원인으로 지목되었다.
* 이로 인해 MySQL 8.0에서는 쿼리 캐시 기능이 완전히 제거되었다.

### 스레드 풀
```
> MySQL 서버의 스레드 풀은 Percona Server와 엔터프라이즈 에디션에서는 지원되나, 커뮤니티 에디션에는 지원되지 않는다.
```
* **스레드 풀은 내부적으로 사용자의 요청을 처리하는 스레드의 개수를 줄여 동시에 처리되는 요청이 많아지더라도 제한된 개수의 스레드 처리에만 집중하도록 한다**.
  * 즉, **스레드 풀은 MySQL 서버의 CPU가 제한된 개수의 스레드에만 집중하여 서버의 자원 소모를 줄이는 데에 목적**이 있다.
  * 이로 인해 실무에서도 스레드 풀이 서비스에서 눈에 띄는 성능 향상을 보여주는 경우는 드물다.
* 상술했듯, 스레드 풀은 동시에 실행 중인 스레드들을 CPU가 최대한 잘 처리할 수 있는 수준으로 줄여 빠르게 처리하도록 하는 기능이다.
  * 이 때, 일반적으로 CPU 코어의 개수와 스레드의 개수를 맞추는 것이 CPU 프로세서의 친화도를 높이기에 좋다.

### 트랜잭션을 지원하지 않는 파일 기반 메타데이터
* **데이터베이스 서버 상에서 테이블의 구조 정보, 또는 스토어드 프로그램 등의 정보는 데이터 딕셔너리 또는 메타데이터라고 지칭**한다.
  * 이 때, **MySQL 서버 5.7 버전까지는 이러한 테이블의 구조와 스토어드 프로그램을 모두 파일 기반으로 관리**했다.
* 그러나 이러한 **파일 기반의 메타데이터는 생성 및 변경 작업에 대한 트랜잭션을 지원하지 않는 치명적인 문제가 존재**하였다.
  * 때문에 **테이블의 생성 또는 변경 도중에 MySQL 서버가 비정상적으로 종료된 경우, 테이블은 일관되지 않는 상태로 남을 가능성이 존재**한다.
  * 이러한 현상은 대부분의 개발자에 의해 `테이블이 깨졌다`는 용어로 표현되었다.

### 시스템 테이블
* **MySQL 8.0 버전부터는 상술한 문제점을 해결하기 위해 메타데이터를 모두 InnoDB의 테이블에 저장하도록 개선**되었다.
  * 이렇듯 **MySQL 서버가 동작하는데에 기본적으로 필요한 사용자의 인증 및 권한 등에 사용되는 필수 테이블들을 묶어 시스템 테이블이라고 지칭**한다.
* **MySQL 8.0 버전부터는 시스템 테이블은 모두 InnoDB 스토리지 엔진을 사용하며, 시스템 테이블과 메타데이터 정보를 모두 모아 mysql DB에 저장**한다.
  * 이 때, mysql DB는 통채로 `mysql.ibd`라는 이름의 테이블 스페이스에 저장된다.
  * 때문에 **MySQL 서버 상의 데이터 디렉토리에 존재하는 해당 파일은 다른 `.ibd` 확장자 파일과 함께 각별히 주의를 기울여 관리**해야 한다.
* 이 때, **mysql DB에 저장된 데이터 딕셔너리 테이블은 사용자가 임의로 수정할 가능성을 제거하기 위해 사용자가 조회할 수 없도록 구성**되어 있다.
  * **대신 MySQL 서버는 데이터 딕셔너리 정보를 조회하기 위해 `information_schema` DB의 `TABLES` 또는 `COLUMNS`와 같은 뷰를 제공**한다.

### 트랜잭션을 지원하는 메타데이터
* **MySQL 8.0 버전부터는 데이터 딕셔너리와 시스템 테이블이 모두 트랜잭션 기반의 InnoDB 스토리지 엔진에 저장되도록 개선**되었다.
  * 때문에 **스키마 변경 작업 도중에 MySQL 서버가 비정상적으로 종료되더라도 스키마의 변경은 원자적으로 처리**된다.
  * 즉, 더 이상 테이블이 깨지는 등 작업 진행 중인 상태로 남아 문제를 유발하는 일은 발생하지 않는다.
* MySQL 서버에서, InnoDB 스토리지 엔진을 사용하는 테이블의 메타데이터는 InnoDB 테이블 기반의 딕셔너리에 저장된다.
  * 반면, **MyISAM 또는 CSV 등과 같은 스토리지 엔진의 메타데이터는 여전히 저장하기 위한 별도의 공간이 필요**하다.
  * MySQL 서버는 이러한 InnoDB 이외의 스토리지 엔진들을 사용하는 테이블들을 위해 `.sdi` 포맷을 지원한다.
* SDI는 직렬화를 위한 포맷이므로, InnoDB 테이블의 구조 역시 `.sdi` 확장자 파일로 전환될 수 있다.
  * 이 때, `ibd2sdi` 유틸리티를 활용하여 InnoDB 테이블 스페이스에서 스키마 정보를 추출할 수 있다.
  * 이 경우, MySQL 서버 상에서 `SHOW TABLES` 명령으로는 확인할 수 없던 `mysql.tables` 딕셔너리 데이터를 위한 테이블 구조 역시 확인이 가능하다.

### InnoDB 스토리지 엔진의 특징
* **InnoDB 스토리지 엔진은 MySQL에서 가장 자주 사용되는 스토리지 엔진**이며, 다음과 같은 특징이 존재한다.
  1. MySQL에서 사용할 수 있는 스토리지 엔진 중 거의 유일하게 레코드 기반의 잠금을 제공한다.
  2. 이로 인해 높은 동시성 처리가 가능하다.
  3. **다른 스토리지 엔진과 비교하여 안정적이며, 성능 역시 뛰어나다**.

### PK에 의한 클러스터링
* **InnoDB 엔진을 사용하는 모든 테이블은 기본적으로 PK를 기준으로 클러스터링되어 저장**된다.
  * **이는 즉 PK의 값 순서대로 디스크에 저장된다는 의미**이기도 하다.
  * **PK가 결국 클러스터링 인덱스이므로, PK를 활용한 레인지 스캔은 상당히 빠르게 처리**된다. 
  * 때문에 **쿼리 실행 계왹에서 PK는 기본적으로 다른 세컨더리 인덱스에 비해 우선 순위가 높게 설정되므로 우선적으로 선택될 확률이 상대적으로 높다**.
* InnoDB 엔진을 사용하는 경우, 모든 **세컨더리 인덱스는 레코드의 주소 대신 PK 값을 논리적인 주소로 사용**한다.

### MyISAM 엔진과 InnoDB의 차이점
* MyISAM 스토리지 엔진의 경우 클러스터링 키를 지원하지 않으므로, PK와 세컨더리 인덱스는 구조적으로 차이가 없다.
  * PK는 단지 UQ 제약 조건을 갖는 세컨더리 인덱스에 지나지 않는다.
* MyISAM 테이블의 경우, PK를 포함하는 모든 인덱스는 물리적인 레코드의 주소인 ROWID 값을 갖는다.
  * 이는 **세컨더리 인덱스에서 ROWID 대신 PK 값을 논리적인 주소로 사용하는 InnoDB 엔진과는 차이가 있다**.

### FK에 대한 지원
* **FK에 대한 지원은 InnoDB 스토리지 엔진 차원에서 제공하는 기능이므로, MyISAM 등 다른 스토리지 엔진을 사용하는 테이블에서는 사용할 수 없다**.
* FK는 DB 서버 운영의 불편함으로 인해 서비스용 DB에서는 생성하지 않는 경우가 잦다.
  * 그러나 개발 환경에서는 가이드 역할로서 유용히 사용될 수 있다.
* **InnoDB 엔진에서의 FK는 다음과 같은 이유에서 관리에 어려움이 따르므로, FK의 존재에 주의해야하는 경우가 많이 발생**한다.
  1. 부모 테이블과 자식 테이블 모두 해당 칼럼에 대한 인덱스 생성이 필수적이다.
  2. 변경 시에는 반드시 부모 테이블 또는 자식 테이블에 데이터가 존재하는지 체크하는 작업이 필요하므로 잠금은 여러 테이블로 전파된다.
  3. 상술한 이유에서, 데드락이 쉽게 발생할 수 있다.

### FK 체크 작업을 일시 중지하기
* 수동으로 데이터를 적재하거나, 스키마를 변경하는 과정은 복잡하게 얽힌 FK 관계로 인해 실패할 수 있다.
  * 물론 부모와 자식 테이블의 관계를 명확히 파악한다면 문제없을 수 있으나, FK 관계 자체가 복잡하다면 이 조차 쉽지 않다.
  * 또한, 서비스에 긴급한 문제가 있어 즉시 조치를 취해야하는 경우에도 엄격한 FK 체크 기능은 마음을 더 조급하게 만들어 실수를 불러일으킬 수 있다.
* 이러한 경우, `SET foreign_key_checks=OFF;` 형태의 명령어로 `foreign_key_checks` 시스템 변수를 변경하여 FK 체크 작업을 일시 중단할 수 있다.
  * 당연하게도 **FK 체크 작업을 일시적으로 해제했다고 해서 부모와 자식 테이블의 관계가 깨진 상태로 유지해도 무방하다는 의미는 아니다**.
  * 예를 들어, FK 체크 기능을 비활성화한 상태에서 FK를 갖는 부모 테이블의 레코드를 삭제했다면 자식 테이블에서도 반드시 일관성을 맞추어야 한다.
* 이러한 **시스템 변수는 적용 범위를 `SET SESSIOn foreign_key_cheks=OFF;`와 같이 GLOBAL 또는 SESSION으로 설정할 수 있는 종류**에 속한다.
  * 때문에 **FK 체크 기능을 일시적으로 비활성화하고자하는 경우, 반드시 현재 작업을 실행하는 세션에서만 적용하는 것이 바람직**하다.
  * 또한, **작업이 완료되는 대로 반드시 현재 세션을 종료하거나 FK 체크 기능을 다시 활성화하는 것을 잊지 않아야 한다**.

### Multi Version Concurrency Control
```
> MVCC란 DBMS 상에서 하나의 레코드에 대해 2개 이상의 버전이 각각 버퍼 풀과 데이터 파일, 언두 영역에서 관리되는 과정을 말한다.
```
* **MVCC는 일반적으로 레코드 레벨의 트랜잭션을 지원하는 DBMS가 제공하는 기능으로, 잠금을 사용하지 않는 일관된 읽기를 제공하는 데에 목적**이 있다.
  * **MVCC의 Multi Version은 하나의 레코드에 대해 여러 버전이 동시에 관리된다는 의미이며, InnoDB는 언두 로그를 활용하여 해당 기능을 구현**한다.

### MVCC 활용 시나리오
* 트랜잭션 격리 수준이 READ_COMMITTED일 때, InnoDB 엔진을 사용하는 테이블에 새로운 레코드 A를 추가하면 데이터베이스의 상태는 다음과 같이 변경된다.
  1. InnoDB 버퍼 풀: 메모리에 포함된 영역이며, 추가된 레코드 A에 대한 정보가 포함된다.
  2. 언두 로그: 메모리에 포함된 영역이지만, 지금 단계에서는 어떠한 데이터도 포함되지 않는다.
  3. 데이터 파일: 디스크 영역이며, 레코드 A에 대한 정보가 쓰여진다.
* 이 때, 레코드 A의 임의의 컬럼 값을 수정하는 UPDATE 문이 실행될 경우 데이터베이스의 상태는 다음과 같이 변경된다.
  1. InnoDB 버퍼 풀: COMMIT 여부와 관계 없이 기존 레코드의 일부 컬럼이 UPDATE 문에 맞추어 수정된다.
  2. 언두 로그: **기존 데이터인 레코드 A의 수정 전 값을 복사하여 갖게 된다**.
  3. 데이터 파일: 스레드에 의해 UPDATE 문에서 변경하는 컬럼의 수정 사항이 반영될 수도, 그렇지 않을 수도 있다.
     * 그러나 **일반적으로는 InnoDB가 ACID 특성을 보장하므로 InnoDB 버퍼 풀과 동일한 상태를 갖는다고 가정해도 무방**하다.
* **이 상태에서, 사용자가 SELECT 문을 통해 레코드 A를 조회하는 경우에는 MySQL 서버에 설정된 트랜잭션 격리 수준에 따라 조회 위치가 달라지게 된다**.
  * 이러한 트랜잭션 격리 수준은 시스템 변수인 `transaction_isolation`에 설정된다.
* 이 때, MySQL 격리 수준에 따른 조회 위치는 크게 다음과 같이 구분된다.
  1. READ_UNCOMMITTED: **InnoDB 버퍼 풀이 갖는 현재 값, 즉 COMMIT 여부와 관계 없이 변경된 현재의 상태를 반환**한다.
  2. READ_COMMITTED 및 그 이상의 격리 수준: **변경 사항이 아직 COMMIT되지 않았으므로, 변경 전 데이터인 언두 영역의 데이터를 반환**한다.
* **이렇듯 하나의 레코드에 대해 2개 이상의 버전이 각각 버퍼 풀과 데이터 파일, 언두 영역에서 관리되는 과정을 DBMS에서는 MVCC라는 용어로 표현**한다.
  * 이 때, **유지되는 2개 이상의 버전 중 필요에 따라 어떤 데이터를 반환할지는 그 때 그 때의 상황에 따라 달라지게 된다**.
* 또한 **언두 영역에서 관리되는 데이터는 상황에 따라 무한히 많아질 수 있다**.
  * 예를 들어 **트랜잭션이 필요 이상으로 길어지는 경우, 언두 영역에서 관리되는 예전 데이터가 적시에 삭제되지 못하고 오랫동안 관리**되게 된다.
  * 이러한 상황에서는 당연히 언두 영역이 저장되는 시스템 테이블 스페이스의 공간이 필요 이상으로 늘어나는 상황이 발생할 수 있다.

### COMMIT과 ROLLBACK에 따른 버퍼 풀과 언두 영역의 동작
* 상술한 시나리오에서, **UPDATE 문이 요청되는 경우 InnoDB의 버퍼 풀은 즉시 새로운 데이터로 변경되는 반면 기존 데이터는 언두 영역으로 복사**된다.
* 이 때, COMMIT 또는 ROLLBACK 중 어떤 명령을 실행하느냐에 따라 InnoDB 엔진은 다음과 같이 동작한다.
  1. **COMMIT 명령을 실행하면 InnoDB는 더 이상의 변경 작업 없이 현재의 상태를 영구적인 데이터로 영속화**한다.
  2. **ROLLBACK을 실행하는 경우 InnoDB는 언두 영역에 백업된 변경 전 데이터를 InnoDB 버퍼 풀로 다시 복구한 후에 언두 영역의 내용은 삭제**한다.
* 반면, ROLLBACK 명령이 실행된 경우와 달리 **COMMIT 명령이 실행되었다고 해서 언두 영역에 백업된 변경 전 데이터가 항상 바로 삭제되지는 않는다**.
  * **대신 해당 언두 영역을 필요로 하는 트랜잭션이 더 이상 존재하지 않는 경우에만 데이터가 삭제**된다.