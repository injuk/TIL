# Basics
## 2022-06-07 Tue

### 클라이언트에서 서버로의 데이터 전송
* 클라이언트에서 서버로 데이터를 전송하는 방식은 크게 다음과 같이 분류된다.
  1. 쿼리 파라미터를 활용한 데이터 전송: 주로 GET 메소드에서 활용되며, 검색어와 같은 정렬 필터를 전달한다.
  2. 메시지 본문을 활용한 데이터 전송: 주로 POST / PUT / PATCH 메소드에서 활용되며, 리소스 생성 또는 변경을 위한 데이터를 전달한다.
* 이를 토대로 다음과 같은 네 가지 시나리오를 고려해볼 수 있다.
  1. 이미지, HTML 등의 정적 데이터 조회
  2. 검색, 정렬 등 동적 데이터 조회
  3. 회원 가입 등 HTML Form을 활용한 데이터 전송
  4. 회원 가입 등 HTTP API를 활용한 데이터 전송

### 정적 데이터 조회
* 이미지, HTML 등은 조회이므로 GET 메소드를 활용하여 요청한다.
* **정적인 데이터는 일반적으로 query 없이 리소스 경로만으로 단순하게 경로 조회가 가능**하다.

### 동적 데이터 조회
* **검색어 등을 통해 동적으로 데이터를 조회하는 경우, query를 통해 데이터를 전달할 필요**가 있다.
  * 서버는 query를 기준으로 필터링하여 결과를 동작으로 생성하여 반환한다.
* 주로 검색, 게시판 목록에서의 정렬 필터, 조회 결과를 줄여주는 필터 등에 사용된다.
* **조회의 목적이 있으므로 GET 메소드를 사용하고, GET은 query를 활용하여 데이터를 전달**한다.
  * 스펙상 GET 역시 메시지 본문을 받을 수 있지만, 이를 지원하지 않는 서버들이 많기에 권장되지 않는 방식이다.

### HTML Form 데이터 전송
* 순수한 HTML만을 사용하여 서버에 HTTP 요청을 전송하는 경우, HTML Form을 사용하게 된다.
* 웹 브라우저는 Form의 submit 버튼이 클릭된 시점에 Form의 데이터를 기반으로 HTTP 요청 메시지를 생성한다.
```
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded

username=kim&age=20
```
* 메시지 본문에서 알 수 있듯, HTML Form 데이터는 `application/x-www-form-urlencoded`에 의해 query와 유사한 형태의 데이터가 만들어진다.
  * POST 메소드는 메시지 본문이 있어야하므로, 해당 방식은 query와 거의 동일한 방식으로 서버에 전송하게 된다.
* `username=kim&age=20` 데이터는 `application/x-www-form-urlencoded` 형식의 Content-Type에 맞게 표현된 데이터이다.
  * 대부분의 웹 서버는 이러한 요청을 파싱하여 처리할 수 있다.
* Form의 method 속성을 post가 아닌 get으로 작성한 경우, 웹 브라우저는 다음과 같은 HTTP 요청 메시지를 생성한다.
```
GET /save?username=kim&age=20 HTTP/1.1
Host: localhost:8080
```
* 이는 GET 메소드가 일반적으로 메시지 본문을 사용하지 않기 때문으로, 데이터를 쿼리로 바꾸어 서버에 전송하게 된다.
* 이렇듯 Form 태그로 데이터를 작성하였더라도 method가 get으로 표현되어 있었다면 GET 메소드에 맞는 요청 메시지를 자동으로 생성한다.
  * 즉, **Form 태그의 method 속성이 post이냐 get이냐에 따라 웹 브라우저가 생성하는 HTTP 요청 메시지의 형태는 달라진다**.
  * 그러나 당연스럽게도 GET 메소드는 조회 용도로만 사용하는 것이 바람직하다.
* HTML Form을 활용한 데이터 전송의 경우, 상술한 바와 같이 form의 내용은 `application/x-www-form-urlencoded`으로 적용된다.
  * 때문에 form의 내용은 메시지 본문을 통해 `key=value`형태로 전송되며 URL encoding 처리된다.
* **HTML Form을 활용한 데이터 전송은 GET과 POST 메소드만을 지원**한다.

### HTTP API 데이터 전송
* 해당 방식은 다음과 같은 경우에 사용될 수 있다.
  1. 서버 간의 통신: 백엔드 시스템 서버는 HTML이 존재하지 않으므로, Form을 활용한 데이터 전송을 사용할 수 없다.
  2. 앱 클라이언트와 서버 간의 통신
  3. 웹 클라이언트와 서버 간의 통신: HTML의 Form 전송 대신 JS를 활용한 통신에 사용한다(=AJAX).
     * React나 Vue.js와 같은 웹 클라이언트의 경우, API 통신 방식을 주로 사용하게 된다.
* **HTTP API의 경우 GET / POST / PUT / PATCH / DELETE 모든 메소드를 활용**할 수 있다.
  * GET 메소드는 조회 용도로 사용되며, query를 통해 데이터를 전달한다.
  * POST / PUT / PATCH 메소드는 메시지 본문을 통해 데이터를 전송한다.
* **해당 방식의 Content-Type은 application/json을 사용하며, 이는 사실상의 표준**이다.
  * 기존에는 XML이 사실상의 표준이었으나, 가독성과 데이터의 크기 문제로 인해 JSON으로 대체되었다.