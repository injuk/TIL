# Basics
## 2022-09-01 Thu

### 느린 디스크 IO
* CPU나 메모리처럼 전기적인 장치의 성능은 매우 빠른 속도로 발전해왔으나, 디스크와 같은 기계식 장치는 상당히 제한적으로 발전해왔다.
  * **최근에는 자기 디스크를 사용하는 HDD가 아닌 SSD가 많이 활용되고 있으나, 여전히 데이터 저장 매체는 컴퓨터에서 가장 느린 부분을 차지**한다.
* 이러한 이유에서, **데이터베이스의 성능 튜닝은 어떻게 디스크 IO를 줄이느냐에 달려 있는 경우가 많다**.

### SSD와 HDD의 비교
* 상술했듯, **CPU나 메모리와 같은 전자식 장치와 비교하여 HDD는 기계식 장치이므로 언제나 데이터베이스 서버의 병목**이 된다.
* 때문에 기존의 기계식 장치인 HDD를 대체하기 위한 전자식 저장 매체인 SSD가 각광받게 되었으며, 빠르게 HDD의 자리를 대체하고 있다.
  * SSD는 전자식 매체이므로 기계식으로 원판을 회전시킬 필요가 없기에 HDD와 비교하여 훨씬 빠른 IO가 가능하다.
* 반면, 디스크의 헤더를 거의 움직이지 않고 한 번에 많은 데이터를 읽어들이는 순차 IO에서는 SSD와 HDD의 성능이 유사하다.
  * 그러나 **SSD는 HDD와 비교하여 랜덤 IO가 훨씬 빠르다는 장점이 존재**한다.
  * **데이터베이스 서버에서는 순차 IO보다는 작은 데이터를 읽고 쓰는 랜덤 IO 작업의 비중이 훨씬 크기에 SSD의 장점은 DBMS용 스토리지에 최적화**되어 있다.

### 랜덤 IO와 순차 IO
* 디스크의 성능은 디스크 헤더의 위치 이동 없이 얼마나 많은 데이터를 한 번에 기록하느냐에 의해 결정된다.
  * 때문에 **여러번 읽기 또는 쓰기 작업을 요청하는 랜덤 IO 작업의 부하가 훨씬 더 크다**.
  * 이는 디스크 원판을 가지지 않는 SSD에서도 마찬가지이며, 때문에 MySQL에는 그룹 커밋 또는 로그 버퍼 등의 기능이 내장되어 있다.
* 사실 **쿼리를 튜닝하여 랜덤 IO를 순차 IO로 변경하는 방법은 그렇게 많지 않다**.
  * **일반적으로 `쿼리 튜닝은 랜덤 IO 자체를 줄이는 데에 그 목적`이 있다**.
  * 이는 즉, **쿼리를 처리하기 위해 꼭 필요한 데이터만 읽도록 쿼리를 개선하는 것을 의미**한다.
* 일반적으로 **인덱스 레인지 스캔은 데이터를 읽기 위해 주로 랜덤 IO를 사용**한다.
  * 반면, **풀 테이블 스캔은 순차 IO를 사용**하므로 커다란 테이블의 레코드를 대부분 읽는 경우 풀 테이블 스캔을 유도하는 경우도 있다.
  * 이러한 유형의 상호작용은 OLTP 유형의 웹 서비스보다는 데이터 웨어하우스, 또는 통계 작업에서 활용된다.

### 인덱스란?
* DBMS 역시 데이터베이스 테이블의 모든 데이터를 검색하여 결과를 가져오기에는 시간이 많이 걸릴 수 밖에 없다.
  * 때문에 **컬럼의 값과 해당 레코드의 주소를 키 값 형태의 인덱스로 만들어둔다**.
* 또한 **책의 인덱스와 목차 페이지와 마찬가지로, DBMS 역시 인덱스를 통해 빠르게 검색할 수 있도록 데이터를 순서대로 정렬하여 보관**한다.

### 자료 구조와 인덱스
* 여러 자료 구조 중 SortedList와 ArrayList는 다음과 같은 차이가 존재한다.
  1. SortedList: 데이터를 항상 정렬된 상태로 유지한다.
  2. ArrayList: 데이터를 항상 저장된 순서 그대로 유지한다.
* 이 중 데이터를 항상 정렬하는 SortedList는 다음과 같은 특징을 갖는다.
  1. 단점: **데이터를 저장할 때마다 값을 정렬해야하므로 저장 과정이 복잡하고 느리다**.
  2. 장점: **데이터가 이미 정렬되어 있으므로, 원하는 값은 매우 빠르게 조회할 수 있다**.
* **인덱스는 SortedList 자료 구조를 따르며, 저장되는 컬럼의 값을 통해 항상 정렬된 상태를 유지**한다.
* 또한, DBMS의 인덱스 역시 SortedList 자료 구조의 장단점을 그대로 계승한다.
  1. 단점: **인덱스가 많은 테이블일수록 INSERT, UPDATE, DELETE 문의 처리는 느려진다**.
  2. 장점: **이미 정렬된 키 - 값 쌍인 인덱스를 가지고 있으므로, SELECT 문의 처리는 매우 빠르다**.

### DBMS에서의 인덱스
* 상술한 이유에서, **DBMS에서의 인덱스는 결국 데이터의 쓰기 성능을 희생하고 읽기 성능을 높이는 기능**이다.
* 때문에 **테이블에 새로운 인덱스를 추가할지 여부는 데이터의 저장 속도를 얼마나 희생하고, 읽기 속도를 얼마나 높여야 하느냐에 따라 결정**된다.
  * SELECT 문의 WHERE 절에서 사용되는 모든 컬럼을 인덱스로 생성하는 경우, 오히려 데이터의 저장 성능이 떨어지고 인덱스의 크기만 커질 수도 있다.

### 프라이머리 키와 보조 키
* 인덱스는 크게 다음과 같이 분류할 수 있다.
  1. 프라이머리 키: **해당 레코드를 대표하는 컬럼 값으로 구성된 인덱스**이다.
  2. 보조 키: **프라이머리 키를 제외한 모든 나머지 인덱스를 의미하며, 세컨더리 인덱스라고도** 한다.
* 프라이머리 키는 테이블에서 해당 레코드를 식별할 수 있는 기준 값이 되므로 식별자라고도 부를 수 있다.
  * 이 때, **프라이머리 키는 NULL과 중복을 허용하지 않는다**.
* **유니크 인덱스는 프라이머리 키와 성격이 유사하며, 프라이머리 키를 대체할 수도 있으므로 대체 키라고도** 부를 수 있다.
  * 그러나 분류에 따라 별도로 분류할 수도 있고, 때로는 보조 키로 분류되기도 한다.
* 또한, 인덱스는 데이터를 저장하는 알고리즘 또는 중복 허용 여부에 따라 분류할 수도 있다.

### 데이터 저장 방식에 따른 인덱스 분류
```
> B-Tree 알고리즘은 컬럼의 값을 변경하지 않고 원본 값으로 인덱싱한다.
> Hash 알고리즘은 컬럼의 값을 해시 값으로 계산하여 인덱싱한다.
```
* DBMS 인덱스에 적용 가능한 데이터 저장 알고리즘은 수많지만, 일반적인 RDBMS에서 적용되는 대표적인 알고리즘은 크게 다음과 같다.
  1. B-Tree: 가장 일반적으로 사용되는 인덱스 알고리즘이며, 오래 전에 도입되었기에 그만큼 성숙되었다.
     * **B-Tree 알고리즘은 컬럼의 값을 변경하지 않고 원래의 값으로 인덱싱**한다. 
  2. Hash: **컬럼의 값을 해시로 계산하여 인덱싱하며, 매우 빠른 검색을 지원**한다.
     * 그러나 **해당 알고리즘은 값을 변형하므로 전방 일치 검색과 같은 일부 검색 또는 범위 검색 기능을 제공할 수 없다**.
     * **해당 방식은 주로 메모리 기반 데이터베이스에서 사용**된다.

