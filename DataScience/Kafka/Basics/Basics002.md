# Basics
## 2023-02-16 Thu
### 카프카 클러스터 운영 방식
* 카프카 클러스터를 직접 운영하는 경우, 다음과 같은 방식을 고려할 수 있다.
  1. 직접 서버에 설치한 후에 운영
  2. 또는 SaaS형 서비스를 도입
* 직접 서버에 설치하는 방식의 경우 설정을 세부적으로 최적화하여 최고의 성능으로 클러스터를 활용할 수 있으나, 운영자의 노하우가 지대한 영향을 미친다.
  * 반면, 컨플루언트 클라우드나 AWS MSK 등의 SaaS 서비스를 사용하는 경우 누구나 쉽고 빠르게 카프카 클러스터를 운영할 수 있다.
* 또한, 카프카는 다시 Apache가 제공하는 오픈소스 카프카와 컨플루언트가 제공하는 기업용 카프카로 분류될 수 있다.
  * 이 경우, 기업용 카프카는 커넥트 또는 모니터링 측면에서 이점이 있다.

### 카프카 클러스터 운영 시 하드웨어 권장 사양
* 컨플루언트의 경우, 상용 환경에서 카프카 클러스터를 운영하는 경우 다음과 같이 비교적 고성능의 브로커 서버 설정을 권장한다.
  1. 메모리: 32GB를 할당하되, 6GB는 힙 메모리로 설정하고 그 외는 OS의 페이지 캐시 용도로 사용
  2. CPU: 24코어를 할당하되, SSL과 같은 보안 설정을 사용하는 경우 더 높은 사양을 적용
  3. 디스크: RAID 10을 사용하는 디스크를 사용하되, NAS는 사용하지 않아야 함
  4. 파일 시스템: XFS 또는 ext4를 사용
* 특히, 페이지 캐시 영역의 메모리는 OS가 관리하며 프로듀서 또는 컨슈머 간의 상호작용시 힙 메모리 대신 사용하여 성능을 높이는 용도로 사용될 수 있다.
  * 이렇듯 대부분의 메모리 영역을 페이지 캐시 용도로 사용하므로, 실제 데이터 처리량은 더 높아지는 효과를 얻을 수 있다.
* 또한 카프카 클러스터에 포함되는 브로커의 개수는 개발 환경인 경우 5대, 운영 환경인 경우 10대가 권장된다. 
  * 그러나 앞서 다룬 바와 같이 카프카 클러스터에 포함된 브로커의 scale out은 쉬운 편에 속하므로, 정확히 몇 대를 유지할지에 대한 많은 고민은 필요가 없다.
* 나아가 디스크의 경우, 정확히 어느 정도의 용량을 권장하는지에 대한 기준은 없으므로 환경 및 실제 사용량에 따라 결정하는 것이 바람직하다.

## 2023-02-17 Fri
### 컨플루언트 카프카
* 컨플루언트는 카프카 생태계를 발전시키는데 핵심적인 역할을 맡는 회사이며, SaaS형 카프카 제품을 제공한다.
  * 예를 들어, 오픈소스 프로젝트가 활성화되려면 해당 프로젝트 뿐만 아니라 이를 돕는 주변 도구들이 많이 개발될 필요가 있다.
  * 카프카 역시 ksqlDB와 커넥터 등 카프카 생태계를 구성하는 여러 도구들이 있으며, 컨플루언트는 이러한 도구들의 개발에도 핵심적인 역할을 수행한다.

### 컨플루언트 카프카 플랫폼
* 컨플루언트 카프카는 다음과 같은 두 가지 플랫폼 형태를 제공한다.
  1. 클라우드 기반 클러스터
  2. 온프레미스 기반 설치형 클러스터
