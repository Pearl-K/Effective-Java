# Item 83. 지연 초기화는 신중히 사용하라
## 지연 초기화를 사용하는 경우
대부분의 경우 지연 초기화를 하지 않는 것이 좋지만, 지연 초기화를 사용할 수 밖에 없는 상황이 있다.


- 무거운 객체를 사용할 일이 거의 없는데 굳이 미리 만들 필요 없을 때
    - 즉, 객체를 미리 만들 필요가 없을 때 (비용 등)
- 순환 참조를 피하고 싶을 때
- 클래스 로딩 시점과 무관하게 초기화하고 싶을 때


---
## 지연 초기화를 안전하게 사용하는 방법
### 1. double-checked locking (이중 검사)
인스턴스 필드에서 안전한 지연 초기화를 한다.


```java
private volatile SomeType field;

SomeType getField() {
    if (field == null) {
        synchronized (this) {
            if (field == null)
                field = new SomeType();  // 여기서 초기화가 일어난다
        }
    }
    return field;
}
```


- volatile 키워드가 반드시 있어야 한다.
- 동기화를 최소화하면서도 스레드 안전을 보장한다.
- 단, 코드가 복잡해지고 유지보수 부담이 커질 수 있다.



### 2. Lazy Initialization Holder 클래스
다음은 정적 필드의 안전한 지연 초기화 방식이다.


```java
private static class Holder {
    static final SomeType INSTANCE = new SomeType();
}

public static SomeType getInstance() {
    return Holder.INSTANCE;
}
```

- Holder 클래스는 getInstance()가 호출되기 전까지 로딩되지 않는다.
- JVM이 클래스 로딩 시점의 스레드 안전성을 보장한다.
- 단순하고 안전함 → 정적 필드엔 이걸 쓰는 것이 좋다.


### 3. 단일 검사
간혹 매번 초기화해도 상관 없는 필드들이 있다.


```java
SomeType getField() {
    return (field == null) ? new SomeType() : field;
}
```


- 필드가 불변하고, 생성 비용이 낮고, 매번 생성해도 문제 없을 때 고려하자.



---
## Summary
- 대부분의 필드는 지연시키지 말고 바로 초기화해야 한다.
- 다중 스레드 환경에서 LazyInit 이 위험하지 않은지 꼭 생각해봐야 한다.
- 성능이나 순환 참조 등을 막기 위해 꼭 지연 초기화를 써야하는 경우, 올바른 방법을 사용해야 한다.
    - 인스턴스 필드에는 이중 검사 관용구 사용하기
    - 정적 필드에는 지연 초기화 홀더 클래스 관용구 사용하기
    - 반복해서 초기화해도 괜찮은 필드에는 단일 검사 관용구도 고려 대상이다.