# Item 01. 생성자 대신 정적 팩토리 메서드를 고려하라
클라이언트가 클래스의 인스턴스를 얻는 기본적인 수단은 `public` 생성자이나, 생성자와 별도로 정적 팩토리 메서드를 제공할 수 있다.


## 정적 팩토리 메서드를 사용했을 때 장점
### 1. 이름을 가질 수 있다.
- 생성자에 넘기는 매개변수와 생성자 자체 만으로 반환될 객체의 특성을 제대로 설명하지 못하는 경우가 많다.
- 이런 경우, 정적 팩토리 메서드의 이름을 잘 지으면 반환될 객체를 쉽게 묘사할 수 있다.


### 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
- 불변 클래스에서, 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
- 대표적인 예시로 `Boolean.valueOf(boolean b)` 메서드가 있다.
    ```java
        public final class Boolean implements java.io.Serializable, Comparable<Boolean> {
            
            // 1. Boolean 패키지 내부에 static으로 선언
            public static final Boolean TRUE = new Boolean(true);
            public static final Boolean FALSE = new Boolean(false);

            // ...
            // 2. Boolean.valueOf() 를 사용하면, 프로그램 시작 시 이미 생성되어
            //    static 영역에 올라가있는 단 하나의 Boolean 객체 인스턴스를 참조한다.
            public static Boolean valueOf(boolean b){
                return b ? TRUE : FALSE;
            }
        }
    ```

