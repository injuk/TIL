# Basics
## 2023-01-31 Tue
### Prisma 실전팁
* `findFirst()`와 `findUnique()` 두 API 모두 하나의 결과만을 반환한다는 공통점을 갖는다.
    * 그러나 `findFirst()` API가 조건에 맞는 레코드 중 첫 번째를 반환한다면 `findUnique()` API는 정말 유일한 조건을 기반으로 레코드를 반환한다.
    * 때문에 **`findUnique()` API는 where 속성에 전달하는 검색 조건 자체가 PK 또는 UQ와 같이 유일성을 보장해야 한다**.
* 조회 시 **select 속성에 원하는 컬럼을 모두 명시하는 것은 반복적인 중복 코드가 될 수 있으므로, 별도의 모듈로 추출하여 사용하는 것이 일반적**이다.
* **include 속성은 select 속성과 함께 사용할 수 없으므로, 대신 select 내부에 가상 컬럼을 명시하여 조인을 유발**시킬 수 있다.
    * 이 경우, 가상 컬럼에 대해서 select 속성을 중첩 작성할 수 있다.
    * 이렇듯 Prisma의 조인 방법은 두 가지가 있으며, include 속성은 조인만 가능한 반면 select 속성은 원하는 컬럼만을 조회하면서도 조인을 수행할 수 있다.
* `createMany()` API의 경우, FK를 활용하는 가상 컬럼을 활용할 수 없다는 단점이 존재한다.

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

### distinct를 활용한 결과값 중복 제거
* `findMany()` API 등을 활용한 레코드 조회시, 결과로부터 특정 컬럼에 대해 중복을 제거하는 distinct 속성은 다음과 같이 사용할 수 있다.
```typescript
getWithDistinct() {
  return this.prisma.user.findMany({
    orderBy: {
      name: 'asc',
    },
    where: {},
    
    // 이 경우, name 컬럼으로부터 중복을 제거한다.
    distinct: ['name'],
  });
}
```
* 또한, **distinct 속성은 배열 형태로 전달되므로 당연히 여러 컬럼에 대한 중복 제거도 수행이 가능**하다.
  * 이 경우, **distinct 속성에 전달된 모든 컬럼을 쌍으로 취급하여 중복되는 조합만을 결과에서 제거**하게 된다.

### Prisma groupBy API를 활용하여 결과 그룹화하기
* 결과 레코드 목록을 특정한 컬럼을 기준으로 그룹화하고자 하는 경우, 다음과 같이 `groupBy()` API 를 활용하여 간단하게 구현할 수 있다.
  * 또한 일반적으로 **중복만을 제거하는 distinct와 달리, groupBy 연산은 집계 함수와 조합하는 등 더 고차원적인 연신을 적용**할 수 있다.
  * 이 과정에서 **having 절 역시 적용이 가능하며, `groupBy()` API 호출 시 속성으로 전달하여 조건에 맞는 그룹만을 선택**할 수 있다.
```typescript
groupBy() {
  return this.prisma.user.groupBy({
    // 이 경우, name 컬럼을 기준으로 그룹화한다.
    by: ['name'],
    
    // 또한, 각 그룹 별로 집계 연산을 적용할 수도 있다.
    _count: {
      _all: true,
    },
    
    // 또는 그룹화된 레코드 집합의 임의의 컬럼에 대해서만 집계 함수를 적용할 수도 있다.
    _sum: {
      userId: true,
    },

    // having 속성을 명시하는 것으로 사용자 Id의 합계가 1만보다 작은 그룹만을 조회할 수 있다.
    having: {
      userId: {
        _sum: {
          lt: 10_000,
        },
      },
    },
  });
}
```