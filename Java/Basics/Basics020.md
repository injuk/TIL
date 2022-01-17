# Java
## 2022-01-17 Sun

### 스트림 
* 스트림은 아래와 같은 문제를 해결하기 위해 고안되었다.
    1. for문이나 Iterator를 사용하는 방식의 가독성이 떨어진다.
    2. 재사용성이 떨어진다.
    3. 원본의 타입마다 다른 방식으로 처리해야 한다.
        * 컬렉션 프레임워크는 표준화되어 있지만, 같은 기능의 메소드들이 중복해서 정의되어 있는 경우가 있다.
* 스트림은 상술한 문제점을 다음과 같이 해소하고자 하였다.
    1. 데이터 원본(데이터 소스)을 추상화한다.
        * 덕분에 데이터 원본의 타입이 무엇이건 간에 같은 방식으로 다룰 수 있으며, 코드의 재사용성이 높아진다. 
    2. 데이터를 다루기 위해 자주 사용될 법한 메소드들을 정의해두었다.
* **스트림의 등장으로 인해 배열, 컬렉션, 파일 데이터 등 모두 같은 방식으로 다룰 수 있게 된다**.
```
class Main {
  public static void main(String[] args) {
    String[] strings = { "eee", "aaa", "ccc", "ddd" };

    Arrays.sort(strings);
    for(String string : strings) {
      System.out.println(string); // 기존 방식
    }

    Arrays.stream(strings) // stream을 사용한 방식
        .sorted()
        .forEach(System.out::println);
  }
}
```
* 위 방식은 for문을 활용한 방식과 동일한 작업을 수행하는 스트림 예시이다.
    * 스트림을 사용한 경우가 더 간결하고, 직관적인 것처럼 보인다.

### 스트림의 특징
* **스트림은 데이터 원본을 수정하지 않는다**.
    * 데이터 원본으로부터 데이터를 읽기만 하고, 원본을 수정하지 않는다.
    * 필요한 경우엔 결과를 다시 원하는 데이터 형태로 반환할 수 있다.
* **스트림은 일회용**이다.
    * 요소를 모두 읽고 나면 재사용이 불가능한 Iterator처럼, 스트림도 한 번 사용하면 닫히므로 재사용할 수 없다.
    * 재사용이 필요한 경우라면 스트림을 다시 생성해야 한다.
* **스트림의 작업은 내부 반복으로 처리**된다.
    * 위 예제에서, 배열의 모든 요소에 대해 수행되는 작업은 코드 블록 상에 노출되어 있다.
    * 반면 스트림의 forEach는 모든 요소에 대해 수행될 실제 작업이 메소드 내부에 숨겨진다.
        * 이러한 특징은 스트림의 작업이 내부 반복으로 처리됨을 의미한다.
        * forEach의 경우, 수행할 작업은 람다식(Consumer)으로 작성하여 매개 변수로 넘겨준다.

### 스트림의 연산
* 스트림의 연산은 **스트림 생성 > 중간 연산 > 최종 연산**의 순서로 진행된다.
* 스트림의 연산은 데이터 소스를 다루는 작업을 말한다.
* 스트림의 연산은 **중간 연산**과 **최종 연산**으로 구분된다.
   1. 중간 연산: **연산의 결과를 스트림으로 반환**하므로, 중간 연산을 연속적으로 연결(**chaining**)할 수 있다.
      * 중간 연산의 결과로 반환되는 스트림은 모두 다른 스트림이다.
      * 예시: distict, filter, sorted, map, flatMap, 등
   2. 최종 연산: 스트림의 **요소를 소모하며 연산을 수행**하므로, **단 한 번만 연산이 가능**하다.
      * 예시: forEach, count, max, min, reduce, collect, 등 
* **지연된 연산**: 스트림에서는 **최종 연산이 수행되기 전까지 중간 연산이 실행되지 않는다**.
    * 중간 연산의 명시는 해당 스트림에서 어떤 작업을 수행해야하는지 명시하는 것이다.
    * 실제 실행은 최종 연산이 수행되는 시점으로, 이 때 중간 연산을 거쳐 최종 연산에서 요소들이 소모된다.