### 데이터 중복 허용 여부에 따른 인덱스 분류
* **중복 허용 여부로 인덱스를 구분할 경우, 유니크 인덱스와 유니크하지 않은 인덱스로 구분**된다.
  * 이는 단순히 중복되는 데이터가 1건만 존재하는지, 그 이상 존재할 수 있는지를 의미한다.
  * 그러나 **이는 동시에 DBMS의 쿼리를 수행할 책임이 있는 옵티마이저에게는 매우 커다란 문제**가 된다.
* **유니크 인덱스는 `=`와 같은 동등 조건으로 검색할 때 1건의 레코드만 찾으면 그 이상 검색할 필요가 없다는 것을 옵티마이저에게 알려주는 효과**를 갖는다.
  * 또한, 이 밖에도 유니크 인덱스로 인한 MySQL 자체의 처리 방식의 변화 또는 차이점 역시 많은 차이점을 보이게 된다. 

## 2022-09-02 Fri
### B-Tree 인덱스란?
```
> B-Tree는 전문 검색과 같은 특수한 요건이 아니라면 대부분의 경우에 사용되는 일반적인 알고리즘이다.
```
* B-Tree 인덱스는 데이터베이스 인덱싱 알고리즘 중 가장 일반적이고, 범용적이며 가장 먼저 도입된 알고리즘이다.
  * 이 때, B는 이진 트리의 Binary가 아닌 Balanced의 앞글자에서 따온 것이다.
* **B-Tree는 기본적으로 칼럼의 앞 부분 일부만을 잘라서 관리하되, 원본 값을 변형하지 않고 항상 정렬된 상태를 유지**한다.

### B-Tree의 구조
* **B-Tree는 최상위에 단일 루트 노드를 갖고, 그 하위에 자식 노드가 붙은 형태**를 갖는다.
  * 이 때, **최하위에 있는 노드는 리프 노드라고 하는 반면 중간의 노드는 브랜치 노드**라고 한다.
* 데이터베이스 상 **인덱스와 실제 데이터는 별도로 관리되지만, 인덱스의 리프 노드는 언제나 실제 데이터 레코드를 찾아낼 수 있는 주소값을 갖는다**.
* 반면, **항상 정렬된 상태를 유지하는 인덱스의 키와는 달리 데이터 파일에 저장되는 실제 레코드는 정렬되지 않고 임의의 순서대로 저장**된다. 
  * 이 때, **데이터 파일의 레코드는 INSERT 만 수행하는 경우가 아니라면 저장된 순서와 맞지 않게 저장될 수 있다**.
  * 이는 **중간 레코드가 DELETE된 경우, DBMS 자체가 빈 공간을 재활용하도록 설계되기 때문**이다.
* 인덱스는 테이블의 키 컬럼만 갖고 있으므로, 나머지 컬럼까지 읽어들이려면 데이터 파일로부터 해당 키를 갖는 컬럼을 찾을 필요가 있다.
  * 이를 위해 MyISAM 엔진의 경우 **리프 노드는 데이터 파일에 저장된 레코드의 실제 주소를 갖는다**.
* **InnoDB의 경우, B-Tree 인덱스와 동일한 구조로 프라이머리 키 인덱스를 갖지만 리프 노드에 실제 레코드가 저장된다는 점에서 차이**가 있다.
  * 이 때, **B-Tree 인덱스의 리프 노드는 프라이머리 키 인덱스의 루트 노드를 가리키는 프라이머리 키를 값으로 저장**한다. 
  * 즉, **MyISAM 엔진의 경우와 달리 InnoDB 엔진은 레코드 주소를 갖지 않는 대신 프라이머리 키 인덱스를 한 번 더 거치게 된다**.

### MyISAM과 InnoDB 인덱스의 차이
* **두 엔진의 인덱스에서 가장 큰 차이는 세컨더리 인덱스를 통해 데이터 파일에 저장된 레코드를 찾아가는 방식**에 있다.
  1. MyISAM: **세컨더리 인덱스의 리프 노드가 물리적인 레코드의 주소를 갖게 되므로, 데이터 파일은 마치 힙 메모리처럼 사용**된다.
  2. InnoDB: **세컨더리 인덱스의 리프 노드는 프라이머리 키를 마치 레코드의 주소처럼 사용하므로, 논리적인 주소**를 갖는다.
* 상술한 이유에서 InnoDB 테이블에서는 인덱스를 통해 레코드를 읽어들일 때 데이터 파일을 바로 찾아갈 수 없다.
  * **대신 B-Tree 인덱스의 리프 노드에 저장된 프라이머리 키로부터 테이블 데이터 레코드의 프라이머리 키 인덱스를 한 번 더 검색**해야 한다.
  * 이후 프라이머리 키 인덱스의 리프 노드에 저장된 레코드를 읽어들이게 된다.
* 이렇듯 **InnoDB 엔진에서는 반드시 세컨더리 인덱스 검색 후 실제 데이터 레코드에 접근하기 위한 프라이머리 키용 B-Tree 인덱스를 한 번 더 검색**한다.
  * 이로 인해 MyISAM보다 성능이 떨어질 것으로 보이지만, 각각의 구조는 서로 다른 장단점을 갖는다.

### B-Tree 인덱스 키 추가
* 테이블 **레코드를 저장하거나 수정하는 경우, 이에 따라 인덱스 키 역시 추가와 삭제 작업이 발생**한다.
  * 이 때, 테이블의 스토리지 엔진 종류에 따라 새로운 인덱스 키 값이 즉시 인덱스에 저장될 수도 있고 그렇지 않을 수도 있다.
  * 일반적으로 **MyISAM 스토리지 엔진을 사용하는 테이블은 즉시 새로운 키 값을 B-Tree 인덱스에 반영**한다.
  * 반면, **InnoDB 스토리지 엔진은 이러한 인덱스 키 추가 작업을 필요한 경우에 지연시키는 등 보다 지능적인 처리가 가능**하다. 
  * **그럼에도 InnoDB 역시 프라이머리 키와 유니크 인덱스는 중복 체크를 위해 즉시 B-Tree 인덱스에 추가하거나 제거**한다.
* **B-Tree 인덱스에 새로운 키가 저장되는 경우, 저장될 키 값을 저장할 적절한 B-Tree 인덱스 위치를 검색하는 작업이 전제**된다.
* 이 때, **저장될 위치가 결정되면 레코드의 키 값과 대상 레코드의 주소 정보를 B-Tree 인덱스의 리프 노드에 저장**한다.
  * 반면, 리프 노드가 이미 꽉 차서 더 이상 저장이 불가능한 경우에는 리프 노드를 분리해야 한다.
  * **이 경우, 상위 브랜치 노드까지 처리 범위가 더 넓어질 수 있으므로 B-Tree 인덱스는 쓰기 작업에 상대적으로 큰 비용이 소모**된다.
* **단순하게 인덱스 키 추가 작업 비용을 계산하는 방법은 레코드 추가를 1로 봤을 때 약 1.5 정도로 예측하는 것**이다. 
  * 다시 말해 **테이블에 인덱스가 없는 경우 작업 비용이 1이라면, 인덱스가 세 개 인 경우에는 4.5를 더한 5.5 정도로 비용을 예측**할 수 있다.
  * 이 때, **해당 비용은 대부분 메모리와 CPU에서 처리되는 시간이 아닌 디스코에 저장된 인덱스 페이지를 읽고 쓰는 데에 소요**된다.

