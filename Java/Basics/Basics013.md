# Java
## 2022-01-09 Sun

### java.time 패키지
* Java의 시간을 다루기 위한 클래스인 Date와 Calendar의 단점들을 해소하기 위해 추가된 패키지이다.
* 기본적으로 java.time 패키지에 속한 클래스들은 String 클래스처럼 불변하다.
* 즉, 날짜나 시간을 변경하는 메소드들은 기존 인스턴스를 변경하는 것이 아닌 새로운 인스턴스를 반환한다.
  * 만약 기존 인스턴스를 수정하는 방식을 사용하면 쓰레드 안전하지 않은 클래스라고 부른다.
* java.time의 날짜와 시간을 표현하는 클래스들은 다음과 같다.
  * LocalTime 클래스: 시간을 표현할 때 사용한다.
  * LocalDate 클래스: 날짜를 표현할 때 사용한다.
  * LocalDateTime 클래스: 날짜와 시간을 모두 표현해야할 때 사용한다.
  * ZonedDateTime 클래스: 시간대까지 다뤄야하는 경우에 사용한다.
* 위 네가지 클래스의 경우, 인스턴스를 생성하기 위해 now 또는 of 메소드를 활용할 수 있다.
  * now(): 현재 날짜와 시간을 저장하는 인스턴스를 생성한다.
  * of(): 필요한 값을 인자로 넘겨 인스턴스를 생성한다.
* java.time의 시간 간격을 표현하는 클래스들은 다음과 같다.
  * Period 클래스: 날짜 간의 차이를 표현한다.
  * Duration 클래스: 시간 사이의 차이를 표현한다.
* 참고로 timestamp란, 날짜와 시간을 초 단위로 표현한 값이다.

### DateTimeFormatter
* java.time 패키지의 클래스를 통해 날짜와 시간을 다룬다면, 포매팅에 관련된 내용은 DateTimeFormatter를 활용한다.
* DateTimeFormatter에는 자주 쓰이는 형식이 이미 정의되어 있으며, LocalDate, LocalTime 등에 포함된 format 메소드를 활용하여 적용할 수 있다.
```
LocalDate date = LocalDate.of(2022, 1, 9);
String formatted = date.format(DateTimeFormatter.ISO_LOCAL_DATE); //2022-01-09
```
* 또는 ofPattern 메소드를 활용하여 원하는 출력 형식을 직접 작성할 수도 있다.
  * ofPattern으로 작성한 출력 형식은 파싱에도 사용될 수 있다.  
```
// DateTimeFormatter.ISO_LOCAL_DATE의 형식을 따르는 예시이다.
DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd");
// 아래는 ofPattern으로 생성한 패턴을 parse 메소드의 인자로 넘겨 문자열을 LocalDateTime으로 파싱하는 예제이다.
LocalDateTime ldt = LocalDateTime.parse("2022-01-09 23:30:31", dtf);
```
