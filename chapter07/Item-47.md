# Item 47. 반환 타입으로는 스트림보다 컬렉션이 낫다
Stream은 Iterable과 달리 반복을 지원하지 않아 for-each문으로 순회할 수 없다.
Stream 인터페이스는 Iterable 인터페이스를 확장(extend)하지 않았기 때문이다.

## Stream이 Iterable을 확장하지 않은 이유
`Stream`이 `Iterable`을 확장했다면 모든 스트림을 for-each로 반복할 수 있었을 것이다.
하지만 `Stream`은 **일회용**이기 때문에 `Iterable`을 확장하지 않았다.

- **Iterable**: 여러 번 반복할 수 있을 것으로 기대된다.
- **Stream**: 한 번 사용하면 닫히므로, 두 번 이상 반복할 수 없다.

만약 Stream이 Iterable을 확장했다면, 개발자들은 자신도 모르게 스트림을 여러 번 순회하려다 `IllegalStateException`을 마주하게 될 것이다.

```java
// Stream이 Iterable을 확장했다고 가정하자.
Stream<String> stream = Stream.of("a", "b", "c");

// 첫 번째 순회 (성공)
for (String s : stream) {
    System.out.println(s);
}

// 두 번째 순회 (IllegalStateException 발생!)
// 이미 스트림이 닫혔기 때문에 다시 순회할 수 없다.
for (String s : stream) {
    System.out.println(s);
}
```

이러한 혼란을 방지하기 위해 Stream은 Iterable을 확장하지 않는다.
그러나 만약, 사용자가 Iterable이 필요하다면 Stream을 반환했을 때 불편해진다.


## Adapter를 사용해서 Stream<E> 를 Iterable<E>로 중개해주기
```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
    // 프로세스 처리..
}
```

이렇게 어댑터를 사용하여 어떤 스트림도 for-each 문으로 반복할 수 있다.
그러나 이렇게 일일히 어댑터를 구현해서 사용하기는 어렵다.


## 사용처에 맞게 반환하기
- 메서드가 오직 스트림 파이프라인에서만 쓰인다면, 마음놓고 스트림을 반환해도 된다.
- 반대로, 반복문에서만 쓰이는 경우 Iterable 반환이 필요하다.
- 만약, 공개 API 라면 두 방식을 모두 고려해야 한다.


### Collection Interface
- Collection 인터페이스는 Iterable 하위 타입이고, stream 메서드도 제공하니 반복과 스트림을 동시에 지원하는 방법이 된다.
- 따라서, 원소 시퀀스를 반환하는 공개 API 에서는 Collection이나 Collection의 하위 타입을 쓰는게 일반적으로 최선이다.


### Arrays
- Arrays 역시 Arrays.asList 와 Stream.of 메서드로 쉽게 반복과 스트림을 지원한다.


---
## Collection 반환 시 주의할 점
1. 반환할 시퀀스가 크다면 메모리에 올리는 걸 주의해야 한다.
2. 반환할 시퀀스가 크지만, 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현할 수도 있다.
    - AbstractList를 이용하여 훌륭한 전용 컬렉션 구현 가능



---
## Stream 반환할 때
### 부분 리스트를 스트림으로 변환하여 처리할 때
명확한 변환 과정이 있고, 평가시점이 명확할 때 스트림을 이용하면 좋다.

```java
static class SubLists {
    public static <E> Stream<List<E>> of(List<E> list) {
        Stream<List<E>> prefixes = prefixes(list);
        System.out.println("prefixes = " + prefixes.toList());

        Stream<List<E>> suffixes = suffixes(list);
        System.out.println("suffixes = " + suffixes.toList());

        return Stream.concat(prefixes(list), suffixes(list));
    }

    private static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                .mapToObj(end -> list.subList(0, end));
    }

    private static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.range(0, list.size())
                .mapToObj(start -> list.subList(start, list.size()));
    }
}
```


## Summary
- 반환 타입으로, 스트림보다 컬렉션이 낫다.
- 원소 시퀀스를 반환하는 메서드를 작성할 때, 이를 스트림으로 처리하기 원하는 사용자와 반복적으로 처리하려는 사용자 모두 있음을 염두에 둬야한다.