### B-Tree 인덱스 키 제거
* **B-Tree 인덱스의 임의의 키 값이 삭제되는 경우, 해당 키 값이 저장된 B-Tree 인덱스의 리프 노드를 찾아 삭제 마킹하는 것으로 작업이 완료**된다.
  * 이렇게 삭제 마킹된 인덱스 키 공간은 그대로 방치하거나, 재활용할 수 있다.
  * 또한, **이러한 마킹 작업 역시 디스크 IO를 요구하는 작업에 해당**한다.
* **InnoDB 엔진에서는 인덱스 키 제거 작업 역시 버퍼링 및 지연 처리가 적용될 수 있으며, 이는 사용자에게 특별한 영향을 주지 않고 내부적으로 처리**된다.
  * 반면, MyISAM 스토리지 엔진의 경우 체인지 버퍼 기능이 존재하지 않으므로 인덱스 키의 제거가 완료된 후에야 쿼리 실행이 완료된다.

### B-Tree 인덱스 키 변경
* B-Tree 인덱스의 키 값은 그 값에 따라 저장될 리프 노드의 위치가 결정된다.
  * 때문에 **B-Tree 인덱스의 키 값이 변경되는 경우는 단순히 인덱스 상의 키 값만을 변경하는 단순한 형태의 처리가 불가능**하다. 
* **B-Tree 인덱스의 키 값 변경 작업은 우선 키 값을 삭제한 후, 새로운 키 값을 추가하는 형태로 진행**된다.
  * 이 때, 이러한 B-Tree 인덱스 키 값의 삭제 및 추가는 상술한 절차대로 진행된다.
  * **InnoDB 엔진을 활용하는 테이블의경우, 이러한 작업 역시 모두 체인지 버퍼를 활용하여 지연 처리**될 수 있다.

### B-Tree 인덱스 키 값의 검색
* **INSERT, DELETE, UPDATE 작업을 처리함에 있어 추가 비용을 감내하면서도 인덱스를 설정하는 이유는 빠른 검색을 위해서**이다.
* B-Tree 인덱스를 검색하는 과정은 다음과 같이 진행된다.
  1. B-Tree 인덱스의 루트 노드로부터 시작한다.
  2. B-Tree 인덱스의 브랜치 노드를 거쳐 최종적으로 리프 노드까지 이동한다.
  3. 이렇듯 이동하는 과정에서 비교 작업을 반복적으로 수행하며, 이를 트리 탐색이라고 한다.
  4. 적절한 리프 노드를 찾아낸 경우, 스토리지 엔진의 종류에 따라 실제 데이터 레코드를 탐색한다.
* **B-Tree 인덱스 트리 탐색은 SELECT 문 뿐만 아니라 UPDATE, DELETE 등 레코드를 검색한 후에 작업을 진행하는 경우에도 발생**한다.
* B-Tree 인덱스를 활용하는 검색은 다음과 같은 조건 하에서만 사용할 수 있다.
  1. 100% 일치
  2. 전방 일치 탐색
  3. <, >와 같은 부등호 비교 조건
* 반면, 다음과 같은 조건에서는 B-Tree 인덱스를 활용할 수 없다.
  1. 후방 일치 탐색
  2. 인덱스의 키 값이 변형된 경우에 대한 비교
* 특히, **함수 또는 연산을 수행하여 한 번 변형된 값은 사실상 B-Tree 인덱스에 존재하는 키가 아니게 되므로 빠른 검색 기능을 활용할 수 없게 된다**.
* **InnoDB 엔진을 사용하는 테이블 인덱스의 경우, 검색을 수행한 인덱스를 잠근 후에 테이블의 레코드를 잠그는 방식으로 구현**된다.
  * 이로 인해 **UPDATE 또는 DELETE 문장을 실행할 때 적절한 인덱스가 존재하지 않는다면 불필요하게 많은, 심지어는 전체 레코드를 잠글 가능성이 있다**.
  * 이렇듯 InnoDB 스토리지 엔진에서는 인덱스의 설계가 그만큼 중요하고, 또 예상치 못한 많은 부분에 영향을 줄 수 있다. 

## 2022-09-03 Sat
### B-Tree 인덱스 사용에 영향을 주는 요소들
* **B-Tree 인덱스는 인덱스를 구성하는 컬럼의 크기와 레코드 수, 그리도 유니크한 인덱스 키 값의 개수 등에 의해 검색 및 변경 작업 성능이 영향**을 받는다.

### 인덱스 키 값의 크기
* **InnoDB 엔진의 경우, 디스크에 데이터를 저장하는 가장 기본적인 단위를 페이지 또는 블록이라고 지칭**한다.
  * 이는 **디스크의 모든 IO 작업에 대한 최소 작업 단위이며, 버퍼 풀에서 데이터를 버퍼링하는 기본 단위**이기도 하다.
  * 또한, **인덱스 역시 페이지 단위로 관리**된다.
* 이진 트리의 경우 각 노드는 자식 노드를 최대 두 개까지만 가질 수 있으므로, B-Tree가 이진 트리의 형태를 따를 경우 검색 효율은 크게 떨어질 것이다.
  * 이렇듯 **B-Tree의 B는 이진을 의미하지 않으며, B-Tree의 자식 노드는 개수가 가변적**이다. 
* **MySQL의 경우, B-Tree 인덱스는 인덱스의 페이지 크기와 키 값의 크기에 따라 자식 노드의 개수를 결정**한다.
  * MySQL 5.7을 기준으로, InnoDB 스토리지 엔진의 페이지 크기는 `innodb_page_size` 시스템 변수를 활용하여 결정할 수 있다.
  * 또한, **페이지의 크기를 의미하는 해당 시스템 변수의 기본 값은 16KB로 할당**된다.

### 인덱스 페이지 별 자식 노드 결정의 예시
```
> 인덱스는 페이지 단위로 관리된다.
> 인덱스의 페이지 별로 관리 가능한 자식 노드의 개수는 인덱스 페이지와 자식 노드 각각을 나타내는 인덱스 키 + 자식 노드 주소의 조합으로 결정된다.
> 때문에 인덱스 키 값이 커질수록 한 페이지 당 관리하는 자식 노드의 개수가 줄어들게 되므로, SELECT 쿼리의 성능은 떨어지게 된다.
```
* MySQL의 기본 설정을 활용하여 인덱스 페이지의 기본 크기는 16KB라고 할 때, 인덱스의 키와 자식 노드 주소를 각각 16 바이트 / 12 바이트로 가정한다.
  * 이 때, 인덱스 키와 자식 노드 주소는 테이블의 형태에 따라 다른 값을 가질 수 있다.
* 이 경우, 인덱스 당 크기는 28 바이트가 되므로 인덱스 페이지 당 자식 노드의 개수는 16 * 1024 / 28인 585가 된다.
* 이 때, 인덱스의 키 값이 두 배인 32 바이트로 늘어난 경우, 한 페이지당 관리 가능한 인덱스의 키는 같은 원리로 372개가 된다.
  * 만일 **자신이 설계한 SELECT 쿼리가 한 번에 레코드를 500개씩 읽어들인다고 가정할 경우, 전자와 달리 후자는 디스크 IO가 반드시 2번 필요**해진다.
* 이렇듯 **인덱스를 구성하는 키 값의 크기가 커질수록 디스크로부터 읽어들어야 하는 횟수가 늘어나며, 그만큼 쿼리의 성능은 떨어지게 된다**.

