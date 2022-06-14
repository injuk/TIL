# Basics
## 2022-06-14 Tue

### 컨트롤러 우선 순위
* 스프링은 요청이 들어왔을 때 관련된 컨트롤러가 있는지 우선 확인하고, 없는 경우에는 `resources/static/index.html` 파일을 웰컴 페이지로 활용한다.
  * 거꾸로 말해, `static` 폴더에 적절한 HTML 파일이 있더라도 요청을 처리하는 컨트롤러가 존재한다면 웰컴 페이지를 사용하지 않는다.

### 컨트롤러의 흐름
* 아래와 같은 컨트롤러를 작성했다고 하자.
```
@Controller
public class MemberController {

    private final MemberService memberService;

    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }

    @GetMapping("/members/new")
    public String memberForm() {

        return "members/createMemberForm";
    }

    @PostMapping("/members/new")
    public String create(MemberForm memberForm) {
        Member member = new Member();
        member.setName(memberForm.getName());
        memberService.join(member);

        return "redirect:/";
    }
}
```
* 사용자가 브라우저에 `/members/new`를 입력하면 스프링은 다음과 같이 동작한다.
  1. 브라우저는 기본적으로 GET 방식을 사용하므로, @GetMapping("/members/new") 어노테이션이 명시된 메소드가 호출된다.
  2. 해당 메소드는 템플릿 파일의 이름을 반환하며, 스프링은 `resources/templates`에서 뷰 리졸버에 의헤 선택된 뷰를 전달받는다.
  3. 해당 뷰는 템플릿 엔진인 타임 리프에 의해 렌더링된다.
  4. 렌더링이 종료된 후, 렌더링된 HTML이 화면에 출력된다.
* 해당 **뷰에 `method="post"`인 form이 존재한다고 할 때, 해당 form에 포함된 input의 `name` 속성이 서버에 넘겨질 키** 값이 된다.
* form에 필요한 정보를 작성하고 제출한 경우, @PostMapping("/members/new") 어노테이션이 명시된 메소드가 호출된다.
* 이 때, create 메소드가 호출될 때 전달된 데이터는 적절한 형식의 객체로 받을 수 있다.
  * 상술한 코드의 경우, `String name` 멤버 변수를 갖는 `MemberForm` 인스턴스에 값이 전달된다.
  * HTML의 form에서 전달한 데이터의 키 역시 name이므로, memberForm의 name 멤버 변수에 해당 값이 설정된다.
  * **즉, 스프링은 form에 포함된 input의 name 속성에 명시된 값을 토대로 memberForm의 멤버 변수를 설정**한다.

## 2022-06-15 Wed
### 모델에 데이터 넘기기
* 다음과 같이 컨트롤러 메소드를 작성한 경우, members라는 키에 List<Member> 변수의 값을 설정한다.
```
@GetMapping("/members")
public String listMembers(Model model) {
    List<Member> members = memberService.findMembers();
    model.addAttribute("members", members);

    return "members/memberList";
}
```
* 이렇게 모델에 전달된 데이터는 타임리프를 통해 다음과 같이 활용할 수 있다.
  * 이렇듯 타임리프는 모델에 전달된 데이터를 활용하여 화면을 렌더링한다.
```
<tr th:each="member : ${members}">
    <td th:text="${member.id}"></td>
    <td th:text="${member.name}"></td>
</tr>
```