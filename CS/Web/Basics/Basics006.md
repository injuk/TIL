# Basics
## 2022-06-07 Tue

### HTTP 상태 코드란?
* **HTTP 상태 코드는 클라이언트가 보낸 요청의 처리 상태를 HTTP 응답 메시지에서 알려주는 기능**을 수행한다.
  1. 1xx: Informational, 요청이 수신되어 처리 중인 경우.
  2. 2xx: Successful, 요청이 정상 처리된 경우.
  3. 3xx: Redirection, 요청을 완료하기 위해 추가적인 행동이 필요한 경우.
  4. 4xx: Client Error, 클라이언트의 오류 또는 잘못된 문법 등으로 인해 서버가 요청을 처리할 수 없는 경우.
  5. 5xx: Server Error, 서버의 오류 등으로 인해 서버가 요청을 정상적으로 처리할 수 없는 경우.
* 이 때, 100번대 상태 코드는 거의 사용되지 않는다.

### 추가될 상태 코드에 대비하기
* 현재에는 존재하지 않는 상태 코드가 추후 서버에 추가되고, 이를 클라이언트에서 처리해야할 필요가 생길 수 있다.
* 클라이언트의 입장에서 이해할 수 없는 코드를 서버가 반환하는 경우, 백의 자리에서 묶어 처리해도 무방하다.
  * 예를 들어 **서버로부터 상태 코드 599가 반환되었으나 클라이언트가 이를 인식할 수 없는 경우, 5xx Server Error로 처리**한다.
* 이러한 방식을 적용한 경우, **미래에 새로운 상태 코드가 추가되더라도 상위 레벨 상태 코드로 처리하므로 추가 작업이 필요 없어진다**.

### 2xx - Successful
* 2xx 상태 코드는 클라이언트의 요청이 성공적으로 처리되었음을 의미하며, 크게 다음과 같이 분류된다.
  1. 200 OK
  2. 201 Created
  3. 202 Accepted
  4. 204 No Content

### 200 OK
* 해당 상태 코드는 요청이 성공했음을 의미한다.

### 201 Created
* 클라이언트의 요청이 성공하여 새로운 리소스가 생성되었음을 의미한다.
  * 이 때, **생성된 리소스는 HTTP 응답의 `Location` 헤더로 식별**한다.
* **클라이언트는 상태 코드가 201로 반환되는 경우, `Location` 헤더가 있을 수도 있다는 사실을 짐작**할 수 있다.

### 202 Accepted
* 클라이언트의 요청이 접수되었으나, 아직 처리가 완료되지 않은 경우에 사용되는 상태 코드이다.
  * 예를 들어 1시간 이후에 시작되는 Batch 프로세스가 요청을 처리하는 경우에 반환할 수 있다.

### 204 No Content
* 해당 상태 코드는 **서버가 요청을 정상적으로 처리하였으나, HTTP 응답 메시지 본문에 첨부할 데이터가 없는 경우에 사용**할 수 있다.
  * 이는 웹 문서 편집기에서 저장 버튼을 클릭한 경우를 예로 들 수 있다.
  * 상술한 시나리오의 경우, 저장 버튼을 클릭한 후 아무런 내용을 반환받을 필요가 없으므로 204 상태 코드를 반환하는 것이 바람직하다.

### 2xx 결론
* 일반적으로 200 OK만 사용하는 경우가 많고, 200과 201을 혼용하는 경우도 있다.
* 이는 **개발 팀에서 개발을 시작하기 전에 합의하고, 합의된 내용에 따라 개발을 진행하는 것이 바람직**하다.

### 3xx - Redirection
* 3xx 상태 코드는 요청을 완료하기 위해 유저 에이전트의 추가적인 조치가 필요한 상황을 의미하며, 크게 다음과 같이 분류된다.
  1. 300 Multiple Choices
  2. 302 Moved Permanently
  3. 302 Found
  4. 303 See Other
  5. 304 Not Modified
  6. 307 Temporary Redirect
  7. 308 Permanent Redirect
* 이 때, **유저 에이전트란 일반적으로 클라이언트 역할을 수행하는 웹 브라우저를 지칭**한다.

### 리다이렉션이란?
* **웹 브라우저는 3xx 상태 코드로 반환된 HTTP 응답의 결과에 `Location` 헤더가 존재하는 경우, `Location` 위치로 자동적으로 이동**한다.
  * 이렇듯 웹 브라우저에 의한 자동 이동을 리다이렉트라고 표현한다.
* 예를 들어 `/event` URI에서 기존 이벤트 페이지를 운영하고 `/new-event` URI에서 신규 이벤트 페이지를 운영한다고 하자.
* 상술한 시나리오에서 기존 이벤트 페이지에 접속한 클라이언트를 신규 이벤트 페이지로 리다이렉트시키는 흐름은 다음과 같다.
  1. 클라이언트는 `GET /event HTTP/1.1`로 요청한다.
  2. 서버는 `HTTP/1.1 301 Moved Permanently`와 `Location: /new-event`로 응답한다.
  3. 클라이언트 측 브라우저는 이를 받아 자동으로 `GET /new-event HTTP/1.1`을 요청한다.
     * **이 과정이 리다이렉션에 해당하며, 웹 브라우저에 의해 자동으로 수행**된다.
  4. 서버는 `HTTP/1.1 200 OK`로 응답하며, 메시지 본문에 신규 이벤트 페이지의 HTML 문서를 첨부하여 반환한다.

### 리다이렉션의 종류
* 리다이렉션은 다시 크게 다음과 같이 분류해볼 수 있다.
  1. 영구 리다이렉션: 특정한 리소스의 URI가 영구적으로 이동한 경우에 해당한다.
  2. 일시 리다이렉션: 일시적인 변경에 해당한다.
  3. 특수 리다이렉션: 예를 들어, 결과 대신 캐시를 사용하는 경우에 해당한다.

### 301 Move Permanently / 308 Permanent Redirect
* 두 상태 코드 모두 리소스의 URI가 영구적으로 이동한 경우에 사용할 수 있다.
  * 즉, **두 코드 모두 기존 URI가 새로운 URI로 영구적으로 변경된 사실을 알리는 역할을 수행**한다.
  * 이 경우 기존 URI를 사용하지 않으며, 이러한 변경 사항은 웹 브라우저에서도 인식할 수 있다.
* **301 상태 코드의 경우, 리다이렉트시 HTTP 요청이 GET 메소드로 변경되며 메시지 본문이 제거될 수도 있다**.
* **308 상태 코드의 경우, 301과 기능은 같으나 리다이렉트시 요청 메소드와 메시지 본문이 유지**된다.
* **301과 308을 비교하는 경우, 영구적으로 페이지가 이동된 경우에는 내부적으로 사용하는 데이터의 형식도 변경될 것이므로 301을 사용하는 것이 바람직**하다.

### 301 Move Permanently의 배경
* 해당 상태 코드의 초기 목적은 전달 받은 메소드를 그대로 유지하여 리다이렉트하는 것이었다.
* 그러나 실제로 브라우저들은 메소드를 유지하지 않고 GET 으로 변환하는 방식으로 구현되었고, 결국 스펙 역시 이에 따라 바뀌게 된 케이스에 속한다.
* 때문에 **상단에서는 요청 메소드가 GET으로 변경되거나 메시지 본문이 버려질 '수도' 있다고 작성되었으나, 실제로 대부분의 브라우저는 이렇게 동작**한다.