* 기본형 스트림: Stream<Integer>와 같은 경우는 데이터 원본의 요소가 기본형인 경우, 불필요한 오토박싱 & 언박싱이 반복된다.
    * IntStream, LongStream과 같은 기본형 스트림은 데이터 원본의 요소를 기본형으로 다루도록 하며, 성능 상 이점이 있다.
    * 기본형을 다루는 데이터 원본에 대한 작업은 기본형 스트림을 사용하는 것이 바람직하다.

## 스트림의 생성
* 스트림 작업을 위해서는 반드시 스트림이 생성되어 있어야 한다.
* 스트림 생성시에는 스트림의 대상이 되는 데이터 원본이 필요하며, 배열 / 컬렉션 프레임워크 / 임의의 수가 원본이 될 수 있다.
### 컬렉션 프레임워크에서의 스트림 생성
* Collection 인터페이스에 stream 메소드가 정의되어 있으므로, Collection의 자식인 List와 Set을 구현한 클래스 모두 해당 메소드를 사용한다.
* **stream 메소드는 해당 컬렉션을 데이터 원본으로 하는 새로운 스트림을 반환**한다.
### 배열에서의 스트림 생성
* 배열의 스트림은 Stream 클래스와 Arrays 클래스 각각에 정의된 두 가지 방법으로 생성할 수 있다.
    1. Stream.of: 가변 인자 또는 배열 인스턴스를 매개 변수로 받아 새로운 스트림을 생성한다.
        * 가변 인자 예시: Stream.of(1, 2, 3);
    2. Arrays.stream: 배열 인스턴스를 매개 변수로 받아 새로운 스트림을 생성한다.
### 특정 범위의 정수 스트림 생성
* 기본형 스트림인 IntStream과 LongStream 등은 다음과 같이 임의의 범위에서 연속된 정수를 새로운 스트림으로 반환하는 메소드를 갖는다. 
    1. range(시작, 끝): 끝 값을 포함하지 않는 연속된 정수 스트림을 반환한다.
        * IntStream.range(1, 3); // 1, 2
    2. rangeClosed(시작, 끝): 끝 값을 포함하는 연속된 정수 스트림을 반환한다.
        * IntStream.rangeClosed(1, 3) // 1, 2, 3
### 난수 스트림 생성
* Random 클래스에는 기본형 스트림을 생성하는 ints, longs, doubles 메소드가 포함된다.
    * 세 메소드는 long 타입 매개 변수를 받아 유한 길이의 스트림을 생성한다.
    * 반면 매개 변수를 지정하지 않은 경우, 세 메소드 모두 해당 타입의 난수로 이루어진 무한한 길이의 스트림을 반환한다.
        * 이 경우에는 스트림의 개수를 지정하는 중간 연산인 limit(개수)를 통해 무한 스트림을 유한한 길이의 스트림으로 바꾸어 연산해주어야 한다.
    * 지정된 범위의 난수 스트림을 발생시키려면 위 세 메소드의 매개 변수를 (시작, 끝)으로 작성한다.
        * 이러한 방식은 range 메소드처럼 끝 값을 스트림 범위에 포함시키지 않는다.
### 람다식을 활용한 스트림 생성
* Stream의 iterate와 generate는 람다식을 매개 변수로 받아, 람다식의 연산 결과를 요소로 하는 무한 길이의 스트림을 생성한다.
    1. iterate(시작값, UnaryOperator): 시작 값으로 시작하여 람다식에 의한 연산 결과를 갖는 무한 길이의 스트림을 생성한다.
    2. generate(Supplier): 이전 결과를 이용해서 다음 요소를 계산하지 않는 연산 결과를 갖는 무한 길이의 스트림을 생성한다.
        * Supplier를 매개 변수로 받으므로, 매개 변수 없이 어떠한 값을 만들어내는 람다식을 활용하여야 한다.
* 위 두 메소드를 활용하여 생성한 스트림은 기본형 스트림일 수 없다.
### 파일 목록을 기반으로 하는 스트림 생성
* Files.list(Path dir): 지정된 dir 디렉토리에 위치한 파일의 목록을 원본으로 하는 스트림을 생성하여 반환한다.
### 빈 스트림 생성
* 아무런 요소를 갖지 않는 빈 스트림을 생성할 수 있다.
    * 스트림의 **연산 수행 결과가 없는 경우, null 보다는 비어 있는 스트림을 반환하는 것이 바람직**하다.
