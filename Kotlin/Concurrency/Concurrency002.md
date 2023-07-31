# Concurrency
## 2023-07-31 Mon
### 그래들로 코루틴 활용하기
* 그래들의 경우, 코루틴을 활용하기 위해 관련된 의존성을 다음과 같이 추가해야 한다.
```
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:$version")
```
* 코루틴은 1.3 버전 이전에는 실험적인 기능으로 별도의 설정이 필요했으나, 현재에는 정식으로 포함되었으므로 필요한 의존성만 추가하여 사용이 가능하다.