### 인덱스 크기와 인덱스 캐싱의 연관 관계
* 상술한 예시와 같이 인덱스의 키 값이 커지는 경우, 개 별 인덱스의 크기 역시 커지므로 전체적인 인덱스의 크기가 커지게 된다.
* 그러나 **인덱스를 캐시하기 위한 InnoDB 엔진의 버퍼 풀 영역은 크기가 제한적**이다.
  * 따라서, **각각의 레코드를 위한 인덱스의 크기가 커질수록 메모리에 캐싱 가능한 레코드의 수는 줄어들게 되어 메모리 효율까지 떨어지게 된다**.

### 인덱스와 페이지 - 중간 정리
* 인덱스는 각 칼럼의 값과 해당 레코드가 저장된 주소를 키 - 값 쌍 형태로 저장하는 개념이다.
* **키 - 값 형태를 갖는 각각의 인덱스는 여럿이 모여 하나의 페이지를 구성**한다.
  * 페이지는 InnoDB 스토리지 엔진 상 모든 디스크 IO의 최소 작업 단위이다.
  * 이 때, 각각의 인덱스는 키에 대응되는 자식 노드 주소 (또는 프라이머리 키) 정보를 값으로 갖게 된다.
* 따라서 **페이지 당 자식 노드 수는 곧 페이지에 포함된 인덱스의 개수를 의미하며, 개 별 인덱스의 크기가 커질 수록 페이지 당 자식 노드의 개수는 줄어든다**.
  * 또한 각각의 인덱스의 크기는 인덱스 별 키와, 각 키에 대응되는 자식 노드 주소 또는 프라이머리 키의 크기에 영향을 받는다.
* 인덱스를 캐시하는 InnoDB 엔진의 버퍼 풀은 크기가 제한적이므로, 인덱스의 크기가 커질수록 캐싱 가능한 인덱스의 수는 줄어들게 되므로 메모리 효율이 떨어진다. 

### B-Tree 인덱스의 깊이
* **B-Tree 인덱스의 깊이는 MySQL 에서 값을 검색하기 위해 몇 번이나 랜덤하게 디스크를 읽어들여야 하는지와 직결되지만, 직접 제어할 방법은 없다**.
* **같은 레코드 개수를 갖는다 하더라도, 인덱스 키 값의 크기가 커지면 이를 표현하기 위해 B-Tree 인덱스의 깊이는 깊어진다**.
  * 이는 앞서 확인했듯 인덱스의 키 값의 크기가 커질수록 개 별 인덱스 페이지가 담을 수 있는 인덱스의 수는 적어지기 때문이다.
  * B-Tree 인덱스의 깊이가 깊어지면 그만큼 디스크 읽기가 더 많이 필요해지므로, 검색 성능은 상대적으로 떨어지게 된다.
* 아무리 대용량 데이터베이스더라도 B-Tree 인덱스의 깊이가 5 이상이 되지는 않지만, 중요한 것은 인덱스 키 값은 가능한 한 작게 만드는 것이 좋다는 것이다.

### 인덱스의 선택도
```
> 선택도, 또는 기수성은 인덱스에서 거의 같은 의미로 사용되며 모든 인덱스 키 값 중 유니크한 값의 수를 의미한다.
```
* 상술한 논리에서 전체 인덱스 키 값 100개 중 유니크한 값의 개수가 10이라면, 기수성은 10이 된다.
  * 당연히 **인덱스 키 값 중 중복되는 값이 많아질수록 기수성과 선택도는 떨어진다**.
* **인덱스는 선택도가 높을수록 검색 대상이 줄어들게 되므로, 처리 속도 역시 빨라질 수 밖에 없다**.
* 즉, 일반적으로 중복을 허용하는 컬럼의 인덱스는 중복을 허용하지 않는 컬럼의 인덱스보다 검색 성능이 떨어진다.
  * 그러나 선택도가 좋지 않은 컬럼에도 정렬 및 그루핑 등의 작업을 위해 인덱스를 추가하는 것이 좋은 경우가 많다.
  * 이렇듯 **인덱스는 반드시 검색 용도로만 사용되는 것이 아니므로, 인덱스 설계 시에는 여러 용도를 고려할 필요**가 있다.

### MySQL과 인덱스의 통계 정보
```
> 인덱스의 통계 정보는 유니크한 값의 개수를 의미한다.
```
* **MySQL에서는 인덱스의 통계 정보가 관리되며, 이에 따른 쿼리 실행 결과를 예측할 수 있다**.
* 이 때, 인덱스된 컬럼에 따라 관리되는 통계 정보는 크게 다음과 같다.
  1. **전체 레코드 개수**
  2. **유니크한 값의 개수**: 선택도
  3. 기타 등등
* **가장 이상적인 것은 정확히 필요한 만큼의 레코드만을 읽어들이는 것이나, 현실적으로 모든 조건을 만족하는 인덱스를 생성하는 것은 불가능**하다.
  * 때문에 **약간의 낭비는 무시하는 것이 바람직**하다.

### 사례 연구 - 동일한 구조의 테이블에서 선택도가 다른 경우
* 국가 별 도시 정보를 10000건 관리하는 테이블이 존재하며, 국가 - 도시 조합은 중복되지 않고 국가 컬럼에만 인덱스가 적용된 상황을 가정한다.
* 이 때, 다음과 같은 두 케이스를 비교할 수 있다.
  1. 국가 컬럼의 유니크한 값이 10개인 경우: 각 국가 별로 대응되는 도시의 개수는 1000개가 된다.
  2. 국가 컬럼의 유니크한 값이 1000개인 경우: 각 국가 별로 대응되는 도시의 개수는 10개가 된다.
* 상술했듯 MySQL 서버는 인덱스된 컬럼에 대해 전체 레코드 수와 선택도와 같은 통계 정보를 관리한다.
  * 때문에 **인덱스 통계 정보의 전체 레코드 수를 선택도로 나누는 것으로 임의의 키로 검색했을 때 몇 건의 레코드가 일치할지 예측할 수 있다**.
* WHERE 절에서 국가와 도시를 명시하여 검색하는 경우 MySQL 서버가 관리하는 인덱스 통계 정보에 따라, 상술한 두 케이스의 조회 결과는 다음과 같이 예측된다.
  1. 임의의 국가를 명시하는 조건으로 인덱스를 검색할 경우 1000 건이 일치하게 되며, 이 중 원하는 도시는 1 건이므로 999 건은 불필요한 결과이다.
  2. 마찬가지 이유에서 임의의 국가를 명시하는 조건으로 검색할 경우 10건이 일치하게 되며, 이 중 원하는 도시는 1 건이므로 9 건은 불필요한 결과이다.
* **이에 따라 첫 번째 케이스의 인덱스는 상대적으로 상황에 적합하지 않은 것으로 판단**할 수 있다.
* 이렇듯 **두 케이스 각각의 테이블로부터 같은 쿼리를 실행하여 같은 결과를 반환받지만, 선택도에 따라 MySQL 서버가 쿼리를 처리하는 작업은 크게 달라진다**.
  * **따라서 인덱스에서 유니크한 값의 개수는 인덱스 또는 쿼리의 효율성에 큰 영향**을 줄 수 있다.

### 읽어들여야 할 레코드 개수
* **인덱스를 통해 테이블의 레코드를 읽는 것은 인덱스를 거치지 않고 테이블의 레코드를 읽는 것보다 큰 비용이 드는 작업**이다.
* 예를 들어 테이블에 저장된 100만 건의 레코드 중 50만 건의 레코드를 읽어들이는 쿼리는 다음과 같은 두 상황을 비교하여 효율적인 것을 골라야 한다.
  1. 전체 테이블을 인덱스 없이 읽어들인 후 필터링하여 50만 건을 버리기
  2. 인덱스를 활용하여 필요한 50만 건만을 읽어들이기
