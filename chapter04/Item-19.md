# Item 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

* 상속은 강력하지만, 올바르게 사용하지 않으면 오류와 취약성을 유발할 수 있다.
* 상속용으로 설계되지 않은 클래스를 하위 클래스에서 사용하면, **예기치 않은 동작이나 깨지기 쉬운 구조**가 될 수 있다.
* 따라서 상속을 허용하려면 **의도적 설계**와 **명확한 문서화**가 필요하다.

---

## 상속 가능한 클래스의 설계 원칙

### 1. 내부 동작 방식 문서화

* \*\*재정의 가능한 메서드(public, protected 중 final이 아닌 것)\*\*가 언제, 어떻게 호출되는지 **문서에 명시**해야 한다.
* 이는 **하위 클래스가 어떤 메서드를 재정의할 수 있고**, **그 메서드가 어떤 타이밍에 호출되는지 예측할 수 있도록** 돕는다.

> Java 8부터 제공되는 `@implSpec` 태그는 해당 메서드의 내부 동작 및 호출 조건을 문서화하는 데 사용할 수 있다.

```java
/**
 * Returns the hash code for this set.
 *
 * @implSpec
 * This implementation returns the sum of the hash codes of the elements in the set.
 */
public int hashCode() { ... }
```

---

## Hook 메서드 설계

* 상속을 고려한 클래스는 **하위 클래스가 끼어들 수 있는 지점**, 즉 **Hook 메서드**를 `protected`로 제공해야 할 수도 있다.
* Hook 메서드는 상속받는 쪽이 핵심 로직의 흐름을 수정하지 않고 **맞춤 동작을 삽입할 수 있게** 도와준다.
* 지나치게 많은 메서드를 `protected` 로 공개하면 오히려 설계가 복잡해지고 결합도가 높아질 수 있다.


### Hook 메서드 예시


```java
public abstract class FileProcessor {
    public final void process(String path) {
        openFile(path);
        readContent();
        if (shouldValidate()) { // shouldValidate()를 호출하여 검사
            validate();
        }
        closeFile();
    }

    protected void openFile(String path) {
        System.out.println("Opening file: " + path);
    }

    protected abstract void readContent();

    protected void validate() {
        System.out.println("Validating content");
    }

    protected void closeFile() {
        System.out.println("Closing file");
    }

    // Hook: 하위 클래스가 원하는 경우에만 끼어들 수 있음
    protected boolean shouldValidate() {
        return true; // 기본 구현
    }
}

// 상속 받아 사용할 때
public class FastFileProcessor extends FileProcessor {
    @Override
    protected void readContent() {
        System.out.println("Reading content quickly");
    }

    @Override
    protected boolean shouldValidate() {
        return false; // 검증 건너뜀
    }
}

```


- `shouldValidate()`는 Hook 메서드로, 하위 클래스가 검증 여부만 조절하도록 허용한다.
- 상위 클래스의 `process()` 로직은 변경되지 않고 유지된다.
- 이렇게 코드를 짜면, 하위 클래스가 불필요하게 상위 클래스 전체 흐름을 오버라이드하지 않아도 된다.
- 상속 설계를 더 안전하고 유연하게 만들어준다.
- 중복 제거와 코드 흐름 제어권 유지를 동시에 가능하게 한다.


---

## 상속용 클래스 만들기

* 상속을 염두에 둔 클래스는 **직접 하위 클래스를 만들어 테스트**해야 한다.

  * 어떤 메서드를 공개해야 하는지,
  * 어떤 메서드는 숨겨야 하는지 판단하는 데 도움이 된다.
* **생성자에서 재정의 가능한 메서드를 호출하지 말 것**

  * 이로 인해 하위 클래스의 상태가 완전히 초기화되기 전에 메서드가 호출되는 일이 발생할 수 있다.

### 잘못된 예시

```java
public class Super {
    public Super() {
        overrideMe(); // 위험한 호출
    }

    public void overrideMe() { }
}

public final class Sub extends Super {
    private final Instant instant;

    Sub() {
        instant = Instant.now();
    }

    @Override
    public void overrideMe() {
        System.out.println(instant); // NullPointerException 발생 가능
    }
}
```

* `Super()`가 먼저 실행되며 `overrideMe()`가 호출된다.
* 하지만 `Sub()`는 아직 `instant`를 초기화하지 않았기 때문에 예외가 발생한다.

---

## 직렬화와 clone에서도 주의

* `readObject()`와 `clone()`도 내부적으로 **재정의 가능한 메서드를 호출하면 안 된다.**
* `readObject()`는 하위 클래스의 역직렬화가 완료되기 전에 메서드 호출을 유발할 수 있고,
  `clone()`은 복제 객체의 상태가 완전히 초기화되기 전에 메서드가 호출될 수 있다.
* 특히 `clone()`은 복제본뿐 아니라 **원본 객체에도 영향을 줄 수 있어 치명적이다.**

---

## Serializable 구현 시 주의

* 상속 가능한 클래스에서 `writeReplace()`나 `readResolve()`를 제공할 경우 **접근 제어자를 반드시 `protected`로 설정**해야 한다.
* 만약 `private`로 선언하면 하위 클래스에서 이 메서드가 무시되어 **의도한 직렬화/역직렬화 흐름이 깨진다.**

---

## 상속을 금지하는 방법

> 상속을 고려하지 않은 클래스라면, 명시적으로 **상속을 막아야** 한다.

1. **`final` 클래스 선언**

   * 가장 확실하게 상속을 금지하는 방법
2. **생성자를 `private` 또는 `package-private`으로 숨기고, 정적 팩터리 메서드만 제공**

   * 외부에서는 인스턴스 생성을 제한할 수 있고, 내부 구현 변경에 유연함

```java
public final class UtilityClass {
    private UtilityClass() {
        throw new AssertionError(); // 인스턴스화 방지
    }

    public static int add(int a, int b) {
        return a + b;
    }
}
```

---

## Summary

* 상속은 강력한 도구지만, **설계와 문서화 없이는 위험하다.**
* 상속 가능한 클래스는 다음을 준수해야 한다:

  * `@implSpec` 또는 JavaDoc을 통해 재정의 가능 메서드의 호출 조건을 명시
  * 필요한 Hook 메서드만 선별적으로 공개
  * 생성자, readObject, clone 내부에서 재정의 메서드 호출 금지
* **직렬화와 복제와 관련된 메서드도 상속 가능성을 고려하여 접근 제어자 설정**
* 상속이 불필요하거나 의도하지 않은 경우 `final`이나 정적 팩터리 방식으로 상속을 명시적으로 막을 것


