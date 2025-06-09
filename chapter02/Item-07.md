# Item 07. 다 쓴 객체 참조를 해제하라
## GC가 있어도 메모리 누수에 신경써야 한다.
- 모든 객체 정리를 GC 가 해줄거라고 믿으면 안된다.
- 본인이 메모리를 관리하고 있는 코드를 사용하는 객체는 해당 부분을 인지하고 명시적으로 해제해야하는 경우가 생긴다.
- But, `null` 을 써서 직접 명시적으로 해제하는 방식은 많이 사용하지 않으므로 주의하자.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = 0;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    // 원소를 위한 공간을 적어도 하나 이상 여유를 두며, 늘려야하는 경우 두배 이상 늘린다.
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```


- 위 코드에서 문제점은 `pop()` 메서드에 있다.
- 스택에서 원소를 꺼낼 때, 꺼낸 객체를 가비지 컬렉터가 회수하지 않는다.
- 이 스택 자체가 꺼낸 객체의 다 쓴 참조를 여전히 가지고 있기 때문이다. (다 쓴 참조: 앞으로 사용하지 않지만 참조는 되어있음)
- 코드를 아래처럼 명시적으로 `null` 처리를 하게 고치면 참조 해제가 된다.


```java
public Object pop() {
    if (size == 0) 
        throw new EmptyStackException();
        
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
```


- 또한, `null` 처리를 명시적으로 하여, 더이상 참조할 수 없는 공간임을 나타내면 이곳에 다시 접근하려고 하는 비정상적인 시도가 있을 때 막을 수도 있다.
- 그러나, 모든 객체를 사용한 후에 바로 `null` 처리 할 필요는 없다. (코드가 지저분해짐)
- 객체 참조를 `null` 처리하는 것은 예외적인 경우에 시도해야 한다.
  - 일반적인 경우는 다 쓴 참조를 유효 범위(scope) 밖으로 밀어내는 방법이 좋다.
  - 명시적인 `null` 처리는 자기 메모리를 직접 관리하는 클래스에서, 메모리 누수가 발생할 위험이 있을 때 하는 것이 좋다.


## 캐시로 인한 메모리 누수
- 캐시로 인한 메모리 누수도 조심하자. (캐싱 필요 없는데 유지하는 경우)
- 해법 중 하나는, 엔트리가 살아 있는 캐시가 필요한 상황에서 `WeakHashMap`을 사용해 캐시를 만드는 것이다.
  - 다 쓴 엔트리는 즉시, 자동으로 제거된다.


- 그러나 다른 어려운 점도 있는데, "캐시 엔트리의 유효 기간을 정확히 정의하기 어렵다는 것"
- 일반적으로 사용하는 방법은, 시간이 지날수록 엔트리의 가치를 떨어트리는 방식이다.
- 이 때, 쓰지 않는 엔트리를 종종 청소해야 하는 상황이 있다. (`Scheduled ThreadPoolExecutor` 등)
- 백그라운드 스레드나 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하기도 한다.
  - `LinkedHashMap`은 `removeEldestEntry` 메서드를 사용해서 후자 방식으로 처리한다.


### WeakHashMap
- `WeakHashMap`은 키를 약한 참조(WeakReference)로 저장하는 Map 구현체다.
- key 객체가 더 이상 강한 참조로 유지되지 않으면, GC 대상이 되고, 해당 엔트리도 자동으로 제거된다.
- 이로 인해 캐시, 리스너 테이블, 내부 참조용 맵 등에서 메모리 누수 방지에 유용하게 쓰인다.


```java
Map<Object, String> map = new WeakHashMap<>();

Object key = new Object();
map.put(key, "value");

System.out.println(map.size()); // 1

key = null; // 강한 참조 제거
System.gc(); // GC 유도

Thread.sleep(1000); // 대기 (GC 시간 확보)
System.out.println(map.size()); // 0 (GC 후 제거됨)
```
- key 객체에 대한 강한 참조가 제거되면 HashMap에 들어있는 값도 필요 없어지기 때문에 자동으로 제거된다.



## Listener or CallBack 으로 인한 메모리 누수
- 클라이언트가 콜백(리스너)을 해지하지 않고 계속 등록 하면, 이벤트 소스가 참조를 계속 유지하게 되어 GC가 수거하지 못한다.
- 이 때, 콜백을 약한 참조(weak reference) 로 저장하면 GC가 즉시 수거해간다.
  - ex. `WeakReference`를 활용하거나 `WeakHashMap`에 등록


## Summary
- 메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 경우도 있다.
- 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 여러 디버깅 도구를 동원해야 발견되기도 한다.
- 메모리 누수가 될 만한 부분을 미리 익혀놓은 후, 예방하는 것이 제일 중요하다.