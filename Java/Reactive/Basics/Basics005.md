# Reactive
## 2023-11-15 Wed
### Subscriber 개념 바로잡기
* **`Subscriber`는 정확히 말해 데이터를 처리하는 쪽만이 아닌, 모든 `Downstream Publisher`를 포함하는 개념**이다.
  * 이렇듯 `Upstream Publisher`로부터 데이터를 전달받아 처리하는 모든 `Publisher`는 `Downstream Consumer`라고도 지칭한다.
* **리액티브 프로그래밍의 경우, 원본 Flux에서 `Emit`된 모든 데이터는 각 `Operator`를 거칠 때마다 다시 Flux 또는 Mono로 반환**된다.
  * 각 **`Operator`가 반환하는 반환 값이 또 다른 `Publisher`인 Flux 또는 Mono이므로, `Subscriber`는 일부 `Publisher`를 포함**할 수 있다.

## 2023-11-16 Thu
### Backpressure란?
```
> Backpressure는 Publisher가 끊임없이 Emit하는 수많은 데이터를 적절히 제어하여 데이터 처리에 과부하를 주지 않는 것을 의미한다. 
```
* 예를 들어 `Publisher`가 데이터를 `Emit`하는 속도가 `Subscriber`의 처리 속도보다 너무 빠른 경우, 작업의 효율성은 떨어질 수 있다.
* **`Backpressure`는 이렇듯 `Publisher`가 끊임없이 `Emit`하는 데이터를 적절히 제어하기 위해 사용되는 개념**이다.
  * 이러한 **배압이 없을 경우 `Publisher`가 계속해서 데이터를 `Emit`하는 것으로 오버플로우가 발생하거나, 최악의 경우에는 시스템 다운**될 수 있다.