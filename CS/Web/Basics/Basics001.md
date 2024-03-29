# Basics
## 2022-06-05 Sun

### IP
* 인터넷 상의 클라이언트와 서버는 일반적으로 서로 물리적으로, 먼 위치에 분리되어 있다.
* IP 주소는 이렇듯 먼 원격지 간의 통신을 위해 사용된다.
* **IP 주소는 지정된 IP 주소에 데이터를 전달하는데에 사용되며, 이 과정에서 데이터 단위를 패킷으로 사용**한다.
  * IP 패킷에는 출발지 IP와 목적지 IP, 그리고 기타 여러 정보가 포함된다.
  * 인터넷 망에 배치된 모든 서버(또는 노드)는 IP 규약을 따르므로, 패킷의 정보를 이해할 수 있다.
* 예를 들어, 인터넷 상의 클라이언트가 서버와 통신하는 흐름은 크게 다음과 같다.
  1. 출발지 IP A와 서버의 목적지 IP B, 그리고 전달할 데이터를 포함한 패킷을 인터넷 망에 송신한다.
  2. 목적지까지의 경로에서 거치는 서버들은 모두 IP를 이해할 수 있으므로, 목적지 IP를 기반으로 계속해서 패킷을 전달해나간다.
  3. 서버가 패킷을 성공적으로 전달받았다면, 패킷에 대한 ACK를 같은 방식으로 클라이언트에게 전달한다.
* **같은 클라이언트와 서버 간의 통신이더라도 복잡한 인터넷 망을 통하므로, 패킷이 경유하는 경로는 달라질 수 있다**.
  * 이 때, 패킷은 수화물을 의미하는 package과 덩어리를 의미하는 bucket의 합성어이다.

### IP의 한계
* IP는 본질적으로 연결성과 신뢰성, 프로그램 구분 관점에서의 한계가 존재한다.
* 연결성 측면에서, **IP는 상대 서버가 실재하는지를 고려하지 않고 우선 패킷을 전달**한다.
  * 예를 들어, 서버가 꺼져 통신이 불가능한 상태일 수도 있으나 IP는 이를 고려하지 않는다.
  * 때문에 **패킷을 전달 받을 대상이 없거나 서비스 불능 상태더라도 패킷 송신자는 이를 알 수가 없다**.
* 신뢰성 측면에서, **패킷이 전달되는 과정에서 유실될 가능성**이 있다.
  * 또한, **여러 패킷을 전달하는 경우 네트워크 이슈로 인해 도착된 패킷의 순서가 바뀔 수도 있다**.
* 프로그램 구분 측면에서, **서버가 제공하는 서비스가 여러 종류인 경우 패킷만으로는 최종 도착지를 구분할 방법이 없다**.

### TCP / IP 4계층
* TCP / IP 4계층은 다음과 같이 구성된다.
  1. 네트워크 인터페이스 계층 - 물리적 전송 신호를 사용한다.
  2. 인터넷 계층 - IP를 사용한다.
  3. 전송 계층 - TCP와 UDP를 사용한다.
  4. 애플리케이션 계층 - HTTP, FTP 등 애플리케이션에서 사용하는 프로토콜을 사용한다.
* 이 중, **전송 계층은 인터넷 계층에서 사용하는 IP의 한계를 보완하기 위해 사용되는 계층**이다.

### 사용자 관점에서의 통신 시작
* 반면, 사용자 관점에서 PC는 크게 다음과 같은 계층으로 분류된다.
  1. 네트워크 인터페이스 계층 - LAN을 위한 물리적인 장비들이 해당된다.
  2. 운영체제 계층 - 해당 계층에서는 TCP나 UDP, IP 등을 지원한다.
  3. 애플리케이션 계층 - 웹 브라우저 등 사용자가 직접 사용하는 애플리케이션이 해당된다.
* 이를 전제로, 사용자가 서버에 `Hello, world!` 메시지를 전달하는 과정은 다음과 같이 시작된다.
  1. 애플리케이션 계층의 애플리케이션이 메시지를 생성한 후, 이를 Socket 라이브러리를 활용하여 전달한다.
  2. 운영체제 계층에서 TCP 정보를 생성하고, 메시지 데이터를 포함시킨다.
  3. 운영체제 계층에서 TCP 정보를 기반으로 IP 패킷을 생성하고, TCP 정보를 포함시킨다.
  4. 네트워크 인터페이스 계층에서 IP 패킷을 전달받고, LAN 카드를 통해 인터넷으로 전달한다.
