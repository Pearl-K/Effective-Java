# Item 04. 인스턴스화를 막으려거든 private 생성자를 사용하라
- 정적 메서드와 정적 필드만을 담은 클래스가 필요할 수 있다.
- 특히, 유틸리티 클래스에서 상태가 변하지 않는 클래스들이 있는데, 이 클래스들의 인스턴스화를 막는 것이 효율적이다.
    - `.class` 파일 로딩 시 `static` 메서드를 바로 사용할 수 있으므로 (static 영역에 적재) 인스턴스를 만들 필요가 없는 것이다.
    - 오히려 인스턴스가 계속 생성되면 메모리 낭비이므로 적절한 방법으로 막을 필요가 있다.
- 인스턴스화를 막는다는 개념은 "아예 인스턴스를 만들지 못하게 하는 것"을 의미한다.
    - 대표적인 예시로 `java.lang.Math` `Collections` 등의 API가 있다.


## `java.util.Collection`
- 특정 인터페이스를 구현하는 객체를 생성해주는 정적 팩토리 메서드를 모아놓을 수 있다.
- Java8부터 이런 메서드를 인터페이스에 넣을 수 있게 되었다.


## `private`으로 인스턴스화를 막는게 필요한 경우
- 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어주는데, 이로 인해 의도치 않은 인스턴스화가 가능하다.
- 종종 OpenAPI 에서도 이런 인스턴스화 할 수 있는 지점이 발견되곤 하는데, 의도치 않은 동작과 메모리 낭비가 생기므로 막아야한다.
- 추상 클래스로 만드는 것도 절대적인 해결법은 아니다. (상속해서 사용할 수 있기 때문)


- 이 때, `private` 생성자를 직접 추가하면 클래스의 인스턴스화를 막을 수 있다.
- 만약, `private` 생성자 안에서도 Exception을 던진다면 클래스 내부에서 실수로 호출되는 상황도 막을 수 있다.
- 이는 상속을 아예 불가능하게 만드니, 꼭 필요한 클래스에서만 수행해야한다.


```java
public class Collections {
    // Suppresses default constructor, ensuring non-instantiability.
    private Collections() {
    }
    
    // ...
}
```


## 추가: Java Spring 에서 Util 클래스
- 스프링에서 유틸 클래스는 정적 메서드로만 구성되어 인스턴스화를 막는 유틸 클래스와, `@Component` 빈 등록을 통해 싱글턴으로 관리 되는 유틸 클래스가 있다.
- 내부가 정적 메서드로만 구성될 수 있으면 전자를, 다른 빈을 주입 받아서 만들어야 하거나 테스트 Mock 대체가 필요할 때는 후자를 택한다.
- 특히, 인스턴스화를 막아야하는 Util 클래스 중, 보안적 의도를 명시하고 싶을 때 `private` 생성자 막기를 작성하는게 좋다.


```java
public final class SecurityPolicyUtils {
    private SecurityPolicyUtils() {
        throw new AssertionError("Cannot be instantiated");
    }

    public static boolean isIpBlocked(String ip) {
        // 보안 정책 적용
        return ip.startsWith("10.");
    }
}
```


이렇게 만들면:
- final로 상속 금지 정책
- private 생성자 + Exception으로 내부 호출도 차단
- 보안 정책을 가진 클래스를 누군가가 상속해서 오버라이드하거나, 인스턴스화해서 다른 동작 유도를 막는다

