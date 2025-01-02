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

## 2025-01-02 Thu
### this와 target 주의사항
* `this`와 `target`은 모두 `*`과 같은 패턴을 사용할 수 없는 대신 적용 대상의 타입을 명확히 지정해야 하며, 부모 타입을 허용하는 특징이 있다.
* 또한, 상술한 바와 같이 두 지시자는 각각 스프링 컨테이너에 등록된 스프링 AOP 프록시 객체 또는 실제 대상 객체를 대상으로 동작하는 조인 포인트에 해당한다.
  * **프록시가 등록되는 경우 프록시 인스턴스가 빈으로 등록되는 반면, 프록시 대상 객체는 엄밀히 말해 빈 객체가 아니라는 점에서 차이**가 있다.
  * 다시 말해 `this`는 스프링 빈에 등록된 프록시 객체를 대상으로 포인트컷을 매칭하는 반면, `target`은 프록시가 참조하는 실제 객체를 대상으로 동작한다.
* 이 때, `this` 표현식의 경우 구체 클래스 기반 프록시를 대상으로 했다면 CGLIB과 JDK 동적 프록시 중 무엇을 사용하느냐에 따라 다른 결과가 나올 수 있다.
  * 반면, `this`와 `target` 지시자는 그 동작 방식을 이해하기 다소 어렵지만 실무적인 관점에서 자주 사용되지는 않는다는 특징을 갖는다.
  * 이러한 특징으로 인해 두 지시자는 각각 단독으로 사용되기보다는 파라미터 바인딩에 활용되는 경우가 많다.