* **일반적인 DBMS 옵티마이저는 인덱스틀 통해 1 건을 조회하는 것이 테이블에서 직접 레코드 1 건을 조회하는 것보다 4 ~ 5배 비용이 드는 작업으로 예측**한다.
  * **따라서 인덱스를 통해 읽어들여야 할 레코드 개수가 전체 테이블 레코드의 20 ~ 25%를 넘어서면 인덱스를 사용하지 않는 것이 이득**이다.
  * 즉, 이러한 경우에는 테이블을 모두 읽어낸 후에 필요한 레코드만을 필터링하는 방식으로 처리하는 것이 효율적이다.
* 이렇듯 **전체 테이블의 20 ~ 25% 이상의 많은 레코드를 읽어들여야하는 경우, 강제로 인덱스를 사용하도록 힌트를 추가하더라도 얻을 수 있는 이점이 없다**.
  * 애초에 이러한 작업은 옵티마이저가 힌트를 무시하고 테이블을 직접 읽어들이는 방식으로 처리할 것이다.

## 2022-09-04 Sun
### B-Tree 인덱스를 활용하는 데이터 조회
* MySQL이 실제 레코드를 얻기 위해 인덱스를 이용하는 대표적인 방법은 크게 다음과 같이 분류된다.
  1. 인덱스 레인지 스캔
  2. 인덱스 풀 스캔
  3. 루스 인덱스 스캔
  4. 인덱스 스킵 스캔
* 어떠한 경우에 인덱스를 사용할지 유도하거나, 사용하지 못하게 하기 위해서는 MySQL이 활용하는 각 방법에 대해서 알고 있어야 한다.

### 인덱스 레인지 스캔
* **인덱스 레인지 스캔은 인덱스의 접근 방식 중 가장 대표적인 접근 방식이며, 다른 인덱스 접근 방식보다 빠른 방법**이기도 하다.
  * 이 때, 인덱스를 통해 레코드를 한 건만 조회하는 경우와 한 건 이상 조회하는 경우 각각은 다른 이름으로 구분된다.
* 예를 들어, `first_name` 컬럼에 인덱스가 적용되었을 때 다음과 같은 쿼리를 요청한다고 가정한다.
  * 이 경우, 인덱스의 루트 노드부터 시작하여 Ebbe와 Gad 인덱스 키를 갖는 리프 노드까지 인덱스를 검색하게 된다.
```
SELECT * FROM employees WHERE first_name BETWEEN 'Ebbe' AND 'Gad';
```
* **상술한 쿼리에서, 리프 노드 페이지가 인덱스 값으로 Ebbe와 Gad를 갖는 인덱스 사이에 위치하는 인덱스를 모두 검색**하게 된다.
* **인덱스 레인지 스캔은 이렇듯 검색 대상 인덱스의 범위가 결정되어 있을 때 사용하는 방식**이다.
  * 이 때, 검색 대상 값의 개수나 검색 결과 레코드 수와 관계 없이 레인지 스캔이라고 표현할 수 있다.
* 인덱스 레인지 스캔의 과정은 크게 다음과 같이 진행된다.
  1. B-Tree 인덱스의 루트 노드로부터 비교를 시작하여 브랜치 노드를 거쳐 리프 노드까지 탐색한다.
  2. 리프 노드까지 탐색한 시점에서야 필요한 레코드의 시작 지점을 알 수 있게 된다.
  3. **한 번 시작해야 할 위치를 찾아냈다면, 이후로는 해당 위치로부터 리프 노드의 레코드를 순서대로 읽어나간다**.
     * 이렇듯 **레코드를 차례대로 읽어나가는 과정을 인덱스 스캔이라고 표현**한다.
  4. **스캔 과정에서 리프 노드 페이지의 끝까지 조회했다면, 리프 노드 간의 링크를 활용하여 다음 리프 노드를 찾아 계속해서 스캔**한다.
  5. 스캔을 멈춰야 하는 위치에 다다른 경우, 현재까지 읽은 레코드를 사용자에게 반환하고 쿼리를 완료한다.
* 이렇듯 B-Tree 인덱스의 루트 / 브랜치 노드를 활용하여 스캔 시작 위치를 찾아내고, 그 후에는 필요한 방향으로 인덱스를 읽어나간다.
  * **인덱스 탐색 방향은 오름차순이거나 내림차순**일 수 있다.
* 이 때, **어떠한 방식으로 인덱스를 스캔하더라도 해당 인덱스를 구성하는 컬럼의 순서대로, 또는 역순으로 정렬된 레코드를 조회**하게 된다.
  * 이는 별도로 정렬을 수행하기 때문이 아니며, 인덱스 자체가 정렬되는 특성을 갖기 때문에 그렇다.
* **중요한 것은 인덱스 리프 노드로부터 스캔하는 과정에서 검색 조건과 일치하는 레코드는 모두 개별로 데이터 파일로부터 조회하는 과정이 필요하다는 점**이다.
* 이 과정에서 **리프 노드에 저장된 레코드 주소로 데이터 파일의 레코드를 읽어들이며, 레코드 한 건당 랜덤 IO가 1회 발생**한다.
  * **이러한 이유에서 인덱스를 활용하여 레코드를 읽어들이는 작업은 비용이 많이 드는 작업으로 분류**된다.
  * 또한, 상술한 바와 같이 인덱스를 통해 읽어들일 레코드가 전체 테이블의 20%를 넘기 시작하면 테이블의 데이터를 직접 읽는 것이 더 효율적일 수 있다.

### 커버링 인덱스
* 상술한 바와 같이, 인덱스 레인지 스캔은 크게 다음과 같은 과정으로 진행된다.
  1. index seek: 인덱스 탐색이라고도 하며, 인덱스의 리프 노드에서 조건을 만족하는 값이 저장된 위치를 찾는다. 
  2. index scan: **탐색된 위치로부터 인덱스를 순차대로 읽어나가며, 1.의 과정과 합쳐 인덱스 스캔으로 통칭하기도 한다**.
  3. 2.의 과정으로부터 읽어 들인 인덱스의 키와 레코드 주소를 활용하여 레코드가 저장된 페이지를 가져온 후, 최종 레코드를 읽어 반환한다.
* 이 때 **쿼리가 요청하는 데이터에 따라 3.의 과정은 필요하지 않을 수도 있으며, 이를 커버링 인덱스**라고 한다.
  * **커버링 인덱스로 처리되는 쿼리는 디스크의 실제 레코드를 읽는 과정이 생략되므로, 랜덤 IO가 크게 줄어들고 성능 역시 그만큼 빨라진다**.

### 인덱스 풀 스캔
* **인덱스 풀 스캔은 인덱스를 사용하여 인덱스의 처음부터 끝까지 모두 읽는 방식을 의미**한다.
* 대표적으로, **쿼리의 조건절에 사용된 컬럼이 인덱스의 첫 번째 컬럼이 아닌 경우 인덱스 풀 스캔이 적용**된다.
  * 예를 들어 인덱스는 컬럼 1, 2, 3 순서로 적용되었으나 WHERE 절에서 2 또는 3 컬럼으로 검색하는 경우를 의미한다.
* 일반적으로 **인덱스의 크기는 테이블 크기보다 작으므로, 직접 테이블 전체를 읽는 것보다는 인덱스만 읽는 것이 항상 효율적**이다.
  * 인덱스의 전체 크기는 테이블 자체의 크기보다 훨씬 작으므로, 인덱스 풀 스캔은 테이블 전체를 읽는 것보다는 적은 디스크 IO로 쿼리를 처리할 수 있다.
  * 이렇듯 **인덱스 풀 스캔은 레인지 스캔보다는 느리지만, 테이블 풀 스캔보다는 효율적**이다.
