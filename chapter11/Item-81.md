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


---
## 2. concurrent collection 동시성 컬렉션
- `List`, `Queue`, `Map` 과 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션을 말한다.
- 동시성 컬렉션은 (개발자들이 수동으로)동기화한 컬렉션을 낡은 유산으로 만들어버렸다.


### ConcurrentHashMap
- 동시성을 보장해주는 HashMap
- 과거의 `Collections.synchronizedMap` 보다 `ConcurrentHashMap`을 사용하는 것이 훨씬 성능에 좋다.




### BlockingQueue
- `Queue`를 확장한 동시성 컬렉션
- 작업이 성공적으로 완료될 때까지 기다리도록 확장되었다.
- 작업 큐(생산자-소비자 큐)로 쓰기 적합하며, `ExecutorService` 구현체에서 작업들을 처리하는 작업 큐로 사용된다.



---
## 3. 동기화 장치
- 동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 해주어, 서로 작업을 조율할 수 있게 만들어준다.
- 자주 쓰이는 동기화 장치는 `CountDownLatch` 와 `Semaphore` 이다.



### CountDownLatch
- 일회성 장벽의 역할로, 하나 이상의 스레드가 또다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.
- `CountDownLatch`의 유일한 생성자는 `int`값을 받고, 이 값이 래치의 `countDown` 메서드를 몇 번 호출해야 대기 스레드를 깨우는지 결정한다.
    - 위 `countDown` 수는 `@Volatile` 키워드로 내부적으로 관리되며 모든 스레드가 동일한 값을 볼 수 있다.


이를 이용해서 유용한 기능을 쉽게 구현해보자.


```java
// 동시 실행 시간을 재는 간단한 프레임워크
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(concurrency);

    // concurrency -> 동시성 수준 만큼 반복문
    for (int i=0; i < concurrency; i++){
        executor.execute(() -> {
            // 타이머에 준비를 마쳤음을 알림
            ready.countDown();

            try {
                start.await();
                action.run();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                done.countDown();
            }
        });
    }

    ready.await(); // 모든 작업자가 준비될 때까지 기다림
    long startNanos = System.nanoTime();
    start.countDown(); // 작업자들 깨우기
    done.await(); // 모든 작업자가 일을 끝마칠 때까지 기다림
    return System.nanoTime() - startNanos;
}
```


위 코드의 흐름을 도식화하면 아래와 같다.


```arduino
────────────────────────────────────────────────────────────────────
  TIME → →
────────────────────────────────────────────────────────────────────

[Main Thread]
    |
    | 생성: ready = new CountDownLatch(N)
    | 생성: start = new CountDownLatch(1)
    | 생성: done  = new CountDownLatch(N)
    |
    |-----------------------\
    |                       | (반복 N회)
    |   executor.execute( worker i )  →→→→→→→→→→→→→→→→→→→→→→→→→→
    |                       |          ⮩ ⬅ 이 시점에 Worker 스레드 등록
    |<----------------------/                    (Runnable 전달)
    |
    | ready.await()  ← 모든 worker가 준비될 때까지 대기
    |
    |                           [Worker Thread i]
    |                             ┌─────────────────────────────┐
    |                             │ ready.countDown()           │  // 준비 완료 알림
    |                             │ start.await()               │  // 메인 스레드 신호 대기
    |                             └─────────────────────────────┘
    |
    | (모든 worker가 ready.countDown 완료 → ready.await() 해제)
    |
    | startTime = System.nanoTime()
    | start.countDown()  ← 모든 worker의 start.await() 해제
    |
    |                           [Worker Thread i]
    |                             ┌─────────────────────────────┐
    |                             │ action.run()                │  // 실제 작업 수행
    |                             │ done.countDown()            │  // 작업 완료 알림
    |                             └─────────────────────────────┘
    |
    | done.await() ← 모든 worker가 done.countDown() 할 때까지 대기
    |
    | return (System.nanoTime() - startTime)

────────────────────────────────────────────────────────────────────

```


### Semaphore
- `Semaphore`는 동시 접근할 수 있는 자원 수를 제한하는 데 사용되는 동기화 도구이다.
- 쓰레드 풀, 연결 제한, API 호출 제한 등 한정된 수의 리소스 접근 제어가 필요한 상황에서 유용하다.
- 특정 개수의 permit을 미리 정해두고, 스레드가 acquire()로 permit을 요청해 얻으면 진입, 얻지 못하면 대기하는 방식이다.


생성자와 파라미터 설명은 아래와 같다.


```java
public Semaphore(int permits)
public Semaphore(int permits, boolean fair)
```


| 파라미터              | 설명                                                                             |
| ----------------- | ------------------------------------------------------------------------------ |
| `permits`         | 허용할 동시 접근 스레드 수. 즉, 초기 permit 개수                                               |
| `fair` (optional) | `true`면 \*\*FIFO(공정한 순서)\*\*로 permit을 할당 (대기열 보장), `false`면 성능 우선 (기본값: false) |



---
## Legacy에서 `wait()` 와 `notify()` 를 다뤄야 하는 경우
### wait
- 스레드가 어떤 조건이 충족되기까지 기다리게 만들 때 사용한다.
- 락 객체의 wait 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다. (Synchronized 블럭 안에서)
- wait 메서드를 사용할 때 반드시 대기 반복문 관용구를 사용하자.
    - 반복문 밖에서 절대 호출하지 말자.
    - 이 반복문은 wait 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다.
    - 대기 전에 조건을 검사하여, 조건이 충족되면 wait 을 건너뛰어서 응답 불가 상태를 막는 것이다 (중요!)
- notify 와 wait의 순서도 조심해야 한다. 
    - 깨운 후에 다시 대기 상태에 들어가는 코드가 있다면, 스레드를 다시 깨울 수 있다고 보장할 수 없다.


### notify, notifyAll
- 되도록 notifyAll을 써서 모든 스레드를 깨우도록하자.
- 가끔 다른 스레드가 wait 공격을 하여 notify() 로만은 스레드를 깨울 수 없는 상태가 될 수도 있다.
- notifyAll을 쓰면 모든 스레드를 깨우는 것을 보장하므로 항상 정확한 결과를 얻을 수 있을 것이다.


---
## Summary
- 새로 작성하는 코드에는 `wait`와 `notify`를 사용하지 말자.
- 만약, 이들을 레거시 코드에서 다뤄야 한다면
    - `wait`는 항상 표준 관용구에 따라 `while`문 안에서 호출하자.
    - 일반적으로 `notify` 보다는 `notifyAll`을 사용하자.
    - 만약, `notify`를 써야하는 상황이라면 응답 불가 상태에 빠지지 않도록 각별히 주의하자.