# Reactive
## 2023-11-26 Sun
### Sinks란?
```
> Sinks는 리액티브 스트림즈의 Signal을 프로그래밍적인 방식으로, 명시적으로 전송할 수 있도록 지원하는 요소에 해당한다. 
```
* 리액티브 스트림즈의 구성 요소 중 하나인 `Processor`는 `Publisher`와 `Subscriber`의 모든 특징을 갖는다.
* 그러나 최근 Reactor에서는 `Processor`의 기능을 개선한 `Sinks`가 지원되고 있으며, `Processor`와 관련된 API는 제거된다.
* 앞서 다룬 내용은 모두 Flux나 Mono가 `onNext() Signal` 등을 내부적으로 전송하는 방식에 해당한다.
  * 반면, **`Sinks`를 활용하는 것으로 `Signal`은 명시적으로 전송**될 수 있다.