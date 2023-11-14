# Reactive
## 2023-11-15 Wed
### Subscriber 개념 바로잡기
* **`Subscriber`는 정확히 말해 데이터를 처리하는 쪽만이 아닌, 모든 `Downstream Publisher`를 포함하는 개념**이다.
  * 이렇듯 `Upstream Publisher`로부터 데이터를 전달받아 처리하는 모든 `Publisher`는 `Downstream Consumer`라고도 지칭한다.
* **리액티브 프로그래밍의 경우, 원본 Flux에서 `Emit`된 모든 데이터는 각 `Operator`를 거칠 때마다 다시 Flux 또는 Mono로 반환**된다.
  * 각 **`Operator`가 반환하는 반환 값이 또 다른 `Publisher`인 Flux 또는 Mono이므로, `Subscriber`는 일부 `Publisher`를 포함**할 수 있다.