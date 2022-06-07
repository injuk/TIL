# Basics
## 2022-06-06 Mon

### HTTP API 설계하기
* **HTTP API의 호출 대상이 되는 URI를 설계하는 경우, 가장 중요한 것은 리소스가 식별 가능해야 한다는 점**이다.
  * 이 때, 리소스란 해당 API가 다루는 자원이 된다.
  * 예를 들어 **회원 목록 조회 API의 경우, '회원 목록을 조회한다'는 행위가 아닌 '회원' 자체가 리소스가 되어야 한다**.
  * **리소스를 적절히 식별하기 위해서는 회원과 관련된 CRUD를 우선은 모두 배제하고, '회원'이라는 리소스를 식별하는데에 집중**해야 한다.
  * 이렇게 식별된 리소스를 URI에 매핑한다.

### 회원 리소스 관리 API의 예시
* 회원에 대한 CRUD API의 경우, 다음과 같은 설계를 적용해볼 수 있다.
  1. 목록 조회: `/members`
  2. 상세 조회: `/members/{id}`
  3. 생성: `/members/{id}`
  4. 수정: `/members/{id}`
  5. 삭제: `/members/{id}`
* 이렇듯 **URI 상의 리소스는 계층 구조상 컬렉션으로 보고 복수형을 사용하는 것이 권장**된다.
* 그러나 **상술한 설계의 경우, 목록 조회를 제외한 모든 API가 구분되지 않는다는 한계가 존재**한다.

### 리소스와 행위의 분리
* 상술한 문제점은 URI가 리소스만을 식별하도록 설계된 것에서 기인한다.
* **URI가 리소스만을 식별하는 것은 바람직하며, 리소스와 해당 리소스에 대한 행위는 분리되어야 한다**.
  * 상술한 회원 시나리오에서, 리소스는 회원이자 명사이며 행위는 조회, 생성, 수정, 삭제 등의 동사가 된다.
* 이렇게 **리소스와 분리된 행위라는 동사는 HTTP 메소드로 표현**한다.
  * 때문에 **URI를 통해서는 리소스만을 식별하면 되며, CRUD 동작은 HTTP 메소드가 대신**할 수 있다.

## 2022-06-07 Tue
### HTTP 메소드
* **HTTP 메소드란, 클라이언트가 서버에 무언가를 요청할 때 기대하는 행동**이다.
  1. GET: 리소스에 대한 조회를 의미한다.
  2. POST: 리소스에 대한 요청 처리를 의미하며, 주로 생성 또는 등록에 사용한다.
     * POST는 클라이언트가 데이터를 담아서 서버에게 무언가를 요청해야 한다.
  3. PUT: 리소스를 대체하고, 없으면 생성한다.
     * 파일을 폴더에 이동시키는 것과 유사하다.
  4. PATCH: 리소스의 부분을 변경한다.
     * 회원의 이름과 같이, 리소스의 특정한 필드를 변경하는 경우에 해당한다.
  5. DELETE: 리소스를 삭제한다.
* 또한, HTTP 메소드에는 다음과 같은 기타 메소드도 존재한다.
  1. HEAD: GET과 동일하지만, **메시지 본문 부분을 제외하고 상태 줄과 헤더만을 반환**한다.
  2. OPTIONS: 대상 리소스에 대한 통신 가능 옵션 메소드들을 설명하기 위해 사용된다.
     * 주로 CORS에서 사용된다.
  3. CONNECT: 대상 자원으로 식별되는 서버에 대한 터널을 설정한다.
  4. TRACE: 대상 리소스에 대한 경로를 따라 메시지 루프백 테스트를 수행한다.
* 이 중 CONNECT와 TRACE는 거의 사용되지 않는다.

### GET 메소드
* **GET은 리소스의 조회에 사용되며, 서버에 데이터를 전달하고 싶은 경우 query를 통해 전달**한다.
* **GET은 query 대신 메시지 본문을 통해 데이터를 전달할 수도 있지만, 지원하지 않는 곳이 많아 권장되지 않는 방식**이다. 
  * GET 메소드에 메시지 본문을 추가하는 것은 최근 스펙에서 허용된 개념이다.
  * **실무에서는 잘 사용되지 않으며, 필요한 경우에는 query를 사용하는 방식이 권장**된다.

### POST 메소드
* **POST는 요청에 대한 데이터 처리를 위해 사용되는 메소드이며, 메시지 본문을 통해 서버로 요청 데이터를 전달**한다.
* **서버는 HTTP 요청의 메시지 본문을 통해 전달된 데이터를 처리하는 모든 기능을 수행**한다.
  * 일반적으로 전달되는 데이터는 신규 리소스를 등록하거나, 프로세스를 처리하기 위해 사용한다.

