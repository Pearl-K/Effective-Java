# Item 45. 스트림은 주의해서 사용하라
## 스트림 개념 및 특징
- **스트림(Stream)**
    - Java 8부터 도입된 데이터 처리 추상화로, 컬렉션·배열·IO 채널 등에서 원소의 **유한·무한 시퀀스**를 선언형으로 다룬다.
    - 내부 반복(Internal Iteration)을 사용해, **for-each** 루프 대신 파이프라인 형태로 연산을 연결한다.
- **파이프라인(Pipeline)**
    - **중간 연산(Intermediate Operation)**: `filter`, `map`, `sorted` 등 → **지연(lazy)** 수행
    - **최종 연산(Terminal Operation)**: `collect`, `forEach`, `reduce` 등 → 파이프라인 실행, 스트림 소비
- **주요 특징**
    1. **지연 연산**: 최종 연산 시점까지 중간 연산은 수행되지 않음 → 불필요한 연산 최소화
    2. **재사용 불가**: 터미널 연산 후 스트림은 닫히므로, 다시 사용하려면 새로운 스트림 생성
    3. **무상태·부수효과 배제 권장**: 중간 연산은 가능한 한 외부 상태 변경 없이 순수 함수형으로 작성
    4. **병렬 처리 용이**: `.parallelStream()` 혹은 `.parallel()` 로 손쉽게 병렬화, 포크/조인 프레임워크 활용
    5. **짧은 연산 단축(short-circuit)**: `findFirst`, `anyMatch` 등에서 전체 파이프라인을 도중에 종료 가능


```java
List<Person> teenagers = people.stream()                     // 소스 스트림
    .filter(p -> p.getAge() >= 13 && p.getAge() <= 19)       // 중간 연산: 필터링
    .map(Person::getName)                                    // 중간 연산: 매핑
    .sorted()                                                // 중간 연산: 정렬
    .collect(Collectors.toList());                           // 최종 연산: 수집
```


## (+추가) 스트림 파이프라인에서 연산의 순서 중요성
### 1. 시맨틱(Semantics) 관점

1. **타입 변화**
    * `.map()` 이후엔 요소 타입이 바뀌기 때문에, 그 뒤로는 원본 타입 메서드를 쓸 수 없다.
    * 예시
   
      ```java
      // ❌ 잘못된 순서: map으로 String으로 바꾼 뒤에 getAge() 호출 불가
      people.stream()
            .map(Person::getName)
            .filter(p -> p.getAge() > 20) // 컴파일 에러
            .collect(Collectors.toList());
      ```
    * **올바른 순서**로 `filter` → `map` 을 해야 한다.

2. **단축(short-circuit) 연산 효과**
    * `findFirst()`, `anyMatch()`, `limit()` 같은 최종 연산은
      앞선 **필터(filter)** 단계가 먼저 작동해야 가능한 한 빨리 종료할 수 있다. (효율적)
    * 예시:

      ```java
      // filter가 마지막에 오면, 모든 요소에 map이 먼저 실행된 뒤에야 종료 검사
      people.stream()
            .map(Person::getName)         // 먼저 실행
            .map(String::length)
            .anyMatch(len -> len > 10)    // 불필요한 연산이 많아짐
      ```
    * 필터를 최대한 앞에 두면, 검사 대상 요소 수를 줄여 성능 향상


---
### 2. **성능(Performance) 관점**
1. **무상태 vs 상태(stateful) 연산**
    * `filter`, `map`, `flatMap` 등은 **무상태(stateless)** → 각 요소만 보고 처리
    * `sorted`, `distinct`, `limit` 등은 **상태(stateful)** → 요소 전체 또는 부분을 메모리에 담음
    * **권장 순서**

      ```text
      (1) filter → (2) map/flatMap → (3) 중간 상태ful(정렬·중복제거) → (4) 최종 연산
      ```
    * 이렇게 하면 stateful 연산이 다룰 요소 수를 최소화할 수 있다.


2. **병렬 스트림에서의 순서 보장**

    * `.parallelStream()` 사용 시, 순서를 보장해야 하는 연산(예: `forEachOrdered`, `limit`)은
      오히려 병렬 효율을 낮춘다.
    * 순서가 필요 없으면 `forEach`와 `unordered()`를 써서 병렬 성능을 더 뽑아낼 수 있다.


---
### 3. **정리 & 권장 패턴**

| 단계 구분                 | 예시 메서드                                      | 권장 위치            |
| --------------------- | ------------------------------------------- | ---------------- |
| 무상태 필터/매핑(stateless)  | `filter`, `map`, `flatMap`                  | **맨 앞**          |
| 상태ful 중간 연산(stateful) | `sorted`, `distinct`, `limit`, `skip`       | 필터·매핑 후, 최종 연산 전 |
| 최종 연산(terminal)       | `collect`, `forEach`, `reduce`, `findFirst` | 스트림 끝            |

* **필터(filter)**: 가장 먼저
* **매핑(map)**: 두 번째
* **상태ful 연산**: 데이터 양이 줄어든 뒤
* **최종 연산**: 맨 마지막

이 순서를 지키면 의도한 대로 동작할 뿐 아니라 성능도 최적화할 수 있으므로 잘 지켜서 코드를 구현하자.



## 코드 블럭 vs 람다 블럭
| 특징                         | 코드 블록                                    | 람다 블록                                                   |
|----------------------------|-------------------------------------------|-----------------------------------------------------------|
| 지역 변수 접근 및 수정            | 범위 안의 지역 변수를 읽고 **수정**할 수 있음            | `final` 또는 **사실상 final** 변수만 읽을 수 있으며, 수정 불가         |
| 제어 흐름 제어(return/break/continue) | `return`으로 메서드 종료, `break`/`continue`로 바깥 반복문 제어 가능 | `return`, `break`, `continue` 사용 불가능                       |
| 검사 예외 던지기               | 메서드 선언에 명시된 **검사 예외**를 던질 수 있음         | 명시된 **검사 예외**를 던질 수 없음                            |
| 특정 작업 수행 적합성            | 블록 내에서만 가능한 복잡한 작업(부수효과, 상태 변경 등) 수행 가능 | 스트림·람다의 순수 함수형 처리와 맞지 않는 작업은 수행 불가               |


- 각 특징을 보고, 코드블록에서만 수행 가능한 경우를 스트림으로 구현하지 않도록 주의하자.


## 스트림이 잘 맞는 경우
- 원소들의 시퀀스를 일관되게 변환하는 경우
- 원소들의 시퀀스를 필터링하는 경우
- 원소들의 시퀀스를 하나의 연산을 사용해 결합하는 경우 (더하기, 연결하기, 최솟값 구하기 등)
- 원소들의 시퀀스를 컬렉션에 모으는 경우 (ex. 공통 속성을 기준으로 묶을 때)
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는 경우


## 스트림으로 처리하기 어려운 일
- 한 데이터가 파이프라인의 여러 단계를 통과할 때, 각 단계에서 값들에 동시에 접근하기 어려운 경우
  - 스트림은 한 값을 다른 값에 매핑하고 나면, 원래의 값은 잃는 구조이기 때문이다.
- 매핑 객체가 필요한 단계가 여러 곳인 경우


---
## Summary
- 스트림 방식과 반복 방식이 각각 알맞은 경우가 다르므로, 판단 기준을 세우고 구현하자.
- 이 둘을 조합해야하는 경우가 있으므로 유동적으로 사용해보자.
- 둘 중에 무엇이 더 나을지 확신하기 어렵다면, 둘 다 해보고 더 나은 것을 택하는 방법이 좋다.