* Stream.empty(): 빈 스트림을 생성하여 반환한다. count 등의 최종 연산으로 확인할 경우 스트림의 길이는 0이 반환된다.
### 두 스트림을 연결하여 새로운 스트림 생성
* Stream의 static 메소드인 concat을 통해 두 스트림을 하나의 스트림으로 연결할 수 있다.
* **연결 대상 스트림은 같은 타입을 다루는 스트림이어야** 한다.
```
class Main {
  public static void main(String[] args) {
    String[] strings = { "eee", "aaa", "ccc", "ddd" };

    Stream<String> stream1 = Arrays.stream(strings);
    Stream<String> stream2 = Stream.of(strings);

    Stream.concat(stream1, stream2).sorted().forEach(System.out::println); // aaa, ccc, ddd, eee가 두 번씩 출력된다.
  }
}
```

## 스트림과 중간 연산
* 중간 연산은 연산 결과로 새로운 스트림을 반환하므로, 아래의 메소드들은 체이닝이 가능하다.
### 스트림 자르기
* skip: long 타입을 매개 변수로 받아 그 값만큼 앞에서부터 요소를 건너뛴다.
* limit: long 타입을 매개 변수로 받아 그 값만큼 스트림의 길이를 제한한다.
### 스트림 거르기
* distinct: 매개 변수를 받지 않으며, 스트림에서 중복된 값을 제거한다.
* filter: Predicate 람다식을을 매개 변수로 받아 스트림에서 그 조건에 맞지 않는 값을 제거한다.
    * 따라서 filter 중간 연산을 거친 스트림에는 조건에 맞는 값만 남게 된다.
    * 중간 연산으로서의 filter 역시 새로운 스트림을 반환하므로, 서로 다른 조건으로 연속해서 filter를 걸 수 있다.
### 스트림 정렬
* sorted: 스트림의 정렬에 사용한다. 매개 변수로 전달된 값에 따라 다음과 같이 수행된다.
    * sorted(): 스트림 요소 별 기본 정렬 기준인 Comparable에 따라 정렬한다.
        * 요소가 Comparable을 구현한 클래스가 아니라면 예외가 발생한다.
    * sorted(Comparator<? super T> comparator): 정렬 기준을 Comparator로 작성하여 정렬한다.
### 스트림 변환
* map: 스트림의 **요소에 저장된 값 중 원하는 멤버 변수만 뽑아내거나, 특정 형태로 변환하고자 하는 경우에 사용**한다.
    * 매개 변수로 Function을 받아 동작한다.
    * map 역시 스트림을 반환하는 중간 연산이므로, filter와 같이 여러 번 연속하여 적용할 수 있다.
* mapToInt, mapToLong, mapToDouble: 스트림의 요소를 숫자로 변환하는 경우, 기본형 스트림을 반환하는 메소드이다.
    * 불필요한 오토 박싱 / 언박싱을 줄여 성능을 향상시키고자 할 때 유용하다.
        * Stream<Integer>에서 숫자로 변환된 스트림을 반환하는 map은 Stream<Integer>를 반환한다.
        * 반면 mapToInt를 사용한다면, 정수 기본형 스트림인 IntStream이 반환된다.
    * Stream<Integer>는 count만 사용할 수 있지만, 기본형 스트림은 다음의 최종 연산을 제공한다.
        * sum, average, min, max
    * 만약 sum과 average를 모두 호출하고자 한다면, 새로운 Stream을 생성하는 것보다 IntSummaryStatistics 클래스를 활용하자.
        * IntSummaryStatistics 인스턴스는 summaryStatistics 메소드를 통해 반환받을 수 있다.
