# Item 42. 익명 클래스보다는 람다를 사용하라
## 익명 클래스
- 익명 클래스는 말 그대로 이름 없는 클래스, 보통 인터페이스나 추상 클래스의 인스턴스를 즉석에서 만들 때 사용한다.
- 이전에 함수 객체를 만드는 주요 수단으로 쓰였었다.
- 과거 디자인 패턴 중, 전략 패턴에서 익명 클래스로 함수 객체를 만들어서 사용했다. (이걸로 충분하다고 여겼음)
- 그러나 Java에 함수형 프로그래밍이 등장하면서, 익명 클래스 방식의 코드가 길고 함수형이지 않은 단점이 부각되었다.


```java
Collections.sort(words, new Comparator<String>() {
      public int compare(String s1, String s2){
          return Integer.compare(s1.length(), s2.length());
    }
});
```


## 익명 클래스를 사용한 코드를 람다 방식으로 전환
- 현재는 함수형 인터페이스라고 부르는, 람다를 사용한 방식으로 전환할 수 있다.
- 람다, 매개변수, 반환값의 타입이 각각 있으나 이를 명시하지 않아도 컴파일러가 추론해준다.
- 상황에 따라 컴파일러가 타입을 결정할 수 없을 때, 프로그래머의 명시가 필요하다.


```java
// 위 코드를 아래 코드로 바꾸어 람다식을 함수 객체로 사용하고, 익명 클래스를 대체할 수 있다.
Collections.sort(words, (s1, s2) -> Integer.compare(sl.length(), s2.length()));
```


> 타입 추론에 대한 조언
>
> Item 26에서는 제네릭의 raw type을 사용하지 말라 했고, Item 29에서는 제네릭을 사용할 것, Item 30에서는 제네릭 메서드를 권장했다.
>
> 이 조언들은 람다와 함께 사용할 때 두 배로 중요해지는데, 컴파일러가 타입 추론할 때 필요한 정보 대부분을 제네릭에서 얻기 때문이다.
> 우리가 이 정보를 제공하지 않으면, 컴파일러가 람다의 타입을 추론할 수 없게 되어 개발자가 일일히 명시해야 한다.


람다를 사용할 때, 타입을 명시해야만 코드가 명확해질 때(코드가 복잡하고, 타입 명시 없이는 코드를 이해하기 어렵거나 컴파일 추론이 불가능 할 때 등등) 외에는 람다의 모든 매개변수 타입을 생략하는 것이 좋다. 람다의 간단 명료한 이점을 살리기 위해서이다.


## 비교자 생성 메서드
- 람다 자리에 비교자 생성 메서드를 사용하면 코드를 더 간결하게 만들 수 있다.
    - `Collections.sort(words, comparingInt(String::length))`
- 더 나아가, Java8 List 인터페이스에 추가된 sort 메서드를 활용하면 더 짧아진다.
    - `words.sort(comparingInt(String::length))`


---
## 함수 객체의 실용적 활용 예시
1. Enum 타입에서 상수별 클래스 몸체를 사용해, 각 상수에서 apply method를 재정의 하는 방식으로 개별 동작을 구현했었다.
2. 열거 타입에 인스턴스 필드를 두고, 필드에 저장된 람다를 호출하여 구현하면 훨씬 깔끔한 코드로 리팩토링 할 수 있다.


```java
// Code 1
public enum Operation {
    /* 각기 다른 Operation 상수별로 클래스 몸체가 있고, 
       각각 apply method를 재정의하여 사용하고 있다. 재정의에 들어가는 반복성 코드가 길어진다. */
    PLUS("+")    {public double apply(double x, double y){return x + y;}},
    MINUS("-")   {public double apply(double x, double y){return x - y;}},
    TIMES("*")   {public double apply(double x, double y){return x * y;}},
    DIVIDE("/")  {public double apply(double x, double y){return x / y;}};

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
    public abstract double apply(double x, double y);
}
```


```java
// Code 2
public enum Operation {
    // 람다 함수로 함수 객체를 넘겨서 인스턴스 필드에 저장한다.
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op; // DoubleBinaryOperator를 인스턴스 필드로 둔다

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public double apply(double x, double y){
        return op.applyAsDouble(x, y);
    };
}
```


이렇게 위 코드처럼, `apply` 메서드에서 필드에 저장된 람다를 호출하는 방식으로 구현할 수 있다.
반복되는 코드가 줄어들고, 람다를 사용하여 함수 객체를 손쉽게 보낼 수 있다.


---
## 람다를 사용하지 말아야 하는 경우
- 람다는 이름이 없고 문서화 불가능하기 때문에, 코드 자체로 동작이 명확하게 설명되지 않거나 코드 줄 수가 많아지면 사용하지 말아야한다.
- 람다는 한 줄 정도가 가장 좋고, 세 줄안에 끝내도록 하자. (이를 넘어가는 순간 가독성 문제 생김)
- 열거 타입 생성자에 넘겨지는 인수들의 타입도 컴파일 타임에 추론된다.
    - 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다. (인스턴스는 런타임에 생성되므로)
    - 상수별 동작을 짧게 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야 한다면 상수별 클래스 몸체(Code 1 ver.)를 사용해야 한다.
- 람다가 익명 클래스를 대체할 수 없는 경우
    - 람다는 함수형 인터페이스에서만 쓰임
    - 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으므로 익명 클래스를 써야한다.
    - 람다는 자신을 참조할 수 없어서, 함수 객체가 자신을 참조해야 하는 경우 익명 클래스를 사용해야 한다.
    - 익명 클래스는 스택 트레이스에 클래스 이름이 찍히지만 람다는 `$$Lambda` 형태로 나오기 때문에, 복잡한 로직일 땐 디버깅이 어려워질 수 있다.
- 람다를 직렬화 하는 일은 극히 삼가해야 한다.
    - 직렬화 형태가 구현별로(ex. 가상머신별로) 다를 수 있기 때문


---
## Summary
- Java8에서 작고 간단한 함수 객체를 구현하는데에 적합한 람다가 도입되었다. (자바 함수형 프로그래밍의 지평을 열었다.)
- 익명 클래스는 함수형 인터페이스가 아닌 타입의 인스턴스를 만들 때만 사용하자.