* 이 때, 클라우드 기반 클러스터는 그 특성에 따라 SaaS형 제품으로서 탄력적인 리소스를 제공한다.
* 반면, IaaS와 온프레미스 기반의 카프카 클러스터는 지속적으로 디스크 사용량을 모니터링하고 적절한 할당량을 고민할 필요가 있다.
  * 그러나 SaaS형 카프카 클러스터를 사용하는 경우 디스크 사용량에 대한 제한이 사실상 없으므로 프로듀서와 컨슈머는 브로커와 제한 없이 상호작용할 수 있다.
* 일반적으로 비즈니스에 아무런 제약이 없는 경우에는 SaaS형 플랫폼이 권장된다.
  * 반면, 데이터가 해외에 저장되지 않아야 하거나 이미 온프레미스 환경을 구축한 경우에는 온프레미스 기반의 카프카 클러스터를 고려할 수 있다.

### AWS MSK란?
* AWS MSK는 컨플루언트와 유사하게 AWS 자체적으로 제공하는 SaaS형 카프카 서비스를 의미한다.
  * 이 때, 해당 서비스는 기본적으로 관리형 서비스이므로 운영자는 간편하게 클러스터를 운영할 수 있다.
  * 예를 들어, 카프카 클러스터 운영 담당자는 주키퍼 등의 애플리케이션을 관리할 필요가 없다.
* 또한 MSK는 AWS에서 제공되는 여러 서비스와 간단한 연동이 가능하므로, 이미 AWS를 사용 중인 기업은 MSK의 사용을 우선적으로 고려할 수 있다.

## 2023-02-18 Sat
### SaaS형 카프카 클러스터의 장점
* SaaS형 카프카 클러스터를 관리할 경우, 기본적으로 최소 3대의 브로커 서버를 운영하는 운영 담당자 입장에서의 부담은 크게 줄어들 수 있다.
  * 예를 들어, 브로커 서버 중 하나에 이슈가 발생하더라도 담당자가 이를 직접 모니터링하고 복구할 필요는 없다.
  * 나아가 브로커 서버의 스케일 아웃 역시 온프레미스 환경에 비해 간단하며, 이로 인해 운영 담당자는 카프카 클러스터의 관리로부터 상당 부분 자유로울 수 있다.
* 또한 운영자가 활용할 수 있는 모니터링 대시보드가 제공되며, 운영에 필요한 여러 지표를 손쉽게 얻을 수 있어 관리 난이도는 상대적으로 낮아질 수 밖에 없다.
  * 반면, 이러한 모니터링 도구의 부재는 오픈소스로 제공되는 카프카 클러스터의 한계이기도 하다.
* 카프카 클러스터 역시 보안을 중시해야 하며, 적절한 설정을 하지 않을 경우 공격자는 호스트 정보와 포트 만으로도 토픽의 모든 데이터를 탈취할 수 있다.
  * 나아가 admin API를 통해 클러스터 자체에 악의적인 공격을 수행할 수도 있다.
  * 이러한 상황을 미연에 방지하기 위해서는 적절한 방어책 설정이 필수적이며, SaaS형 플랫폼은 이러한 보안 설정을 기본적으로 제공한다는 점에서 의미가 있다.

### SaaS형 카프카 클러스터의 단점
* 이러한 관리형 서비스의 특징이 으레 그렇듯, SaaS형 카프카 클러스터는 많은 편의 기능과 운영 용이성을 제공하는 대신 높은 유지 비용을 필요로 한다.
  * 예를 들어, 동일 사양의 브로커 서버를 구성할 때 AWS EC2로 직접 구성하여 운영하는 비용에 비해 AWS MSK로 운영하는 비용은 2배에 달한다.
* 또한 SaaS형 카프카 클러스터는 직접 운영하는 것에 비해 세부적인 설정을 적용하는 것이 어려우며, 멀티 클라우드 또는 하이브리드 클라우드 구성이 불가능하다.
  * 이로 인해 클라우드 종속성이 발생할 수 밖에 없으며, 마이그레이션 역시 쉽지 않은 문제가 될 수 있다.

