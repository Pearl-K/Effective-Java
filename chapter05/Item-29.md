# Item 29. 이왕이면 제네릭 타입으로 만들라
- 제네릭 타입을 새로 만드는 일은 어렵

---
## 1. 일반 클래스를 제네릭 클래스로 만들기
- 클래스 선언에 타입 매개변수를 추가하여 제네릭 클래스로 만들 수 있다.
- 그러나 제네릭 클래스로 변경하면서 생길 수 있는 오류가 있다.
  - 배열을 사용하는 코드를 제네릭으로 변경하려고 할 때 생기는 문제이다.


### 방법 1. 제네릭 배열 생성 금지 제약 우회, `@SuppressWarnings` 사용하여 경고 숨기기
- 비검사 형변환이 안전한지 직접 증명하고 제네릭 배열 생성 금지 제약을 우회할 수 있다.
- 우회하여 생기는 경고를 어노테이션으로 막아 경고를 숨길 수 있다.
- 현업에서 실제로 선호하는 방식이지만, 우회로 인해 생기는 힙 오염 가능성 때문에 이 방법을 피하는 사람도 있다. (방법 2 사용)


```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 15;

    // elements 배열은 push(E)로 넘어온 E 인스턴스만 담는다.
    // 따라서, 타입 안정성을 보장하지만, 런타임 타입은 E[]가 아닌 Object[]이다.
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }

        E result = elements[size--];
        elements[size] = null;
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

### 방법 2. `Object[]` 사용해서 타입 변경
- 이 방법은 배열에서 원소를 읽을 때마다 형변환을 해주며, 가독성도 이전 방법보다 좋지 않다. 
- 하지만, 힙 오염을 일으키지 않는다는 점에서 채택하는 사람들이 있다.


```java
public class Stack<E> {
    // private으로 저장
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 15;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }

        // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
        @SuppressWarnings("unchecked") E result = (E) elements[size--];
        elements[size] = null;
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

---
## 타입 매개변수에 제약을 두는 제네릭 타입: 한정적 타입 매개변수
- `java.util.concurrent.DelayQueue` 클래스의 선언을 보자. 
  - `class DelayQueue<E extends Delayed> implements BlockingQueue<E>`
- 여기서 한정적 타입 매개변수를 사용하여, `Delayed` 클래스의 하위 타입만 받도록 선언하고 있다.
- 이렇게 하면, `DelayedQueue` 원소에서, `Delayed` 클래스의 메서드를 바로 호출할 수 있기 때문에 안전하고 편하게 쓸 수 있다.

---
## Summary
- 클라이언트에서 직접 형변환해야 하는 타입보다, 제네릭 타입이 더 안전하고 쓰기 편하다.
- 새로운 타입을 설계할 때, 형변환 없이도 사용할 수 있도록 하라. (이렇게 하려면 제네릭 타입으로 만들어야 하는 경우가 많음)
- 기존 타입 중, 제네릭이었어야 하는게 있다면 제네릭 타입으로 변경하자.
- 기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자를 편하게 해주는 방법이다.
