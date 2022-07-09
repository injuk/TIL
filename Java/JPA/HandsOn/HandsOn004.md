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
  * 이를 억지로 녹여내는 경우, 유지보수 과정에서 엔티티와는 관계 없는 화면 관련 필드가 지속적으로 추가되게 된다.
  * **때문에 폼 형태로 데이터를 전달 받아 컨트롤러 단에서 필요한 값을 통해 엔티티의 필드를 채운 후에 계층 별로 넘겨주는 것이 바람직**하다. 
* 이렇듯 **JPA를 사용하는 경우, 엔티티는 다른 계층에 의존하는 일 없이 최대한 순수하게 유지**되어야 한다.
  * 엔티티는 오직 핵심 비즈니스 로직에만 의존되어야 한다.
  * 반면, **화면 종속적인 객체는 폼 객체 또는 DTO를 사용하는 것이 바람직**하다.

### 회원 목록 기능 구현하기
* 회원 서비스로부터 받아온 회원 목록을 화면에 반환하는 컨트롤러 메소드는 다음과 같이 간단하게 작성할 수 있다.
```
@GetMapping("/members")
public String memberList(Model model) {
    model.addAttribute("members", memberService.findMembers());
    return "members/memberList";
}
```
* 그러나 이러한 메소드는 엔티티를 그대로 뷰 계층에 반환하며, 이는 상술한 DTO와 엔티티의 구분 원칙에 어긋난다.
  * 정말 **요구사항이 단순한 경우라면 상술한 방식을 도입하여도 무방하지만, 일반적으로는 별도의 DTO를 두고 객체를 바꾸어 반환하는 것이 바람직**하다.
* 만약 템플릿 엔진과 서버사이드 렌더링을 사용하는 경우라면, 엔티티를 그대로 반환하는 방식을 사용하더라도 큰 문제는 발생하지 않을 수 있다.
  * 물론 아무리 간단한 요구사항에서도 가능하다면 DTO를 도입하는 것이 권장된다.
* 그러나 **API를 작성하는 경우, 절대 엔티티 객체를 그대로 반환하지 않아야 한다**.
  * **API는 기본적으로 스펙이지만, 엔티티를 그대로 반환하게 되면 엔티티의 구조가 수정된 경우에 API의 스펙이 덩달아 변경되버리는 문제가 발생**한다.
  * 엔티티의 로직 변경 등 수정 작업이 API의 스펙을 변경하게 되면 API는 그만큼 불안정해지며, 협업 관점에서도 큰 불편을 초래할 수 있다.

## 2022-07-09 Sat
### path variable 받아들이기
* path variable 정보는 컨트롤러에서 다음과 같이 @PathVariable 어노테이션을 통해 받아들일 수 있다. 
```
@GetMapping("/items/{itemId}/edit")
public String updateItem(@PathVariable(value = "itemId") Long id, Model model) {

    return "items/updateItemForm";
}
```

### JPA로 update하기
* JPA에서의 update는 merge 메소드를 활용한다.
  * 아래의 코드에서, save는 서비스 내부적으로 merge 메소드를 활용하는 리포지토리를 호출하도록 동작한다.
```
@PostMapping("/items/{itemId}/edit")
public String updateItem(@PathVariable("itemId") Long id, @ModelAttribute("form") BookForm bookForm) {
    Book book = new Book();
    book.setId(bookForm.getId());
    book.setName(bookForm.getName());
    book.setPrice(bookForm.getPrice());
    book.setStockQuantity(bookForm.getStockQuantity());
    book.setAuthor(bookForm.getAuthor());
    book.setIsbn(bookForm.getIsbn());

    itemService.saveItem(book);

    return "redirect:/items";
}
```
* 상술한 코드의 경우, itemId 정보가 path variable을 통해 그대로 드러나는 취약점이 존재한다.
  * 따라서 실무에서는 요청한 사용자가 해당 item에 대한 권한이 있는지 없는지를 체크하는 로직이 반드시 추가되어야 한다.

### 변경 감지, Dirty checking
* **엔티티가 영속 상태로 관리되는 경우, 엔티티 상에서 변경사항이 발생했다면 JPA는 이를 감지하여 트랜잭션 커밋 시점에 반영**한다.
  * 예를 들어 아래의 코드에서 JPA에 의해 관리되는 엔티티는 코드 상에서 변경되고, 이는 트랜잭션 커밋 시점에 쿼리를 통해 데이터베이스에 반영된다.