### 카프카 클러스터 운영 형태 결정하기 
* SaaS형 카프카 클러스터는 명확한 장단점을 갖기에, 카프카 클러스터 운영 노하우 자체가 부족한 경우라면 좋은 선택지가 될 수 있다.
  * 그러나 이 역시도 카프카 클러스터라는 본질은 다름이 없으므로, 카프카에 대한 노하우가 없다면 세세한 운영에 어려움을 겪을 수 있다.
  * 반면, 카프카 클러스터 운영에 대한 이해가 깊은 사용자라면 SaaS형 카프카 클러스터가 제공하는 여러 편의 기능을 안전하게 누릴 수 있게 된다.
* 카프카 클러스터의 운영 형태를 결정하는 경우 반드시 운영 담당자의 이해도와 운영 비용, 추후 발생할 여러 이슈를 모두 고려하여 검토하는 것이 바람직하다.
  * 이렇듯 신중하게 진행하는 검토 과정은 곧 안정적인 서비스 운영의 밑바탕이 될 수 있으므로 매우 중요하다.

## 2023-02-19 Sun
### 카프카 CLI란?
* 카프카를 운영하는 과정에서 카프카 CLI는 필수적인 요소이며, 이를 통해 브로커 운영에 필요한 여러 명령을 전달할 수 있다.
  * 예를 들어, 토픽 또는 파티션의 개수를 조정하는 작업을 처리할 수 있다.
* CLI 명령어들은 기본적으로 길어 부담을 느끼기 쉬우나, 익숙해진 후에는 카프카 클러스터 운영에 지대한 영향을 줄 수 있다.
* CLI 명령어를 통해 토픽을 관리할 경우 필수 옵션과 선택적 옵션을 전달할 수 있으며, 선택적 옵션은 기본적으로 디폴트 값을 사용한다.
  * 이 때, 이러한 선택적 옵션은 브로커 서버 또는 CLI 명령어 자체적으로 설정된 디폴트 설정을 따르므로 디폴트 설정을 미리 이해하는 것 역시 중요하다.

## 2023-02-20 Mon
### 로컬 환경에 카프카 구축하기
* 카프카를 로컬 환경에서 설치하는 경우, 반드시 주키퍼와 브로커 서버의 설치가 선행되어야 한다.
  * 이 때, 기본적으로 0 버전부터 3 버전까지는 모두 호환되나 아직은 3 버전의 운영 사례가 많지 않아 사실상 2 버전이 LTS에 해당한다.
  * 또한, `kafka_2.12-2.5.0.tgz`와 같은 바이너리 이름에서 `2.12`의 경우 Scala 버전을 의미한다.
  * 그러나 Scala는 일반적으로 빌드 버전만 다르며, 기능 상 차이가 없으므로 무엇을 사용해도 무방하다고 볼 수 있다.
* 카프카의 다운로드 사이트에서 바이너리 압축 파일을 다운로드한 경우, 다음과 같은 흐름으로 카프카를 로컬 환경에 구축할 수 있다.
  1. 카프카 바이너리 압축을 해제한다.
  2. 압축을 해제한 후, 주키퍼를 실행한다.
  3. 카프카 바이너리를 실행한다.
* 이 경우, **카프카 바이너리는 곧 로컬 환경에서 구동되는 브로커 서버의 역할을 수행**하게 된다.
* 또한, 주키퍼와 카프카 바이너리는 모두 JVM 환경에서 동작하므로 반드시 1.8 버전 이상의 JDK를 준비해야 한다.

## 2023-02-21 Tue
### 카프카 바이너리 파일 구조
* 카프카 바이너리 압축을 해제할 경우, 다음과 같은 폴더 구조를 확인할 수 있다.
```shell
[kafka_2.12-2.5.0] tree -L 1
.
├── LICENSE
├── NOTICE
├── bin
├── config
├── libs
└── site-docs
```
* 이러한 구조에서, 각각의 폴더는 다음과 같은 파일을 포함한다.
  1. bin: 실행할 바이너리 및 쉘 스크립트들이 포함되며, 브로커 등을 실행하는 경우에도 해당 폴더를 활용한다.
  2. config: 설정에 필요한 `server.properties` 등의 파일을 포함한다.
  3. libs: 브로커 실행을 위해 필요한 라이브러리들을 포함한다.
