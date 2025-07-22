# Item 78. 공유 중인 가변 데이터는 동기화해 사용하라
## Synchronized와 모니터 락
많은 프로그래머들은 동기화, `Synchronized` 키워드를 해당 메서드나 블록을 한번에 한 스레드만 수행하도록 막는 **"배타적 실행"** 용도로만 생각한다.


그러나 동기화에는 중요한 기능이 더 있는데, 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 제대로 확인하지 못할 수 있다.


즉, **스레드 간의 통신 용도** 로도 쓰인다. 이 두 가지는 동기화를 다룰 때 둘 다 만족하거나, 둘 중 하나만 만족하거나 하는 상황을 구분해야 하므로 꼭 알아두어야 한다.


동기화 동작을 다시 정리하면, 일관성이 깨진 상태를 볼 수 없게 하는 것은 물론이고, 동기화된 메서드나 코드 블럭에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.



## 변수를 읽고 쓰는 원자적 동작
Java 언어 명세상, long 과 double 외에 변수를 읽고 쓰는 동작은 원자적이다. 여러 스데르가 같은 변수를 동기화 없이 수정하고 있더라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 것이다.


그러나 이렇게 원자적 데이터를 읽고 쓸 때, 동기화가 필요하지 않다는 뜻은 아니다. 왜냐하면 위에서 설명했던 **스레드 간 통신** 문제가 있기 때문이다.


스레드가 필드를 읽을 때, 항상 수정이 완전히 반영된 값을 얻는 것은 맞다. 그러나 한 스레드가 저장한 값이 다른 스레드에게 "보이는지"(메모리 가시성 문제)는 보장하지 않는다.


따라서 이 가시성 문제를 해결하기 위해서는 동기화가 필요하다. **(배타적 실행 & 스레드 사이 안정적 통신)**


## 메모리 가시성 문제
- 한 스레드에서 변경한 값을 다른 스레드에서 즉시 읽어올 수 없을 때 생기는 문제이다.


```java
// Code 1. 메모리 가시성 문제
// 스레드를 멈출 때 Thread.stop() 을 사용하면 안되기 때문에
// 아래 방식으로 다루고자 했으나, 적절한 시기에 스레드가 멈추지 않는다.
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread background = new Thread(() -> {
            int i = 0;
            while (!stopRequested) i++;
        })

        background.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```


- 제대로 동기화 하지 않으면, 메인 스레드가 수정한 값을 백그라운드 스레드가 읽지 못한다. (백그라운드 스레드가 언제 보게 될 지 보증할 수 없음 - 캐시 구조 때문에)
- [메모리 가시성 문제 - 도식 참고](https://github.com/Pearl-K/operating-system/blob/master/week4_concurrency/java_concurrent_package/Volatile_n_AQS.md)



```java
// Code 2. 적절한 동기화를 하면 기대한 시간에 작업이 종료된다.
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread background = new Thread(() -> {
            int i = 0;
            while (!stopRequested) i++;
        })

        background.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

- 위 Code 2 에서는 쓰기 메서드와 읽기 메서드를 모두 동기화했다.
- 쓰기와 읽기가 모두 보장되지 않으면 동작을 보장하지 않는다는 점을 주의하자.
- 위 코드에서는 배타적 수행 & 스레드 간 통신 두 가지 기능 중에 "통신" 목적만 사용했다.
- 스레드간 통신(메모리 가시성 보장)만 보장하면 될 때, `volatile` 키워드를 쓸 수 있다.


## `volatile` 키워드
- 이 키워드는 메모리 가시성을 보장하여, 어느 스레드에서든지 메인 메모리에 직접 접근해서 값을 읽도록 한다.
- 스레드 간 일관성이 깨지는 문제가, 스레드 간 서로 다른 cpu 캐시 계층에 저장된 값을 읽기 때문이라 이 메모리 가시성을 하나로 통일 시키는 방법이다.
- 물론, 캐시 메모리를 활용하지 못하고 계속 메인 메모리에 접근한다는 문제 때문에 약간의 성능 저하가 있다.


```java
// 이 코드는 스레드 간 통신(메모리 가시성)만 지원되면 정상 동작한다.
// 그래서 stopRequested 를 volatile 키워드로 선언하면 된다.
// 만약, 통신 외에 추가로 배타적 수행이 필요한 코드가 있다면 동기화를 꼭 해줘야한다.
public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread background = new Thread(() -> {
            int i = 0;
            while (!stopRequested) i++;
        })

        background.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

- 마지막으로, `volatile` 키워드의 증가 연산자 사용에 주의해야한다.
- 증가 연산자(`++`)는 코드상으로 단순해 보이나, 실제로는 증가하려는 필드에 두 번 접근한다.
    - 값을 읽어올 때 (1)
    - 값을 1 증가시킨 새로운 값으로 저장할 때 (2)
- 만약, 다른 스레드가 이 1번과 2번 작업 사이에 침투하면 잘못된 결과를 낸다. (safety-failure)



## `java.util.concurrent.atomic`을 이용한 Lock-free 동기화
- `volatile` 키워드는 스레드 간 통신만 지원하지만, `Atomic` 패키지는 원자성(배타적 실행)까지 제공한다.


```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```



## 근본적인 문제 해결: 가변 데이터를 여러 스레드 간 공유하지 말자
- 가변 데이터를 사용할 때는 단일 스레드에서만 사용하자.
- 또한, 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때 공유하는 부분의 동기화도 중요하다.
    - 사실상 불변과 안전 발행의 개념
- 객체를 안전하게 발행하도록 하자.



---
## Summary
- 여러 스레드가 가변 데이터를 공유한다면, 그 데이터를 읽고 쓰는 동작을 반드시 동기화해야 한다.
- 동기화하지 않으면, 한 스레드가 수행한 변경을 다른 스레드가 보지 못할 수 있다.
- 공유되고 있는 가변 데이터를 동기화하는 데 실패하면, 응답이 불가능해지거나 안전 실패(safety failure)로 이어질 수 있기에 조심해야 한다.


> Safety Failure
>
> 멀티 스레드 환경에서 정상적인 동작 보장이 깨진 상태 -> 잘못된 상태임을 의미한다.


- 동기화 방식에는 배타적 실행(Lock), volatile(메모리 가시성 해결), Atomic 등의 Lock-free 연산자 사용, 동시성 컬렉션 사용 등이 있다.