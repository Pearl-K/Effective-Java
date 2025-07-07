# Item 46. 스트림에서는 부작용 없는 함수를 사용하라

## 스트림: 함수형 프로그래밍 패러다임의 핵심

스트림은 데이터 처리 및 계산을 위한 **함수형 프로그래밍** 패러다임에 기초를 둔다. 함수형 프로그래밍은 프로그램을 상태를 변경하는 명령어의 연속으로 보는 대신, 순수한 함수들의 조합으로 간주하여 **부작용(Side Effect)을 최소화**하는 것을 목표로 한다.

-   **함수형 프로그래밍이란?**
    -   계산을 수학적 함수의 평가로 취급하고, 상태 변경과 가변 데이터를 피하는 프로그래밍 스타일이다. 
    - 프로그램의 동작을 예측하기 쉽고, 테스트와 병렬 처리를 용이하게 만든다.

-   **부작용(Side Effect)이란?**
    -   함수가 결과값을 반환하는 것 외에 외부 상태를 변경하는 모든 행위를 의미한다. 
    - 예를 들어, 함수 외부의 변수 수정, 데이터베이스나 파일에 내용 쓰기, 콘솔에 로그 출력 등이 부작용 예시이다.
    - 부작용이 있는 코드는 코드의 동작을 이해하고 예측하기 어렵게 만들기 때문에 스트림에서 주의해야한다.

스트림 패러다임의 핵심은 "계산을 일련의 변환 과정"으로 재구성하는 것이다. 이때 각 변환 단계는 이전 단계의 결과를 받아 처리하는 **순수 함수**여야 한다.

-   **순수 함수**: 오직 입력만이 결과에 영향을 주는 함수로, 외부의 가변적인 상태를 참조하지도, 스스로 다른 상태를 변경하지도 않음

따라서 스트림 연산에 전달되는 모든 함수 객체는 부작용이 없어야 한다. 이는 스트림 파이프라인이 예측 가능하고, 안정적이며, 병렬 실행에 최적화될 수 있도록 보장하는 핵심 원칙이다.

---
## 부작용 없는 함수는 무엇일까?
### 1. 스트림 패러다임을 이해하지 못한 코드


```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```


위 코드는 스트림, 람다, 메서드 참조를 사용했지만 적절한 스트림 코드는 아니다. (스트림 코드를 가장한 반복적 코드임)


이 코드의 모든 작업은 종단 연산인 `forEach` 에서 일어나는데, 이때 외부 상태(빈도표)를 수정하는 람다를 실행하면서 문제가 생긴다.


`forEach` 가 그저 스트림이 수행한 연산 결과를 보여주는 일 이상을 하고 있다. (람다가 상태를 수정하고 있음) 


또한, `forEach` 로 외부 컬렉션을 수정하는 코드를 병렬 스트림으로 실행할 경우 race condition에 빠질 수 있어 예상치 못한 결과가 나올 수 있다. (추후 Item 48에서 스트림 병렬화에 대해 자세히 설명한다.)


### 2. 스트림을 제대로 활용한 코드


```java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(
        groupingBy(String::toLowerCase, counting()));
}
```


앞선 1번 코드와 같은 일을 하지만, 이번에는 스트림 API를 제대로 사용한 코드이다.


`for-each` 반복문과 `forEach` 종단 연산을 같은 코드 스타일로 사용해서는 안된다. `forEach` 연산은 스트림 계산 결과를 보고할 때만 사용하고, 직접 계산해서 값을 변경하는 데에 사용하지 말자.


### 3. Collector 개념
- 스트림을 사용하려면 꼭 배워야하는 새로운 개념이다.
- java.util.stream.Collectors 에서 확인할 수 있다.
- Collector를 이해할 때는, 축소 전략을 캡슐화한 블랙박스 객체라고 생각하면 편하다.


> 축소(reduction) 전략?
>
> 스트림의 원소들을 객체 하나에 취합한다는 의미이다.
> 
> 수집기가 생성하는 객체는 일반적으로 컬렉션이기 때문에 `collector` 라는 이름을 사용한다.


### 4. Collector 사용하기
수집기를 사용하면 스트림의 원소들을 손쉽게 컬렉션으로 모을 수 있다.


수집기에는 대표적으로 세 가지가 있다. 지정한 컬렉션 타입을 반환한다.
1. `toList()`
2. `toSet()`
3. `toCollection(collectionFactory)`


```java
// 스트림 파이프라인 예시 1. 빈도표에서 topTen 단어 꺼내오기
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
```


```java
// 스트림 파이프라인 예시 2. 스트림을 맵으로 취합
private static final Map<String, Operation> stringToEnum =
    Stream.of(values()).collect(
        toMap(Object::toString, e -> e));
```


위 코드는 스트림의 각 원소가 고유한 키에 mapping 되어 있을 때 적합하다. 스트림 원소 다수가 같은 키를 사용하면 해당 파이프라인이 `IllegalStateException` 을 던지며 종료될 것이다.


```java
// 스트림 파이프라인 예시 3. toMap을 유용하게 사용하는 경우
Map<Artist, Album> topHits = albums.collect(
    toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```