* 이 때, **1.의 과정에서 생성되는 메시지가 HTTP 메시지라면 해당 메시지는 HTTP 계층에서만 사용될 정보들을 포함**한다.
  * 그 후 **운영체제 계층, 네트워크 인터페이스 계층을 거쳐가며 각 계층에서만 사용될 정보들을 하나씩 덧붙이는 캡슐화 과정을 거치게 된다**.

### TCP / IP 패킷
* TCP는 IP의 본질적인 한계를 기능적으로 보완하기 위한 정보를 포함하며, 계층적으로는 IP의 상위 계층이다.
* **TCP는 상위 계층으로부터 전달 받은 데이터에 더해 출발지 포트와 목적지 포트, 전송 제어, 순서, 검증 정보 등을 포함**한다.
  * 이렇게 사용자 데이터에 더해 여러 정보를 포함하는 TCP의 전송 단위는 세그먼트라고 지칭한다.
  * **IP는 TCP 세그먼트를 전달 받아, 자신이 사용할 정보인 출발지 / 목적지 IP 정보를 추가하여 캡슐화**한다.

### TCP의 특징
* TCP는 연결성을 지향하고, 데이터가 순서대로 전달되는 것을 보장하는 신뢰성 있는 프로토콜이다.
  * 현재의 애플리케이션은 대부분 TCP를 사용하도록 동작한다.
* 연결성 측면에서, **TCP는 가상의 연결 상태인 `3 way handshake`를 활용**한다.
  * 이는 데이터를 본격적으로 전송하기 전에 우선 가상의 연결을 시작한다는 의미를 갖는다.
  * IP의 한계점에서 다루었던 상황 중 **상대 서버가 서비스 불능 상태에 빠진 경우, `3 way handshake`가 불가능하므로 데이터 전송은 시작되지 않는다**.
* 데이터 전달 측면에서, **TCP는 유실된 패킷이 존재하는 것을 알 수 있는 방법을 제공**한다.
* 순서 보장 측면에서, **TCP는 패킷의 순서를 알 수 있는 방법을 제공**한다.

### 3 way handshake
* `3 way handshake`는 다음과 같은 방식으로 연결을 시작한다.
  1. 클라이언트는 서버로 `SYN` 메시지를 전송한다.
     * 클라이언트는 이를 통해 서버에게 통신 시작 의사를 밝힌다.
  2. 서버는 이를 받아 `SYN + ACK` 메시지를 클라이언트에게 전송한다.
     * 서버는 이를 받아 클라이언트의 통신 시작 의사를 수락(`ACK`)하고, 자신 역시 클라이언트와 통신을 시작하겠다는 의사(`SYN`)를 밝힌다.
  3. 클라이언트는 이를 받아 서버에세 `ACK`를 전송한다.
     * 클라이언트는 서버의 통신 시작 의사를 수락(`ACK`)하는 것으로 연결이 시작된다.
* 이 때, TCP는 충분히 최적화된 프로토콜이므로 3.의 과정에서 `ACK`와 함께 데이터를 전송할 수 있다.
* 예를 들어 **서버가 서비스 불가능한 경우, 클라이언트는 `SYN`을 전송하더라도 `SYN + ACK`를 수신할 수 없으므로 응답이 없음을 미리 알 수 있게 된다**.
* **`3 way handshake`는 클라이언트와 서버 간의 논리적인 연결을 의미하며, 실제로는 중도에 경유하는 수 많은 인터넷 상의 노드의 존재를 알 수는 없다**.
  * 즉, `3 way handshake`가 성공했다고 하더라도 클라이언트와 서버 전용의 물리적인 회선이 생성된 것은 아니다.

### 데이터 전달과 순서의 보장
* **TCP를 활용하는 클라이언트가 데이터를 서버에 전송한 경우, 서버는 해당 데이터를 정상적으로 수신했다는 사실을 알려준다**.
  * 이를 통해 클라이언트는 데이터가 잘 전송되었는지를 알 수 있게 된다.
* **TCP를 활용하는 클라이언트가 전송하는 데이터의 크기가 너무 커 패킷이 셋으로 나뉜 경우, 서버는 전송받아야 할 패킷의 순서를 알 수 있다**.
  * 이는 TCP 세그먼트에 순서 정보가 포함되기 때문이다.
  * **클라이언트가 패킷 1, 2, 3을 전송했지만 서버가 1, 3, 2의 순서로 전달 받은 경우, 서버는 3 이후의 모든 패킷을 버리고 2부터 재전송을 요청**한다.
    * 이는 기본적인 방식이며, 서버가 내부적으로 최적화 로직을 사용하는 경우에는 순서가 잘못 된 시점부터의 모든 패킷을 버리지 않을 수도 있다.

