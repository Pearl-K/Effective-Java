# Item 31. 한정적 와일드카드를 사용해 API 유연성을 높이라
- Java의 제네릭 타입시스템에는 변성 개념이 존재하고, 변성에 대해서 알아야 공변과 불공변을 설명할 수 있다.

## 변성 `variance`
변성은 `List<String>`과 `List<Object>`와 같이 base 타입이 같고, 타입 인자가 다를 때 서로 어떤 관계를 갖는지 설명하는 개념이다. 


변성에는 다음 3가지 종류가 있다.

1. 공변성 covariant
   - A가 B의 하위타입일 때, `A[]`가 `B[]`의 하위타입인 경우이다.


2. 반공변성 contravariant
   - 말그대로 공변성의 반대 경우이다. 
   - 즉, B가 A의 하위타입일 때 `A[]`가 `B[]`의 하위타입인 경우이다.


3. 불공변성 invariant
    - A가 B의 하위타입이지만 `List<A>`와 `List<B>`는 아무 관계가 없는 경우이다.
    - 자바의 제네릭은 기본적으로 무변성을 따른다.

즉, `List<String>`과 `List<Object>`는 아무런 관계가 없다. (`String` 이 `Object` 의 하위 타입임에도 성립x)


## 한정적 와일드카드 타입 지원
- 이렇게 불공변성으로 인해, 상속 관계 등을 받아야하는 경우 문제가 생긴다.
- 따라서, 객체 지향의 특징을 어느정도 지원하기 위해 제네릭에서는 한정적 와일드카드 타입을 지원한다.
- `<? extends E>` 와 `<? super E>` 를 사용하여 유연성을 극대화할 수 있다.


## PECS
어떤 와일드 카드를 사용해야하는지 기억하려면, PECS 공식이 도움이 된다.


#### PECS : producer - extends, consumer-super


즉, 매개변수화 타입 T가 생산자인 경우 상한 경계 와일드 카드`<? extends E>`를 사용하고, 소비자이면 하한 경계 와일드카드`<? super E>`를 사용하라는 것이다.


### 예시
```java
// Code 1: 한정적 와일드카드 타입을 사용하지 않은, 경직된 메서드
public static <E extends Comparable<E>> E max(List<E> c) {
  if (c.isEmpty()) {
    throw new IllegalArgumentException("collection is empty");
  }
  E result = null;

  for (E e : c) {
    if (result == null || e.compareTo(result) > 0) {
      result = Objects.requireNonNull(e);
    }
  }

  return result;
}

```


```java
// Code 2: 한정적 와일드카드 타입을 적용하여 유연한 사용 가능 + 타입 한계 완화
public static <E extends Comparable<? super E>> E max(List<? extends E> c) {
    if (c.isEmpty()) {
        throw new IllegalArgumentException("collection is empty");
    }
    E result = null;

    for (E e : c) {
        if (result == null || e.compareTo(result) > 0) {
            result = Objects.requireNonNull(e);
        }
    }

    return result;
}
```


`List<? extends E> c`
생산자: 메서드 내부에서 c 에서 값을 꺼내(E를 생산) 사용만 하므로 extends 사용


`Comparable<? super E>`
소비자: `compareTo` 메서드의 파라미터(`? super E`)는 E를 소비하는 쪽이므로 super 사용


comparable이 언제나 소비자라고 하는 이유는 비교할 때 항상 값을 소비(값을 인자로 넘기기)하기 때문이다.
`E.compareTo(x)` 에서 x 는 `E` 자신 또는 `E`의 슈퍼타입이 될 수 있어야 안전하다.


### 생산자, 소비자 구분 방법 정리
모든 것은 `E`를 중심으로 생각하면 된다.


| 역할              | 타입 위치                   | 실제 사용 예                           | 왜 그 와일드카드인가?                  |
| --------------- | ----------------------- | --------------------------------- | ----------------------------- |
| 생산자(생산된 값 꺼내기)  | `List<? extends E> c`   | `for (E e : c)`로 `c`에서 `E` 꺼냄       | “리스트에서 꺼낼 요소”이므로 `extends` 적용 |
| 소비자(값을 인자로 넘기기) | `Comparable<? super E>` | `e.compareTo(result)` 에 `result` 전달 | “메서드에 넘겨주는 값”이므로 `super` 적용   |


---
## private static helper 메서드
- 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.
- 와일드카드 타입의 실제 타입을 알려주는 private helper 메서드를 작성하여 안전을 추구할 수 있다.
- 클라이언트는 복잡한 helper 메서드의 존재를 모른 채, 깔끔히 컴파일되는 혜택을 누릴 수 있다.


```java
// Code 1: 기존 swap 코드
public static void swap(List<?> list, int i, int j){
    list.set(i, list.set(j, list.get(i)));
};

// Code 2: helper 사용한 swap 코드
// 클라이언트에게는 와일드카드 타입으로 API를 열어놓고, swapHelper가 안전하게 타입을 한정해주는 방식
// 클라이언트는 타입 파라미터가 들어가지 않은 깔끔한 시그니처를 본다.
public static void swap(List<?> list, int i, int j){
    swapHelper(list, i, j);
};

// 이 과정을 와일드카드 캡처(wildcard capture)라고 한다.
public static <E> void swapHelper(List<E> list, int i, int j){
    list.set(i, list.set(j, list.get(i)));
};
```


---
## Summary
- 조금 복잡하더라도 와일드카드 타입을 적용하면 API가 유연해진다.
- 널리 쓰일 오픈 라이브러리를 작성할 때, 와일드카드 타입을 적절히 사용해야한다.
- PECS 공식을 기억하자: producer - extends, consumer-super
  - Comparable과 Comparator는 모두 소비자라는 사실을 잊지 말자.