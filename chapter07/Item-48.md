# Item 48. 스트림 병렬화는 주의해서 적용하라
동시성 프로그래밍 측면에서 Java는 항상 앞서가고, 다양한 기능을 제공하고 있다. Java8+ 부터는 parallel 메서드만 호출하면 파이프라인을 병렬 실행할 수 있는 스트림을 지원하고 있다.


동시성 프로그래밍 코드 작성이 점점 쉬워지지만, 올바르고 빠른 성능을 추구하며 작성하는 방법은 어렵다. 아래에 바람직하지 못한 상황(🚨) 및 사용을 추천하는 상황(💚)을 나누었다. 


## 🚨 1. 데이터 소스가 Stream.iterate 이거나 중간 연산에 limit 있는 경우
- 파이프라인 병렬화로 성능 개선을 기대할 수 없다.
- 아래 코드는 두 가지 문제를 모두 지니고 있어, cpu 사용률이 치솟으면서도 완료되지 않는일이 발생한다.


```java
public static void main(String[] args) {
  primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
        .filter(mersenne -> mersenne.isProbablePrime(50))
        .limit(20)
        .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
  return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```


## 💚 2. 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스 이거나, 배열, int 범위, long 범위일 때
- 이 자료구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있다.
- 즉, 일을 다수의 스레드에 분배하기 좋다는 특징이 있다.
- 나누는 작업은 Spliterator 가 담당하고, 이 객체는 Stream 이나 Iterable의 spliterator 메서드로 얻어올 수 있다.
- 이 자료구조들의 또다른 중요 공통점은, 원소들을 순차적으로 실행할 때 참조 지역성이 뛰어나다는 것이다.
    - 참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 매우 중요한 요소로 작용한다.
    - 참조 지역성이 가장 뛰어난 자료구조는 기본 타입 배열이다. (데이터 자체가 연속적으로 메모리에 저장)


## 3. 스트림 파이프라인의 종단 연산 동작 방식 (축소 연산 💚)
- 스트림 파이프라인의 종단 연산 동작 방식도 병렬 수행의 효율성에 영향을 미친다.
    - 종단 연산 중 병렬화에 가장 적합한 것은 축소(reduction) 연산이다. 
    - `sum()`, `reduce()` 등
    - 가변 축소를 수행하는 `collect` 메서드는 컬렉션들을 합치는 부담이 크기때문에 병렬화에 적합하지 않다.
    - 파이프라인에서 만들어진 모든 원소를 하나로 합칠 때 병렬 스트림에서는 전체 데이터를 나눠서 각각 처리한 다음, 중간 결과들을 합치는 식으로 기능한다.
- 조건에 맞으면 바로 반환되는 메서드도 병렬화에 적합하다.
    - `anyMatch()`, `findAny()`, `allMatch()`
    - 이런 연산은 전체 요소를 다 처리하지 않아도, 중간에 조건이 만족되면 연산을 즉시 종료할 수 있어서, 여러 부분들을 동시에 검사하다가 조건을 만족하는 값이 하나라도 나오면 바로 중단 가능하여 효율적으로 기능한다.



## 스트림 병렬화는 오직 성능 최적화 수단
- 직접 구현한 `Stream`, `Iterable`, `Collection` 이 병렬화의 이점을 제대로 누리게 만들고 싶다면, spliterator 메서드를 반드시 재정의하고 결과로 받는 스트림의 병렬화 성능을 강도 높게 테스트해야 한다.
- 스트림을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과가 잘못되거나 예상 못한 동작을 할 수 있다.
- `Stream` 에서 사용되는 엄중한 규약을 어기면 안된다.
    - `Stream` `reduce` 연산의 `accumulator` 와 `combiner` 함수는 반드시 결합 법칙을 만족하고, 간섭받지 않고, 상태를 갖지 않아야한다.
- 반드시 테스트를 통해 병렬화의 가치가 있는지 확인해야 한다.


---
## Summary
- 확신 없이 스트림 파이프라인 병렬화를 시도하지 말자.
    - 계산을 올바르게 수행하면서, 성능이 빨라질거라는 확신이 있을 때 도입
- 스트림을 잘못 병렬화하면 프로그램 오동작, 성능 악화를 발생시킨다.
- 병렬화가 이론적으로 좋더라도, 수정 후에 운영 환경과 유사한 조건에서 수행하면서 성능 지표를 관찰해야 한다.
    - 꼼꼼한 관찰 후 운영 코드에 반영해야한다.