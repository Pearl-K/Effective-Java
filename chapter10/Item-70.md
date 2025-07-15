# Item 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

Java에는 문제 상황을 알리는 타입(throwable)으로 검사 예외, 런타임 예외, 에러 3가지를 제공한다.

- **검사 예외 (Checked Exception)**
    - `Exception` 클래스를 상속하지만 `RuntimeException`은 상속하지 않는 예외들이다.
    - 이름 그대로, 컴파일러가 예외 처리 여부를 **검사**한다.
    - 즉, `try-catch`로 잡거나 `throws`로 명시해서 호출한 쪽으로 전파하지 않으면 컴파일 오류가 발생한다.
- **비검사 예외 (Unchecked Exception)**
    - `RuntimeException` 클래스를 상속하는 예외들과 `Error`를 통칭한다.
    - 컴파일러가 예외 처리 여부를 검사하지 않는다. 개발자가 알아서 처리해야 한다.


여러 가지 문제 상황에서, 언제 무엇을 사용해야 하는지 참고할만한 지침을 소개한다.


---
## 1. 호출하는 쪽에서 복구할 것이라고 여겨지는 상황에서는 검사 예외를 사용하라.
- 이는 검사 예외와 비검사 예외를 구분하는 기본 규칙이다.
- 검사 예외를 던지면, 호출자가 해당 예외를 `catch` 로 잡아서 처리하거나 더 바깥으로 전파하도록 강제하게 된다.
- 따라서, 메서드 선언에 포함된 각각의 검사 예외들은 그 메서드를 호출했을 때 발생할 수 있는 유력한 결과임을 API 사용자에게 명시하는 것과 같다.
    - API 설계자가 API 사용자 측에 검사 예외를 던져서 해당 상황에서 회복(복구)하라고 요구하는 것이다.

- 예를 들어, 계좌에서 돈을 출금할 때 잔고가 부족한 상황은 충분히 예측 가능하고, 클라이언트가 처리할 수 있는 **복구 가능한 상황**이라고 할 수 있다. 이런 경우 검사 예외를 사용하는 것이 좋다.


```java
public class InsufficientFundsException extends Exception {
    private final double shortfall;

    public InsufficientFundsException(double shortfall) {
        super("잔고가 " + shortfall + "원 부족합니다.");
        this.shortfall = shortfall;
    }

    // 복구에 필요한 정보(부족 금액)를 반환한다.
    public double getShortfall() {
        return shortfall;
    }
}

// API - 출금 메서드는 검사 예외를 던지도록 설계
public class BankAccount {
    private double balance;
    // ...
    public void withdraw(double amount) throws InsufficientFundsException {
        if (balance < amount) {
            throw new InsufficientFundsException(amount - balance);
        }
        balance -= amount;
    }
}

// 클라이언트 - 예외를 잡아서 복구를 시도한다.
try {
    account.withdraw(10000);
} catch (InsufficientFundsException e) {
    System.err.println(e.getMessage());
    // 예외가 제공하는 정보를 바탕으로 사용자에게 구체적인 안내를 할 수 있다.
    System.err.println("부족한 금액을 채우려면 " + e.getShortfall() + "원을 더 입금해주세요.");
}
```


- 위 코드처럼 `withdraw` 메서드는 잔고 부족 시 `InsufficientFundsException`이라는 검사 예외를 던진다.
- 클라이언트는 이 예외를 `try-catch`로 반드시 처리해야 하며, `getShortfall()` 같은 메서드로 복구에 필요한 구체적인 정보를 얻어 후속 조치를 취할 수 있다. (복구 단계)



## 2. 비검사 throwable: 런타임 예외와 에러
- 이 둘은 프로그램에서 잡을 필요가 없거나, 통상적으로 잡지 말아야하는 경우이다.
- 프로그램에서 이 두 가지를 던졌다는 것은 복구가 불가능하거나, 더 실행해봐야 득보다 실이 많다는 뜻이다.


### 프로그래밍 오류를 나타낼 때는 런타임 예외를 사용하라.
- 런타임 예외의 대부분은 전제조건을 만족하지 못했을 때 발생한다. (ex. 클라이언트가 API 명세의 제약 조건을 지키지 않았을 때)
- 해당 상황은 복구 가능한 상황인지 or 프로그래밍 오류인지 판별하기 어렵다. (확신하기 어렵기 때문에 검사 예외로 던지기 어려운 상황)


### 에러는 보통 JVM 상의 자원 부족, 불변식 깨짐 등 수행이 불가능한 상황을 나타낸다.
- `Error` 클래스를 상속해 Custom Error를 만드는 짓은 하지 말자 (^.^)
- `AssertionError` 외에 직접 에러를 `throw` 하지도 말자.



---
## Summary
- 복구 할 수 있는 상황이면 검사 예외를 던지자.
- 프로그래밍 오류가 예상되거나 복구 가능한지 확신할 수 없다면 비검사 예외를 던지자.
- 검사 예외도 아니고 런타임 예외도 아닌 throwable 은 정의하지 말자. (성능 저하 문제)
- 검사 예외라면, 복구에 필요한 정보를 알려주는 메서드도 제공하자.
