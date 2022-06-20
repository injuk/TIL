# Core
## 2022-06-20 Mon

### 프로젝트 설정
* `spring.start.io`에서 별도의 종속성 없이 프로젝트를 생성할 경우, 스프링 코어와 관련된 종속성만이 다음과 같이 포함된다.
```
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```
* 이 경우, `starter-web` 과 같은 웹 관련 종속성이 없으므로 프로젝트를 실행해도 exit code 0으로 종료된다.

### 확정되지 않은 요구사항
* 실무에서는 요구사항이 모두 확정되지 않은 상태에서 개발을 시작해야할 수도 있다.
  * 예를 들어, 주문 시스템의 할인 정책은 최대한 숙고하기 위해 확정이 늦춰질 수도 있다.
  * 최악의 경우에는 할인을 적용하지 않을 수도 있으나, 정책이 결정될 때까지 개발을 미룰 수는 없다.
* **인터페이스를 적절히 활용하여 역할과 구현을 성공적으로 분리할 경우, 이러한 상황 속에서도 필요한 다른 부분들을 개발하는 것이 가능**하다.

### 다이어그램 활용하기
* 요구사항을 토대로 도메인을 설계하는 경우, 기본적으로 다음과 같은 다이어그램을 활용해볼 수 있다.
  1. 회원 도메인 협력 관계 다이어그램: 회원 도메인에 포함되는 여러 객체 간의 협력을 개략적으로 묘사한다.
     * 해당 다이어그램은 기획자가 함께 공유할 수도 있다.
  2. 회원 클래스 다이어그램: 실제 구현 단위로 구현할 경우 어떠한 인터페이스와 구현체들이 필요한지 묘사한다.
     * 개발자는 1.을 토대로 내용을 구체화한다.
  3. 회원 객체 다이어그램: 서버에서 애플리케이션이 동작할 때, 실제로 만들어진 객체 간의 참조 관계를 묘사한다.
* 회원 객체 다이어그램의 경우, 서버에서 동작하는 애플리케이션에서 동적으로 결정되는 구현체를 한 눈에 보기 위해 작성한다.
  * 구현체는 동적으로 결정되므로, 회원 클래스 다이어그램만으로는 실제로 어떠한 객체와 협력하는지 판단하기 어려울 수 있기 때문이다.

### MemberRepository의 구현
* 인메모리 회원 리포지토리의 경우, 다음과 같이 구현할 수 있다.
  * 이 때, **인메모리 저장소 역할을 수행하는 `storage` 변수는 실무의 경우 동시성 이슈로 인해 ConcurrentHashMap을 사용**해야 한다. 
```
public class InMemoryMemberRepository implements MemberRepository {
    
    private static Map<Long, Member> storage = new HashMap<>(); 
    
    @Override
    public void save(Member member) {
        storage.put(member.getId(), member);
    }

    @Override
    public Member findById(Long id) {
        return storage.get(id);
    }
}
```

### MemberService의 구현
* 회원 서비스의 경우, 인터페이스인 `MemberService`를 구현하는 `MemberServiceImpl` 클래스를 다음과 같이 구현할 수 있다.
  * 이 때, **특정 인터페이스에 대한 구현체가 단 하나 뿐인 경우 관례상 `Impl`이라는 접미사를 붙여 작성**할 수 있다.
```
public class MemberServiceImpl implements MemberService {
    
    private final MemberRepository memberRepository = new InMemoryMemberRepository();
    
    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member find(Long id) {
        return memberRepository.findById(id);
    }
}
```