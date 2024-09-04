# Spring Advanced
## 2024-09-04 Wed
### ThreadLocal이란?
```
> 컨버터나 포맷터는 모두 등록 방법만 준수한다면 ConversionService를 통해 쉽게 사용이 가능하지만, 메시지 컨버터에는 이러한 기능이 적용되지 않는다.
> ConversionService는 @RequestParam이나 @ModelAttribute, @PathVariable 또는 뷰 템플릿 등에서 사용된다.
> 반면, ResponseEntity와 HTTP 응답 본문 변환과 같이 HttpMessageConverter가 동작하는 부분에 대해서는 ConversionService가 동작하지 않는다.
```
* **`HttpMessageConverter`와 같은 메시지 컨버터의 역할은 HTTP 요청 또는 응답 본문을 객체로 변환**하는 데에 있다.
* 이를 위해 내부적으로는 `Jackson`과 같은 라이브러리를 사용하므로, 객체와 JSON 문서 간의 변환은 순수히 라이브러리의 동작에 달렸다.
  * 때문에 JSON 변환의 결과로 생성된 숫자 또는 날짜에 포맷팅을 적용하고자 하는 경우, 해당 라이브러리의 설정을 활용하여 이를 지정해주어야 한다.
  * 즉, **`HttpMessageConverter`와 같은 메시지 컨버터와 `ConversionService`의 동작에는 서로 아무런 연관성이 없음에 주의**해야 한다.