* **인덱스 풀 스캔은 쿼리가 인덱스에 명시된 컬럼만으로 모든 조건을 처리할 수 있는 경우에 사용**된다.
  * 반면, **인덱스에 명시된 컬럼만으로는 쿼리를 처리할 수 없어 실제 데이터 레코드를 읽어들어야 한다면 인덱스 풀 스캔으로 처리될 수 없다**.
* 인덱스 풀 스캔은 다음과 같은 흐름으로 진행된다.
  1. 우선 인덱스 리프 노드의 처음 또는 가장 마지막으로 이동한다.
  2. **인덱스의 리프 노드 각각을 연결하는 링크드 리스트를 따라 처음부터 끝까지 스캔**한다.

### 인덱스 레인지 스캔과 인덱스 풀 스캔의 비교
* 인덱스 레인지 스캔은 검색 대상 인덱스의 범위가 결정되었을 때 사용하는 방식이다.
  * 인덱스 풀 스캔은 조건 절에 사용된 컬럼이 인덱스의 첫 컬럼이 아니지만, 인덱스에 명시된 컬럼만으로 조건을 처리할 수 있는 경우에 적용되는 방식이다.
* 인덱스 레인지 스캔은 인덱스 범위를 순차 탐색하며 조건에 맞는 레코드를 확인할 때마다 랜덤 디스크 IO를 사용한다.
  * 인덱스 풀 스캔은 인덱스에 포함된 컬럼만으로 쿼리를 처리할 수 있다면 테이블 레코드를 읽지 않으므로, 테이블 전체를 읽는 것보다 적은 디스크 IO를 사용한다. 

### 효율적이지 못한 인덱스의 사용
* **일반적으로 `인덱스를 사용한다`는 표현은 인덱스 레인지 스캔, 또는 루스 인덱스 스캔 방식으로 인덱스를 사용하는 것을 의미**한다.
* 반면, **`인덱스를 사용하지 못한다` 또는 `인덱스를 효율적으로 사용하지 못했다`는 표현은 테이블 전체를 읽거나 인덱스 풀 스캔을 사용한 경우를 의미**한다.
  * **인덱스 풀 스캔 역시 인덱스를 활용하지만 효율적인 방식은 아니며, 일반적으로 인덱스를 사용하는 목적으로 볼 수는 없다**.

### 루스 인덱스 스캔
* **루스(loose) 인덱스 스캔은 이름 그대로 느슨하게, 또는 듬성듬성하게 인덱스를 읽는 것을 의미**한다.
  * 앞서 다룬 **인덱스 레인지 스캔과 인덱스 풀 스캔은 루스 인덱스 스캔과 대비되는 의미에서 타이트(tight) 인덱스 스캔으로 분류**된다.
  * 이러한 방식은 다른 DBMS의 인덱스 스킵 스캔과 유사하며, MySQL 8.0 버전 이후부터 지원되기 시작한 인덱스 방식이다.
  * 루스 인덱스 스캔을 사용하려면 여러 조건을 만족해야 한다.
* **루스 인덱스 스캔은 인덱스 레인지 스캔과 유사하게 동작하지만, 중간에 필요하지 않은 인덱스 키 값은 무시하고 다음으로 넘어가는 형태**를 취한다.
  * 일반적으로 GROUP BY 또는 MAX / MIN 함수에 대한 최적화에 사용될 수 있다.
* 예를 들어, dept_emp 테이블은 dept_no와 emp_no라는 두 컬럼으로 인덱스가 생성되어 있을 때 다음과 같은 쿼리를 요청한다고 가정한다.
```
SELECT dept_no, MIN(emo_no)
FROM dept_emp
WHERE dept_no BETWEED 'd002' AND 'd004'
GROUP BY dept_no;
```
* 이 때, **해당 테이블은 dept_no와 emp_no를 인덱스 컬럼으로 사용하므로 dept_no 별로 emp_no를 기준으로 정렬한 결과를 인덱스로 갖는다**. 
  * 따라서 dept_no 그룹 별로 첫 번째에 위치한 레코드의 emp_no 값만을 읽으면 해당 dept_no에 위치한 다른 레코드들을 읽을 필요가 없다.
* 이렇듯 **옵티마이저는 인덱스에서 WHERE 조건을 만족하는 모든 범위를 다 스캔할 필요가 없다는 사실을 알게 된다**.
  * 때문에 **조건에 만족하지 않는 레코드는 스캔 대상에서 제외하여 무시하고, 다음 레코드로 이동하는 식으로 필요한 부분만을 읽어들인다**.

## 2022-09-05 Mon
### MySQL 8.0의 인덱스 스킵 스캔
```
> 데이터베이스에서 인덱스의 핵심은 값이 정렬되어 있다는 점이다.
> 때문에 인덱스를 구성하는 컬럼의 순서 역시 매우 중요하다.
```
* 예를 들어 gender와 birth_date 컬럼으로 구성된 인덱스를 갖는 테이블의 경우, WHERE 절에서 birth_date 만을 사용하는 쿼리는 인덱스를 사용할 수 없다.
  * 이 경우, 일반적으로는 birth_date 컬럼부터 시작하는 인덱스를 새로 생성해야만 했다.
* **MySQL 8.0 부터는 옵티마이저가 gender를 건너 뛰고 birth_date 컬럼만으로도 인덱스를 사용할 수 있도록 하는 인덱스 스킵 스캔 최적화가 추가**되었다.
* MySQL 8.0 이전부터 유사한 최적화를 수행하던 루스 인덱스 스캔과는 다음과 같은 차이점이 존재한다.
  1. **루스 인덱스 스캔은 GROUP BY 작업을 처리하기 위해 인덱스를 사용하는 경우에만 적용이 가능**하다.
  2. 반면, **인덱스 스킵 스캔은 WHERE 절의 검색을 위해서도 사용이 가능**하다.

### 인덱스 스킵 스캔을 적용하지 않는 경우
* 상술한 바와 같이, gender와 birth_date 컬럼에 대해 인덱스가 적용된 테이블에 대해 다음과 같은 쿼리를 요청한다고 가정한다.
```
SELECT gender, birth_date
FROM employees
WHERE birth_date >= '1965-02-01'
```
* 해당 쿼리의 경우, **다음과 같은 이유에서 인덱스 풀 스캔을 사용**하게 된다.
  1. 쿼리에서 요구하는 결과가 gender와 birth_date 컬럼 뿐이다.
     * 만약 **`SELECT *`과 같이 모든 컬럼을 요구한 경우, 인덱스 풀 스캔이 아닌 테이블 풀 스캔이 실행**된다. 
  2. **두 컬럼은 모두 인덱스가 적용되어 있으므로, 인덱스 만으로 쿼리의 요청 결과를 반환할 수 있다**.
  3. 그러나 **인덱스의 순서는 gender, birth_date 이므로 WHERE 절의 비교 조건을 기존 인덱스만으로 효율적으로 해결할 수 없다**.
* 즉, **기존 인덱스와 상술한 쿼리에서는 인덱스 풀 스캔 방식을 적용하므로 인덱스를 효율적으로 이용하지 못하는 경우**가 된다.

### 인덱스 스킵 스캔이 활성화되어 적용된 경우
* **인덱스 스킵 스캔이 활성화된 경우, 상술한 쿼리에서도 옵티마이저는 기존 인덱스에 대해 인덱스 스킵 스캔을 활용하여 필요한 데이터를 조회**한다.
* 이 때, MySQL 옵티마이저는 다음과 같은 순서로 쿼리를 실행한다.
  1. 우선 **루스 인덱스 스캔과 유사한 방식으로 gender 컬럼으로부터 유니크한 값을 모두 조회**한다.
  2. 사용자가 요청한 **쿼리에 대해 조회한 모든 값을 AND 조건으로서 추가**한다.
  3. 쿼리를 실행한 후 결과를 반환한다.