```
public class Study {
    public static void main(String[] args){
        IntStream stream = IntStream.range(1, 100);
        // IntSummaryStatistics 클래스를 통해 stream을 닫지 않고 여러 통계 정보를 얻을 수 있다.
        IntSummaryStatistics statistics = stream.summaryStatistics(); // summaryStatistics 메소드 활용
        System.out.println("average: " + statistics.getAverage());
        System.out.println("sum: " + statistics.getSum());
        System.out.println("count: " + statistics.getCount());
    }
}
```
* boxed: 기본형 스트림을 다시 래퍼 클래스 스트림으로 변환한다.
* mapToObj: Function을 매개 변수로 받아 래퍼 클래스가 아닌 <T> 클래스로 변환할 때 사용하는 메소드이다.
* flatMap: 스트림의 요소가 배열인 경우, 배열의 결과를 펼친 스트림으로 변환한다.
    * 즉, Stream<String[]>을 펼쳐 Stream<String> 형태로 변환한다.
    * 배열을 요소로 갖는 Stream에 대한 map(Arrays::stream) 연산이 Stream<String[]>을 Stream<Stream<String>>으로 만들어준다면,
    * flatMap(Arrays::stream)은 Stream<String[]>을 Stream<String>으로 만들어준다.
### 스트림 조회
* peek: 스트림의 중간 연산 결과를 보고 싶을 때 유용하며, 주로 map 또는 filter의 결과를 확인하기 위해 추가한다.
    * peek은 중간 연산이므로 요소를 소모하지 않는다.
    * peek은 Consumer를 매개 변수로 받아 동작한다.

### Optional<T>와 OptionalInt
* Optional<T>는 T 타입의 인스턴스를 감싸는 래퍼 클래스이다.
    * 따라서 Optional 타입의 인스턴스는 모든 타입의 참조 변수를 감쌀 수 있다.
* 스트림의 경우 최종 연산의 결과를 Optional 인스턴스에 담아 반환한다.
    * 이를 통해 매번 반환 결과의 null 체크를 if 문으로 수행할 필요 없이, Optional 클래스의 메소드를 활용할 수 있다.
* Optional 인스턴스를 생성하는 방법은 크게 다음과 같은 두 가지로 나뉜다.
    1. of: 참조 변수의 값이 null일 경우 NullPointerException이 발생한다.
    2. ofNullable: 참조 변수의 값이 null일 가능성이 있는 경우에 사용한다.
    * ofNullable에 null을 넘긴 경우, equals 메소드 등에 의해 Optional.empty()와 비교할 경우 true가 반환된다.
* Optional<T> 타입의 참조 변수를 기본값으로 초기화할 때는 Optional.empty() 메소드를 활용한다.
    * 그냥 null 로 초기화하는 것보다 empty 메소드를 통해 빈 인스턴스로 초기화하는 것이 바람직하다.
* Optional 인스턴스의 값을 가져오기 위해 다음의 메소드가 제공된다.
    1. get(): Optional 인스턴스의 값을 가져오며, null인 경우 예외가 발생한다.
    2. orElse(값): 값을 가져오며, null인 경우 '값'을 반환한다.
    3. orElseGet(Supplier): 값을 가져오며, null일 경우 Supplier 람다식을 통해 특정한 값을 반환한다.
    4. orElseThrow(Supplier): 값을 가져오며, null일 경우 Supplier 람다식을 통헤 특정한 예외를 발생시킨다.
* Optional 인스턴스 값의 null 여부에 따라 수행할 수 있는 동작이 달라지는 메소드가 제공된다.
    1. isPresent(): null이면 false를, null이 아니면 true를 반환한다.
    2. ifPresent(Consumer): null이면 아무 일도 하지 않지만, null이 아닌 경우에는 Consumer를 실행한다.
        * ifPresent는 Optional<T>를 반환하는 findAny, findFirst, min, max, reduce 등의 최종 연산과 함께 활용할 수 있다.

## 스트림과 최종 연산
* 최종 연산은 스트림의 요소를 소모하여 결과를 만들고 스트림을 닫는다.
    * 닫힌 스트림은 더 이상 사용할 수 없게 된다.
* 최종 연산의 결과는 count와 같은 단일 값이거나, 배열 또는 컬렉션 프레임워크일 수 있다.
### 조건 검사를 위한 최종 연산
* forEach: 반환 타입이 void인 최종 연산이며, 주로 스트림의 요소를 출력하는 용도로 사용된다.
* allMatch, anyMatch, noneMatch, findFirst, findAny
    * 스트림의 요소에 대해 지정된 조건과의 일치성을 확인하기 위해 사용되는 메소드이다.
    * 해당 메소드들은 모두 매개 변수로서 Predicate를 받아 검증한 후, 결과로 boolean을 반환한다.
    * findFirst는 주로 filter와 함께 사용되어 조건에 맞는 요소가 있는지 확인하기 위해 사용한다.
        * 스트림에 적절한 요소가 없는 경우 내부적으로 null을 저장하는 '비어 있는 Optional 인스턴스'를 반환한다.
