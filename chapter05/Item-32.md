# Item 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라
- 가변인수(varargs)는 메서드를 호출할 때 인수(파라미터)의 개수를 유연하게 지정할 수 있게 해 주는 문법이다.
- 이는 메서드에 넘기는 인수의 개수를 유연하게 조절할 수 있어 편리하지만, 구현 방식에서 발생하는 **힙 오염(heap pollution)** 문제로 인해 타입 안전성이 깨질 수 있다.

---

## 위험 예제 1: 힙 오염 발생

```java
static void dangerous(List<String>... stringLists) {
    List<Integer> ints = List.of(42);
    Object[] objs = stringLists;
    objs[0] = ints;                // (1) 힙 오염: Integer 리스트가 저장됨
    String s = stringLists[0].get(0); // (2) ClassCastException
}
```

* (1)에서 `Object[]`로 변환된 가변인수 배열에 다른 타입을 저장해 힙 오염이 발생한다.
* 결국 (2)에서 `String`이 아닌 `Integer`를 꺼내려 하며 런타임 에러가 발생한다.

---

## 위험 예제 2: 빈 배열 반환도 힙 오염

```java
static <T> T[] toArray(T... args) {
    return args;  // 경고: Possible heap pollution from parameterized vararg type
}

static <T> T[] pickTwo(T a, T b, T c) {
    switch (ThreadLocalRandom.current().nextInt(3)) {
        case 0: return toArray(a, b);  // warning: Unchecked generics array creation
        case 1: return toArray(a, c);
        case 2: return toArray(b, c);
    }
    throw new AssertionError();
}

// calling code
String[] attrs = pickTwo("좋은", "빠른", "저렴한");
// 런타임에 ClassCastException 발생
```

* 컴파일 시점에 varargs 배열의 타입이 `Object[]`로 소거되어, 호출 측에서 `String[]`에 저장하려 할 때 실패한다.

---

## 안전한 `@SafeVarargs` 사용 조건

1. **가변인수 배열에 아무것도 저장하지 않는다.**
2. **배열 참조를 외부에 노출하지 않는다.**
3. **재정의할 수 없는 메서드**(`static`, `final`, Java9부터는 `private`)만

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists) {
    result.addAll(list);
  }
  return result;
}
```

* varargs 배열 자체를 반환하거나 수정하지 않고, **읽기 전용**으로만 사용하므로 타입 안전.
* `@SafeVarargs`를 달아 컴파일 경고를 억제할 수 있다.

---

## 대안: List로 받기

```java
static <T> List<T> flatten(List<List<? extends T>> lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists) {
    result.addAll(list);
  }
  return result;
}

// 호출 예시
audience = flatten(List.of(friends, romans, countrymen));
```

* `@SafeVarargs` 없이도 컴파일러가 타입 안정성을 검증한다.
* 단점: 호출부가 다소 장황해지고, 내부에서 배열 대신 컬렉션을 사용해 퍼포먼스에 약간의 오버헤드가 있을 수 있다.

---

## Summary
* 가변인수와 제네릭은 궁합이 좋지 않다.
* 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하고, 배열과 제네릭의 타입 규칙이 서로 다르기 때문이다.
* 제네릭 varargs 매개변수는 타입 안전하지 않으나 허용은 된다.
  * 이를 사용하고자 한다면, 먼저 그 메서드가 타입 안전한지 확인이 필요하다.
  * 확인 후, `@SafeVarargs`를 달아 사용해야 한다.
