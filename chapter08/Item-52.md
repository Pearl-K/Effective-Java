# Item 52. 다중정의(OverLoading)는 신중히 사용하라

다중정의(OverLoading) 된 메서드 중 어느 메서드를 호출할지가 **컴파일 타임**에 정해진다.
반면, 재정의(Overriding)한 메서드는 **런타임**에 선택된다.

---
## 재정의(Overriding) vs 다중정의(Overloading)

- **재정의(Overriding)**: 상위 클래스의 메서드를 하위 클래스에서 재정의한 경우, **실제 인스턴스의 타입(런타임 타입)**에 따라 호출될 메서드가 결정된다. (동적 디스패치)
- **다중정의(Overloading)**: 같은 이름의 여러 메서드가 있을 경우, **매개변수의 컴파일 타임 타입**에 따라 호출될 메서드가 결정된다. (정적 디스패치)


아래 코드는 `CollectionClassifier` 예시의 문제를 명확히 보여준다.
`collections` 배열의 컴파일 타임 타입은 `Collection<?>`이다.
따라서 for문 안에서 `classify` 메서드를 호출할 때, 항상 `classify(Collection<?> c)` 메서드만 호출된다.
각 원소의 런타임 타입(`HashSet`, `ArrayList`)은 무시된다.


```java
public class CollectionClassifier {

    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }
}

public class OverloadingTest {
    @Test
    void overloadingTest() {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        // for문 안에서 c의 컴파일 타임 타입은 항상 Collection<?> 이다.
        for (Collection<?> c : collections) {
            // 항상 classify(Collection<?> c)만 호출된다.
            System.out.println(CollectionClassifier.classify(c));
        }
        // 기대와 달리 "그 외"만 세 번 출력된다.
    }
}
```

---

## 다중정의의 혼란스러운 예시: `List.remove`

`List<Integer>`에서 특정 정수를 제거하려고 할 때, 다중정의된 `remove` 메서드는 혼란을 야기할 수 있다.

- `remove(int index)`: 특정 인덱스의 원소를 제거한다.
- `remove(Object o)`: 특정 객체와 일치하는 첫 번째 원소를 제거한다.

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));

// list.remove(2)를 호출하면?
// 컴파일러는 int 타입의 2를 보고 remove(int index)를 선택한다.
// 따라서 인덱스 2의 값인 '3'이 제거된다.
list.remove(2); 
System.out.println(list); // [1, 2, 4, 5]

// 값 '2'를 제거하고 싶다면?
// Integer.valueOf(2) 또는 (Integer) 2로 형변환하여
// remove(Object o) 메서드가 선택되도록 해야 한다.
list.remove(Integer.valueOf(2));
System.out.println(list); // [1, 4, 5]
```

이처럼 다중정의는 개발자의 의도와 다르게 동작할 여지가 있어 API 사용을 어렵게 만들 수 있다.


---
## 다중정의, 언제 사용해야 할까?

무조건 피해야 하는 것은 아니다. 다중정의를 사용하는 좋은 예시도 있다.
**"서로 다른 타입의 매개변수를 받지만, 근본적으로 같은 기능"**을 수행할 때 유용하다.


예를 들어, `String.valueOf()` 메서드는 다양한 기본 타입을 받아 문자열로 변환하는 다중정의 메서드의 좋은 예시다.
어떤 타입을 넘기든 "해당 값을 문자열로 바꾼다"는 동일한 기능을 수행할 것이라고 예측할 수 있다.


```java
String.valueOf(true);   // boolean
String.valueOf(10);     // int
String.valueOf(10.5);   // double
String.valueOf('a');    // char
```


---
## 다중정의가 위험할 가능성이 있는 상황에서, 피하는 방법
### 1. 메서드 이름 다르게 짓기

가장 좋은 방법은 다중정의 대신, **메서드 이름을 다르게 짓는 것**이다.
`ObjectOutputStream`의 `write` 메서드들이 좋은 예시다.

- `writeBoolean(boolean val)`
- `writeByte(int val)`
- `writeChar(int val)`


만약 이 메서드들이 모두 `write`로 다중정의되었다면, `CollectionClassifier`와 같은 문제를 겪었을 것이다.


### 2. `instanceof`로 명시적 타입 검사

`CollectionClassifier` 예시처럼 하나의 메서드 안에서 `instanceof`를 사용하여 타입을 검사하고 분기 처리하는 방법도 있다.
이는 다중정의의 문제를 피할 수 있지만, 코드가 다소 지저분해질 수 있다.

```java
public class CollectionClassifier {
    public static String classify(Collection<?> c) {
        return c instanceof Set ? "집합" :
               c instanceof List ? "리스트" : "그 외";
    }
}
```


### 3. 생성자 대신 정적 팩터리 메서드 사용 (Item 1)

생성자는 이름을 다르게 지을 수 없으므로, 같은 시그니처를 갖는 생성자를 여러 개 만들 수 없다.
이때 **정적 팩터리 메서드**를 사용하면, 이름을 자유롭게 붙일 수 있어 다중정의를 피하고 의도를 명확히 드러낼 수 있다.


---
## Summary
- 재정의(Overriding)는 **런타임**에 실제 인스턴스 타입에 따라, 다중정의(Overloading)는 **컴파일 타임**에 선언된 타입에 따라 결정된다.
- 다중정의는 API 사용을 혼란스럽게 만들 수 있으므로 신중하게 사용해야 한다. 특히 매개변수 수가 같을 때는 피하는 것이 좋다.
- 다중정의를 피하는 가장 좋은 방법은 **메서드 이름을 다르게 짓는 것**이다.
- 헷갈릴 만한 다중정의가 이미 존재한다면, 정확한 타입으로 형변환하여 올바른 메서드가 선택되도록 유도해야 한다.