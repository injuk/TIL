# CleanCode
## 2022-01-28 Fri

## 함수
* 어떤 프로그램이든지 간에, 가장 기본적인 단위는 함수가 된다.
* 함수는 읽고 이해하기 쉬워야 하며, 의도를 분명히 표현해야 한다.

### 작게 만들기
* 함수를 만드는 첫번째 규칙은 '더 작게 만들기'이다.
* 함수는 100줄을 넘어서면 안되며, 20줄도 길다.
* if / else / while 문 등에 들어가는 블록은 한 줄이 이상적이다.
  * 대개 이 한 줄은 다른 함수를 호출하는 문장이다.
  * 이를 통해 외부 함수가 짧아질 뿐 아니라, 블록 내부에서 호출하는 함수의 이름이 명백하다면 이해하기도 쉬워진다.
* **함수는 중첩 구조가 생길 만큼 커져서는 안 된다**.
  * 함수의 들여쓰기 수준은 가독성 향상을 위해서라도 1단, 많아도 2단을 넘기면 안 된다.

### 하나만 하기
* 여러 가지를 하느라 바쁜 함수는 이해하기 어렵다.
```
> 함수는 하나만 해야 한다.
> 함수는 하나만 잘 해야 한다.
```
* 명백한 이름을 갖는 함수 내부에서 추상화 수준이 하나인 단계만 수행한다면, 함수는 한 가지 작업만 한다고 볼 수 있다.
  * 즉, 추상화 수준을 더 이상 나눌 수 없는 함수는 하나의 일만 하는 함수라고 생각할 수 있다.
* 단순한 동의어를 사용하는 경우가 아닌, **의미 있는 이름으로 다른 함수를 추출할 수 있다면 여러 작업을 하는 함수**이다.

### 추상화 수준을 하나로 만들기
* 함수가 **하나의 작업만 하려면 함수 내에 포함된 모든 문장의 추상화 수준이 동일**해야 한다.
* 함수에 여러 추상화 수준이 혼합되어 있으면 함수를 이해하기 어려워진다.
  * 이는 함수에 포함된 표현 각각이 근본적인 개념을 말하는지, 세부사항을 표현하는지 알기 어렵기 때문이다.
  * 또한 추상화 수준이 혼합된 함수를 오래 방치할 경우, 깨진 유리창 이론에 따라 다른 개발자들이 죄책감 없이 세부사항을 추가하기 쉽다.
* 코드는 위에서 아래로 이야기처럼 읽혀야 좋다.
  * 이를 위해서는 함수의 다음에 추상화 수준이 한 단계 낮은 함수가 오고, 이러한 연쇄가 반복되어야 한다.
  * 즉, 위에서 아래로 읽히는 코드는 함수의 추상화 수준이 한 번에 한 단계씩 낮아진다.
* 추상화 수준이 하나인 함수를 구현하는 것은 쉽지 않지만, 매우 중요한 규칙이다.
  * 한 가지만 하는 함수는 코드를 읽어 내려가듯 구현하면 추상화 수준을 일관되게 유지할 수 있다.
  * **각 함수는 다음 함수를 소개**하고, 일정한 추상화 수준을 유지한다.

### Switch 문
* switch 문은 태생적으로 작게 만들기 어렵고, 본질적으로 두 개 이상의 작업을 처리한다.
  * 일반적으로 선호되는 단일 블록, 단일 함수를 만들기 어렵다.
* switch 문을 사용하면,
  1. 함수가 길어진다.
  2. 함수가 하나의 작업만 처리하지 않는다.
  3. SRP를 위배하게 된다.
  4. OCP를 위배하게 된다.
  5. 이러한 함수 구조가 다른 함수에서도 발견될 수 있다.
* **Java의 경우, 추상 팩토리 속에 switch 문을 숨기고 적절한 클래스의 인스턴스를 반환하는 것으로 해결**할 수 있다.
  * switch 문을 통해 다양한 함수를 호출하던 함수는 추상 팩토리 속으로 switch 문을 옮기고, 케이스 별 함수 호출이 아닌 적절한 인스턴스를 반환받아 함수를 호출하는 식으로 수정되어야 한다.
  * 즉, 다형성을 활용하여 해결한다.

### 서술적인 이름 사용하기
* 함수가 하는 일을 명확하게 표현할 수 있는 서술적인 이름으로 명명한다.
```
> 코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라고 부를 수 있다.
```
* 하나의 작업만 처리하는 함수에 서술적인 이름을 붙인다면 상술한 원칙을 달성하는 데에 있어 절반은 성공한 셈이다.
* **함수의 이름이 길어도 상관 없다**.
  * **길고 서술적인 이름은 언제나 길고 서술적인 주석보다 좋다**.
  * **이름을 정하느라 긴 시간을 보내도 좋다**.
  * **이런 저런 단어를 사용하면서 코드를 읽어보고, 최대한 서술적인 이름을 고를 수 있다면 더 좋다**.