```
@Test
public void updateItem() {
    // member는 JPA에 의해 관리되는 엔티티이다.
    Member member = em.find(Member.class, 1L);
    member.setName("injuuuuk");
    
    // transaction commit
    // Dirty checking, JPA는 트랜잭션 커밋 시점에 엔티티의 변경 사항을 감지하여 쿼리를 통해 반영한다.
}
```
* 이렇듯 **JPA가 트랜잭션 커밋 시점에 변경 사항을 확인하여 영속화하는 `변경 감지` 전략을 통해 기본적으로 변경 사항을 데이터베이스에 반영**할 수 있다.
  * 즉, **flush 시점에 발생하는 더티 체킹을 통해 영속화된 데이터의 변경이 가능**하다.

### 준영속 엔티티
* **준영속 엔티티는 영속성 컨텍스트가 더 이상 관리하지 않는 객체**를 말한다.
```
public String updateItem(@PathVariable("itemId") Long id, @ModelAttribute("form") BookForm bookForm) {
    Book book = new Book();
    book.setId(bookForm.getId());
    book.setName(bookForm.getName());
    book.setPrice(bookForm.getPrice());
    book.setStockQuantity(bookForm.getStockQuantity());
    book.setAuthor(bookForm.getAuthor());
    book.setIsbn(bookForm.getIsbn());

    itemService.saveItem(book);

    return "redirect:/items";
}
```
* 예를 들어, 상술한 Book 객체는 new 연산자를 통해 새로이 생성되었으나 식별자인 id 값이 존재한다.
  * 이는 곧 **Book 객체의 내용은 이미 JPA를 통해 한 번 관리되었던 객체임을 의미하며, 이를 준영속 상태의 객체라고 표현**할 수 있다.
  * 이러한 엔티티는 영속성 컨텍스트가 더 이상 관리하지 않는다.
* 이렇듯 **개발자가 new 연산자를 통해 임의로 생성한 객체이더라도, 세터를 통해 식별자 정보를 설정하였다면 준영속 엔티티**로 볼 수 있다.
* 그러나 **준영속 상태의 엔티티는 말 그대로 JPA 영속성 컨텍스트에 의해 관리되지 않고 있다는 문제점이 존재**한다.
  * 영속성 컨텍스트에 의해 관리되는 엔티티는 JPA에 의해 관리되며, 변경 감지의 대상이 된다.
  * 때문에 트랜잭션 커밋 시점에 변경 사항만 골라내어 쿼리를 통해 데이터베이스에 반영할 수 있게 된다.
* 반면, 상술한 Book 객체의 경우 JPA에 의해 관리되지 않고 개발자가 인스턴스화한 엔티티이므로 아무리 코드를 통해 변경하더라도 변경 사항은 영속화되지 않는다.
  * 이 경우, 트랜잭션이 실제로 커밋된다고 한들, JPA는 해당 객체를 영속화해야한다는 사실 자체를 인식할 수 없다.
  * 때문에 준영속 상태의 엔티티를 관리할 수 있는 별도의 방안이 필요하게 된다.

### 준영속 엔티티를 수정하는 방법
* 준영속 엔티티의 수정 사항을 영속화하는 방법은 크게 다음과 같이 분류할 수 있다.
  1. 변경 사항 감지 기능 사용하기
  2. merge 사용하기

### 준영속 엔티티와 변경 사항 감지
* 준영속 엔티티의 수정 사항을 영속화하기 위해 변경 사항 감지를 사용하는 경우, 다음과 같은 코드를 작성할 수 있다.
```
@Transactional
public Item updateItem(Long itemId, Item item) {
    // 이 상태에서 Item 객체는 영속 상태이다.
    Item found = itemRepository.findOne(itemId);
    found.setName(item.getName());
    found.setPrice(item.getPrice());
    found.setStockQuantity(item.getStockQuantity());
    // 이 상태에서 itemRepository.save 또는 em.persist를 호출할 필요조차 없다!
    return found;
}
```
* 상술한 코드에서, **내부적으로 EntityManager를 사용하는 itemRepository를 활용하여 가져온 Item 객체는 영속 상태에 해당**한다.
* 또한 **updateItem 메소드가 마지막 라인까지 실행할 완료한 경우, @Transactional 어노테이션에 의해 트랜잭션 커밋이 발생**하게 된다.
  * **트랜잭션 커밋이 발생하면 JPA는 flush를 진행하게 되며, 이를 통해 영속성 컨텍스트에서 관리되는 엔티티 중 변경 사항이 발생한 항목을 확인**한다.
  * updateItem 메소드의 경우 Item 객체인 found에 변경 사항이 발생하였으므로, JPA는 변경 사항을 영속화하기 위한 쿼리를 통해 데이터베이스에 반영한다.
