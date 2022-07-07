# HandsOn
## 2022-07-08 Fri

### validation 추가하기
* DTO로 사용되는 객체에 필수 값을 지정해야하는 경우, @NotEmpty 어노테이션을 활용할 수 있다.
  * 해당 어노테이션을 명시할 경우, 스프링에 의한 유효성 검증이 가능하다.
```
@Getter @Setter
public class MemberForm {

    @NotEmpty(message = "회원 이름은 필수입니다.")
    private String name;

    private String city;
    private String street;
    private String zipcode;
}
```
* 이렇게 **추가한 `javax.validation`의 어노테이션들은 다음과 같이 @Valid 어노테이션을 컨트롤러에 명시하는 것을 통해 검증**할 수 있다.
```
@PostMapping("/members/new")
public String createMember(@Valid MemberForm memberForm) {

    Member member = new Member();
    member.setName(memberForm.getName());
    member.setAddress(
            new Address(memberForm.getCity(), memberForm.getStreet(), memberForm.getZipcode())
    );

    Long id = memberService.join(member);
    return "redirect:/";
}
```
* 그러나 이 경우, 에러에 대한 아무런 처리가 되어 있지 않으므로 입력 폼에 회원 명을 입력하지 않게 되면 바로 Whitelabel 페이지로 리다이렉트된다.
  * 이는 컨트롤러에서 오류가 발생한 경우 스프링이 기본적으로 오류를 전파하기 때문에 발생하는 현상이다.
* 이는 바람직하지 못한 방식이며, 오류가 발생한 경우에 대한 조치를 추가하기 위해 컨트롤러 메소드의 매개변수에 `BindingResult` 객체를 추가할 수 있다.
  * 이렇듯 **메소드 매개변수에 `BindingResult`가 명시된 경우, 스프링은 오류가 발생하면 자동적으로 오류를 해당 객체에 담아 코드를 계속해서 실행**한다.
  * **정확히는 @Valid와 같이 validation 대상이 된 매개변수 다음에 `BindingResult` 객체가 명시**되어야 한다.
  * 즉, 오류로 인해 Whitelabel 페이지로 리다이렉트하지 않고도 후속조치를 취할 수 있게 된다.
```
@PostMapping("/members/new") // BindingResult를 추가하는 것만으로 오류는 해당 객체에 담기게 된다.
public String createMember(@Valid MemberForm memberForm, BindingResult bindingResult) {
    // @Valid에 의해 오류가 발생한 경우, 다시 회원 가입 화면을 반환한다.
    if(bindingResult.hasErrors()) {
        return "members/createMemberForm";
    }

    Member member = new Member();
    member.setName(memberForm.getName());
    member.setAddress(
            new Address(memberForm.getCity(), memberForm.getStreet(), memberForm.getZipcode())
    );

    Long id = memberService.join(member);
    return "redirect:/";
}
```

### Model 클래스란?
* 일반적인 컨트롤러에는 다음과 같이 Model 객체를 매개변수로 받는 메소드를 작성하게 된다.
```
@GetMapping("/members/new")
public String createForm(Model model) {
    return "create";
}
```
* 이 때, Model 클래스는 컨트롤러로부터 뷰로 반환할 때 데이터를 포함시키기 위해 사용한다.

### DTO와 엔티티의 구분
* 뷰 계층에서 컨트롤러 계층으로 DTO를 넘기고, 컨트롤러는 해당 객체의 유효성을 검증할 수 있다.
* 이 때, DTO와 엔티티가 포함하는 필드가 유사해보인다고 해서 모든 것을 엔티티로 사용하는 실수를 범하지 않아야 한다.
* 이러한 방식은 다음과 같은 문제점을 유발할 수 있다.
  1. 엔티티 객체가 유효성 검증 코드로 오염된다.
  2. **컨트롤러 단에서 요구하는 유효성과 핵심 비즈니스 로직에서 요구하는 유효성은 다를 수 있지만, 이를 엔티티에 반영하기 어려워진다**.
* 특히 실무에서는 뷰 계층 역시 복잡해지므로, 뷰로부터 전달받은 데이터를 그대로 엔티티에 녹여내기 어렵기도 하다.
  * **때문에 폼 형태로 데이터를 전달 받아 컨트롤러 단에서 필요한 값을 통해 엔티티의 필드를 채운 후에 계층 별로 넘겨주는 것이 바람직**하다. 