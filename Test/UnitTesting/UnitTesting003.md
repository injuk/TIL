# UnitTesting
## 2022-05-07 Sat

## junit 단언
* junit에서 단언은 테스트에 작성할 수 있는 정적 메소드 호출이다.
* **각각의 단언은 어떠한 조건이 참인지 검증하며, 참이 아닌 경우에는 테스트가 멈추며 실패를 보고**한다.
  * 반면, junit이 테스트를 실행하며 예외가 발생했으나 잡지 않은 경우에 오류를 보고한다.
* junit은 다음과 같은 두 가지 단언 스타일을 제공한다.
  1. 전통적인 단언: junit의 원래 버전에 포함된다.
  2. 햄크레스트 단언: 새롭고, 표현력이 더 좋은 단언이다.
* 두 단언 스타일은 각각 다른 환경에서 다른 방식으로 제공된다.
* **두 단언 스타일을 섞어 사용할 수도 있으나, 일반적으로는 둘 중 하나를 선택하여 사용하는 것이 바람직**하다.

### assertTrue
```
> org.junit.Assert.assertTrue(boolean 표현식);
```
* 가장 기본적인 단언이며, 단언은 junit 테스트에서 광범위하게 사용되므로 일반적으로 다음과 같은 정적 임포트를 활용한다.
```
> import static org.junit.Assert.*;
```
* **테스트 메소드의 이름은 검증하려는 동작에 관한 일반적인 설명이며, 단언 역시 검증하려는 동작을 확인하는 방식으로 작성**되어야 한다.
* **테스트 코드는 특정한 사례에 해당하므로 검증하는 기대값 역시 명시하는 것이 바람직**하다.
  * 상술한 두 원칙의 예시는 depositIncreasesBalance 메소드로 들 수 있다.
  * 해당 메소드의 이름은 예금이 잔액을 증가시키는 것을 의미한다.
  * 때문에 단언 역시 assertTrue(account.getBalance() > 0)을 검증해야 한다.
  * 그러나 **테스트 코드는 특정한 사례에 해당하므로, 기대값인 0대신 명시적인 변수 이름인 initialBalance 등으로 표현하는 것이 바람직**하다.

### 햄크레스트 단언 - assertThat
* 대부분의 단언은 기대하는 값과 반환되는 실제 값을 비교하는 식으로 동작한다.
* **상술한 예금 시나리오를 예로 들어, 단지 잔고가 0보다 크다는 표현식을 작성하는 것보다 명시적으로 기대하는 잔고를 작성**할 수도 있다.
```
> assertThat(account.getBalance(), equalTo(100));
```
* assertThat은 햄크레스트 단언의 예시이며, 각 인자는 다음을 의미한다.
  1. 첫 번째 인자: **실제 표현식이며, 개발자가 검증하고자 하는 값을 의미**한다.
  2. 두 번째 인자: **matcher이며, 실제 값과 표현식의 결과를 비교하여 테스트의 가독성을 크게 높여준다**.
* matcher가 도입되는 햄크레스트 단언은 일반 문장처럼 왼쪽에서 오른쪽으로 읽을 수 있으므로 가독성을 높여준다.
* junit에서 제공하는 핵심적인 햄크레스트 matcher를 사용하려면 코드에 다음과 같은 정적 임포트를 추가해야 한다.
```
import static org.hamcrest.CoreMatchers.*;
```
* 예를 들어, **equalTo matcher는 어떠한 Java 인스턴스나 값을 작성해도 무방**하다.
  * equalTo matcher는 비교 기준으로 equals 메소드를 사용한다.
* **assertTrue와 같은 전통적인 단언보다 assertThat과 같은 햄크레스트 단언이 실패 시 오류 메시지로부터 더 많은 정보를 제공**한다.
* assertTrue 역시 표현식의 다음 인자로 matcher를 전달할 수 있으나, 이는 불필요한 정보를 장황하게 작성하게 된다.
  * **가독성을 떨어트리므로, 차라리 단순한 assertTrue를 사용하는 것이 바람직**하다.

### 중요한 햄크레스트 matcher
* junit에 포함되는 햄크레스트 CoreMatchers 클래스는 유용한 matcher 모음을 제공한다.
* 이 중 **몇 개만 사용해도 되지만, 더 많은 햄크레스트 matcher를 도입할수록 테스트 코드의 표현력은 깊어진다**.
* equalTo 메소드는 Java의 배열 또는 컬렉션 객체를 비교할 때에도 사용할 수 있으며, 예상한 대로 동작한다.
  * 이 경우, 비교하는 컬렉션이 일치한다면 단언은 통과한다.
* is 장식자는 아무 작업도 하지 않고 전달받은 matcher를 그대로 반환하지만, 다음과 같이 matcher 표현의 가독성을 높여준다.
  * **장식자의 사용 여부는 철저히 개인 취향이지만, 장식자를 빼고 equalTo matcher 만 사용하는 것이 바람직**하다.