* 만역 gender 컬럼이 M과 F만을 갖는 이넘이라면 옵티마이저는 내부적으로 `gender = 'M' AND`와 `gender = 'F' AND`를 추가한 쿼리를 요청하게 된다.
  * 컬럼이 **이넘이 아니더라도 MySQL 서버는 기존 인덱스를 루스 인덱스 스캔 방식으로 읽어들여 인덱스에 존재하는 모든 값을 추출**한다.
  * 이후 해당 결과를 활용하여 인덱스 스킵 스캔을 실행한다.

### 인덱스 스킵 스캔의 단점
* 인덱스 스킵 스캔은 MySQL 8.0에서야 추가된 기능으로, 다음과 같은 단점 역시 존재한다.
  1. 상술한 **gender 컬럼과 같이, WHERE 절에 조건이 없는 인덱스의 선행 컬럼이 갖는 유니크한 값의 종류가 적어야** 한다. 
  2. **쿼리가 인덱스에 존재하는 컬럼만으로도 처리가 가능한 커버링 인덱스 형태여야 한다**. 
* 인덱스 선행 컬럼이 갖는 유니크한 값의 개수가 매우 많은 경우, MySQL 옵티마이저는 인덱스에서 스캔을 시작한 지점을 검색하는 작업을 많이 수행하게 된다.
  * 예를 들어 레코드 수 만큼 선행 컬럼이 유니크한 값을 갖는 경우에 해당하며, 이 경우에는 쿼리의 처리 성능이 오히려 느려질 수도 있다.
  * 즉, **인덱스 스킵 스캔은 인덱스의 선행 컬럼이 갖는 유니크한 값의 종류가 적을 때에만 적용 가능한 최적화에 해당**한다.
* **WHERE 절은 동일하지만 `SELECT *`과 같이 테이블의 모든 컬럼을 조회하는 쿼리의 경우, 인덱스에 포함된 컬럼 정보만으로는 결과를 반환할 수 없다**.
  * 예를 들어, 상술한 시나리오에서는 gender와 birth_date 이외의 컬럼을 요청하는 경우 인덱스 정보만으로 응답할 수 없게 된다.
  * 따라서 **인덱스 스킵 스캔을 적용하지 못하며, 풀 테이블 스캔으로 쿼리 실행 계획을 수립**하게 된다.

### 다중 컬럼 인덱스, 또는 복합 컬럼 인덱스
* **실무에서는 데이터베이스에 단일 컬럼 인덱스 보다는 두 개 이상의 컬럼을 포함하는 인덱스를 더 많이 사용**하게 된다.
  * 이 때, 둘 이상의 컬럼이 연결되므로 `concatenated index`라는 용어로 지칭하는 경우도 있다.
* 다중 컬럼 인덱스에서는 데이터 레코드의 개수가 적은 경우, 브랜치 노드가 없을 수도 있다.
  * **반면, 루트 노드와 리프 노드는 반드시 존재**한다.
* **다중 컬럼 인덱스에서 가장 중요한 것은 두 번째 인덱스 컬럼이 첫 번째 컬럼에 의존하여 정렬된다는 사실**이다.
* 예를 들어, number 와 alphabet 컬럼의 순서대로 인덱스를 적용한 경우 리프 노드에는 다음과 같이 정렬될 수 있다.
```
number  | alphabet
1       | E
1       | J
1       | N
1       | U
2       | B
2       | D
2       | P
2       | T
3       | A
... 이하 생략
```
* 이렇듯 **인덱스의 두 번째 컬럼의 정렬은 첫 번째 컬럼이 동일한 레코드에서만 의미가 있다**.
* 인덱스의 컬럼이 둘 이상으로 적용되는 경우 역시 N 번째 컬럼은 항상 N-1 번째 컬럼에 의존하여 정렬된다.
  * 즉, 컬럼이 네 개인 인덱스의 경우 우선 첫 번째 컬럼이 정렬된 후 두 번째 컬럼은 첫 번째 컬럼에, 세 번째 컬럼은 두 번째 컬럼에 의존하는 식으로 정렬된다.
  * 이와 같은 이유에서 상술한 인덱스와 같이 순서상 앞서는 첫 번째 알파벳 A의 순서가 뒤로 밀려날 수 있게 된다.
* **다중 컬럼 인덱스에서는 인덱스 내에 설정된 각 컬럼의 순서가 매우 중요하며, 이는 반드시 신중히 숙고하여 결정되어야 한다**.

### B-Tree 인덱스의 내림차순, 또는 오름차순 정렬
```
> 오름차순으로 정렬되어 저장된 인덱스는 거꾸로 읽는 것으로 내림차순처럼 사용될 수 있다.
> 이러한 인덱스 스캔 방향은 쿼리에 따라 옵티마이저가 생성하는 실행 계획에 따라 결정된다.
```
* **인덱스를 생성했을 때 설정한 정렬 방식에 따라 인덱스의 키는 항상 오름차순이거나 내림차순으로 정렬되어 저장**된다.
* 그러나 **중요한 것은 임의의 인덱스가 오름차순으로 정렬되었다고 해서 인덱스를 내림차순으로 읽을 수 없다는 의미는 아니라는 사실**이다.
  * 예를 들어, 인덱스를 마지막에서부터 역순으로 읽어올라오는 경우 마치 내림차순으로 정렬된 인덱스처럼 사용할 수 있다.
* 이 때, **인덱스의 스캔 방향은 쿼리에 따라 옵티마이저가 실시간으로 작성하는 실행 계획에 따라 결정**된다.

### 인덱스의 정렬
* **일반적인 DBMS에서는 인덱스의 생성 시점에 인덱스를 구성하는 각 컬럼의 정렬을 ASC 또는 DESC로 설정**할 수 있다.
* MySQL의 경우, 5.7 버전까지는 인덱스 단위로 오직 한 종류의 정렬 방식만을 사용할 수 있다.
  * 예를 들어, 상술한 예시에서 number 컬럼은 오름차순으로 정렬하되 alphabet 컬럼은 내림차순으로 정렬할 수는 없다.
  * 정확히는 컬럼 별로 다른 정렬 방식을 적용한 인덱스를 생성하더라도  SQL 구문 오류가 발생하지는 않으나, 실제로는 모두 오름차순이 적용된다.
  * **이는 추후에 추가될 버전에 대한 호환성을 미리 보장하기 위해 문법적으로만 표면적으로 지원한 기능에 해당**한다.
* 반면, **MySQL 8.0 버전 이후로는 인덱스를 구성하는 컬럼 별로 정렬 순서를 혼합하는 인덱스를 생성할 수 있게 되었다**.
  * 따라서 상술한 바와 같이 number 컬럼은 오름차순으로 정렬하되 alphabet 컬럼은 내림차순으로 정렬하는 식의 설정이 가능하다.

## 2022-09-06 Tue
### 인덱스의 스캔 방향
* 오름차순으로 정렬된 인덱스에 포함된 컬럼에 대해 `ORDER BY column_name DESC LIMIT 1;` 쿼리를 실행할 경우, 언뜻 풀스캔이 필요한 것처럼 보인다.
  * 이는 해당 쿼리가 column_name이 가장 큰 값을 구하는 용도로 사용되기 때문이다.
* 그러나 **MySQL 옵티마이저는 인덱스를 역순으로 읽으면 내림차순 효과를 얻을 수 있다는 사실을 이미 알고 있으며, 이를 실행 계획에 반영**한다.
  * 따라서, 위와 같은 쿼리는 인덱스를 역순으로 접근하여 최초의 레코드 하나만 읽어 들이는 식으로 동작한다.
