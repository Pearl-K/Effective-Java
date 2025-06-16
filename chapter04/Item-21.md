# Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라
- Java8+ 이전에는 인터페이스에 메서드를 추가할 방법이 없었다.
- Java8+ 이후, 디폴트 메서드가 추가되면서 기존 구현체를 깨뜨리지 않고 인터페이스에 메서드를 추가할 수 있게 되었다.
- 그러나 이를 사용할 때 조심해서 사용해야 한다.


## 디폴트 메서드를 사용할 때 주의할 점

### 1. 기존 구현체가 안전하게 동작할지 반드시 고려

* 디폴트 메서드는 기존 구현체에도 자동으로 적용된다.
* 그러나 모든 기존 구현체가 디폴트 구현과 호환된다는 보장은 없다.
* 예: `removeIf()`는 내부 동기화가 필요한 클래스에서 문제를 일으킬 수 있다.


1. Java8 Collections 기본 구현

```java
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
        if (filter.test(each.next())) {
            each.remove(); // 여기서 remove()는 iterator 기반 remove
            removed = true;
        }
    }
    return removed;
}

```
- 동기화 전혀 없음
- 멀티스레드 환경에서는 `iterator()`로 순회하면서 동시에 구조가 바뀌면 `ConcurrentModificationException` 발생 가능


2. Apache Commons 4.4의 `SynchronizedCollection`의 명시적 override


```java
public boolean removeIf(Predicate<? super E> filter) {
    synchronized(this.lock) {
        return this.decorated().removeIf(filter);
    }
}
```

- `this.lock`으로 전체 연산을 원자적으로 감싼다.
- `decorated()`는 실제 내부 컬렉션 (ArrayList, Set 등)을 반환
- 동기화를 강제함으로써 `ConcurrentModificationException` 방지
- 그러나 이전 버전(4.3이하)을 사용하는 경우 removeIf를 재정의하지 않아, 디폴트 구현을 물려받는다.
- 따라서, 모든 메서드 호출을 알아서 동기화해주지 못하게 된다.
- `removeIf()`의 구현은 동기화에 대해 모르므로 락 객체를 사용할 수 없다.
- `SynchronizedCollection` 인스턴스를 여러 스레드가 공유하는 환경에서 `removeIf를` 호출하면 `ConcurrentModificationException이` 발생하거나 다른 예상치 못한 결과를 낳는다.



---

### 2. 디폴트 메서드는 구현 세부사항(lock, 상태 등)을 알 수 없다

* 디폴트 메서드는 클래스가 아닌 인터페이스에서 정의되기 때문에 해당 클래스의 상태(예: 락 객체)에 접근할 수 없다.
* 따라서, 동기화, 상태 공유, 리소스 관리가 필요한 클래스는 디폴트 메서드의 일반적인 구현을 안전하게 사용할 수 없다.
* 디폴트 메서드는 인터페이스 수준의 일반 논리만 작성 가능하며, 세부 동작은 구현체마다 다를 수 있다.


---

### 3. 컴파일은 되더라도 런타임 오류가 발생할 수 있다

* 컴파일러는 디폴트 메서드가 안전한지까지는 판단하지 못함.
* 따라서, 구현체의 내부 상태/제약을 고려하지 않고 메서드가 실행될 수 있다. 
* 특히, Java 8 이전에 작성된 구현체에서는 더 큰 리스크

---

### 4. 꼭 필요한 경우가 아니라면 기존 인터페이스에 디폴트 메서드 추가는 피하라

* 라이브러리처럼 이미 배포된 API에 디폴트 메서드 추가는 위험하다.
* 신중하게, 많은 테스트를 거쳐서 결정해야 하며, 하위 호환성을 철저히 고려해야 한다. 
* 리팩토링 편의를 위해 디폴트 메서드를 남용하면 오히려 API 품질이 나빠질 수 있다.

---

### 5. 디폴트 메서드는 "추가" 용도로만 사용하라

* 기존 메서드를 제거하거나 시그니처를 바꾸는 "수정" 용도로 사용하면 안 된다.
* 디폴트 메서드는 호환성을 지키며 **기능을 확장할 때만** 사용해야 한다.

> 기존 메서드를 디폴트 메서드로 대체하는 건 API 안정성에 위험요소

---

## Summary
- 디폴트 메서드는 신중히 사용해야 하며, 기존 구현체의 동작을 깨뜨리지 않도록 철저히 고려해야 한다.


