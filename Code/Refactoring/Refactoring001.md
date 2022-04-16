# Refactoring
## 2022-04-11 Mon

## 리팩토링
* 간단한 프로그램은 사실 굳이 리팩토링 원칙 모두를 적용할 필요가 없다.
* 그러나 코드가 대규모 시스템의 일부라면 리팩토링의 적용 여부는 차이가 매우 크므로, 반드시 리팩토링하는 것이 바람직하다.
* 컴파일러는 코드가 깔끔한지 신경쓰지 않는데, 코드를 미적인 기준으로 볼 필요가 있을까?
  1. 그러나 코드의 수정에는 언제나 사람인 개발자가 투입되며, 사람은 코드의 미적 상태에 민감하다.
  2. 설계가 나쁜 시스템은 수정이 어렵다.
  3. 수정하기 어려운 코드는 실수를 통해 버그를 발생시킬 확률이 높다.
* 때문에 **수백 줄짜리 코드를 수정하려면 우선 프로그램의 작동 방식을 쉽게 이해할 수 있도록 여러 함수와 프로그래밍 요소로 재구성하는 것이 바람직**하다.
  * 프로그램의 구조가 빈약한 경우, 우선 구조를 바로잡은 후에 기능을 수정하는 것이 작업하기 더 수월하다.
```
> 프로그램이 새로운 기능을 추가하기 편한 구조가 아니라면, 우선 기능을 추가하기 쉬운 형태로 리팩토링한 후에 원하는 기능을 추가한다.
```

### 리팩토링의 필요성
* **리팩토링이 필요한 이유는 다가올 변경에 쉽게 대응하기 위해서**이다.
  * 어떤 방식으로 구현하든 반드시 반 년 내에 변경사항이 발생하며, 새로운 요구사항은 한 웅큼씩 몰려오기 마련이다.
* 잘 동작하고 변경할 일이 '절대로' 없다면 코드를 방치해도 좋지만, **누군가가 읽고 이해할 일이 필요한데도 로직을 파악하기 어렵다면 대책을 마련해야 한다**.

### 리팩토링의 첫 단계
* **리팩토링의 첫 단계는 언제나 테스트 코드의 작성으로 똑같다**.
  * 리팩토링 대상 코드 영역을 꼼꼼히 검사해줄 테스트 코드를 우선 작성한다.
  * 리팩토링 기법들이 정립이 되어있다고 한들, 실제 작업은 사람인 개발자가 수행하므로 테스트의 역할은 매우 중요하다.
* **테스트는 반드시 성공과 실패를 스스로 판단하는 자가진단 테스트**여야 한다.
  * 자가진단을 지원하지 않는다면 매 테스트 케이스의 성공시 값을 기억하고 비교 대조해야 한다.
* **리팩토링은 테스트에 크게 의지하며, 테스트를 작성하는 시간이 좀 걸리더라도 디버깅 시간이 줄어들기에 전체적인 작업 시간은 단축**된다.

### 긴 함수 쪼개기
* **긴 함수를 리팩토링하는 과정에서, 우선 전체적인 동작을 각각의 부분으로 나눌 수 있는 지점을 찾는다**.
  * switch 또는 if 분기는 쪼개기 좋은 지점의 예시이다.
* 긴 함수를 읽고 분석한 후, 나누기 적절한 지점을 찾았다면 이를 빠르게 코드에 반영한다.
  * **나누기 적절한 지점을 찾은 이유는 내 머릿속에서 금방 잊혀질테지만, 코드에 반영해둔 정보는 이후에 코드를 읽을 때 그 이유를 설명**해줄 수 있다.
* 나눠야 할 코드 조각은 별도의 함수로 추출하는 방식으로 파악한 정보를 코드에 반영할 수 있으며, 이 때 적절한 함수의 이름을 명명한다.
* 별도의 함수로 추출한 코드 조각에서 유효 범위를 벗어나는 변수를 찾는다.
  * 예를 들어, 나눌 코드 조각을 그대로 함수로 빼냈을 때 해당 함수에서 접근할 수 없는 변수를 말한다.
  * 이 때, **접근할 수 없는 변수를 추출된 함수 내에서 수정하지 않는다면 매개 변수로 전달**한다.
  * 반면, **변수를 수정해야 하는 경우라면 함수의 반환 값으로 사용**한다.