### POST 메소드 동작 예시
* 예를 들어, /members URI에 POST 메소드와 메시지 본문을 전달할 경우 서버는 회원 리소스를 생성한다고 사전에 약속되었다고 하자.
  * 즉, 클라이언트 역시 해당 API를 호출했을 때 서버가 회원 리소스를 생성하는 처리를 수행한다는 사실을 알고 있다.
* 클라이언트는 다음과 같은 HTTP 요청을 전달한다.
```
POST /members HTTP/1.1
Content-Type: application/json

{
  "username": "hello",
  "age": 20
}
```
* 서버는 이를 받아 회원을 신규 생성해야한다는 사실을 알게 되며, 신규 회원을 등록하는 처리를 진행한다.
  * 일반적으로 데이터베이스에 신규 회원 정보를 추가하는 등의 처리를 진행하게 된다.
* 서버는 상술한 과정을 거쳐 회원 생성이 완료되었다는 응답을 주기 위해, 다음과 같은 HTTP 응답 메시지를 반환한다.
```
HTTP/1.1 201 Created // 200 OK를 사용해도 무방하다.
Content-Type: application/json
Content-Length: 34
Location: /members/100 // 자원이 생성된 경로는 일반적으로 Location 헤더로 전달한다.

{
  "username": "hello",
  "age": 20
}
```

### POST 메소드의 요청 데이터 처리 예시
```
> POST 메소드는 대상 리소스가 리소스의 고유한 의미에 따라 요청에 포함된 표현을 처리하도록 요청한다.
```
* 즉, **임의의 리소스 URI에 POST 메소드 요청이 수신된 경우 이를 어떻게 처리할지는 리소스마다 결정되어야 한다**.
  * 결국, POST 메소드에 대응되는 행위는 사전에 정의된 것이 없으므로 요청을 처리하기 위한 무엇이든 적용이 가능하다.
* 따라서 POST 메소드를 통해서 다음과 같은 기능을 구현할 수 있다.
  1. 새로운 리소스 생성: 서버가 아직 식별하지 않은 새로운 리소스를 생성한다.
  2. 요청 데이터의 처리: 단순한 데이터 생성 또는 변경을 넘어 프로세스 자체를 처리해야하는 경우에 해당한다.
     * 예를 들어, **주문 리소스의 상태가 결제완료, 배달시작, 배달완료 등 값 변경을 넘어 프로세스의 상태가 변경되는 경우**가 있다.
     * **POST 처리의 결과로 새로운 리소스가 생성되지 않을 수도 있으며, 그렇다고 해도 서버에서 큰 변화를 일으킨다면 POST를 적용한다**.
     * **POST /orders/{orderId}/start-delivery 등은 제어 기능을 수행하는 컨트롤 URI이며, 이 역시 POST를 활용하는 예시**이다.
  3. **다른 메소드를 적용하기 애매한 경우에도 POST 메소드를 적용**할 수 있다.
     * 예를 들어 JSON 형태의 조회 데이터를 넘겨야하지만 GET 메소드를 사용하기 어려운 경우에도 POST 메소드를 활용한다.
     * 이는 상술했던 'GET 메소드에 메시지 본문을 포함하는 것은 스펙은 허용하지만, 서버들이 이를 지원하지 않는 경우'의 대체제로 사용할 수 있다.

### 컨트롤 URI란?
* URI는 기본적으로 리소스를 식별하는 단위로 설계되어야 하지만, 모든 것을 리소스로 식별할 수 있는 것은 아니다.
  * **리소스만으로 식별할 수 없는 경우, start-delivery와 같은 동사 형태의 URI를 설계해도 무방**하다.
  * 이러한 개념을 컨트롤 URI라고 하며, 실무에서는 종종 사용되는 방식이다.
* **실무에서는 리소스만을 통해 모든 URI를 설계할 수는 없으므로, 기본적으로 최대한 리소스만으로 URI를 설계하되 필요한 경우에 컨트롤 URI를 정의**한다.
* 컨트롤 URI에는 주로 POST 메소드가 사용된다.

### 모든 것을 POST 메소드로 정의해도 될까?
* POST 메소드는 메시지를 내부에 담아 처리하는 모든 것을 수행할 수 있다.
* 그러나 **조회시에는 GET 메소드를 활용하는 것이 유리**하다.
  * 예를 들어, **서버 간에는 GET 메소드로 전달된 HTTP 요청을 캐싱**하는 약속을 하는 경우가 많다.
  * 반면, POST 메소드로 전달된 HTTP 요청은 캐싱하기가 어려우므로 조회 데이터는 GET을 사용하는 것이 바람직하다.
  * 그 외에 데이터가 변경되거나, 프로세스가 진행되거나, 정말 어쩔 수 없는 경우에는 POST를 사용하는 것이 좋다.

