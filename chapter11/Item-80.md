# Item 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라
고수준의 실행자 프레임워크(Executor Framework)가 나오기 전에는, 로우레벨로 직접 다루는 코드를 작성해야 했다.


예를 들면, 직접 스레드에 작업을 할당하고 실행시키고 실패나 복구 코드까지 모두 개발자가 작성해야 했는데, 이제 concurrent 패키지의 Executor Framework를 사용하면 고수준으로 쉽게 작업 큐, 할당 등을 구현할 수 있다.


---
## Executor Framework
ExecutorService 는 Java에서 비동기 작업 실행을 위한 고수준 추상화를 제공하는 인터페이스다. 개발자는 작업을 `Runnable` 또는 `Callable` 으로 정의하고, 이를 실행자에게 넘기면 내부적으로 스레드 풀을 사용해 적절히 실행된다.


직접 스레드를 만들고 관리하는 대신, 작업 제출만 하면 되고 실행 방식은 Executor가 알아서 처리한다.


> 즉, 작업 실행의 책임을 프레임워크에 위임하는 방식이다.


아래에서 몇 개의 ExecutorService 사용 예시를 볼 수 있다.

```java
// 1. 작업 큐 생성
ExecutorService exec = Executors.newSingleThreadExecutor();

// 2. 실행할 테스크 넘기기
exec.execute(runnable);

// 3. Executor 종료시키기
exec.shutdown();
```

### ExecutorService의 주요 기능
- 특정 태스크가 완료되기를 기다린다.
- 태스크 모음 중, 아무거나 하나(`invokeAny()`) or 모든 태스크(`All()`)가 완료되기를 기다린다.
- ExecutorService 가 종료하기를 기다린다. (`awaitTermination()`)
- 완료된 태스크들의 결과를 차례대로 받는다. (`ExecutorCompletionService`)
- 태스크를 특정 시간에 or 주기적으로 실행하게 만든다. (`ScheduledThreadPoolExecutor` 사용)


만약, 특정한 작업 큐를 둘 이상의 스레드가 병렬 소비하게 만들고 싶다면, 간단히 다른 정적 팩토리를 이용하여 다른 종류의 ExecutorService(스레드 풀)을 생성하면 된다.


필요한 실행자 대부분은 `java.util.concurrent.Executors` 내 정적 팩토리를 이용하여 생성할 수 있다.


만약, 스레드 풀 동작을 결정하고자 세부적인 속성을 설정하고 싶다면 `ThreadPoolExecutor` 를 직접 사용할 수도 있다.


### ExecutorService 사용 시 주의 조건
1. 작은 프로그램이나 가벼운 서버
    - `Executors.newCachedThreadPool`이 일반적으로 좋은 선택이다.


2. 무거운 프로덕션 서버
    - `CachedThreadPool` 은 좋지 못한 선택이다.
    - 위 스레드 풀에서는 요청받은 태스크들이 큐에 쌓이지 않고, 즉시 스레드에 위임되어 실행되기 때문이다.
    - 가용 스레드가 없다면 새로 하나를 생성하게 되는데, 서버가 무거운 상태(즉, CPU 이용률이 충분한 상태) 라면 억지로 스레드를 하나 더 만들어서 과도한 context switching이 일어난다.
    - 즉, CPU 이용률이 100% 수준으로 치솟을 것이며, 이 상황에서 태스크가 계속 추가된다면 Thrasing 현상이 발생할 수 있다. (위험!)
    - 따라서, 스레드 개수를 고정해야 한다. (`Executors.newFixedThreadPool`)
    - 아니면, 완전히 통제할 수 있는 `ThreadPoolExecutor` 를 직접 사용해야 한다.


---
## 스레드를 직접 다루지 말자
스레드를 직접 다루면 `Thread`가 수행하는 작업 단위와 수행 메커니즘 두 가지 역할을 모두 수행하게 된다.


반면, `ExecutorService`에서는 작업 단위와 실행 메커니즘이 분리된다.


### 작업 단위 & 실행 메커니즘은 각각 무엇인가?
- 작업 단위(Task)
    - 실제로 수행하고자 하는 로직 또는 연산
    - 보통 `Runnable` 또는 `Callable` 객체로 표현


- 실행 메커니즘(Execution Mechanism)
    - 작업을 언제, 어떤 방식으로, 어떤 스레드에서 실행할지를 결정하는 로직
    - `Executor`, `ThreadPoolExecutor`, `ForkJoinPool` 등이 이 역할을 맡음



### 작업 단위와 실행 메커니즘의 분리가 왜 필요한가?
- 유연성: 같은 작업 단위를 서로 다른 실행 방식(단일 스레드, 병렬 스레드, 스케줄링 등)으로 실행할 수 있다.


- 재사용성: 작업 단위를 재사용하거나 테스트하기 쉬워진다.


- 관심사의 분리: 비즈니스 로직과 실행 전략을 명확히 나눌 수 있다. 결과적으로 유지보수성이 높아지고, 코드 품질이 향상된다.


- 자원 관리 최적화: 실행 메커니즘에서 스레드 풀 크기, 큐 전략, 타임아웃 등을 통해 시스템 자원을 효과적으로 제어할 수 있다.




---
## Java7+ ExecutorService의 fork-join 기능
ForkJoinPool은 Java 7부터 도입된 고성능 병렬 처리용 스레드 풀이다.


Fork/Join 패턴은 큰 작업을 재귀적으로 쪼개고(fork), 각각 병렬로 실행한 뒤(join) 결과를 합치는 방식이다.


이 구조는 CPU 코어를 최대한 활용할 수 있게 해주며, 특히 CPU 바운드 작업에서 효과적이다.


내부적으로는 워크 스틸링(work stealing) 기법을 사용하여, 유휴 스레드가 다른 작업자 큐에서 작업을 가져와 실행함으로써 전체 효율을 높인다.


RecursiveTask<V> 또는 RecursiveAction을 상속받아 태스크를 정의한다.

```java
ForkJoinPool pool = new ForkJoinPool();
pool.invoke(new YourRecursiveTask());
```


fork-join task를 직접 작성하고 튜닝하기는 어렵지만, fork-join pool을 이용해서 만든 병렬 스트림을 이용하면 적은 노력으로 그 이점을 얻을 수 있다.