* 또한, `mkdir data`와 같은 형태의 명령어를 통해 로컬 환경에서 동작하는 카프카 브로커 서버가 데이터를 적재할 공간을 생성해줄 수 있다.

## 2023-02-22 Wed
### server.properties 설정 파일이란?
* `server.properties` 파일은 카프카 바이너리 파일의 압축을 해제했을 경우에 확인할 수 있는 config 디렉토리에 포함되며, 브로커 서버의 동작을 설정한다.
* 이 때, 해당 파일의 여러 설정 중 눈여겨볼만한 것은 다음과 같다.
  1. broker.id: **해당 브로커 서버의 식별자를 의미하며, 같은 클러스터에 배치된 각각의 브로커 서버를 구분하는 역할을 수행**한다.
  2. log.dirs: 카프카 브로커 서버가 사용할 파일 시스템 경로를 명시하며, 프로듀서가 전달한 레코드는 해당 경로에 저장된다.
  3. num.partitions: 카프카 토픽 생성 시 기본적으로 적용되는 파티션 개수이며, 해당 설정에 명시된 수만큼 토픽의 파티션이 생성된다.
  4. log.retention. 및 log.segment. 설정: 카프카 브로커가 파일 시스템에 데이터를 저장할 때, 세그먼트와 관련된 설정을 명시한다.
  5. zookeeper. 설정: 해당 브로커 서버와 연결될 주키퍼와 관련된 설정을 명시한다.
* 이러한 설정을 기반으로, 카프카 클러스터를 실행하기 전에 우선 주키퍼를 실행시킬 필요가 있다.
  * 이 경우, 주키퍼와 관련된 설정을 동일한 config 디렉토리의 `zookeper.properties` 파일에 명시할 수 있다.
  * 그러나 주키퍼 설정은 별도로 수정하지 않은 상태에서 bin 디렉토리의 `zookeeper-server-start.sh`을 실행해도 무방하다.
* 반면, **운영 환경에서 카프카 브로커 서버를 실행하는 경우에는 해당 쉘 스크립트보다는 주키퍼 앙상블을 우선 설정하는 것이 바람직**하다.
  * 즉, 주키퍼 실행을 위한 쉘 스크립트는 어디까지나 로컬 환경 테스트 등의 활용을 위해 제공되는 것으로 이해할 수 있다.
* 상술한 과정을 통해 주키퍼를 실행한 이후, bin 디렉토리의 `kafka-server-start-sh` 쉘 스크립트를 통해 비로소 카프카 브로커 서버를 실행할 수 있다.

## 2023-02-25 Sat
### server.properties 설정하기
* config 폴더에 위치한 해당 파일의 경우, 다음과 같은 구성 요소를 확인해줄 필요가 있다.
  1. broker.id: 브로커 서버의 id이며, 로컬 환경의 경우 0으로 설정되었는지 확인한다.
  2. listeners, advertised.listeners: 카프카 클라이언트가 통신하는 과정에서 접근하고자 하는 브로커 서버의 주소를 인지할 수 있도록 설정한다.
     * 로컬 환경의 경우, 테스트를 위해 `PLAINTEXT://localhost:9092`와 같은 형태로 입력한다.
  3. log.dirs: 레코드 등의 로그 세그먼트가 브로커 서버 상에서 저장될 경로를 명시한다.

### 카프카 정상성 확인하기
* bin 디렉토리의 `kafka-broker-api-version.sh` 스크립트와, 다음과 같은 옵션을 활용하는 것으로 브로커 서버가 정상적으로 실행되었는지 활용할 수 있다.
  * --bootstrap-server: 해당 스크립트를 통해 확인할 카프카 브로커 서버의 주소를 명시하며, 로컬 환경의 경우 `localhost`가 된다.
