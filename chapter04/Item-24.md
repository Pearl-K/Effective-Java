# Item 24. 멤버 클래스는 되도록 static으로 만들라

* 중첩 클래스(Nested Class)란 다른 클래스 안에 정의된 클래스를 말한다.
* 중첩 클래스는 자신을 감싼 바깥 클래스에서만 사용되어야 하고, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

---

## 중첩 클래스의 종류 4가지

* 여기서 첫 번째를 제외한 나머지 클래스는 inner class에 해당한다.
* 각각의 중첩 클래스를 언제, 그리고 왜 사용해야 하는지 설명한다.

---

### 1. 정적 멤버 클래스

* 바깥 클래스의 인스턴스와 독립적으로 동작
* 바깥 클래스의 static 멤버에는 접근 가능
* 보통 바깥 클래스의 **도우미 역할**을 할 때 사용
* 바깥 클래스와 논리적으로만 연관되어 있고, 물리적으로 참조할 필요가 없을 때 유용

```java
public class Outer {
    static class StaticNested {
        void hello() {
            System.out.println("Hello from static nested class!");
        }
    }

    public static void main(String[] args) {
        StaticNested nested = new StaticNested();
        nested.hello();
    }
}
```

---

### 2. (비정적) 멤버 클래스

* 바깥 클래스의 인스턴스에 암묵적으로 연결되어 있음 (`Outer.this`)
* 바깥 클래스의 모든 멤버(필드, 메서드 포함)에 접근 가능
* 주로 바깥 클래스의 **인스턴스와 긴밀하게 연결된 상태**를 표현할 때 사용

```java
public class Outer {
    private String greeting = "Hi";

    class Inner {
        void hello() {
            System.out.println(greeting); // 바깥 클래스 필드 참조 가능
        }
    }

    public static void main(String[] args) {
        Outer outer = new Outer();
        Outer.Inner inner = outer.new Inner();
        inner.hello();
    }
}
```

---

### 3. 익명 클래스

* 이름이 없고, 바깥 클래스의 멤버가 아니다.
* 선언과 동시에 인스턴스를 생성하며, 한 번만 사용될 때 적합
* 특정 메서드 구현을 간단히 전달하고 싶을 때 자주 사용됨 (ex: 리스너, Comparator 등)

```java
import java.util.Comparator;

public class Example {
    public static void main(String[] args) {
        Comparator<String> comp = new Comparator<>() {
            @Override
            public int compare(String a, String b) {
                return b.length() - a.length();
            }
        };

        System.out.println(comp.compare("hello", "hi")); // 결과: -3
    }
}
```

---

### 4. 지역 클래스

* 지역 변수처럼 특정 메서드 안에 정의되는 클래스
* 이름이 있는 클래스지만, 해당 블록 바깥에서는 사용할 수 없음
* 가독성을 해치지 않는 선에서 짧게 작성되어야 함

```java
public class Example {
    public void greet(String name) {
        class Greeter {
            void sayHello() {
                System.out.println("Hello, " + name);
            }
        }

        Greeter greeter = new Greeter();
        greeter.sayHello();
    }

    public static void main(String[] args) {
        new Example().greet("Jinju");
    }
}
```

---

## Summary

* 중첩 클래스에는 4가지 종류가 있으며, 각각 쓰임이 다르다.
* 메서드 밖에서도 사용해야 하거나, 메서드 안에 정의하기에 너무 길다면 멤버 클래스로 만든다.
* **비정적 멤버 클래스**: 인스턴스 각각이 바깥 인스턴스를 참조한다면 선택
* **정적 멤버 클래스**: 위 비정적 클래스 조건에 해당하지 않을 때 선택
* **익명 클래스**

    * 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고,
    * 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있을 때 선택
* **지역 클래스**: 위 익명 클래스 조건에 해당하지 않을 때 선택

