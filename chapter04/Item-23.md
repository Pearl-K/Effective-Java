# Item 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

## 1. 태그 달린 클래스의 문제점
- 두 가지 이상의 의미를 표현할 수 있으며, 그 중에 현재 표현하는 의미를 태그 값으로 알려주는 클래스가 있다.
- 아래 코드는 태그 달린 클래스의 예시이다.

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 
    final Shape shape;

    // 모양이 RECTANGLE일때만 사용
    double length;
    double width;

    // 모양이 CIRCLE 일때만 사용
    double radius;

    // 원 생성자
    Figure(double radius){
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형 생성자
    Figure(double length, double width){
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape){
          case RECTANGLE:
              return length * width;
          case CIRCLE:
              return Math.PI * (radius * radius);
          default:
              throw new AssertionError(shape);
        }
    }
}
```


### 단점

- 태그달린 클래스에는 단점이 많다.
- 열거 타입 선언, 태그 필드, switch 문 판별 등 쓸데 없는 코드가 길어진다.
- 여러 구현이 한 클래스에 혼합되어 있어 가독성이 나쁘다.
- 만약, 해당 필드를 `final`로 선언하려면, 쓰지 않는 필드까지도 모두 초기화하는 불필요한 코드가 늘어난다.

> 즉, 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.
> 
> 클래스 계층 구조를 활용하는 SubTyping을 사용하는 것이 훨씬 낫다. 태그 달린 클래스는 클래스 계층 구조를 어설프게 따라한 것이다.



## 2. 태그 달린 클래스를 클래스 계층 구조로 바꾸는 방법
1. 계층 구조의 루트가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들은 추상 메서드로 선언한다.

```java
abstract class Figure {
    abstract double area();
}
```

2. 태그 값에 상관없이 동작이 일정한 메서드들은 일반 메서드로 추가한다.


3. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드도 루트 클래스에 추가한다.


4. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의하고, 각 하위 클래스에 각자 해당하는 데이터 필드를 추가한다.


5. 루트 클래스가 정의한 추상 메서드를 하위 클래스에서 각자 의미에 맞게 구현한다.


```java
class Circle extends Figure {
    final double radius;

    Circle(double radius){
        this.radius = radius;
    }

    @Override double area(){
        return Math.PI * (radius * radius);
    }
}

class Ractangle extends Figure {
    final double length;
    final double width;

    Ractangle(double length, double width){
        this.length = length;
        this.width = width;
    }

    @Override double area(){
        return length * width;
    }
}
```

- 이런 클래스 계층 구조는 태그 달린 클래스의 단점을 모두 해소한다.
- 간결하고 명확하며, 쓸데 없는 코드도 사라졌다.
- 특히, 타입이 의미 별로 존재하므로 변수의 의미를 명시하거나, 제한할 수 있고, 특정 의미만 매개변수로 받을 수도 있다.
- 타입 사이의 자연스러운 계층 관계를 반영할 수 있다. (유연성, 컴파일타임 타입 검사 능력 높임)

> 이번 예시에서 접근자 메서드 없이 필드 직접 노출은 원래는 피해야하는 방식이며, 예시로 적절하게 보여주기 위해 코드 간편화로 사용했다.


## Summary
- 태그 달린 클래스를 써야하는 상황은 거의 없고, 피해야 한다.
- 태그를 없에고 클래스 계층 구조로 대체하는 것을 추천한다.