* 또한, 같은 디렉토리의 `kafka-topics.sh` 명령어와 --list 옵션을 병용하여 임의의 브로커 서버와 관련된 토픽의 목록을 조회할 수 있다.
  * 이 떄, 브로커 서버를 처음 실행한 경우라면 아무런 토픽이 없을 것이므로 빈 결과가 출력된다.
  * 그러나 이렇듯 빈 결과가 출력되는 것 역시도 브로커 서버가 정상적으로 실행된 이후에 가능한 것이므로, 역시 카프카의 정상성을 확인하는 지표가 될 수 있다. 

## 2023-02-26 Sun
### 카프카 브로커 서버 실행하기
* 카프카 서버를 실행하는 경우 우선 주키퍼를 실행해야 하며, 그 이전에 1.8 버전 이상의 Java가 설치되어 있어야 한다.
  * 이 때, 주키퍼의 실행은 상술한 쉘 스크립트를 사용하여 진행한다.
* 모든 준비가 완료된 경우, bin 디렉토리로부터 `./zookeeper-server-start.sh ../config/zookeeper.properties`와 같은 명령어를 실행한다.
  * 이러한 명령어를 실행할 경우, 아래와 같은 로그를 확인하는 것으로 주키퍼가 실행된 것을 알 수 있다.
```shell
[2023-02-26 22:37:51,926] INFO Server environment:java.io.tmpdir=/var/folders/06/kk42q66107zc4v6g8n656d6m0000gn/T/ (org.apache.zookeeper.server.ZooKeeperServer)
[2023-02-26 22:37:51,926] INFO Server environment:java.compiler=<NA> (org.apache.zookeeper.server.ZooKeeperServer)
# ...생략
[2023-02-26 22:37:51,926] INFO Server environment:os.memory.free=498MB (org.apache.zookeeper.server.ZooKeeperServer)
[2023-02-26 22:37:51,926] INFO Server environment:os.memory.max=512MB (org.apache.zookeeper.server.ZooKeeperServer)
[2023-02-26 22:37:51,926] INFO Server environment:os.memory.total=512MB (org.apache.zookeeper.server.ZooKeeperServer)
[2023-02-26 22:37:51,927] INFO minSessionTimeout set to 6000 (org.apache.zookeeper.server.ZooKeeperServer)
[2023-02-26 22:37:51,927] INFO maxSessionTimeout set to 60000 (org.apache.zookeeper.server.ZooKeeperServer)
[2023-02-26 22:37:51,927] INFO Created server with tickTime 3000 minSessionTimeout 6000 maxSessionTimeout 60000 datadir /tmp/zookeeper/version-2 snapdir /tmp/zookeeper/version-2 (org.apache.zookeeper.server.ZooKeeperServer)
[2023-02-26 22:37:51,933] INFO Using org.apache.zookeeper.server.NIOServerCnxnFactory as server connection factory (org.apache.zookeeper.server.ServerCnxnFactory)
[2023-02-26 22:37:51,935] INFO Configuring NIO connection handler with 10s sessionless connection timeout, 2 selector thread(s), 16 worker threads, and 64 kB direct buffers. (org.apache.zookeeper.server.NIOServerCnxnFactory)
[2023-02-26 22:37:51,937] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)
[2023-02-26 22:37:51,943] INFO zookeeper.snapshotSizeFactor = 0.33 (org.apache.zookeeper.server.ZKDatabase)
[2023-02-26 22:37:51,944] INFO Snapshotting: 0x0 to /tmp/zookeeper/version-2/snapshot.0 (org.apache.zookeeper.server.persistence.FileTxnSnapLog)
[2023-02-26 22:37:51,946] INFO Snapshotting: 0x0 to /tmp/zookeeper/version-2/snapshot.0 (org.apache.zookeeper.server.persistence.FileTxnSnapLog)
[2023-02-26 22:37:51,956] INFO Using checkIntervalMs=60000 maxPerMinute=10000 (org.apache.zookeeper.server.ContainerManager)
```
* 이후 새로운 터미널을 열어 bin 디렉토리로부터 `./kafka-server-start.sh ../config/server.properties` 명령어를 통해  브로커를 실행한다.
  * 이 경우, 아래와 같은 로그를 통해 카프카 브로커가 정상 실행되었음을 알 수 있다.