### 통계를 위한 최종 연산
* 기본형 스트림이 아닌 경우, count / max / min 을 활용할 수 있다.
    * 기본형 스트림의 통계 정보는 summaryStatistics 메소드를 통해 IntSummaryStatistics 인스턴스를 활용할 수 있다.
### reduce 메소드
* 스트림의 요소를 소모해가며 연산을 수행하고, 최종 연산 결과를 반환한다.
    * 초기값을 갖는 reduce(값, BinaryOperator) 형태의 메소드도 있다. 
* 때문에 매개 변수는 BinaryOperator이다. 
    * 매 연산마다 이전 값까지의 연산 결과와 현재 값을 이용하여 연산을 수행하고, 하나의 값을 다음 연산으로 넘긴다.
* 스트림의 요소를 모두 소모하게 되면 결과를 반환한다.
## 최종 연산 - collect
* 스트림의 최종 연산 중 가장 복잡하면서도 유용한 연산이다.
* reduce와 유사하지만, collect는 요소를 수집하기 위한 수집 방법이 사전 정의되어 매개 변수로 전달되어야 한다.
    * 이러한 '수집 방법'을 컬렉터라고 한다.
    * 컬렉터는 Collector 인터페이스를 구현해야 한다.
    * Collectors는 사전 정의된 Collector 구현체들을 static 메소드로 제공하는 클래스이다.
### 스트림을 컬렉션 / 배열로 변환
* Collectors.toList(), toSet(): 스트림의 요소들을 List나 Set으로 수집한다.
* Collectors.toCollection(ArrayList::new): 특정한 컬렉션을 사용하고 싶은 경우, 컬렉션의 생성자 참조를 매개 변수로 넣어준다.
* Collectors.toMap(): 스트림의 요소들을 Map으로 수집한다.
    * Map의 경우, 키와 값의 쌍으로 반환되어야 하므로 요소의 어떤 필드를 각각 키 / 값으로 사용할지 명시해야 한다.
    * 예시: HashMap<String, Integer> map = ...생략.. .toMap(e->e.getName(), e->e.getAge());
    * e->e와 같은 형태를 사용하는 경우, 항등 함수를 의미하는 Function.identity()를 사용할 수 있다.
* Collectors.toArray(): 스트림의 요소들을 배열로 수집한다.
    * 적절한 배열의 타입을 생성자 참조로 인자에 넘겨주어야 한다.
        * 넘겨주지 않는 경우, 반환되는 배열의 타입은 Object[]이다.
    * 예시: toArray(String[]::new)
### 스트림의 통계 결과 반환
* 앞서 다룬 min, max, count와 같은 역할을 하는 컬렉터를 Collectors 클래스가 제공한다.
* Collectors.counting()
* Collectors.summingInt()
* Collectors.averagingInt()
* Collectors.maxBy()
* Collectors.minBy()
### reducing, joining
* reduce와 같은 역할을 수행하는 Collectors.reducing() 메소드도 제공된다.
* 문자열 스트림의 모든 요소를 하나의 문자열로 연결하기 위해서는 Collectors.joining() 메소드를 사용할 수 있다.
    * 매개 변수로 결합 문자를 넘겨줄 경우, 해당 문자를 기준으로 결합된다.
    * 예시: Collectors.joinint(",");은 문자열을 쉼표로 결합한다.
### 그룹화와 분할
* 그룹화: 스트림의 요소를 일정 기준으로 그룹화한다.
    * 대응되는 매개 변수는 groupingBy로, 매개 변수로 Function을 받는다.
* 분할: 스트림의 요소를 지정된 조건에 일치하는 그룹과 일치하지 않는 그룹으로 분할한다.
    * 대응되는 매개 변수는 partitioningBy로, 매개 변수로 Predicate를 받는다.
* 그룹화와 분할의 결과는 모두 Map의 형태로 반환된다.