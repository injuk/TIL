# Spring Advanced
## 2025-01-01 Wed
### this와 target의 차이
* `this`와 `target` 포인트컷 지시자는 각각 다음과 같은 형태로 사용이 가능하다.
```java
class Temp {
    @Before("allMember() && this(obj)")
    public void thisArgs(JoinPoint joinPoint, MyService obj) {
        System.out.println("[this] obj=" + obj.getClass());
    }
    @Before("allMember() && target(obj)")
    public void targetArgs(JoinPoint joinPoint, MyService obj) {
      System.out.println("[target] obj=" + obj.getClass());
    }    
}
```
* 이 때, 로그의 결과를 토대로 확인했을 경우 각각의 포인트컷 지시자는 다음과 같은 의미를 갖는 것을 이해할 수 있다.
  * `this`: 스프링 컨테이너에 등록된 실제 인스턴스로, 일반적으로 프록시 객체가 된다.
  * `target`: 어드바이스 적용 대상 객체로, 이 경우 프록시가 내부적으로 실제로 호출하게 될 실제 대상 객체인 `MyService` 클래스 자체가 된다.