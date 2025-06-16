# Item 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
- 만약, `public` 클래스가 있다면, 접근자와 변경자 메서드를 활용해 데이터를 캡슐화 하는 것이 좋다.


## 접근자 메서드 활용

```java
class Point{
    private double x;
    private double y;

    public Point(double x, double y){
        this.x = x;
        this.y = y;
    }

    public double getX(){ return x; }
    public double getY(){ return y; }
    public void setX(double x){ return this.x = x; }
    public void setY(double y){ return this.y = y; }
}
```


이런식으로 구성하면, 내부 표현을 고치고 싶을 때 얼마든지 고칠 수 있다.


하지만, `package-private`, `private` 중첩 클래스의 경우, 데이터 필드를 노출해도 그 클래스가 표현하려는 추상 개념만 올바르게 표현한다면 문제가 없다. 


접근자 방식보다 훨씬 깔끔하며, 패키지 바깥 코드를 신경쓰지 않고 데이터 표현 방식을 바꿀 수 있다.


## 불변 필드의 `public` 제한자 문제


```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_DAY = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute){
        if(hour < 0 || hour >= HOURS_PER_DAY){
            throw new IllegalArgumentException("시간 : "+ hour);
        }
        if(minute < 0 || minute >= MINUTES_PER_DAY){
            throw new IllegalArgumentException("분 : "+ minute);
        }
        this.hour = hour;
        this.minute = minute;
    }
}
```

위 코드와 같이 `public` 클래스의 필드가 불변이라면 불변식은 보장할 수 있다. 즉, 직접 노출할 때의 단점은 줄지만, 여전히 단점이 존재한다.
1. 내부 표현 변경시 API 수정 필요
2. 필드를 읽을 때 부수 작업을 수행할 수 없다.


## Summary
- `public` 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
- 불변 필드라면 노출해도 덜 위험하나, 완전히 안심할 수 없다.
- 단, `package-private` 클래스나 `private` 중첩 클래스에서는 종종 필드를 노출하는 편이 편할 때도 있다. (적절하게 사용 필요)