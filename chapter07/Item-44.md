# Item 44. 표준 함수형 인터페이스를 사용하라
- 람다 지원이 되면서 API 작성의 모범 사례도 바뀌었다.
    - 기존 방식: 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴 활용
    - 현대 방식: 같은 효과의 함수 객체를 받는 정적 팩토리나 생성자 제공


현대적 방식을 유지하기 위해서는, 함수 객체를 매개변수로 받는 생성자 메서드를 더 많이 만들어야 한다.
이때 함수형 매개변수 타입을 올바르게 선택해야 한다.

---
## 함수형 인터페이스란?
- 추상 메서드가  단 하나만 선언된 인터페이스를 말한다.
- 이를 구현한 객체(== 함수 객체)는 람다식이나 메서드 참조로 간결하게 생성할 수 있다.


### `LinkedHashMap` 의 `removeEldestEntry` 예시
- 이 클래스의 protected 메서드인 removeEldestEntry 메서드를 재정의하면 캐시로 사용할 수 있다.
- 즉, 캐시의 LRU 정책으로 활용이 가능하다.

```java
// Code 1. 100개의 엔트리를 넘으면 eldest가 삭제되는 정책을 일반적인 재정의로 구현함
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size() > 100;
}
```


- 위 메서드는 인스턴스 메서드이기 때문에 size()를 호출하여 맵 안의 원소 수를 알아낼 수 있다.
- 하지만, 생성자에 넘기는 함수 객체는 이 맵의 인스턴스 메서드가 아니다. (팩토리나 생성자를 호출할 때는 맵의 인스턴스가 존재하지 않기 때문)
    - 따라서 이 경우에는, 자기 자신도 함수 객체에 건네줘야한다.
    - 이를 반영한 함수형 인터페이스를 아래와 같이 만들 수 있으나, 이는 표준 함수형 인터페이스는 아니다.


    ```java
    @FunctionalInterface interface EldestEntryRemovalFunction<K, V> {
        boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
    }
    ```

위 인터페이스도 잘 동작하지만, 이미 자바 표준 라이브러리에 같은 모양의 인터페이스가 있기 때문에 표준 함수형 인터페이스를 사용하는 것이 좋다.


## 표준 함수형 인터페이스
- `java.util.function` 패키지에 정의된, 람다와 메서드 참조의 타깃이 되는 인터페이스 모음을 의미한다.
- 표준 함수형 인터페이스들은 유용한 디폴트 메서드들을 많이 제공하여 다른 코드와의 상호 운용성이 좋아진다.
- Java 스트림(Stream API), `Optional`, `CompletableFuture` 등 표준 라이브러리와 자연스럽게 연동되어, 커스텀 함수형 인터페이스를 만들 필요가 거의 없다.
- 해당 패키지에는 총 43개의 인터페이스가 있으나, 기본 인터페이스 6개만 기억하면 나머지를 충분히 유추할 수 있다.


### 기본 인터페이스 6개
| 인터페이스 이름               | 함수 시그니처          | 사용 예시                                             |
| -------------------------- | --------------------- | ----------------------------------------------------- |
| `Supplier<T>`              | `T get()`             | `Supplier<LocalDate> now = LocalDate::now;`           |
| `Consumer<T>`              | `void accept(T t)`    | `Consumer<String> printer = System.out::println;`     |
| `Function<T,R>`            | `R apply(T t)`        | `Function<String,Integer> parse = Integer::parseInt;` |
| `Predicate<T>`             | `boolean test(T t)`   | `Predicate<String> isEmpty = String::isEmpty;`        |
| `UnaryOperator<T>` (자기 자신) | `T apply(T t)`        | `UnaryOperator<String> trim = String::trim;`       |
| `BinaryOperator<T>` (두 개)  | `T apply(T t1, T t2)` | `BinaryOperator<Integer> sum = Integer::sum;`        |


- `Supplier`: 값을 공급만 하고, 입력 파라미터 없음
- `Consumer`: 값을 소비(처리)하고, 반환값 없음
- `Function`: 입력을 받아 다른 형태로 변환한 뒤 반환함
- `Predicate`: 입력을 받아 `boolean` 결과(조건 판단)를 반환함
- `UnaryOperator`: `Function<T,T>`의 특수화, 같은 타입 간 변환(연산) 전용
- `BinaryOperator`: `BiFunction<T,T,T>`의 특수화, 두 개의 동일 타입 값을 받아 하나로 합치는 연산 전용


---
## 표준 함수형 인터페이스 사용 시 주의점
- 대부분 기본 타입만 지원한다.
- 또한, Boxing 된 기본 타입을 넣어서 사용하지 말자.


---
## `@FunctionalInterface` 사용
- 이 어노테이션을 사용하는 이유는 `@Override` 를 사용하는 이유와 비슷하다.
- 프로그래머의 의도를 명시하기 위한 목적으로 쓰인다.
    1. 해당 클래스의 코드나 설명 문서를 읽는 사람에게 해당 인터페이스가 람다용임을 명시한다.
    2. 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일 되게 해준다.
    3. 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다. (직접 만든 함수형 인터페이스에 이 어노테이션을 붙여라!)


---
## Summary
- API를 설계할 때 람다도 염두에 두어야 한다.
- 입력값과 반환값에 함수형 인터페이스 타입을 활용하자.
- 보통, java.util.function 패키지의 표준 함수형 인터페이스를 사용하는 것이 가장 좋은 선택이다.
    - 흔치는 않지만, 직접 새로운 함수형 인터페이스를 만들어 쓰는 편이 나을수도 있다.