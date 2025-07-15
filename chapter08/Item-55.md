# Item 55. 옵셔널 반환은 신중히 하라
Java8+ 부터 메서드가 특정 조건에서 값을 반환할 수 없을 때 Optional 을 반환할 수 있다.


`Optional<T>` 는 `null` 이 아닌 `T` 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다. 아무것도 담지 않은 상태는 "비었다"라고 표현한다. 옵셔널은 원소를 최대 1개 가질 수 있는 불변 컬렉션이다. `Optional<T>` 가 `Collection<T>` 를 구현하지는 않았으나 원칙적으로는 컬렉션이라고 부른다.


> 옵셔널이 불변 컬렉션으로 불리는 이유?
>
> - 실제 상속 관계가 없지만, 개념적인 유사성과 동작 방식 때문이다.
>
> 옵셔널은 최대 1개의 원소를 갖는 컨테이너로서의 역할을 한다. 또한, 불변하고 Stream API와 유사한 API를 많이 제공하기 때문에 Optional을 마치 원소가 하나뿐인 Stream 처럼 다룰 수 있게 된다.
>
> 즉, 함수형 스타일의 선언적 프로그래밍을 가능하게 만들어, 함수형으로 데이터를 다룰 수 있게 해주는 특별한 컨테이너 -> 개념적으로 "불변 컬렉션" 이라고 한다.


옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, `null` 반환하는 메서드보다 오류 가능성이 적다.


아래 코드를 비교해보자.


```java
// Code 1. 컬렉션이 비었을 때 예외 던짐
public static <E extends Comparable<E>> E max(Collection<E> c){
  if (c.isEmpty())
    throw new IllegalArgumentException("빈 컬렉션");

  E result = null;
  for (E e : c){
    if (result == null || e.compareTo(result) > 0){
      result = Objects.requireNonNull(e);
    }
  }
  return result;
}

// Code 2. 컬렉션에서 최댓값을 구해 Optional<E> 로 반환
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c){
  if (c.isEmpty())
    return Optional.empty();

  E result = null;
  for (E e : c){
    if (result == null || e.compareTo(result) > 0){
      result = Objects.requireNonNull(e);
    }
  }
  return Optional.of(result);
}
```


---
## 옵셔널 활용
- 적저한 정적 팩토리를 사용해 옵셔널을 생성해주고, 반환해주면 된다.
- 빈 옵셔널은 `Optional.empty()` 로 만들고, 값이 든 옵셔널은 `Optional.of(value)` 로 만들면 된다.
- 특히, 옵셔널을 반환하는 메서드에서는 절대 `null` 을 반환하지 말자. 옵셔널 도입 취지를 완전히 무시하는 행위이다.


### 옵셔널은 Stream과 비슷한 메서드를 많이 제공한다.
- `orElse` : 기본 값 설정

- `orElseThrow` : 원하는 예외 설정 - 실제 예외가 아닌 예외 팩토리를 생성해, 실제로 예외가 발생하지 않는 한 예외 생성 비용은 들지 않는다.

- `get` : 항상 값이 채워져 있다고 가정하고, 바로 값을 꺼내서 사용

- `orElseGet` : 기본값을 설정하는 비용이 부담스러울 때 `Supplier<T>` 를 인수로 받는 메서드 
    - 사용 값이 처음 필요할 때 `Supplier<T>` 를 사용해 생성하므로 초기 설정 비용을 낮출 수 있다.

- `isPresent` : 옵셔널이 채워져 있으면 true, 빈 값이면 false 반환 
    - 이 메서드는 다른 메서드들로 대체할 수 있으며, 대체 사용할 때 더 짧고 명확하고 용법에 맞는 코드가 될 수 있으므로 신중히 사용해야 한다.


---
## 옵셔널 사용 시 주의점
### 1. 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입을 옵셔널로 감싸지 마라
- `List<T>`에 아이템이 없는 경우, `Optional`로 감싸서 반환하기 보다는 빈 컬렉션(`Collections.emptyList()`)을 반환하는 것이 좋다.
- 클라이언트가 `Optional`이라는 wrapper 한 겹을 더 벗겨내야 하는 불필요한 복잡성을 추가할 뿐이다.
- 빈 컨테이너 자체로 '값이 없음'을 충분히 표현할 수 있기 때문이다.


### 2. 박싱된 기본 타입을 담은 옵셔널을 만들지 말자
- `Optional<Integer>` 대신 `OptionalInt`를 사용하는 식으로, 기본 타입 전용 옵셔널을 사용하는 것이 좋다.
- `Optional<Integer>`는 두 단계의 래핑(wrapping)이 필요해 성능적으로 손해이다. 기본 타입 옵셔널은 박싱/언박싱 오버헤드를 피할 수 있게 해준다.
- `OptionalInt`, `OptionalLong`, `OptionalDouble` 클래스를 적극적으로 활용하면 된다.


### 3. 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 것은 대부분 적절치 않다
- `Map`의 키로 `Optional`을 사용하면 키 비교 시 혼란을 야기하며, 코드의 복잡성만 가중시킨다.
- 옵셔널을 인스턴스 필드에 저장하는 것 역시 좋지 않은 설계일 가능성이 높다. 
- 필드가 '없을 수도 있다'는 사실은 보통 더 나은 클래스 구조(예: 클래스 분리)로 해결할 수도 있다. (꼭 필요한 경우가 아니라면 사용하지 말자)


---
## Summary
- 값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환 값이 없을 가능성이 있다면 Optional 을 반환해야 적절한 상황일 수도 있다.
- 하지만, Optional 반환에는 성능 저하가 뒤따르므로 성능에 민감한 메서드라면 주의해야 한다. (null을 반환하거나 예외 던지기)
    > 개인적인 궁금증: 예외를 던지는것도 비용 꽤 드는데, 무슨 기준으로 Optional 과 예외 중에 선택할까?
- 그리고, Optional 을 반환값 이외의 용도로 사용하는 것은 혼란을 야기하기 때문에 매우 드문 경우이다. (꼭 Optional로 나타낼 필요가 있는 경우 아니면 안씀)