* 수정한 후에는 반드시 테스트 코드를 통해 테스트하여 실수를 확인한다.
  * **아무리 간단한 수정이라도 리팩토링 직후에 테스트하는 습관을 들이는 것이 바람직**하다.
  * **조금씩 변경하고 매 번 테스트하는 것은 리팩토링의 핵심 절차**이다.
* 함수 추출하기 기법인 IDE 차원에서 자동으로 지원해주며, 습관적으로 눌러주는 것이 좋을 수 있다.
```
> 리팩토링은 프로그램의 수정을 작은 단계로 나누어 진행하므로, 중간에 실수하더라도 버그와 원인을 쉽게 찾아낼 수 있다.
```
* 상황이 복잡한 경우라면, 여러 리팩토링 기법을 도입할 수 있도록 단계를 작게 나누는 것에서부터 시작한다.
  * 이를 통해 리팩토링 중간에 테스트에 실패하고 원인을 즉시 파악할 수 없는 경우에 최근 커밋으로 돌아가 단계를 더 작게 나누어 리팩토링을 재시도할 수 있다.
  * **일반적으로 코드가 복잡할수록 단계를 작게 나누면 작업 속도가 빨라지곤 한다**.

### 리팩토링과 커밋
* **수정 사항을 테스트하고 난 후 문제가 없음이 확인되었다면, 변경 사항을 로컬 버전 관리 시스템에 커밋**한다.
* 하나의 리팩토링 작업을 문제 없이 완료할 때마다 커밋하는 것으로 중간에 문제가 발생하더라도 이전의 정상적인 상태로 쉽게 돌아갈 수 있도록 한다.
* 자잘한 수정사항들이 리팩토링을 통해 어느 정도 의미 있는 단위로 쌓인 후에는 공유 저장소로 푸시하도록 한다.

### 추출된 함수도 검토하기
* **긴 함수로부터 코드 조각을 별도의 함수로 추출했다면, 추출된 함수에서도 더 명확하게 표현할 수 있는 부분이 있는지 확인하는 것이 바람직**하다.
  * 대표적으로는 **의미가 불분명한 변수 이름 등**을 명확하게 바꾸어줄 수 있다.
  * 예를 들어, 함수의 반환 값에는 항상 result라는 변수 명을 사용할 수 있다.
* 이렇듯 **간단한 변수 이름의 변경에도 반드시 테스트와 커밋 과정을 거치도록 한다**.
* **좋은 코드라면 본디 하는 일이 명확히 드러나야 하며, 이 과정에서 변수 이름은 매우 큰 역할을 하므로 이름을 변경할 만한 가치는 차고 넘친다**.
  * 찾아 바꾸기 기능을 사용하면 어렵지도 않으므로, 변수 이름은 망설임 없이 더 명확하게 바꾸어주도록 한다.
```
> 컴퓨터가 이해하는 코드는 바보라도 작성할 수 있다.
> 사람이 이해하도록 작성하는 프로그래머가 진정한 실력자다.
```

### 임시 변수는 질의 함수로 바꾸기
* 다음과 같은 코드에서, key는 애초에 sizeFor 함수의 매개변수로 전달할 필요가 없다.
  * 이는 sizeFor 내부에서 계산해도 무방하며, **이러한 변수들은 로컬 범위에 존재하는 이름의 개수를 늘려 작업을 복잡하게 하므로 제거되어야 한다**.
```
for(const object of objects) {
  const key = object.key;
  const size = sizeFor(object, key);
  //.. 생략
}
```
* 이러한 코드를 다음과 같이 object.key를 반환하는 질의 함수를 추가하여 리팩토링할 수 있다.
  * 이로 인해 object 변수를 조회하는 횟수가 늘게 되더라도, 성능에 큰 영향을 주지 않을 확률이 높다.
  * 만약 **해당 리팩토링으로 인해 성능이 처참히 느려졌더라도 제대로 리팩토링된 코드는 그렇지 않은 코드보다 성능을 개선하기가 훨씬 쉽다**.
* **임시 지역 변수를 제거하여 얻는 가장 큰 이점은 유효범위를 신경 써야 할 대상이 줄어들기 때문에 추출 작업이 쉬워진다는 점**이다.
  * 따라서, 제거할 수 있는 지역 변수는 최대한 제거하는 것이 바람직하다.