```shell
[2023-02-26 22:41:45,992] INFO Kafka version: 2.5.0 (org.apache.kafka.common.utils.AppInfoParser)
[2023-02-26 22:41:45,992] INFO Kafka commitId: 66563e712b0b9f84 (org.apache.kafka.common.utils.AppInfoParser)
[2023-02-26 22:41:45,992] INFO Kafka startTimeMs: 1677418905990 (org.apache.kafka.common.utils.AppInfoParser)
[2023-02-26 22:41:45,993] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```
* 이 때, 상술한 바와 같이 `kafka-broker-api-version.sh`과 `kafka-topics.sh`을 통해 브로커의 정상 실행을 확인할 수 있다.

### 카프카 브로커 서버 실행하기 - 중간 정리
* 상술한 방식을 통해, 로컬 환경에서는 주키퍼와 카프카 브로커 서버라는 두 프로세스가 동작하게 된다.
  * 이 경우, 브로커 서버의 주소는 `localhost:9092`가 된다.
  * 또한, 브로커 서버는 물론 주키퍼 역시도 JVM 상에서 동작하므로 1.8 버전 이상의 JDK 설치는 필수적이다.
* 이 때, 개발 환경이 아닌 로컬 환경에서 카프카 브로커를 실행하는 것은 프로듀서 및 컨슈머 애플리케이션 개발 또는 테스트를 위해 필요한 경우가 많다.
* 나아가 로컬 환경에서 카프카를 동작시키는 경우, 편의성을 위해 `127.0.0.1`을 my-kafka 등의 host로 설정해둘 수 있다.
  * macOS의 경우, 해당 설정은 `sudo vi /etc/hosts`를 활용하여 적용할 수 있다.

## 2023-02-27 Mon
### kafka-topics.sh 활용하기
* `kafka-topics.sh` 스크립트를 활용하여 토픽을 생성할 수 있으며, 파티션이나 복제 개수 등의 설정을 적용할 수 있다.
  * 이 때, 이러한 설정 값들은 기본적으로 브로커에 설정된 기본 값이 적용되는 반면 카프카 클러스터 정보와 토픽의 이름은 반드시 사전에 결정된 상태여야 한다.
  * 해당 **쉘 스크립트는 카프카의 토픽을 관리할 수 있으므로, 카프카 애플리케이션을 개발하고 운영하는는 과정에서 자주 사용**되곤 한다.
* 토픽에 대한 CRUD는 옵션으로 결정되며, 생성을 위한 `--create`에 더해 상세 조회를 위한 `--describe`와 목록 조회를 위한 `--list`가 존재한다.
* 반면, 이미 생성된 토픽에 대해 파티션의 개수를 늘리는 등 수정 작업을 적용하기 위해서는 `--alter` 옵션을 사용한다.
  * 이렇듯 **파티션 개수와 컨슈머의 수를 늘리는 것은 컨슈머의 데이터 처리량을 선형으로 늘리는 가장 쉬운 방법이므로 해당 옵션 역시 자주 사용**될 수 있다.
  * 그러나 분산 시스템에서 분산된 데이터를 줄이는 방법은 너무 복잡하기에 카프카에서는 늘린 파티션의 개수는 다시 줄일 수 없으며, 삭제 후 재생성만이 지원된다.
  * 때문에 **`--alter` 옵션을 토대로 더 적은 파티션 개수를 명시할 경우, `InvalidPartitionsException` 에외가 발생**하게 된다.
  * 이로 인해 파티션의 개수를 늘리거나 새 토픽을 생성하는 경우, 파티션의 개수를 다시 줄이는 상황이 발생하지는 않을지 충분한 논의를 거치는 것이 바람직하다.