### PUT 메소드
* PUT 메소드는 **기존 리소스를 완전히 대체하기 위해 사용하며, 일반적으로 다음과 같이 덮어쓰기와 유사하게 동작**한다.
  1. 리소스가 있으면 대체한다.
  2. 리소스가 없으면 생성한다.
* PUT 메소드의 경우, 리소스를 완전히 대체하므로 부분 속성에 대한 변경에 사용할 수 없다.
  * 예를 들어 username과 age 속성을 갖는 기존 리소스에 대한 age 수정을 위해 PUT을 활용하는 경우, 기존 리소스의 username은 제거된다.
  * 이는 기존 리소스가 age 속성만을 갖는 신규 리소스로 완전히 대체되기 때문이다.
  * 즉, **PUT 메소드는 리소스를 대체하기 위해 사용하는 것이지 리소스의 부분을 수정하기 위해 사용하는 것이 아니다**.
* **PUT 메소드에서 중요한 키워드는 `기존 리소스를 대체`하는 것과, `클라이언트가 리소스의 위치를 인지`하는 것**이다.

### POST 메소드와 PUT 메소드의 차이점
* **POST 메소드는 클라이언트가 리소스의 위치를 알 필요가 없지만, PUT은 반드시 리소스의 위치를 안 상태에서 요청**해야 한다.
  * 따라서, POST 메소드의 경우 신규 회원을 생성하기 위해 `POST /members` URI를 호출한다.
  * 반면, PUT 메소드의 경우 기존 회원을 덮어쓰거나 신규 회원을 생성하기 위해 `PUT /members/{id}` URI를 호출한다.
* 결국 **POST 메소드는 클라이언트가 회원이 생성되는 `Location`을 알 수 없지만, PUT 메소드는 정확한 `Location`의 리소스에 대한 요청을 호출**한다.

### PATCH 메소드
* **기존 리소스의 부분을 변경하기 위해 사용하는 메소드는 PATCH 메소드**이다.
* **레거시 서버 중에는 PATCH를 받아들이지 못하는 경우도 있으며, 이러한 경우에도 POST를 활용해서 대응**할 수 있다.
  * POST는 무적이다.

### DELETE 메소드
* **기존 리소스를 제거하기 위해 사용되는 메소드는 DELETE 메소드**이다.

### HTTP 메소드의 속성
* HTTP 메소드의 속성은 안전, 멱등, 캐시 가능으로 분류할 수 있다.
* **안전이란, 호출하더라도 리소스를 변경하지 않는 속성을 의미**한다.
  * 안전은 해당 리소스만을 고려하며, 지속적인 호출로 인해 로그가 쌓여 발생한 장애는 고려하지 않는다.
  * 예를 들어, GET 메소드는 단순 조회이므로 안전을 보장한다.
  * 반면, POST / DELETE / PUT 등은 리소스의 변경을 수반하므로 안전하지 않다.
* **멱등이란, 몇 번 호출하든지 결과가 같은 특징을 의미**한다.
  * 예를 들어, **GET / PUT / DELETE는 멱등이 보장되는 메소드**이다.
  * 반면, POST는 멱등이 보장되지 않는 메소드이다.
  * **멱등은 자동 복구 메커니즘 등에 활용할 수 있으며, 예를 들어 클라이언트가 삭제 요청에 대한 응답을 받지 못했더라도 재시도를 가능케하는 근거**가 된다.
  * 멱등은 외부 요인으로 인한 리소스 변경은 고려하지 않으므로, GET > PUT > GET 순서로 호출된 리소스의 응답이 변경되는 것으로 멱등이 깨진 것은 아니다.
    * 이렇듯 **멱등은 동일한 사용자 한 명이 동일한 요청을 여러 번 송신했을 때의 결과로 판단하는 속성**이다.
* **캐시 가능이란, 응답 결과 리소스를 캐시해서 사용해도 되는지를 결정하는 속성**이다.
  * 예를 들어, 너무 커다란 이미지를 요청한 경우 이를 매 요청마다 다운로드하는 것은 손해가 크다.
  * 때문에 이러한 큰 이미지를 웹 브라우저가 로컬에 저장해두고, 다음 요청에서 재활용한다.
  * 이렇듯 캐시 가능이란 웹 브라우저가 내 요청을 저장하고 있어도 되는지를 결정하는 속성이다.
  * 기술적으로는 GET / HEAD / POST / PATCH를 캐시할 수 있지만, **실무에서는 GET / HEAD 정도만 캐시를 사용**한다.
    * 이는 캐시를 위해 같은 리소스임을 보장하는 캐시 키가 사용되어야 하기 때문이다.
    * POST / PATCH는 본문 내용까지 캐시 키로 고려해야 하지만, 이를 구현하기가 쉽지 않아 캐싱이 사용되지 않는 편이다.
    * 반면, **GET의 경우 URI만으로도 캐시 키를 결정할 수 있기 때문에 실무에서도 캐싱이 자주 적용**된다. 