```
function keyFor(object) {
  return object.key;
}
```
* 리팩토링 후에는 다시 테스트와 커밋 과정을 거치도록 한다.

### 변수 인라인하기
* **임의의 함수가 변수의 값을 설정하기 위해 사용되지만, 해당 변수의 값이 다시 바뀌지 않는다면 변수 인라인하기 기법을 적용**하도록 한다.
  * 기준은 단순히 함수에 의해 반환된 값이 변경될 가능성이 있는지이다.
  * 따라서, 값이 여러 번 호출되더라도 수정되지 않는다면 함수의 반환 값을 변수에 저장하는 것보다는 함수를 여러 번 호출하는 방식을 선택하도록 한다.
```
for(const object of objects) {
  // const size = sizeFor(object);
  // console.log(size);
  console.log(sizeFor(object); // 어차피 수정되지 않을 값이므로 변수에 저장할 필요가 없다.
  //.. 생략
}
```

### 임시 변수 제거하기
* 임시 변수는 일반적으로 자신이 속한 루틴에만 의미가 있으므로 루틴을 불필요하게 길고 복잡하게 만들기 때문에 문제가 되기 쉽다.
  * 임시 변수의 예시로는 일반 변수에 대입된 함수 등이 있으며, 잠재적인 문제의 발생 가능성을 줄이기 위해 최대한 제거해주는 것이 좋다.

### 함수 선언 바꾸기
* **함수의 이름이 기능을 충분히 해결할 수 있을 정도로 명확하지 않은 경우, 적절한 이름을 갖는 포워딩 함수를 작성하여 문제를 해결**할 수 있다.
  * **아래의 함수는 시그니쳐만으로는 무슨 기능을 수행하는지 전혀 알 수 없으므로, 적절한 이름을 가진 포워딩 함수로 래핑하도록 리팩토링**한다.
```
new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', minimumFractionDigits: 2}).format(number / 100);
```
* **명명은 중요하지만 쉽지 않은 작업이며, 특히 긴 함수를 작게 쪼개는 리팩토링은 이름을 잘 지었을 때에만 효과가 있다**.
  * 따라서 **처음에는 당장 떠오르는 최선의 이름으로 사용하되, 나중에 떠오를 때 더 좋은 이름으로 바꾸는 방식으로 작업하는 것이 효율적**이다.

### 반복문 쪼개기
* **반복문 내부의 로직이 복잡한 경우, 차라리 반복문의 내용을 나누어 새로운 반복을 돌리는 것이 좋은 경우도 있다**.
* **반복문을 쪼개는 것으로 성능이 느려질 것을 우려할 수도 있으나, 작은 반복문을 따로 빼는 것은 성능에 미치는 영향이 미미한 경우가 많다**.
  * 경험 많은 개발자들 또한 실제 성능을 정확히 예측할 수 없으며, 컴파일러들은 최신 캐싱 기법으로 무장하여 우리의 직관을 초월하곤 한다.
  * **소프트웨어의 성능은 대체로 코드의 몇몇 작은 부분에 의해 결정되므로, 그 외의 부분은 수정한다고 해도 성능의 차이를 체감할 수 없다**.

### 리팩토링으로 인한 성능 저하
* 물론 **때로는 리팩토링이 실제로 성능에 악영향을 주는 경우도 있지만, 그래도 개의치 않고 리팩토링을 수행**하도록 한다.
  * 잘 다듬어진 코드라야 리팩토링 후의 개선 작업도 수월하게 진행이 가능하기 때문이다.
* **리팩토링 과정에서 성능이 크게 떨어졌다면, 리팩토링 이후에 시간을 내어 성능을 개선하면 그만**이다.
  * 리팩토링을 롤백하는 경우도 있어 일을 두 번 하는 것으로 생각할 수도 있지만, 대체로 리팩토링 덕분에 성능 개선은 더욱 효과적으로 수행할 수 있다.
  * 그 결과, 더 깔끔하면서 더 빠른 코드를 얻을 수 있게 된다.
```
> 리팩토링으로 인해 발생한 성능 문제는 특별한 경우가 아니라면 일단 무시하도록 한다.
> 리팩토링으로 인해 발생한 성능 문제는 우선 지금 하던 리팩토링 작업을 마무리한 후에 성능을 개선하도록 한다.
```