```
assertThat(account.getName(), is(equalTo("ingnoh")));
```
* **not matcher는 다음과 같이 부정하는 단언**을 만들어줄 수 있다.
```
assertThat(account.getName(), not(equalTo("injuk")));
// is 장식자를 도입할 수도 있다.
// assertThat(account.getName(), is(not(equalTo("injuk"))));
```
* nullVale와 notNullValue matcher는 null이거나 그렇지 않은 경우를 검사하기 위해 사용된다.
  * null이 아님을 자주 검사하는 것은 설계 문제이거나, 너무 많이 걱정하는 경우에 해당한다.
  * **대부분의 경우, 상술한 이유에서 null이 아님을 체크하는 검사는 불필요하며 가치가 없다**.
```
assertThat(account.getName(), is(not(nullValue())));
assertThat(account.getName(), is(notNullValue()));
```
* 일반적으로 예외를 던지는 null 참조 예외는 테스트 오류가 발생하며, 테스트의 실패는 발생하지 않는다.
  * 이 경우, junit은 발생한 예외를 테스트 코드에서 잡지 않았다면 오류를 보고한다.

### 햄크레스트 matcher로 할 수 있는 것
* junit 햄크레스트 matcher를 활용하면 다음의 일을 수행할 수 있다.
  1. 객체의 타입을 검사
  2. 두 객체의 참조가 같은 인스턴스인지 검사
  3. 다수의 matcher를 결합하여 둘 다, 또는 둘 중 하나라도 성공하는지 검사
  4. 어떤 컬렉션이 요소를 포함하거나, 임의의 조건에 부합하는지 검사
  5. 어떤 컬렉션이 몇 개의 요소를 모두 포함하는지 검사
  6. 어떤 컬렉션에 있는 모든 요소가 matcher를 준수하는지 검사
* 상술한 작업 이외에도 훨씬 많은 작업을 수행할 수 있다.
* 햄크레스트 matcher를 파악하는 가장 좋은 방법은 직접 IDE에서 사용해보며 동작을 파악하는 것이다.
* 제공되는 matcher만으로 원하는 작업을 단언할 수 없는 경우, 도메인에 맞는 사용자 정의 matcher를 정의할 수도 있다.

### 부동소수점 수의 비교
* 컴퓨터는 모든 부동소수점 수를 표현할 수 없으며, Java에서 float과 double과 같은 어떠한 수들은 근사치로 구해야 한다.
* 이로 인해 단위 테스트에서는 두 부동소수점 수를 비교해도 항상 원하는 결과가 나오지 않을 수 있다.
  * 예를 들어, 다음의 테스트는 실패한다.
```
@Test
public void test() {
    assertThat(2.32 * 3, equalTo(6.96));
}
```
* 두 개의 float 또는 double을 비교할 때는 두 수 사이의 공차, 또는 허용 오차를 지정해주어야 한다.
  * 예를 들어, assertTrue를 활용하면 다음과 같이 표현된다.
```
@Test
public void test() {
//  assertThat(2.32 * 3, equalTo(6.96));
    assertTrue(Math.abs(2.32 * 3) - 6.96 < 0.0005);
}
```
* 그러나 **이는 가독성이 떨어지므로, 햄크레스트 matcher인 closeTo를 활용하여 두 부동소수점 수를 훨씬 수월하게 비교**할 수 있다.
  * **junit에 포함되는 햄크레스트 matcher는 더 큰 matcher 집합의 부분 집합**이다.
  * closeTo와 같은 유용한 matcher를 프로젝트에서 추가로 사용하려면 별도로 제공되는 원본 햄크레스트 matcher를 다운로드 하여 프로젝트에 추가해야 한다.
```
@Test
public void test() {
//  assertThat(2.32 * 3, equalTo(6.96));
//  assertTrue(Math.abs(2.32 * 3) - 6.96 < 0.0005);
    assertThat(2.32 * 3, closeTo(6.96, 0.0005));
}
```

### 단언의 설명
* 전통적인 fail과 햄크레스트 assertThat과 같은 모든 junit 단언의 형식 message라는 선택적인 첫 번째 인자가 존재한다.
  * message 인자는 마치 주석처럼 단언의 근거를 설명하도록 작성할 수 있다.
```
@Test
public void testWithMessage() {
    assertThat("is two", 1, equalTo(1));
    assertTrue("is two",1 == 1);
}
```
* 그러나 **잘 관리되지 않는 주석처럼 message 인자 역시 테스트를 정확히 설명하지 못할 수 있다**.
  * 구현 세부사항을 설명하는 주석은 코드와 일치하지 않는 것으로 악명이 높다.
* **설명 있는 주석문을 선호한다면 단언에 message를 작성할 수 있지만, 더 좋은 것은 테스트 코드 자체만으로 이해할 수 있도록 하는 것**이다.
* **단언에 message를 작성하는 것보다는 다음의 방식을 선택하여 테스트 코드의 가독성을 개선하는 것이 훨씬 바람직**하다.
  1. 테스트의 이름을 더 의미 있게 변경한다.
  2. 의미 있는 상수를 도입한다.
  3. 변수의 이름을 개선한다.
  4. 복잡한 초기화 작업은 의미 있는 이름을 갖는 도우미 메소드로 추출한다.
  5. 가독성이 우수한 햄크레스트 단언을 활용한다.