* 서술적인 이름은 개발자의 설계를 더 뚜렷하게 하므로, 자연스럽게 코드를 개선하기도 쉬워진다.
* 서술적인 이름의 명명은 일관성이 있어야 한다.
  * **같은 모듈에서의 함수 이름은 같은 문구, 명사, 동사를 사용**한다.

### 함수의 인수
* **함수에서 이상적인 인수의 개수는 0개**이다.
  * 인수가 1개씩 늘어갈 때마다 이상은 멀어진다.
  * 최선은 0개이며, 차선은 1개이다.
* 1개의 인수
  1. 인수에 질문을 던지는 경우: boolean isExists(File)
  2. 인수를 변환하여 반환하는 경우: InputStream open(File)
  3. 이벤트 함수의 경우: passwordAttemptFailedNTimes(int attempts)
  * 위 경우를 제외하고는 1개의 인수는 가급적 지양하도록 한다.
* 플래그 인수
  * 플래그 인수는 그 자체로 함수가 여러 작업을 수행함을 나타내므로 좋지 않다.
  * boolean 플래그에 따라 **서로 다른 두 작업을 수행하는 함수라면, 서로 다른 두 작업을 별도의 두 개의 함수로 나누어야** 한다.
* 2개의 인수
  * 2개의 인수를 갖는 함수는 1개의 인수를 갖는 함수보다 이해하기 어렵다.
  * new Point(0, 0)과 같이 자연적인 순서가 있는 코드의 경우에는 괜찮지만, 두 **인수 사이의 순서를 인위적으로 기억해야 하는 경우는 문제**가 있다.
  * 2개의 인수를 갖는 함수가 반드시 나쁘고, 사라져야만 한다는 의미는 아니다.
    * 다만 어쩔 수 없는 경우에는 사용하더라도 위험이 수반된다는 점을 이해하고, 가능하다면 1개의 인수를 갖는 함수로 바꿀 수 있도록 노력해야 한다.
* 3개의 인수
  * 인수가 3개인 함수는 2개인 함수보다 이해하기 훨씬 어렵다.
* 인수 객체
  * 반드시 인수가 2개 이상 필요한 경우라면 별도의 클래스로 선언하는 것을 고려한다.
  * 객체를 통해 인수를 줄이는 방법이 눈속임이라고 여길 수 있지만, 그렇지 않다.
    * **변수를 클래스로 묶는 행위 자체에 이름을 붙이게 되므로, 결국은 개념을 표현하게 되기 때문**이다.
* 인수 목록
  * 때로는 가변 인수를 사용하는 함수도 필요하다.
  * 가변 인수를 취하는 함수는 가변 인수 전부를 동등하게 취급하고, 그 형태에 따라 1개, 2개, 3개의 인수를 사용하는 함수로 해석할 수 있다.
    * 예를 들어, String format(String format, Object... args)는 2개의 인수를 사용하는 함수로 해석할 수 있다.
    * 이 경우에도 4개 이상의 인수를 사용하는 함수는 적절하지 못하다.
* **함수의 의도, 또는 인수의 순서와 의도를 제대로 표현할 수 있는 좋은 함수 이름은 필수적**이다.
  * 1개의 인수를 갖는 함수는 함수와 인수가 동사 / 명사 쌍을 이루어야 한다.
    * writeField(name): name은 필드이며, 이 함수는 name을 설정한다는 의도가 명확하다.
  * **2개의 인수를 갖는 함수는 키워드를 추가하여 함수 이름에 인수 이름을 넣어줄 수 있다**.
    * assertExpectedEqualsActual(expected, actual): 인수의 순서를 기억할 필요가 없어진다.

### 부수효과 지양하기
* 부수효과는 하나의 일만 하기로 한 함수가 몰래 다른 일도 수행하는, 일종의 거짓말이다.
* 대부분의 경우 부수효과는 시간적 결합, 또는 순서 종속성을 초래한다.
  * 시간적 결합: 함수가 특정한 상황과 결합되어 재사용성을 잃는 경우
    * 시간적 결합이 정말 필요한 함수는 함수 이름에서 명시되어야 한다.

### 명령과 조회를 분리하기
* 함수는 뭔가를 수행하거나(명령) 답할 수만(조회) 있어야 한다.
  * 둘 중 하나만 해야하며, 둘다하는 것은 바람직하지 못하다.
* 명령과 조회가 혼합된 함수는 함수의 역할을 판단하기 힘들다.
  * 예시: 인수로 전달받은 값을 선언하고, 성공 여부를 반환하는 경우.

### 오류 코드보다는 예외를 사용하기
* 함수의 실행 결과를 정해진 오류코드로 반환한다면, 외부 함수에서는 오류 처리를 위한 코드를 필수적으로 작성해야 한다.
* 대신 예외를 사용하면 오류 처리 코드가 try - catch 블록으로 분리되므로 깔끔하다.
  * try - catch 블록은 코드 구조에 혼란을 주며, 정상 동작과 오류 처리 동작을 섞는다.
  * try - catch 블록은 이렇듯 추하므로, 별도의 메소드로 뽑아내는 것이 좋다.