## 2022-04-12 Tue
### 리팩토링 중간 점검하기
* 리팩토링 과정에서, 데이터는 가능하다면 불변으로 취급한다.
* **리팩토링 과정에서, 새로운 함수를 많이 만들게 되어 코드의 길이는 길어질수도 있다**.
  * 추가된 코드 덕분에 전체 로직을 구성하는 요소는 각각이 뚜렷하게 부각되고, 기능 별로 모듈화할 수 있다.
  * 모듈화된 코드는 각 부분이 하는 일과 코드들이 맞물려 돌아가는 과정을 파악하기 쉬워진다.
  * 이러한 **모듈화를 통해 얻어내는 명료함은 진화 가능한 소프트웨어의 정수**이기도 하다.
* **리팩토링을 적용하는 과정에서, 항상 리팩토링과 기능 추가 사이에서 균형을 맞추어야 한다**.
  * **균형점을 판단하기 어렵다면 보이스카웃 규칙을 적용**하여 언제나 코드가 점점 나아지도록 만들 수 있다.
```
> 캠핑장은 언제나 도착했을 때보다 깔끔하게 정돈하고 떠나야 한다.
```

### 다형성을 활용하여 코드 재구성하기
* 여러 **조건에 따른 조건부 분기 로직이 포함되는 코드는 수정에 수정을 거듭함에 따라 골칫거리로 전락하기 쉽다**.
  * 이를 방지하려면 프로그래밍 언어가 제공하는 구조적인 요소로 보완해야 한다.
* 조건부 로직을 명확한 구조로 보완하는 방법은 다양하며, 대표적인 예시로 객체지향의 핵심인 다형성을 활용하는 방법이 있다.
  * 자바스크립트의 경우, 커뮤니티에서 전통적인 객체지향의 지원은 오랜 시간 동안 논란거리였으나 ES6부터 이를 지원하는 문법과 구조가 도입되었다.
  * 따라서, **용도에 맞는 경우라면 이렇듯 언어 차원에서 제공되는 기능을 적극적으로 활용하는 것이 바람직**하다.
* **같은 타입의 다형성을 기반으로 실행되는 함수가 많을수록 서브클래스를 활용하여 구성하는 것이 유리**하다.

### 리팩토링 II
* 리팩토링은 대부분 코드가 하는 일을 파악하는 데에서 시작한다.
  * **코드를 읽고, 개선점을 찾고, 리팩토링을 통해 개선점을 코드에 반영할 수 있어야 한다**.
  * 이를 통해 **코드는 점점 명확해지고 이해하기 더 쉬워지며, 리팩토링 작업 도중 또 다른 개선점이 떠올라 선순환이 형성**된다.
* **좋은 코드를 판가름하는 확실한 방법은 얼마나 수정사항을 반영하기 쉬운가**이다.
  * 코드는 미적인 관점으로 접근하면 좋고 나쁨이 명확하지 않아 개인 취향 말고는 어떠한 지침도 적용할 수 없게 된다.
  * 때문에 코드의 수정하기 쉬운 정도야말로 좋은 코드를 가늠하는 확실한 방법이다.
* 코드는 명확해야하며, 수정할 상황이 닥치면 수정할 곳을 쉽고 오류 없이 빠르게 수정할 수 있어야 한다.
* 건강한 코드는 생산성을 높이고, 필요한 기능을 더 빠르고 저렴한 비용으로 제공할 수 있도록 한다.
  * **코드를 건강하게 관리하기 위해서 개발 팀의 이상에 가까워지도록 끊임없이 리팩토링해야 한다**.

### 리팩토링의 리듬
* 리팩토링에서 중요한 것은 리팩토링하는 리듬을 깨닫는 것이다.
* 리팩토링 과정에서 **각 단계는 굉장히 작게 나누어지고, 매 번 컴파일과 테스트를 통해 동작하는 상태를 유지할 수 있어야 한다**.
* 효과적인 리팩토링의 핵심은,
  1. 단계를 작게 나누어야 빠르게 처리할 수 있고,
  2. 코드는 절대 깨지지 않으며,
  3. 작게 나누어진 여러 단계들이 모여서 큰 변화를 이룰 수 있다
* 는 **사실을 몸소 깨닫는 것**이다.