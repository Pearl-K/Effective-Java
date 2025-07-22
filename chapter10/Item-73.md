# Item 73. 추상화 수준에 맞는 예외를 던져라

수행하려는 일과 관련 없어 보이는 예외가 튀어나올 경우, 당황스러울 것이다. 이런 일이 발생하는 이유는 메서드가 저수준의 예외를 처리하지 않고 바깥으로 전파할 때 종종 생긴다.


이런 경우, 단점이 크리티컬 한데 내부 구현 방식을 드러내어서 윗 레벨의 API를 오염시키기 때문에 이 문제를 피해야한다.


---
## Solution: Exception Translation (예외 번역)
상위 계층에서 저수준 예외를 잡아서 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다. 아래와 같은 구조로 예외 번역을 진행할 수 있다.


```java
try {
    // ... 저수준 추상화 이용
} catch (LowerLevelException e) {
    // 추상화 수준에 맞게 번역 진행
    throw new HigherLevelException(...);
}
```


실제로 위와 같은 구조를 자바 내부 `AbstractSequentialList` 에서 찾을 수 있다. `AbstractSequentialList` 는 `List` 인터페이스의 골격 구현으로, 아래 코드에서 수행된 예외 번역은 `List<E>` 인터페이스의 `get` 메서드 명세에 명시된 필수 사항이다.


```java
// i.next() 가 존재하지 않는 경우
// NoSuchElementException 이라는 저수준의 추상적인 예외를 고수준으로 다시 던진다.
public E get(int index) {
    ListIterator<E> i = listIteator(index);
    try {
        return i.next();
    } catch (NoSuchElementException e) {
        throw new IndexOutOfBoundsException("인덱스: " + index);
    }
}
```



---
## Exception-Chaining (예외 연쇄)
예외를 번역할 때, 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄를 사용하는 것이 좋다.


```java
try {
    // ... 저수준의 추상화
} catch (LowerLevelException cause) {
    // 저수준 예외 cause를 고수준 예외에 실어 보낸다.
    throw new HigherLevelException(cause);
}
```


예외 연쇄란, 문제의 근본 원인(cause)인 저수준의 예외를 고수준 예외에 실어서 보내는 방식이다. 별도의 접근자 메서드(`Throwable` 의 `getCause` 메서드)를 사용하여 언제든지 저수준 예외를 꺼내서 볼 수 있다.


고수준 예외의 생성자는(예외 연쇄용으로 설계된 생성자) 상위 클래스의 생성자에 해당 원인을 건네주어, 최종적으로 Throwable(Throwable) 생성자까지 건네지게 한다.


```java
// 예외 연쇄용 생성자
class HigherLevelException extends Exception {
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}
```


대부분의 표준 예외는 예외 연쇄용 생성자를 갖추고 있으며, 갖추고 있지 않다고 해도 `Throwable` 의 `initCause` 메서드를 이용해 원인을 직접 정하고 초기화해줄 수 있다.


예외 연쇄를 사용하면, `getCause` 메서드를 이용해 프로그램에서 문제의 원인을 접근할 수 있어서 정보가 명확히 전달되고, 원인과 고수준 예외의 StackTrace를 잘 통합해준다.



---
## 예외 번역, 예외 연쇄 남용 금지
정보를 제공한다는 좋은 기능을 보고, 무턱대고 예외를 전파하는 것은 프로그래밍 최적화 상 맞지 않다. 예외 번역이 우수한 방식이긴 하지만, 남용하면 안된다. (예외를 던지는 것 자체가 리소스가 큼)


가능하면 저수준 메서드가 반드시 성공하도록 하여, 아래 계층에서 예외가 발생하지 않도록 최대한 막는것이 좋다.


만약, 예외를 절대 피할 수 없는 상황이라면 API 호출자까지 전파하지 않도록 하자. 저수준의 예외가 발생했다면 그 상위 계층에서 예외를 조용히 처리하여 문제를 전파하지 않는 방식으로 대응할 수 있다.


이때 발생한 예외를 적절한 로깅 기능을 활용하여 기록하면, 프로그래머가 분석해 추가 대응을 할 수 있다.


---
## Summary
- 아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 곤란하다면 예외 번역을 사용하자.
- 이때 예외 연쇄를 이용하면 상위 계층에는 맥락에 어울리는 고수준의 예외를 던지면서, 근본 원인도 함께 알려줄 수 있어 오류 분석에 좋다.