* **변경 사항 감지를 활용한 준영속 엔티티의 수정 사항 반영 방식이 더 권장되며, 가능하다면 변경 사항 감지 기능을 활용하는 것이 바람직**하다.
  * 즉, **해당 방식은 트랜잭션 안에서 엔티티를 조회한 후 값을 수정하고, 트랜잭션을 커밋하여 변경 사항 감지가 발생하도록 유도하는 방식**이다.

### 준영속 엔티티와 merge
* **병합은 준영속 상태의 엔티티를 영속 상태로 변경하기 위해 사용하는 기능**이다.
  * 단적으로 말해 병합은 상술한 updateItem 메소드와 같은 기능을 JPA가 알아서 수행해주는 것으로 이해할 수 있다.
* 병합을 위해 merge를 호출한 경우, JPA는 다음과 같이 동작한다.
  1. merge 메소드의 매개변수로 전달된 객체의 식별자 값을 토대로 영속성 컨텍스트의 1차 캐시를 조회한다.
     * 1차 캐시에서 조회하지 못한 경우, 데이터베이스에서 조회한다.
  2. **조회한 엔티티가 갖는 모든 값을 merge 메소드의 매개변수로 전달된 객체의 모든 값으로 변경한다**.
  3. 모든 값이 바뀐 엔티티는 영속 상태이며, 이를 merge 메소드 호출자에게 반환한다.
* 이 떄, **중요한 것은 merge 메소드는 영속성 컨텍스트로부터 조회한 값을 영속 상태로 만들어 값을 수정한 후에 반환한다는 점**이다.
  * 다시 말해, **merge 메소드에 전달한 객체는 영속화되지 않는다!**
```
// merged 겍체는 영속성 컨텍스트에서 관리되는 객체이다.
// 반면, item 객체는 merge 메소드의 매개변수로 전달된 준영속 엔티티이다. 
Item merged = em.merge(item);
```

### merge 기능의 주의사항
* **변경 감지를 활용하는 방식은 엔티티의 원하는 값만을 수정할 수 있는데 비해, 병합 기능은 병합 대상 엔티티의 모든 값을 교체**한다.
  * 때문에 병합 기능의 사용은 주의를 기울여야 하며, 최악의 경우 값이 없는 피드들이 모두 null로 교체될 수도 있다.
* 때문에 **준영속 엔티티의 수정 사항을 영속화하고자 하는 경우에는 merge 기능보다는 변경 감지를 적용하는 방식을 우선 고려하는 것이 바람직**하다.
  * 실무에서는 간단한 요구사항이 없을 가능성이 높으므로, merge 기능만으로 깔끔하게 해결되지 않는 경우가 많다.
  * **사실상 실무에서는 merge 기능을 사용하지 않는다는 마음가짐을 갖는 것도 좋다**.

### update 기능과 best practice
* 상술한 예시에서는 세터를 사용하였지만, **가능하다면 엔티티 자체에 값의 수정과 관련된 메소드를 추가하여 이를 호출하는 것이 바람직**하다.
  * 세터는 되도록이면 사용을 지양해야 한다.
  * 또한 세터 대신 별도의 메소드를 두는 방식은 어느 지점에서 엔티티의 수정이 발생하는지 추적하여 디버깅하는 상황에서도 더욱 유용하다.
* **컨트롤러에서 직접 엔티티를 생성하는 것보다는 파라미터 또는 DTO를 통해 서비스 계층에 식별자와 변경사항을 명확하게 전달**한다.
* 또한 **트랜잭션이 적용된 서비스 계층에서는 merge 기능보다는 영속 상태의 엔티티를 조회하고, 엔티티의 데이터를 직접 변경**하도록 한다.
  * 이를 통해 트랜잭션 커밋 시점에 변경 감지에 의해 수정 사항이 데이터베이스에 영속화된다.