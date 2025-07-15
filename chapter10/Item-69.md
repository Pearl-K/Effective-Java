# Item 69. 예외는 진짜 예외 상황에만 사용하라
예외 발생을 트리거로 하여, 특정 로직을 처리하는 식의 코드는 좋지 못한 코드이다. 예외를 이런 트리거 형식으로 사용하지 않고, 실제 예외 상황에서만 쓰는 것이 좋기 때문이다.


```java
// Code 1. 예외를 잘못 사용한 예시
try {
    int i=0;
    while(true) range[i++].climb();
} catch (ArrayIndexOutOfBoundsException e) {

}

// Code 2. 1번의 코드를 표준 관용구대로 작성
for (Mountain m : range) {
    m.climb();
}
```


## 이유 1. 예외는 예외 상황에 쓸 용도로 설계되었다.
- JVM 구현자 입장에서는 예외는 예외 상황에 사용하는 용이다.
- 따라서, 명확한 검사 상황이나 어떤 로직의 트리거로 사용될 것을 고려하여 설계되지 않았다.
- 즉, 성능 최적화(빠른 속도)를 추구해야할 동기가 약하다.


## 이유 2. try-catch 블록은 최적화 제한
- 코드를 try-catch 블록 안에 넣으면 JVM이 적용할 수 있는 최적화가 제한된다.


## 이유 3. 예외를 사용한 트리거 처리보다, 표준 관용구가 더 최적화 되어있다.
- 배열을 순회하는 표준 관용구는 중복 검사를 수행하지 않는다.
- JVM이 알아서 최적화하여 없애주기 때문이다.


따라서, 예외를 트리거처럼 사용한 코드가 표준 반복 관용구보다 훨씬 느리다.


---
## 예외에 대한 원칙
위 사례를 통해 예외에 대한 원칙을 두 가지 짚을 수 있다.


1. 예외는 반드시 예외 상황 처리 용도로 의도에 맞게 사용해야하며, 절대 일상적인 제어 흐름에 쓰여선 안된다.
2. 잘 설계된 API는 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 만든다. (따라서, API 설계할 때 예외 사용할 일 없게 만들기)


---
## 상태 검사 메서드
만약, 특정 상태에서만 호출할 수 있는 "상태 의존적"인 메서드를 제공하는 클래스가 있을 수 있다.


이 경우엔 "상태 검사" 메서드도 함께 제공해야 한다. 예를 들면 `Iterator` 인터페이스의 `next` 와 `hasNext` 는 각각 상태 의존적 메서드 & 상태 검사 메서드에 해당한다.


> `next` 는 다음이라는 상태가 있어야 하기 때문에 상태 의존, `hasNext` 는 다음 상태를 가지는지 상태를 검사하는 것이라고 이해하면 된다.


표준 관용구 (ex. for-each) 들은 이러한 별도의 상태 검사 메서드를 활용하여 반복되는 검사 로직을 줄이고 깔끔한 코드를 유지한다. 따라서 이런 관용구를 활용하는 것이 좋다.


---
## 상태 검사 메서드 대신 사용할 수 있는 선택지: `Optional` 등의 특수 값 반환
올바르지 않은 상태일 때를 표시하는 특수 값을 반환할 수 있다. 그러나 이전 Item에서 보았던 것처럼 null을 반환하는 것은 위험하므로 주의하자.


아래와 같은 지침을 참고하여 코드를 짤 수 있다.


1. 외부 동기화 없이 여러 스레드가 동시 접근 가능하거나, 외부 요인으로 상태가 변할 수 있다면 Optional 이나 특정 값을 사용한다.
    - 상태 검사 메서드와 상태 의존적 메서드 호출 사이에 객체 상태가 변화할 가능성이 있기 때문
2. 성능이 중요한 상황에서, 상태 검사 메서드가 상태 의존적 메서드의 작업 일부를 중복 수행한다면 Optional 등의 특정 값을 선택한다.


### 2번 코드 예시
예를 들어, 어떤 키에 해당하는 데이터를 외부 파일이나 데이터베이스에서 찾아오는 클래스를 설계하자. 

`hasData(key)`로 존재 여부를 확인하고 `getData(key)`로 데이터를 가져오는 메서드가 있을 때, 두 메서드 모두에서 파일을 열고 키를 검색하는 비싼 작업(I/O)을 중복으로 수행하게 될수도 있다.


```java
// (1) 상태 검사 메서드와 상태 의존적 메서드가 작업을 중복 수행하는 경우

public class DataRepository {

    // 상태 검사 메서드: 키 존재 여부 확인
    public boolean hasData(String key) {
        // 파일을 열고, 내용을 검색하는 작업 수행
        System.out.println("hasData: Performing expensive search for key: " + key);
        // ...
        return true;
    }

    // 상태 의존적 메서드: 실제 데이터 가져오기
    public String getData(String key) {
        // 파일을 열고, 내용을 검색하는 비싼 작업을 중복 수행 (중복 I/O)
        System.out.println("getData: Performing expensive search AGAIN for key: " + key);
        // ...
        return "some data";
    }
}

// 클라이언트: 동일한 검색 두 번 수행
DataRepository repo = new DataRepository();
String key = "db.url";
if (repo.hasData(key)) {
    String data = repo.getData(key);
}
```

- 이런 경우, `Optional`을 사용하면 검색을 한 번만 수행하고 결과를 안전하게 처리할 수 있어 성능상 이점을 가진다.


```java
// (2) Optional을 사용하여 작업 중복을 없앤 경우
import java.util.Optional;

public class DataRepository {
    public Optional<String> findData(String key) {
        // 파일을 열고, 내용을 검색하는 비싼 작업을 한 번만 수행
        System.out.println("findData: Performing expensive search ONCE for key: " + key);
        // ...
        
        boolean found = true; // 찾았다고 가정
        if (found) {
            return Optional.of("some data");
        } else {
            return Optional.empty();
        }
    }
}

// 클라이언트 코드: 검색을 한 번만 수행하고 결과를 안전하게 처리
DataRepository repo = new DataRepository();
String key = "db.url";
repo.findData(key).ifPresent(data -> {
    System.out.println("Data found: " + data);
});
```


3. (위에 1, 2 번 경우 외에) 다른 모든 경우에는 상태 검사 메서드 방식이 좀 더 낫다고 볼 수 있다.
    - 가독성이 좋고, 잘못 사용했을 때 발견하기 쉽다.
    - 상태 검사 메서드 호출을 잊었다면, 상태 의존적 메서드가 예외를 던져 버그를 명시할 것이기 때문에 발견하기도 쉽다.



---
## Summary
- 예외는 반드시 예외 상황에만 사용하자. (설계 의도대로 사용)
- 정상적인 제어 흐름에서 사용해선 안되며, 프로그래머에게 사용을 강요하는 API를 만들어서도 안된다.