- 어떤 키와 그 키에 연관된 원소들 중 하나를 골라서 연관 짓는 맵을 만들 때 유용하다.
- 여기서 비교자로 사용한 함수는 `BinaryOperator` 에서 정적 임포트한 `maxBy` 라는 정적 팩토리 메서드이다.
- 자신의 키 추출 함수로는 `Album::sales` 를 받았다.
- 이렇게 인수가 3개인 toMap은 충돌이 나면 마지막 값을 취하는(last-write-wins) 수집기를 만들 때도 유용하다.


```java
// 마지막에 쓴 값을 취하는 수집기 코드
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```


인수가 4개인 `toMap` 은 마지막 인수로 맵 팩토리르 받아, 인수로 EnumMap 이나 TreeMap 처럼 원하는 특정 맵 구현체를 지정할 수 있다.

- `toMap(keyMapper, valueMapper, mergeFunction, mapFactory)`


### `toMap` 인수 개수 정리
   1. `toMap(keyMapper, valueMapper)` (인자 2개)
       * 가장 기본적인 `toMap` 사용
       * `keyMapper`: 스트림의 원소를 맵의 키(key)로 변환하는 함수
       * `valueMapper`: 스트림의 원소를 맵의 값(value)으로 변환하는 함수
       * 문제점: 만약 스트림을 처리하다가 중복된 키가 나오면 `IllegalStateException` 예외를 던지며 멈춤


   2. `toMap(keyMapper, valueMapper, mergeFunction)` (인자 3개)
       * 위의 1번 메서드에 중복 키 처리 기능이 추가된 버전
       * `mergeFunction`: 키가 중복될 때 기존 값과 새로 들어온 값을 어떻게 합칠지(merge) 결정하는 함수
       * 예를 들어 `(oldValue, newValue) -> newValue` 라고 하면 중복 시 새 값으로 덮어쓰고, `Long::sum `처럼 두 값을 더할 수도 있음


   3. `toMap(keyMapper, valueMapper, mergeFunction, mapFactory)` (인자 4개)
       * 2번 메서드에 맵 구현체 선택 기능이 추가된 버전
       * `mapFactory`: 최종 결과를 담을 맵의 종류를 직접 지정하는 함수
       * 예를 들어 `TreeMap::new`를 전달하면, 결과가 알파벳 순서로
         정렬된 `TreeMap` 에 담기게 된다. (기본적으로는 `HashMap` 사용)


### (+ 추가) `toMap` 의 변종 `toConcurrentMap`
   * toConcurrentMap은 toMap의 변종(variant) 이다.
   * 이름에서 알 수 있듯이, 병렬 스트림(parallelStream) 처리 시에 안전하고 효율적으로 동작하도록 설계되었다.
   * 결과물로 `ConcurrentHashMap` 이라는 동시성(concurrency) 처리에 특화된 맵을 생성한다.
   * toConcurrentMap도 마찬가지로 인자 2개, 3개, 4개를 받는 오버로딩된 버전들이 존재한다.


### `groupingBy` 메서드
- 이 메서드는 입력으로 분류 함수를 받고, 출력으로 원소들을 카테고리로 그룹핑한 수집기를 반환한다.
- 그리고 이 카테고리가 해당 원소의 map key로 쓰인다.


```java
words.collect(groupingBy(word -> alphabetize(word)))
```


위 코드는 알파벳화한 단어를 알파벳화 결과가 같은 단어들의 리스트로 매핑하는 맵을 생성하는 코드이다.


groupingBy 도 인수를 여려 개 받아서 반환할 수 있다.
   * 1개: 단순 그룹화 (`Map<K, List<T>>`)
   * 2개: 그룹화 + 그룹별 추가 연산 (`Map<K, D>`)
   * 3개: 그룹화 + 그룹별 추가 연산 + 결과 맵 타입 지정


groupingBy 역시 groupingByConcurrent 메서드도 사용할 수 있다.


### 다운 스트림 수집기
- `groupingBy` 에서 1차적으로 분류한 것을 가지고 추가적인 작업을 하는 것
- `toList()`: 기본 값으로 수집한 것을 리스트로 만든다.
- `counting()`: 개수를 세어 반환한다.
- `groupingBy()`: 한 번 분류한 결과를 다시 그룹화 할 수 있다. (ex. Map 안에 Map 담는 계층 구조처럼 반환됨)



### `joining` 메서드
- 이 메서드는 CharSequence 인스턴스의 스트림에만 적용 가능하다.
- 매개변수가 없는 `joining` 은 단순히 원소들을 연결하는 수집기를 반환한다.
- 구분문자를 매개변수로 받으면 연결 부위에 해당 문자를 삽입해준다.


---
## Summary
- 스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다.
- 스트림뿐 아니라 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다.
- 종단 연산 중에 `forEach` 는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다. (계산 자체에는 이용 x)
- 스트림을 올바르게 사용하려면 `Collectors` 에 대해서 잘 알아둬야 한다.
    - 가장 중요한 수집기 팩토리: `toList`, `toSet`, `toMap`, `groupingBy`, `joining` 등이 있다.