* 단언의 message는 테스트의 실패시 유용한 정보를 더 빠르게 알려줄 수 있으나, 군더더기 없는 코드를 만드는 것과는 트레이드오프 관게에 있다.

## 예외를 기대하기
* junit은 적어도 세 가지 다른 방식으로 기대한 예외를 던지는지 명시할 수 있다.

### 어노테이션 활용하기
* junit의 @Test 어노테이션은 기대한 예외를 지정할 수 있는 expected 인자를 제공한다.
  * 해당 방식은 junit에서 제공하는 단순한 방식이다.
  * 다음의 메소드에서, 명시된 예외가 발생하면 테스트는 통과한다.
  * 반면, **예외가 발생하지 않거나 다른 종류의 예외가 발생하면 테스트는 실패**한다.
```
@Test(expected = NullPointerException.class)
public void exceptionTest() {
    throw new NullPointerException();
}
```

### try - catch 방식
* **발생한 예외를 처리하는 오래된 방식인 try - catch를 활용하며, 예외가 발생하지 않은 경우에 fail을 통해 강제로 실패**시킨다.
* 아래의 예시에서, 예외가 발생한 경우 제어권은 catch 블록으로 넘어가 테스트가 종료된다.
  * 반면 예외가 발생하지 않는 경우, 제어권은 fail 문으로 넘어가 테스트는 실패한다.
  * 이러한 try - catch 블록은 희귀한 경우이며, 비어 있는 catch 블록을 허용한다.
  * 또한, **예외 변수를 expected로 명명하는 것으로 해당 예외가 예상된 것임을 강조**할 수 있다.
```
@Test
public void tryCatchTest() {
    try {
        int a = 1 / 0;
        fail();
    } catch (ArithmeticException expected) {
    }
}
```
* **오래된 방식은 예외가 발생한 경우의 assertThat 등의 단언을 통해 예외 메시지 등의 상태를 검사할 때 유용**하다.

### 새로운 방식 - ExpectedException 규칙
* junit은 커스텀 규칙을 정의하는 것으로 테스트가 실행되는 흐름 동안 발생할 수 있는 일에 대한 더 큰 통제권을 부여할 수 있다.
* 또한 junit의 규칙은 AOP와 유사한 기능을 제공하며, 자동으로 테스트 집합에 종단 관심사를 부착할 수 있게 한다.
* **junit은 몇몇 유용한 규칙을 제공하며, 특히 ExpectedException 규칙은 예외를 검사하는 데에 있어 상술한 두 방식의 장점만을 취합**한다.
  * 예를 들어 아래의 코드에서, thrown 규칙 인스턴스는 특정한 예외가 어떤 메시지와 함께 발생하는지 명시한다.
  * 규칙에 정의된 모든 기대 사항이 충족되면 테스트에 통과하며, 그렇지 않은 경우에는 실패한다.
```
private Account account;

@Before
public void create() {
    account = new Account("ingnoh");
}

@Rule
public ExpectedException thrown = ExpectedException.none();

@Test
public void ruleTest() {
    thrown.expect(InsufficientFundsException.class);
    thrown.expectMessage("balance only 0");

    account.withdraw(100);
}
```

### 예외 무시하기
* 개발자가 작성하는 대부분의 테스트는 행복한 경로만을 다루는 테스트일 가능성이 높다.
  * 즉, 예외가 거의 발생하지 않는 상황을 다루곤 한다.
* 그러나 Java는 checked exception을 반드시 처리하도록 강제한다.
  * 예를 들어, 아래의 코드는 IOException을 적절히 처리하기 전까지는 컴파일할 수 없다.
```
@Test
public void ioTest() {
    String fileName = "test.txt";
    BufferedWriter writer = new BufferedWriter(new FileWriter(fileName));
    writer.write("test!!!");
    writer.close();
    
    assertTrue(true);
}
```
* **이러한 상황을 처리하기 위해 테스트 코드에 무의미한 try - catch 블록을 넣는 것보다는 발생하는 예외를 다시 던지는 것이 바람직**하다.
```
// throws를 활용하여 예외를 다시 던져주면 컴파일이 가능하다.
@Test
public void ioTest() throws IOException { 
    String fileName = "test.txt";
    BufferedWriter writer = new BufferedWriter(new FileWriter(fileName));
    writer.write("test!!!");
    writer.close();

    assertTrue(true);
}
```
* 개발자는 긍정적인 테스트를 설계한 경우, 정말 예외적인 상황을 제외하고는 IOException이 발생하지 않음을 알 수 있으므로 이러한 상황을 걱정할 필요가 없다.
  * 아주 드물게 **예상치 못한 예외가 발생하더라도 junit은 개발자 대신 이를 처리**해줄 수 있다.
  * 이 경우, **junit은 예외를 잡아 테스트 실패가 아닌 테스트 오류로 보고**해준다.