## 2023-02-28 Tue
### 카프카 토픽 생성하기
* 토픽 생성의 경우, `kafka-topics.sh`과 `--create` 옵션을 아래와 같이 활용할 수 있다.
  * 또한, 이렇게 생성된 토픽은 `--list` 옵션을 통해서 확인할 수 있다.
```shell
[bin] ./kafka-topics.sh --bootstrap-server my-kafka:9092 --create --topic hello.kafka
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic hello.kafka.
[bin] ./kafka-topics.sh --bootstrap-server my-kafka:9092 --list
hello.kafka
[bin]
```
* 또는 `--describe` 옵션을 명시하여 다음과 같이 생성한 토픽의 세부 정보를 확인할 수도 있다.
```shell
[bin] ./kafka-topics.sh --bootstrap-server my-kafka:9092 --describe --topic hello.kafka
Topic: hello.kafka	PartitionCount: 1	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: hello.kafka	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
```

### 설정과 함께 토픽 생성하기
* 반면, 파티션 개수 또는 복제 개수를 옵션 형태로 명시하여 다음과 같이 토픽을 생성할 수도 있다.
  * 이 경우, `--describe` 옵션을 통해 자신이 설정한 토픽의 정보를 확인할 수 있게 된다.
```shell
[bin] ./kafka-topics.sh --bootstrap-server my-kafka:9092 --create --partitions 10 --replication-factor 1 --config retention.ms=172800000 --topic advanced.hello.kafka
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic advanced.hello.kafka.
[bin] ./kafka-topics.sh --bootstrap-server my-kafka:9092 --describe --topic advanced.hello.kafka
Topic: advanced.hello.kafka	PartitionCount: 10	ReplicationFactor: 1	Configs: segment.bytes=1073741824,retention.ms=172800000
	Topic: advanced.hello.kafka	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: advanced.hello.kafka	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: advanced.hello.kafka	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
	Topic: advanced.hello.kafka	Partition: 3	Leader: 0	Replicas: 0	Isr: 0
	Topic: advanced.hello.kafka	Partition: 4	Leader: 0	Replicas: 0	Isr: 0
	Topic: advanced.hello.kafka	Partition: 5	Leader: 0	Replicas: 0	Isr: 0
	Topic: advanced.hello.kafka	Partition: 6	Leader: 0	Replicas: 0	Isr: 0
	Topic: advanced.hello.kafka	Partition: 7	Leader: 0	Replicas: 0	Isr: 0
	Topic: advanced.hello.kafka	Partition: 8	Leader: 0	Replicas: 0	Isr: 0
	Topic: advanced.hello.kafka	Partition: 9	Leader: 0	Replicas: 0	Isr: 0
[bin]
```
* 이 때, 상술한 바와 같이 **`--partitions` 옵션 등을 명시하지 않는 경우 server.properties에 설정된 기본 값이 적용된 토픽이 생성**된다.