* 이렇듯 **인덱스는 생성시 한 방향으로 정렬되지만, 쿼리가 해당 인덱스를 사용하는 시점에 읽는 방향에 따라 반대 방향으로 정렬되는 효과**를 얻을 수 있다.
* 예를 들어, 오름차순으로 생성된 인덱스를 정방향으로 읽으면 오름차순 정렬이며, 역방향으로 읽어들이면 내림차순 정렬과 같다.
  * **MySQL 옵티마이저는 쿼리의 ORDER BY 또는 MIN / MAX 함수 등에서 최적화가 필요한 경우에도 인덱스의 읽기 방향을 전환하여 실행 계획을 작성**한다.

### 내림차순 인덱스 살펴보기
```
> 내림차순 인덱스든 오름차순 인덱스든, 정순으로 스캔하는 것이 더 좋은 성능을 낸다.
```
* 상술한 바와 같이, MySQL에서는 인덱스의 정렬 방식에 관계 없이 읽는 순서를 전환하는 것으로 오름차순과 내림차순을 적절하게 선택할 수 있다.
  * 오름차순 인덱스와 내림차순 인덱스의 차이는 작은 값의 인덱스 키가 B-Tree의 어느 방향으로 정렬되는지 차이에 있다.
  * 예를 들어, **오름차순 인덱스는 작은 값의 인덱스 키가 B-Tree의 왼쪽으로 정렬되는 반면 내림차순 인덱스에서는 오른쪽으로 정렬**된다.
* 이 때, **InnoDB에서는 리프 노드의 오른쪽 페이지부터 스캔하는 인덱스 역순 스캔이 왼쪽부터 스캔하는 인덱스 정순 스캔에 비해 느리다**.
  * 즉, 하나의 인덱스를 정순으로 읽어들이는지 역순으로 읽어들이는지에 따라 가시적인 성능 차이를 확인할 수 있다.
* InnoDB에서 인덱스 역순 스캔이 정순 스캔보다 더 느릴 수 밖에 없는 이유는 크게 다음과 같다.
  1. **페이지 잠금이 인덱스 정순 스캔에 더 적합한 구조**를 갖는다.
  2. **페이지 내부에서 인덱스 레코드는 단방향 링크만을 갖는 구조**이다.
     * **반면, 페이지 간에는 양방향 연결 고리**를 갖는다.
* 일반적으로 ORDER BY 절에 의해 내림차순 정렬하는 쿼리가 적은 레코드에 가끔 실행되는 경우, 굳이 내림차순 인덱스를 고려할 필요는 없다.
* 그러나 이러한 **쿼리가 많은 레코드를 조회하면서도 자주 실행되는 경우, 상술한 이유에서 오름차순 인덱스보다 내림차순 인덱스가 더 효율적**이다.

### 인덱스 정렬을 활용한 병목 현상 완화
* 대다수의 쿼리가 인덱스의 앞 쪽만, 또는 뒤 쪽만 집중적으로 읽어들이는 탓에 인덱스의 특정 페이지 잠금이 병목의 원인이 될 것으로 '예상'되는 경우가 있다.
* 이 경우, **쿼리에서 자주 사용되는 정렬 순서에 맞추어 인덱스를 생성하는 것이 페이지 잠금에 의한 병목 현상을 완화하는데 도움**이 될 수 있다.

## 2022-09-07 Wed
### 비교 조건의 종류와 효율성
* 예를 들어 `SELECT * FROM dept_emp WHERE dept_no='d002' AND emp_no >= 10114;`와 같은 쿼리에 다음의 인덱스를 적용한다고 가정하자.
  1. INDEX (dept_no, emp_no)
  2. INDEX (emp_no, dept_no)
* 1.의 경우, 우선 `dept_no='d002' AND emp_no >= 10114`를 만족하는 최초의 레코드를 찾은 후 dept_no가 d002가 아닐 때까지 인덱스를 읽어 나간다.
* 반면, 2.의 경우 우선 `emp_no >= 10114 AND dept_no='d002'`인 레코드를 찾고 나서 이후의 모든 레코드에 대해 dept_no를 비교한다.
  * 이렇듯 **인덱스를 통해 읽은 레코드가 나머지 조건에 맞는지 비교하여 걸러내는 작업을 필터링**이라고도 한다.
* 이 때, 1.의 인덱스의 두 번째 컬럼인 emp_no는 비교 작업의 범위를 좁혀주지만 2.의 인덱스에서 사용된 dept_no는 아무런 영향을 주지 못한다.
* 이렇듯 WHERE 절에서 사용되는 조건 절은 다음과 같이 분류될 수 있다.
  1. 작업 범위 결정 조건: 작업의 범위를 실제로 결정할 수 있는 조건을 의미한다.
  2. 필터링 조건: 비교 작업의 범위를 줄이지 못하며, 단순한 거름망의 역헐만을 수행한다.
* 이 때, 1.의 dept_no와 emp_no는 모두 작업 범위 결정 조건이지만 2.에서는 emp_no 만 작업 범위 결정 조건에 해당한다.
* **작업 범위 결정 조건은 많을수록 쿼리의 처리 성능을 높이나, 필터링 조건은 많다고 해서 쿼리의 성능을 높이지 못한다**.
  * **많은 필터링 조건은 오히려 쿼리의 실행을 더 느리게 만들 가능성**이 있다.

### 왼쪽 기준 정렬 기반의 B-Tree 인덱스
```
> 인덱스 키 값이 정렬된다는 특징은 그 자체로 빠른 검색을 가능케하는 전제 조건이 된다.
```
* **B-Tree 인덱스의 가장 큰 특징은 왼쪽 값에 기준하여 오른쪽 값이 정렬된다는 사실**이다.
  * 예를 들어 단일 컬럼 인덱스의 경우, 문자열 컬럼을 예로 들면 왼쪽 첫 글자에 의존하여 두 번째 글자가 정렬되는 식이다.
  * 다중 컬럼 인덱스의 경우, 왼쪽 컬럼에 의존하여 두 번째 컬럼이 정렬된다.
* 때문에 **하나의 컬럼으로 검색하더라도 값의 왼쪽 부분이 없는 경우, 인덱스 레인지 스캔 방식의 검색은 불가능**하다.
  * 이는 **다중 컬럼 인덱스에서도 마찬가지이며, 왼쪽 컬럼의 값을 알지 못하는 경우 인덱스 레인지 스캔은 불가능**하다.

### 단일 컬럼 인덱스에서의 가용성
* 예를 들어, **문자열 컬럼을 대상으로 한 `LIKE '%검색어'` 형태의 검색 조건은 값의 왼쪽이 없으므로 인덱스 레인지 스캔을 적용할 수 없다**.
  * 이렇듯 왼쪽 기준 정렬 기반의 인덱스인 B-Tree 에서는 정렬 우선 순위가 낮은 뒷 부분만으로는 인덱스의 효과를 제대로 누릴 수 없다.

### 다중 컬럼 인덱스에서의 가용성
* **다중 컬럼 인덱스의 경우, 선행 컬럼 조건 없이 후행 컬럼의 값만으로 검색할 경우 인덱스를 효율적으로 사용할 수 없다**.
  * **B-Tree 인덱스는 왼쪽 값에 기준하여 정렬되므로, 인덱스의 키 값은 항상 선행 컬럼이 모두 정렬된 후에 후행 컬럼이 순서대로 정렬**된다.
  * 이는 **WHERE 절은 물론 GROUP BY, ORDER BY 절에서도 모두 동일하므로 가능한 한 조건 절에는 되도록 인덱스의 선행 컬럼을 사용하는 것이 유리**하다.