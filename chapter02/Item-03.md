# Item 03. private 생성자나 enum 타입으로 싱글턴임을 보증하라

## 싱글턴(Singleton)이란?
- 인스턴스를 오직 하나만 생성하도록 보장하는 패턴
- 애플리케이션 전역에서 동일한 인스턴스를 공유할 수 있음
- 주로 무상태(stateless) 유틸리티 객체, 공통 설정 객체 등에 사용됨


## 자바에서 싱글턴을 구현하는 방법
### 1. `private` 생성자 + `static` 필드
- `private` 생성자를 통해 외부에서 객체 생성 호출을 막는다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }; // Elvis.INSTANCE를 초기화 할 때 딱 한 번 호출
}
```


- 장점: 간단하고 명시적이다.
- 단점: 리플렉션, 역직렬화 공격에 의해 인스턴스가 추가로 생성될 수 있음
    - 이를 막기 위해서는 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.



### 2. 정적 팩토리 방식의 싱글턴


```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {} // 외부에서 new 금지

    public static Singleton getInstance() {
        return INSTANCE; // 항상 같은 객체를 반환
    }
}
```


### 3. `Enum` 타입으로 싱글턴 구현

```java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        // ...
    }
}
```

- 자바가 Enum 타입을 싱글턴으로 보장한다. (아래 자세한 설명)
    - 자바는 Enum 타입을 클래스 로딩 시점에 단 한 번만 초기화한다.
    - Enum 타입은 리플렉션에 안전하고, 추가적인 노력 없이 직렬화가 가능하다.
- 단점으로는 Enum은 상속 불가능, 복잡한 클래스는 Enum으로 표현하기 어렵다.


## Enum으로 싱글턴을 보장하는 이유
- 컴파일러가 `Enum`을 **final 클래스**로 컴파일함 (`abstract methods` 허용은 됨)
- 내부적으로 `INSTANCE`는 `private static final Singleton INSTANCE = new Singleton();`과 유사하게 동작
- JVM 레벨에서 인스턴스를 단 1개만 생성하도록 보장 (리플렉션도 막힘)


---

## 싱글턴으로 만들어도 되는 객체 조건
### 무상태 (Stateless)

- 필드에 상태가 없고, 동시 접근에도 상태가 변하지 않는 객체일 때
    - ex. 암호화 유틸, 포맷터, Validator 등
- 상태를 가진 클래스는 싱글턴으로 만들면 위험하다.
    - 여러 스레드가 공유 객체에 접근하여 경합, 불일치, race condition 발생 가능


## 테스트와 싱글턴의 상충
- 테스트 환경에서는 **Mock/Stub 대체가 필요**할 수 있는데, 직접 구현한 싱글턴은 이를 어렵게 만든다.
- **DI (Dependency Injection)** 사용하면 **싱글턴처럼 하나만 공유**하면서도, **테스트에서는 교체 가능**하다.
    - 따라서 Spring에서 컨테이너한테 DI, 싱글턴 관리를 맡기는 경우가 많다.



## Spring Framework에서의 싱글턴 관리

| 스코프           | 설명                     | 사용 예               |
| ------------- | ---------------------- | ------------------ |
| `singleton`   | 애플리케이션 전체에 1개    | 대부분의 서비스 빈         |
| `prototype`   | 요청할 때마다 새 인스턴스         | 요청마다 상태가 달라야 할 때   |
| `request`     | HTTP 요청마다 새 인스턴스       | 웹 컨트롤러 요청 처리       |
| `session`     | HTTP 세션마다 새 인스턴스       | 사용자 인증 정보 관리       |
| `application` | ServletContext 당 1개    | 전체 애플리케이션에 공유되는 객체 |


>
> 대부분의 경우
>
> 직접 싱글턴을 구현하기보다 Spring Bean으로 등록하고 `@Service`, `@Component` 등으로 DI받아 사용한다.
> 개발할 때, 상태가 없는 단순 Util 클래스들을 직접 수동으로 싱글턴 관리하는 것보다 `@Component` 등의 어노테이션을 통해 스프링에 맡기는 경우가 대부분이다. 그러나 왜 이런 코드가 관습처럼 사용되는지, 스프링 컨테이너의 싱글톤 관리가 어떤지, 한 번 쯤 다시 생각해볼 필요가 있다.
>