### 카프카 토픽 수정하기
* 상술했듯, 아무런 옵션을 병기하지 않고 생성한 카프카 토픽의 설정은 server.properties의 설정을 적용하며, 이는 `--alter` 옵션을 통해 수정할 수 있다.
```shell
[bin] ./kafka-topics.sh --bootstrap-server my-kafka:9092 --create --topic update.candidate
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic update.candidate.
[bin] ./kafka-topics.sh --bootstrap-server my-kafka:9092 --describe --topic update.candidate
Topic: update.candidate	PartitionCount: 1	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: update.candidate	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
[bin] # 상술한 바와 같이, 아무런 옵션을 병기하지 않고 생성한 토픽의 파티션 개수는 기본 설정값인 1이 적용된다.
[bin] ./kafka-topics.sh --bootstrap-server my-kafka:9092 --alter --partitions 10 --topic update.candidate
[bin] # --alter를 활용한 토픽의 수정에는 별다른 응답이 출력되지 않는다.
[bin] ./kafka-topics.sh --bootstrap-server my-kafka:9092 --describe --topic update.candidate
Topic: update.candidate	PartitionCount: 10	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: update.candidate	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: update.candidate	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: update.candidate	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
	Topic: update.candidate	Partition: 3	Leader: 0	Replicas: 0	Isr: 0
	Topic: update.candidate	Partition: 4	Leader: 0	Replicas: 0	Isr: 0
	Topic: update.candidate	Partition: 5	Leader: 0	Replicas: 0	Isr: 0
	Topic: update.candidate	Partition: 6	Leader: 0	Replicas: 0	Isr: 0
	Topic: update.candidate	Partition: 7	Leader: 0	Replicas: 0	Isr: 0
	Topic: update.candidate	Partition: 8	Leader: 0	Replicas: 0	Isr: 0
	Topic: update.candidate	Partition: 9	Leader: 0	Replicas: 0	Isr: 0
[bin] # 반면 --alter 옵션을 통해 파티션의 개수를 수정한 경우, 상술한 바와 같이 파티션의 개수가 늘어나게 된다.
```
* 이 때, 이러한 **파티션의 증설은 컨슈머를 함께 증설시켜 처리량을 선형으로 늘리는 장점이 있지만 다시 파티션을 줄일 수는 없다는 점을 반드시 고려**해야 한다.
  * 만일 `--alter` 옵션을 통해 파티션의 개수를 줄이는 경우, 다음과 같은 `InvalidPartitionsException` 예외가 발생하는 것을 확인할 수 있다.
```shell
[bin] ./kafka-topics.sh --bootstrap-server my-kafka:9092 --alter --partitions 1 --topic update.candidate
Error while executing topic command : org.apache.kafka.common.errors.InvalidPartitionsException: Topic currently has 10 partitions, which is higher than the requested 1.
[2023-02-28 22:52:13,436] ERROR java.util.concurrent.ExecutionException: org.apache.kafka.common.errors.InvalidPartitionsException: Topic currently has 10 partitions, which is higher than the requested 1.
	at org.apache.kafka.common.internals.KafkaFutureImpl.wrapAndThrow(KafkaFutureImpl.java:45)
	at org.apache.kafka.common.internals.KafkaFutureImpl.access$000(KafkaFutureImpl.java:32)
	at org.apache.kafka.common.internals.KafkaFutureImpl$SingleWaiter.await(KafkaFutureImpl.java:89)
	at org.apache.kafka.common.internals.KafkaFutureImpl.get(KafkaFutureImpl.java:260)
	at kafka.admin.TopicCommand$AdminClientTopicService.alterTopic(TopicCommand.scala:270)
	at kafka.admin.TopicCommand$.main(TopicCommand.scala:64)
	at kafka.admin.TopicCommand.main(TopicCommand.scala)
Caused by: org.apache.kafka.common.errors.InvalidPartitionsException: Topic currently has 10 partitions, which is higher than the requested 1.
 (kafka.admin.TopicCommand$)
[bin]
```

## 2023-03-01 Tue
### kafka-configs.sh이란?
* 해당 쉘 스크립트는 토픽의 옵션 일부를 설정하기 위해 사용할 수 있으며, `--alter` 또는 `--add-config` 옵션을 병기할 수 있다.
  * 예를 들어, 이러한 명령어를 통해 `min.insync.replicas` 옵션을 토픽에 설정할 수 있다.
  * 이 때, 해당 옵션은 프로듀서가 데이터를 보내거나 컨슈머가 이를 읽어들일 때 워터마크 용도 등 안전한 데이터 전송을 보장하기 위해 사용될 수 있다.
* 또한, 이렇게 수정된 토픽의 설정은 kafka-topics.sh 쉘 스크립트와 `--describe` 옵션을 활용하여 확인할 수 있다.
* 나아가 `--broker [브로커ID] --all --describe`와 같은 옵션을 병기하는 것으로 브로커에 설정된 여러 옵션의 기본 값을 확인할 수 있다. 