### 신뢰할 수 있는 프로토콜로서의 TCP
* 상술한 **연결성과 데이터 전달, 순서의 보장은 마법처럼 이루어지는 것이 아닌 TCP 세그먼트에 포함된 TCP 관련 정보를 토대로 이루어지게 된다**.
* 이로 인해 TCP는 신뢰할 수 있는 프로토콜로 지칭된다.

### UDP
* UDP는 TCP와 같은 계층에 위치한 프로토콜이다.
* **UDP는 TCP가 연결성과 데이터 전달, 데이터 순서를 보장하기 위해 제공하는 어떠한 기능도 제공하지 않는다**.
  * UDP는 IP와 거의 유사하지만, 추가적으로 포트 정보와 checksum과 같은 데이터를 포함한다.
  * 포트는 동일한 PC에서 실행되는 각 애플리케이션을 구분하기 위해 사용하는 주소이며, checksum은 각 메시지의 위변조를 검증하기 위한 정보이다. 
* 이러한 특징으로 인해 UDP는 신뢰성이 없는 프로토콜이다.
* 반면, **아무런 기능을 제공하지 않는 UDP는 TCP에 비해 단순하고 빠른 전달이 가능**하다.
  * TCP가 제공하는 신뢰성은 그 자체가 추가적인 시간을 사용하는 작업이다.
  * UDP는 이러한 기능을 제공하지 않으므로, 자연스레 TCP보다 더 빠를 수 밖에 없다.

### TCP의 한계와 UDP의 장점
* TCP는 신뢰성을 보장하기 위해 많은 기능을 제공하므로, 최적화에는 한계가 있다.
* 또한 **방대한 인터넷이 이미 TCP를 기반으로 동작하고 있으므로, 섣불리 손대기가 어려운 프로토콜**이기도 하다.
* 때문에 **개발자가 TCP를 최적화하는 경우, TCP보다는 사실상 아무런 기능을 제공하지 않는 UDP에 기능을 덧붙이는 방식으로 접근하는 것이 바람직**하다.
* 이러한 특징으로 인해 최근에는 UDP가 각광받고 있으며, 실제로 HTTP3 스펙의 경우 `3 way handshake` 등의 과정을 최적화하기 위해 UDP를 사용하고 있다.

### 포트란?
```
> 포트는 동일한 IP를 갖는 서버에서 동작하는 각각의 프로세스를 구분하기 위해 사용되는 주소 체계이다.
```
* 클라이언트와 서버 간의 통신에서, 서버가 여러 애플리케이션을 실행 중인 경우 클라이언트가 무엇을 대상으로 하는지 결정할 수 있는 방법이 있어야 한다.
* 포트는 이러한 경우를 위해 사용되는 또 다른 주소 체계이며, TCP 계층에서 사용된다.
  * **IP는 목적지 서버를 찾는 용도이며, 서버 내부에서 동작하는 개별 애플리케이션을 구분하는 것이 포트**이다.
* 이를 통해 **클라이언트와 서버는 TCP / IP 패킷에 포함된 포트 정보를 토대로 서로의 종단 애플리케이션을 인지**할 수 있게 된다.
* 포트 주소는 0 ~ 65535 범위를 갖는다.
  * 이 때, **0 ~ 1023 범위는 well-known 포트이므로 사용을 지양하는 것이 바람직**하다.

### DNS란?
```
> IP 주소는 기억하기가 어렵다.
```
* **IP를 활용하면 클라이언트와 서버의 위치를 파악할 수 있지만, 이는 사용자 친화적이지 못한 주소 체계**이다.
  * 또한, IP 주소는 고정적이지 않고 변경될 가능성도 있는 주소이다.
* **DNS는 이러한 문제를 해결하기 위한 기술이며, 도메인 주소를 IP 주소로 변환하는 기능을 수행**한다.
* DNS 서버에는 도메인 명과 그에 대응하는 IP 주소를 등록할 수 있으며, 사용자는 도메인을 토대로 통신하기 위해 다음의 순서를 따르게 된다.
  1. 클라이언트는 통신 대상 도메인에 대한 실제 IP 주소를 DNS 서버에 질의한다.
  2. DNS 서버는 대응되는 IP 주소를 응답한다.
  3. 클라이언트는 전달 받은 IP 주소를 토대로 통신을 시작한다.
* 도메인 명을 사용하는 특정 서버의 IP 주소가 변경된 경우, 서버 소유자는 DNS 서버에 수정을 요청하게 된다.
* 이렇듯 **DNS를 활용하면 기억하기 어렵고 변경에 대응하기 힘든 IP 주소의 본질적인 한계를 극복**할 수 있게 된다.