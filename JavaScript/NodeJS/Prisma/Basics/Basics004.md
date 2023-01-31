# Basics
## 2023-01-31 Tue
### Prisma에서 혼동하기 쉬운 개념들
* `findFirst()`와 `findUnique()` 두 API 모두 하나의 결과만을 반환한다는 공통점을 갖는다.
    * 그러나 `findFirst()` API가 조건에 맞는 레코드 중 첫 번째를 반환한다면 `findUnique()` API는 정말 유일한 조건을 기반으로 레코드를 반환한다.
    * 때문에 **`findUnique()` API는 where 속성에 전달하는 검색 조건 자체가 PK 또는 UQ와 같이 유일성을 보장해야 한다**.
* 조회 시 **select 속성에 원하는 컬럼을 모두 명시하는 것은 반복적인 중복 코드가 될 수 있으므로, 별도의 모듈로 추출하여 사용하는 것이 일반적**이다.
* **include 속성은 select 속성과 함께 사용할 수 없으므로, 대신 select 내부에 가상 컬럼을 명시하여 조인을 유발**시킬 수 있다.
    * 이 경우, 가상 컬럼에 대해서 select 속성을 중첩 작성할 수 있다.
    * 이렇듯 Prisma의 조인 방법은 두 가지가 있으며, include 속성은 조인만 가능한 반면 select 속성은 원하는 컬럼만을 조회하면서도 조인을 수행할 수 있다.

### Prisma 집계 API 사용하기
* `aggregate()` API는 개수나 합계 또는 최대값 / 최소값 및 평균 등을 손쉽게 구할 수 있도록 도우며, 다음과 같이 간단하게 사용이 가능하다.
```typescript
getAggregates() {
  return this.prisma.user.aggregate({
    // user의 레코드 수를 산출한다.
    _count: true,
    
    // userId의 평균 값을 산출한다.
    _avg: {
      userId: true,
    },
    
    // userId의 합계를 산출한다.
    _sum: {
      userId: true,
    },

    // 각각 userId의 최대값과 최소값을 산출한다.
    _max: {
      userId: true,
    },
    _min: {
      userId: true,
    },
  });
}
```