# Item 30. 이왕이면 제네릭 메서드로 만들라
- 클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다.

---
## 제네릭 메서드
- Raw Type을 사용하는 기존에 안전하지 않던 메서드들을 제네릭 메서드로 변경할 수 있다.
  - 직접 형변환 하지 않아도 오류나 경고 없이 컴파일 되어 편하게 사용할 수 있다.


```java
// 기존에 경고가 발생하던 Raw Type 사용 메서드
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1); // Raw Type
  result.addAll(s2);
  return result;
}

// 아래와 같이 제네릭 메서드로 만들면 타입 안전하고, 쓰기 쉽게 바뀐다.
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```


---
## 제네릭 싱클톤 팩토리 패턴
- 때때로, 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 한다.
- 제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다는 장점이 있다.
- 하지만, 이렇게 하려면 요청한 타입 매개변수에 맞게, 매번 그 객체의 타입을 바꾸는 정적 팩토리를 만들어야 한다.
  - 이 패턴을 제네릭 싱글톤 팩토리라고 한다.
  - `Collections.reversOrder` 같은 함수 객체나 `Collections.emptySet` 등 컬렉션 용으로 사용한다.
- 제네릭 싱글톤 팩토리는 단 하나의 불변 객체를, 요청된 그 어떤 제네릭 타입으로도 재사용할 수 있게 한다.

```java
// emptySet 예시
private static final List<?> EMPTY_LIST = new EmptyList<>();

@SuppressWarnings("unchecked")
public static <T> List<T> emptyList() {
    return (List<T>) EMPTY_LIST;   // 단 한 번 생성된 객체를 여러 타입으로 재사용
}

// 호출 예시
List<String> s1 = Collections.emptyList();   // List<String>
List<Integer> s2 = Collections.emptyList();  // List<Integer>

```


### 작성 요령 & 주의점
- 단 하나의 `static final` 인스턴스 보관.
- 외부에는 `<T>`를 선언한 정적 팩터리만 노출한다.
- 내부 캐스팅에는 `@SuppressWarnings("unchecked")` 를 붙이되 메서드 본문 한 곳으로 범위를 국한하는 것이 좋다.


---
## 재귀적 타입 한정
- 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정하는 것을 말한다.
- 주로, 타입의 자연적 순서를 정하는 `Comparable` 인터페이스와 함께 쓰인다.


```java
// Comparable 예시
public class MyNumber implements Comparable<MyNumber> {
    private final int value;

    @Override
    public int compareTo(MyNumber o) {
        return Integer.compare(value, o.value);
    }
}

```



---
## Summary
- 제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하고 사용하기도 쉽다.
- 타입과 마찬가지로, 메서드도 형변환 없이 사용할 수 있는 편이 좋다.
- 많은 경우, 위 내용을 지원하려면 제네릭 메서드가 되어야 한다.
- 즉, 형변환을 해줘야하는 기존 메서드를 제네릭하게 만들어 편리함을 추구하자는 것이다.
  - 기존 클라이언트를 그대로 둔 채, 새로운 사용자의 삶을 더 편하게 해줄 것이다.