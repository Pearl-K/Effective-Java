# Item 34. int 상수 대신 열거 타입을 사용하라
## 기존 정수 열거 패턴의 한계
Java1.5+ 에서 Enum 타입이 생기기전에는 정수 열거 패턴을 사용했었다.


```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```


위 예시 코드를 보면, 정수 열거 패턴의 단점을 한눈에 볼 수 있다. 우선 정수 열거 패턴을 위한 별도의 namespace가 존재하지 않아서 개발자가 수동으로 접두어를 사용해야한다. (충돌을 막기 위해서)


또한, 아래 코드를 보면 서로 지칭하는 것이 달라도, 값이 같을 때 true로 취급된다.


```java
APPLE_FUJI == ORANGE_NAVEL; // 결과 true
```


단점을 정리해보면 다음과 같다.
1. 타입 안전성이 지켜지지 않고, 개발자가 수동으로 관리하고 규칙을 세우기에 복잡하다.


2. 상수의 값이 바뀌면 반드시 다시 컴파일해야 한다.
    - 평범한 상수를 나열한 것이기 때문에 컴파일할 때 값이 클라이언트 파일에 그대로 들어가기 때문이다.


3. 정수 상수는 문자열로 출력하기가 어렵다. 
    - 그 값을 출력하거나 디버거로 살펴볼 때, 특정 의미가 아닌 숫자로만 보인다.
    - 또한, 같은 정수 열거 그룹에 속한 상수가 몇개인지도 알 수 없다.
    - 같은 namespace로 묶이는게 아니라 그저 나열한 상수 목록이기 때문이다.


---
## 기존 문자열 열거 패턴의 한계
위에서 상수의 의미를 표현할 수 없다는 단점을 극복하기 위해 문자열 열거 패턴을 도입하기도 했다.(과거)


```java
public static final String APPLE_FUJI           = "apple fuji";
public static final String APPLE_PIPPIN         = "apple pippin";
public static final String APPLE_GRANNY_SMITH   = "apple granny smith";
```


이 방법은 의미를 알 수 있는 점에서 좋지만, 아래와 같은 추가적인 단점이 있다.

1. Human Error에 취약하다
    - 하드코딩한 문자열에 오타가 있어도 컴파일러는 확인할 길이 없다.
    - 오직 개발자가 코드를 읽어보고 체크해야 한다.


2. 문자열 비교에 따른 성능저하가 생길 수 있다.
    - 문자열이 길거나, 비교 작업이 반복되면 숫자를 비교하는 것에 비해 성능이 떨어진다.



---
## 열거 타입 Enum Type
### 개념
열거 타입은 서로 관련된 상수들을 하나의 타입으로 묶어 표현하는 참조(Reference) 타입이다. 위에서 참고했던 기존 열거 패턴들의 단점을 해결하기 위해 나왔다.


열거 타입은 클래스처럼 동작하며, 내부에 필드/메서드/생성자까지 정의할 수 있어서 단순한 값의 집합을 넘어 상태와 동작을 함께 표현할 수 있는 수단이다.



### 특징
1. 타입 안전성 보장
    - 열거 타입은 명확한 타입을 가지므로, 잘못된 비교나 대입을 컴파일 시점에 잡아낼 수 있다.


2. 네임스페이스 제공
    - 열거 타입은 자신만의 이름 공간을 제공하여 상수 간 충돌을 방지한다. (이름이 겹쳐도)


3. `toString()`과 `name()` 메서드 제공
    - 열거 상수는 명시적인 이름을 해당 메서드로 출력할 수 있다.
    - 디버깅이나 로깅에도 유용하게 쓰인다.


4. switch 문에서 사용 가능
    - 로직 분기 처리를 위해 swtich 문에서 함께 사용할 수 있다.


5. 내부 상태와 동작 정의 가능
    - Enum 안에 생성자/필드/메서드를 정의하여 단순 상수 이상의 역할을 수행할 수 있다. (아래에서 추가 설명)



### 내부 구조
Enum 타입은 컴파일 시 내부적으로 클래스로 변환되며, 각 상수는 해당 클래스의 `public static final` 인스턴스로 생성된다.


- 열거 상수는 기본적으로 싱글톤으로 보장된다. (이 특징으로 인해, 간단한 싱글톤 객체 생성과 관리가 필요할 때 Enum으로 만들기도 한다.)
- JVM이 클래스 로딩 시점에 인스턴스를 생성해둔다.
- Enum 클래스는 `java.lang.Enum` 을 암묵적으로 상속하며, 개발자는 이를 상속하거나 확장할 수 없다.


생성 시 내부적으로 아래 코드와 유사한 구조가 생성된다고 보면 된다.


```java
public final class Day extends Enum<Day> {
    public static final Day MONDAY = new Day("Monday", 0);
    ...
}
```


---
## Enum Type이 제공하는 다양한 정적 메서드
- `values()` : 열거 타입에 정의된 모든 상수를 배열로 반환한다.
    ```java
    // 반복문에 자주 활용
    for (Day days : Day.values()) {
        System.out.println(days);
    }
    ```


- valueOf(String name) : 문자열 이름에 대응하는 열거 상수를 반환한다.
    ```java
    // 만약, 잘못된 이름을 넘기면 IllegalArgumentException 반환함
    Day nowday = Day.valueOf("Monday")
    ```


---
## Enum Type이 제공하는 추상 메서드
열거 타입은 추상 메서드를 선언할 수 있으며, 각 상수에서 이를 개별적으로 구현하게 만들 수도 있다.


이런 동작은 상수마다 동작이 달라야 할 때 유용하게 쓰인다. `apply()`를 통해서 상수별 메서드를 구현할 수 있다.


```java
public enum Operation {
    PLUS {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS {
        public double apply(double x, double y) { return x - y; }
    };

    public abstract double apply(double x, double y);
}
```




---
## 전략 열거 타입 패턴
여러 열거 상수가 일부 로직을 공유하는 경우, 공통 동작을 하나의 전략 객체(인터페이스 or 추상 클래스)로 추출하여 위임하는 패턴이다.


열거 타입에서 전략 객체를 필드로 가지고, 각 상수가 해당 전략을 선택하도록 만든다.


```java
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    double pay(double hoursWorked, double payRate) {
        return payType.pay(hoursWorked, payRate);
    }

    // 내부 전략 객체 (열거 타입)
    private enum PayType {
        WEEKDAY {
            double pay(double h, double r) {
                return h <= 8 ? h * r : 8 * r + (h - 8) * r * 1.5;
            }
        },
        WEEKEND {
            double pay(double h, double r) {
                return h * r * 1.5;
            }
        };

        abstract double pay(double hours, double rate);
    }
}
```



---
## Summary
- 열거 타입은 이전의 열거 패턴들보다 더 뛰어나고, 읽기 쉽고, 안전하고, 강력하다.
- 대부분의 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 필요한 경우도 있다.
    - 열거 타입을 특정 데이터와 연결 짓거나, 상수마다 다르게 동작할 때 활용하면 좋다.
    - `switch` 문 대신에 상수별 메서드 구현을 사용하면 된다.
    - 만약, 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.