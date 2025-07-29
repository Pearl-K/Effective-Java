# Item 81. wait와 notify 보다는 동시성 유틸리티를 애용하라
과거에는 wait와 notify를 올바르게 사용하는 방법을 소개했지만, 현재는 이 둘을 사용해야할 이유가 줄었다.

> `wait()` 와 `notify()`는 기존에 무슨일을 했을까?
>
> `wait()`와 `notify()`는 Java에서 스레드 간 협력을 가능하게 해주는 기본적인 저수준 동기화 메커니즘이다.
> 
> 주로 공유 자원에 대한 조건 대기나 상태 변화 알림에 사용한다. 
> 예를 들어, 생산자-소비자 패턴에서 소비자가 큐가 비어 있으면 `wait()`으로 대기하고, 생산자가 데이터를 추가한 후 `notify()`로 소비자에게 알리는 식으로 사용한다.
>
> 그러나 이 메커니즘은 아래와 같은 문제점이 있다.
> 
>   1. 명시적 동기화 블록(`synchronized`) 안에서만 호출 가능
>   2. `notify()`가 임의의 스레드 하나만 깨우므로, 정확한 스레드 선택이 불가능
>   3. wait 조건 재확인 필요 (`while` 루프로 감싸야 함)
>   4. 복잡한 설계가 필요하고, 구현해야 할 게 많아서 디버깅이 어려움
>


따라서, wait와 notify를 사용해서 커스텀 구현 + 하드코딩 하기 보다, 이를 대신 처리해주는 고수준의 동시성 유틸리티를 사용하자.


---
## 고수준 유틸리티의 세 범주
1. 실행자 프레임워크 (ex. `ExecutorService`)
2. 동시성 컬렉션 (ex. `BlockingQueue`)
3. 동기화 장치 (ex. `Synchronizer`)


아래에서 각각 범주를 나눠서 좀 더 자세히 설명하겠다.


## 1. 실행자 프레임워크
- [Item-80](/chapter11/Item-80.md)에서 설명한다.



## 2. concurrent collection 동시성 컬렉션
- `List`, `Queue`, `Map` 과 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션을 말한다.


### ConcurrentHashMap


### BlockingQueue




## 3. synchronizer 와 같은 동기화 장치


### synchronizer


### CountDownLatch


### Semaphore


---
## Legacy에서 `wait()` 와 `notify()` 를 다뤄야 하는 경우



---
## Summary
- 새로 작성하는 코드에는 `wait`와 `notify`를 사용하지 말자.
- 만약, 이들을 레거시 코드에서 다뤄야 한다면
    - `wait`는 항상 표준 관용구에 따라 `while`문 안에서 호출하자.
    - 일반적으로 `notify` 보다는 `notifyAll`을 사용하자.
    - 만약, `notify`를 써야하는 상황이라면 응답 불가 상태에 빠지지 않도록 각별히 주의하자.