- 비슷한 기법으로 [Flyweight 패턴이 있다. (아래에 설명)](#flyweight-패턴이란)


#### 정적 팩토리 메서드를 통해 클래스의 인스턴스를 통제할 수 있다.
- "인스턴스 통제"에 대한 개념이 처음에는 잘 안와닿을 수 있는데, 이는 클래스 외부에서 객체를 어떻게 생성하고 사용할지 -> 직접 설계자가 통제할 수 있다는 뜻이다.
- 즉, 객체 생성을 직접 관리할 수 있는 권한이 클래스에 있다는 의미이다.
- 단순히, `new` 생성자 방식만 제공하면, 외부는 통제 없이 언제든 `new`로 객체를 계속 만들 수 있다.
- 만약, 정적 팩토리 메서드를 만들면, 내가 원하는 동작/스펙을 하도록 유도할 수 있게 된다.
    - 이 부분을 책에서 "통제" 한다고 표현한 것이다.
    - ex. 내부적으로 이미 같은 값의 객체가 있을 때 그것을 재사용하라고 설계자가 유도할 수 있다. (위 `Boolean.valueOf()` 예시 참고)
    - 즉, 객체 재사용이 가능해지는 설계이다. (캐싱, 싱글턴)
    - 또한, 싱글턴 측면에서 볼 때 생성할 수 있는 인스턴스 수도 제한할 수 있게 된다.
    - 생성 로직에 부가 로직을 넣어서 설계자가 원하는 동작을 추가로 수행하도록 만들 수도 있다.


    ```java
        public final class Singleton {
            private static final Singleton INSTANCE = new Singleton();
            private Singleton() {} // private으로 생성자 감추고, 정적 팩토리 메서드로만 접근
            public static Singleton getInstance() { return INSTANCE; }
        }
    ```


    - 마지막으로, 정적 팩토리 메서드로 불변 객체를 반환하도록 만들면, 같은 값에 대해 항상 동일한 인스턴스를 반환하므로 → 동치인 인스턴스가 하나뿐임을 보장할 수 있다.


    ```java

        Integer a1 = new Integer(100);
        Integer b1 = new Integer(100);
        System.out.println(a.equals(b)); // true (값이 같음)
        System.out.println(a == b);      // false (다른 객체)

        // 동치 객체를 valueOf()를 통해 하나의 인스턴스를 반환하도록 통제
        Integer a2 = Integer.valueOf(100);
        Integer b2 = Integer.valueOf(100);
        System.out.println(a.equals(b)); // true
        System.out.println(a == b);      // true

    ```


### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
- 반환할 객체의 클래스를 자유롭게 선택할 수 있게 만드는 유연성을 제공한다.
- 이런 유연성은 반환 타입을 "인터페이스 or 부모 클래스"로 선언하고, 어떤 구현체든 자유롭게 반환할 수 있도록 만들었을 때 생긴다.
- 즉, API 스펙 자체를 구체 클래스에 종속되지 않게 만들 수 있다는 것이다.
- 책에서 인터페이스나 내부 컬렉션 프레임워크에 대한 설명이 나오는 이유도, Java 내부에 이를 활용한 추상화가 잘 진행되어있기 때문에 이를 서술 형식으로 설명하고 있는 것것


#### (1) 정적 팩토리 메서드가 인터페이스를 반환하는 경우
- 정적 팩토리 메서드가 인터페이스를 반환 타입으로 선언하고, 내부적으로 해당 인터페이스를 구현한 구체 클래스 인스턴스를 반환하는 형식이다.


```java
    // 반환 타입은 List 인터페이스
    // 실제 반환되는 객체는 Collections$UnmodifiableRandomAccessList 내부 클래스의 인스턴스
    // -> 원본 리스트를 수정 못하게 Read-Only로 Wrapper를 씌운다.

    // 클라이언트는 List 인터페이스만 알고 사용하면 된다.
    // 당연히 내부 구현도 숨길 수 있다.
    List<String> list = Collections.unmodifiableList(Arrays.asList("a", "b"));
```


#### (2) 정적 팩토리 메서드가 부모 클래스를 반환하는 경우
- 정적 팩토리 메서드가 부모 클래스(추상 클래스 or 일반 클래스) 타입을 반환하고, 내부적으로는 해당 부모 클래스를 상속한 하위 클래스의 인스턴스를 반환한다.


```java
    // 반환 타입은 NumberFormat 추상 클래스
    // 내부적으로는 DecimalFormat 같은 하위 클래스의 인스턴스 반환
    NumberFormat nf = NumberFormat.getInstance();
```


```java
    // 반환 타입은 EnumSet<Day> 추상 클래스
    // -> 여기서 Day는 사용자가 정의한 enum 타입

    // 내부적으로는 RegularEnumSet이나 JumboEnumSet 같은 구체 클래스의 인스턴스를 반환한다.
    // enum 크기에 따라 최적화된 구현체 사용 가능하다.
    EnumSet<Day> days = EnumSet.of(Day.MONDAY, Day.WEDNESDAY);

```
- 위에 EnumSet은 아래 4번 장점에서도 볼 수 있다.


### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
- 3번 장점과 이어지는 내용인데, 인터페이스나 부모 클래스를 반환하는 정적 팩토리 메서드에서 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환하게 만들 수 있다.
- 반환 타입의 하위 타입이기만 하면 된다.
- `EnumSet<T extends Enum<T>>` 은 추상 클래스이고, 아래와 같이 구현되어 있다.


```java

    enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY }

    public class Main {
        public static void main(String[] args) {
            EnumSet<Day> workdays = EnumSet.of(Day.MONDAY, Day.TUESDAY, Day.WEDNESDAY);
            System.out.println(workdays); // [MONDAY, TUESDAY, WEDNESDAY]
        }
    }

    // → 내부적으로 noneOf()로 빈 EnumSet 생성 후, 남은 element를 add()
    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E... rest) {
        EnumSet<E> result = EnumSet.noneOf(e1.getDeclaringClass());
        result.add(e1);
        result.add(e2);
        for (E e : rest)
            result.add(e);
        return result;
    }

    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum[] universe = elementType.getEnumConstants();
        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType); // 비트 마스크 1개 (long)
        else
            return new JumboEnumSet<>(elementType);   // 비트 마스크 배열 (long[])
    }

```


- 그러나 이 4번째 장점의 핵심은, 클라이언트가 이 두 클래스 `RegularEnumSet` `JumboEnumSet` 의 존재를 모른다는 것이다.
- 클라이언트는 팩토리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수 없고, 알 필요도 없다.
- 다만, Java 개발자는 지속적으로 성능을 생각하면서 내부 구현체를 추가, 수정, 삭제할 수 있는 것이다.


### 5. 정적 팩토리 메서드를 작성하는 시점에 반환할 객체의 클래스가 존재하지 않아도 된다.
- 이는 서비스 제공자 프레임워크를 만드는 예시로 설명된다.
- 서비스 제공자 프레임워크 =
    > 클라이언트 코드가 특정 구현 클래스를 직접 알지 않아도,
    >
    > 인터페이스(또는 추상 클래스)와 정적 팩토리 메서드를 통해 구현체를 선택하고 사용하게 해주는 구조를 말한다.


계층 구성을 조금 더 추상화해서 간단하게 표현하면 다음과 같다.


| 계층                         | 설명                                  |
| -------------------------- | ----------------------------------- |
| **클라이언트 (Client)**         | 서비스를 사용하는 사용자 코드                    |
| **서비스 접근 API (Framework)** | 정적 팩토리 메서드 제공, 인터페이스 또는 추상 타입에 의존 |
| **서비스 제공자 (Provider)**     | 실제 기능을 구현하는 클래스들 (나중에 추가되어도 됨)      |


여기서, 서비스 접근 API를 정적 팩토리 메서드로 제공하면 클라이언트가 명시한, 원하는 조건에 해당하는 구현체를 반환할 수 있다. (ex. JDBC)


왜 클라이언트에 제공할 API를 만들 때, 정적 팩토리 구조가 유리한지 다음과 같은 비교를 통해 알 수 있다.

| 생성자 방식             | 정적 팩토리 방식                            |
| ------------------ | ------------------------------------ |
| new로 직접 구현체를 알아야 함 | 구현체를 몰라도 정적 메서드가 반환함               |
| 의존성이 강하게 결합됨       | 느슨하게 결합됨 (추상화된 인터페이스로만 의존)           |
| 확장성 낮음             | 나중에 구현체를 추가하기 쉬움 → 서비스 제공자 구조 |



## 정적 팩토리 메서드의 단점
### 1. 상속을 시도할 수 없다
- 정적 팩토리 메서드를 사용할 때는 보통 생성자를 `private`이나 `package-private`으로 제한한다.
- 이는 외부에서 상속한 하위 클래스를 만들 수 없게 한다.
- 상속을 허용하려면 `public` 또는 `protected` 생성자가 필요하므로, 정적 팩토리만 제공하는 클래스는 사실상 상속이 불가능하다.
- 하지만 이 제약은 단점인 동시에, 상속 대신 컴포지션을 유도하고, 불변 객체를 설계할 때 유용한 제약 조건이 될 수 있다.


### 2. 정적 팩토리 메서드는 프로그래머가 찾기 어려울 수 있다.
- 프로그래머가 사용하기 쉽도록, 규약(관습)을 따른 명명 방식을 사용하는 것이 좋다.


| 메서드 이름     | 용도 및 의미 설명                                                                           |
| ---------- | ------------------------------------------------------------------------------------ |
| `from`     | 하나의 매개변수로부터 인스턴스를 반환할 때                           |
| `of`       | 여러 개의 인자를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드     |
| `valueOf`  | `from`과 유사하지만, 변환 또는 캐스팅의 의미가 포함 된 더 자세한 버전  |
| `instance`, `getInstance` | (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다. |
| `create`, `newInstance`   | 매번 새로운 인스턴스를 생성한다는 의도를 나타낼 때      |
| `getType`  |  다른 클래스의 정적 메서드를 통해, 특정 타입의 인스턴스를 반환 `FileStore fs = Files.getFileStore(path)` |
| `newType`  |  다른 클래스의 정적 메서드를 통해, 특정 타입의 인스턴스를 생성 `BuffredReader br = Files.newBufferedReader(path)` |
| `type`     | `getType` 이나 `newType` 의 간결한 버전  |



## Flyweight 패턴이란?
- Flyweight 패턴은 동일한 객체를 공유해서 메모리 사용을 최소화하는 구조 패턴
    
### 개념
- 다수의 객체가 같은 상태(값)를 공유할 때, 공통된 객체를 재사용하여 메모리 낭비를 줄이는 것이 목적이다.
- 자주 사용되는 값 객체`Value Object` 들을 캐싱하고 공유해서 많은 중복 인스턴스 생성을 피한다는 점에서 효율을 추구하는 패턴이다.
    

### Java에서 대표적인 Flyweight 예시
- `Boolean.valueOf()`, `Integer.valueOf()` (특히 -128 ~ 127 범위)
- `String.intern()`


### Flyweight 패턴 활용 예시
- `String.intern()` 이 고유한 스트링 객체를 재사용한다는 점을 이용해, 하나의 JVM 위에서 고유한 값(ex. userId)에 대해서 모니터 락을 잡게 해서 간단한 동기화를 구현하는데 쓰이기도 한다.
- 원래 기본적으로, `lockMap` 등을 이용해 고유 user를 관리하는 방식이 자주 쓰이는데, 토이 프로젝트나 작은 규모의 서비스에서 간이 동기화를 구현할 때 활용할 수 있다.
- 대신, string이 많아질 때에 대비하여 메모리 관리에 주의하고, 작은 규모에서 동기화 방식으로 한정 됨을 이해해야한다.


```java
    public UseCouponResponseDto useCoupon(Long userId, Long price) {
        validateOriginalPrice(price);

        synchronized (lockKey(userId)) {
            List<Coupon> coupons = getUserCouponsOrThrow(userId);
            Coupon bestCoupon = selectBestCouponOrThrow(coupons, price);
            long discountAmount = bestCoupon.getType().calculateDiscount(price);
            couponRepository.delete(bestCoupon);
            return UseCouponResponseDto.of(price, discountAmount, bestCoupon.getType());
        }
    }

    // intern으로 고유한 String 객체에만 접근해서 락을 건다
    // 고유 userId에 대해서 락을 잡으면 해당 유저가 동시에 여러번 시도해도,
    // 모니터락을 잡은 하나의 요청만 처리된다.
    private String lockKey(Long userId) {
        return userId.toString().intern();
    }

```