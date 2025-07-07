# Item 33. 타입 안전 이종 컨테이너를 고려하라

일반적인 제네릭 컨테이너(`Set<E>`, `Map<K,V>`, `ThreadLocal<T>`, `AtomicReference<T>` 등)는 컨테이너 자신이 타입 매개변수를 갖는다. 이 경우 컨테이너가 다룰 수 있는 타입의 수가 고정되어 있다.

하지만 **타입 안전 이종 컨테이너(type-safe heterogeneous container)** 패턴을 사용하면, 컨테이너가 아닌 **키(key)를 타입 토큰(type token)** 으로 활용해 다양한 타입을 한 컨테이너에 저장할 수 있다.

---

## 패턴 개요

1. **키를 제네릭 타입 토큰**으로 쓴다 (`Class<T>`, 또는 직접 정의한 키 타입).
2. 값을 넣을 때는 키와 함께 저장하여 타입 정보를 유지한다.
3. 값을 꺼낼 때는 키의 `cast()` 메서드로 안전하게 형변환한다.

```java
public class Favorites {
    private final Map<Class<?>, Object> favorites = new HashMap<>();

    // 키와 값의 타입이 일치하는지 검증하며 저장
    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }

    // 키를 이용해 저장된 값을 안전하게 꺼낸다
    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }

    public static void main(String[] args) {
        Favorites f = new Favorites();
        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 0xcafebabe);
        f.putFavorite(Class.class, Favorites.class);

        String s = f.getFavorite(String.class);
        int    i = f.getFavorite(Integer.class);
        Class<?> c = f.getFavorite(Class.class);

        System.out.printf("%s %x %s%n", s, i, c.getName());
    }
}
```

* `String.class`는 `Class<String>`이고, `Integer.class`는 `Class<Integer>`다.
* **타입 토큰(type token)** 이란 메서드에 전달되는 `Class<T>` 같은 런타임 타입 정보를 가리킨다.
* `Map<Class<?>, Object>`에 저장된 값이 타입 안전한 이유는 `getFavorite`에서 `Class.cast()`로 검증하기 때문.

---

## 주요 요소

| 구성 요소                 | 역할                                      |
| --------------------- | --------------------------------------- |
| 키: `Class<T>`         | 타입 토큰, 값을 저장·검색할 때 타입 정보를 제공            |
| 값: `Object`           | 모든 타입을 저장 가능하지만, 캐스팅 책임은 `Class.cast()` |
| `type.cast(instance)` | 런타임 타입 안전 검사 및 형변환                      |

---

## 주의 사항 & 제약

* **로타입(raw type) 키 입력**: 클라이언트가 `(Class) Integer.class`처럼 로타입을 사용하면 컴파일 경고 없이 잘못된 값을 넣을 수 있다.
* **무결성 확보**: `putFavorite`에서 `type.cast(instance)`를 활용해, 키와 값의 타입이 일치하는지 항상 검증하자.
* **실체화 불가 타입**: `List<String>` 같은 제네릭 클래스는 `List.class` 한 종류만 존재하므로, 타입 토큰으로 사용 불가.

---

## 한정적 타입 토큰

특정 키 타입만 허용하고 싶을 때는 한정적 타입 토큰을 사용한다.

```java
public interface Annotation { /* ... */ }

// 한정적 타입 토큰 활용 예
<T extends Annotation>
T getAnnotation(Class<T> annotationType);
```

* `getAnnotation(Class<T extends Annotation>)`은 `Annotation` 서브타입만 키로 허용한다.
* `Class.asSubclass()`를 활용해 런타임에 안전한 다운캐스트가 가능하다.

```java
public static Annotation getAnnotation(AnnotatedElement el, String name) {
    try {
        Class<?> cls = Class.forName(name);
        // 런타임에 Annotation 서브타입으로 검증
        Class<? extends Annotation> token = cls.asSubclass(Annotation.class);
        return el.getAnnotation(token);
    } catch (ClassCastException | ClassNotFoundException ex) {
        throw new IllegalArgumentException(ex);
    }
}
```

---

## Summary
* 컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다.
* 하지만 타입 안전 이종 컨테이너를 활용하여 제약을 없앨 수 있다.
  * **타입 안전 이종 컨테이너** 패턴은 키를 타입 토큰으로 사용해, 컨테이너가 다양한 타입을 저장하도록 한다.
  * `Class` 를 키로 사용하고, 여기 쓰이는 `Class` 객체를 타입 토큰이라 한다.

* 이 패턴을 사용하면, 한 컨테이너 안에 여러 종류의 원소를 타입 안전하게 보관할 수 있다.
