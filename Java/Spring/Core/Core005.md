# Core
## 2022-06-24 Fri

### 싱글톤 컨테이너
* 스프링은 기본적으로 기업의 서비스 애플리케이션을 지원하기 위해 탄생하였다.
  * 대부분의 스프링 애플리케이션은 웹 애플리케이션이지만, 굳이 웹이 아닌 애플리케이션도 개발이 가능하다.
  * 반면, **기업용 웹 애플리케이션은 태생적으로 많은 고객이 동시에 요청을 전달하기 쉽다**.

### 기존 방식의 한계
* 스프링 없이 구현한 순수 DI 컨테이너 예시의 경우, 매 의존성 주입 요청마다 새로운 객체가 생성되는 한계가 존재한다.
  * 예를 들어 아래의 테스트 코드는 `MemberService`를 요청할 때마다 새로운 객체가 생성되어 반환되므로 반드시 실패한다.
```
@Test
@DisplayName("스프링 없는 순수한 DI 컨테이너의 한계.")
void pureContainer() {
    AppConfig appConfig = new AppConfig();

    MemberService memberService1 = appConfig.memberService();
    MemberService memberService2 = appConfig.memberService();

    System.out.println("memberService1 = " + memberService1);
    System.out.println("memberService2 = " + memberService2);
    
    assertThat(memberService1).isEqualTo(memberService2);
}
```
* 이 경우, 매 사용자의 요청마다 새로운 객체가 생성되어 JVM 메모리에 올라가게 된다.
  * 때문에 **기존 방식에서는 초당 1000건의 요청이 발생한다면 새로운 객체는 매 초 1000개가 생성되어야하며, 이는 매우 비효율적인 방식**이다.
  * 엎친 데 덮친 격으로, 만약 각 의존성이 또 다른 의존성 객체를 요청한다면 초당 생성되는 객체 수는 기하급수적으로 늘어갈 수 있다.
* 이렇듯 기존 방식은 메모리 낭비가 매우 심하므로, 싱글톤 패턴을 도입하여 문제를 해결할 필요성이 있다.
  * **싱글톤 패턴을 도입하는 경우, 객체가 항상 단 1개만 생성되는 것을 보장하여 모든 사용자의 요청에서 같은 객체를 공유하도록 할 수 있다**.