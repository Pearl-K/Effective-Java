# Item 53. 가변인수는 신중히 사용하라
메서드에서 인수를 넘길 때, 가변인수를 넘기고 싶을 수 있다. 특히, 인수의 개수/길이를 정할 수 없을 때 가변인수를 넘기는 선택을 할 수 있으나. 이는 신중하게 결정해야 한다.


---
## 이유 1. 인수가 1개 이상이어야 할 때
- 만약, 인수를 무조건 1개 이상으로 받아야 실행할 수 있는 메서드가 있다.
- 해당 메서드가 받는 인수를 가변인수로 만들면, 인수가 0개일 때 문제가 생긴다.
- 이는 런타임 오류를 발생시키고, args 유효성 검사를 명시적으로 추가해야하는 비용이 든다.
- 이를 해결하기 위해선 아래 방법을 사용할 수 있다.


```java
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs) {
        if (arg < min) min = arg;
    }
    return min;
}
```

> 이렇게 매개변수를 2개 받도록 만들면 된다.
>
> 첫 번째로 평범한 매개변수를 하나 받고, 가변 인수를 두 번째로 받아 개수를 보장해주는 것이다. (`min` 변수 초기화 시 문제도 안생김)


---
## 이유 2. 성능에 민감한 상황이라면 가변인수가 걸림돌이 된다.
- 가변인수가 도입되면서 Java의 `printf`, 리플렉션 기능도 가변인수를 사용하여 재정비되었다.
- 그러나 성능에 민감한 상황이라면 가변인수 사용을 조심해야 하는데, 이는 "가변 인수의 동작 방식" 때문이다.


### 가변인수의 동작 방식
가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화 한다. 따라서, 메서드 호출이 많고 가변인수 개수가 많아질수록 비용이 부담될 것이다.


만약, 가변인수의 유연성이 필요하면서도 사용하는 인수 개수가 적다면 다중정의(Overloading) 메서드를 통해 최적화 할 수 있다. (아래 예시 참고)


```java
public void example() { }
public void example(int a1) { }
public void example(int a1, int a2) { }
public void example(int a1, int a2, int a3) { }
public void example(int a1, int a2, int a3, int... rest) { }
```


이렇게 다중정의를 하면, 마지막 메서드를 호출할 때만 배열이 생성될 것이다. 이 기법이 평소에는 별 이득이 없더라도, 꼭 필요한 특수 상황에서 적용하면 좋을 것이다. (가변적으로 사용하는 인수의 절대적인 개수가 적으면서도, 배열을 일일히 생성하는 비용을 줄이고 싶은 경우 등)


EnumSet의 정적 팩토리도 해당 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다. 아래는 `EnumSet`의 실제 코드 일부이다.


```java
// 인수가 2개일 때
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2) {
    EnumSet<E> result = noneOf(e1.getDeclaringClass());
    result.add(e1);
    result.add(e2);
    return result;
}

// 인수가 3개일 때
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3) {
    EnumSet<E> result = noneOf(e1.getDeclaringClass());
    result.add(e1);
    result.add(e2);
    result.add(e3);
    return result;
}

// ...

// 인수가 6개 이상일 때 가변인수 버전 호출
public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest) {
    EnumSet<E> result = noneOf(first.getDeclaringClass());
    result.add(first);
    for (E e : rest)
        result.add(e);
    return result;
}
```



## Summary
- 인수 개수가 일정하지 않은 메서드를 정의하려면 가변인수를 사용하면 된다.
- 그러나 가변인수가 불러올 수 있는 유효성 문제, 성능 문제를 고려해야 한다.