```
public void delete(Page page) {
  try {
    // 분리된 try - catch 블록
    deletePageAndAllReferences(page);
  } catch (Exception e) {
    log.error(e);
  }
}
public void deletePageAndAllReferences(Page page) throws Exception {
  // 실제 동작
}
```
* 오류 처리 코드(try - catch)와 정상 동작 코드가 별도의 메소드로 분리된 코드는 읽기 쉽고, 이해하기 쉽다.
  * 오류 처리 코드는 delete 메소드에서, 정상 동작은 deletePageAndAllReferences 메소드에서 수행한다.
* **오류 처리 작업도 하나의 작업이다. 함수는 하나의 작업만** 해야 한다!
* 오류 코드를 반환하는 것은 클래스, 또는 enum에 오류 코드들이 정의되어 있다는 의미이다.
  * 이러한 클래스는 **의존성 자석**이다.
* 의존성 자석: 다른 클래스에서 의존성 자석 클래스를 import했을 때, 의존성 자석 클래스가 수정되면 이를 사용하는 모든 클래스도 재컴파일해야 한다.
  * 프로그래머는 번거로운 재컴파일을 원하지 않으므로, 오류 코드를 추가하지 않고 기존 코드를 재사용하곤 한다.
* **오류 코드 대신 예외를 사용하면, 새로 추가된 예외도 Exception 클래스를 상속받으므로 재컴파일이 필요 없다**.

### 반복을 피하기
* 여러 곳에서 반복되는 알고리즘의 사용은 잠재적인 문제를 포함한다.
  1. 코드의 길이가 늘어난다.
  2. 알고리즘이 변하면, 이 알고리즘을 사용하는 다른 모든 코드도 수정해야 한다.
  3. 코드를 수정하는 과정에서, 수정 작업을 깜빡하여 오류가 발생할 확률이 높다.
* **중복은 소프트웨어에서 발생하는 모든 악의 근원**이다.
* 소프트웨어 개발에서 여지껏 있어 왔던 혁신은 모두 중복을 제거하려는 지속적인 노력이기도 하다.

### 구조적 프로그래밍
* 다익스트라의 구조적 프로그래밍은 모든 함수와, 함수 내의 블록에 입구와 출구는 반드시 하나만 존재해야한다고 주장한다.
  * 출구를 여러개 만드는 break, continue, goto의 사용을 지양하며 단 하나의 return 문만을 허용한다.
* 그러나 **함수가 작아질수록 구조적 프로그래밍은 큰 이점이 되지 못한다**.
* **함수가 충분히 작다면 여러 개의 return, break, continue를 사용해도 무방**하다.
  * 때로는 오히려 함수의 의도를 표현하기에 더 쉬워진다.

### 함수를 작성하는 방법
* 소프트웨어 작성은 글짓기와 유사하다.
1. 먼저 생각을 기록하고, 읽기 좋게 다듬는다.
2. 초안은 대개 미숙하므로, 원하는대로 읽힐 때까지 말을 다듬고, 문장을 고치고, 문단을 정리한다.
* 함수 작성 역시 마찬가지이다.
1. 길고 복잡한 함수를 만든다. 들여쓰기와 중복 루프, 긴 인수 목록을 포함할 수 있다.
2. 이 코드를 빠짐없이 테스트하는 단위 테스트 케이스를 작성한다.
3. 코드를 다듬고, 함수를 따로 빼고, 이름을 바꾸고, 중복을 제거하는 과정에서 끊임없이 메소드와 클래스를 쪼갠다.
4. 이러한 과정에서도 끊임없이 단위 테스트를 통과한다.
* 이를 통해 이상적인 함수에 가까워질 수 있다. 한 번에 이상적인 함수를 만들 수 있는 사람은 없다!

### 결론
* 모든 시스템은 DSL로 만들어지며, 함수는 동사이고 클래스는 명사의 역할을 수행한다.
* 숙련된 프로그래머는 시스템을 구현해야만 하는 프로그램이 아니라, 풀어나가야 할 이야기로 생각한다.
  * 프로그래밍의 기술은 언제나 언어 설계의 기술이다.
* 상술한 원칙들은 함수를 잘 만드는 원칙이며, 이를 따른다면 길이가 짧고, 직관적인 이름을 갖는 체계적인 함수가 작성된다.
  * 그러나 이러한 **함수 역시 시스템의 이야기를 풀어나가기 위한 도구인 점을 명확하게 이해**해야 한다.
  * 분명하고 정확히 표현된 함수는 쉽게 이야기를 풀어나갈 수 있도록 만든다.
* 여럿으로 쪼개어진 함수는 아래와 같이 작성하고, 위에서부터 읽히는 순서로 작성하는 것이 좋다.
```
(() => {
    func1();
    func2();
    func3();
    func4();
})();
// func1이 가장 먼저 읽히므로, 가장 위에 작성한다.
function func1() {
    func5();
}
// func1을 추적하면 func5를 우선 호출하게 되므로, func5가 func2보다 먼저 작성된다.
function func5() {
};
function func2() {
    func6();
    func7();
}
function func6() {
    func8();
}
function func8() {
}
function func7() {
}
function func3() {
}
function func4() {   
}
```