# Item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라
- 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.
- 즉, 자신의 인스턴스로 무엇을 할 수 있는지, 클라이언트에게 알려주는 역할이다.
- 인터페이스는 오직 이 용도로 사용해야 한다.

## 안티 패턴
- 상수 인터페이스는 이 지침에 맞지 않는 안티패턴이다.
- 이는 인터페이스를 잘못 사용한 예로, 클래스 내부에서 사용하는 상수는 내부 구현에 해당한다.
- 이는, 내부 구현을 클래스 API로 노출하는 행위이고, 사용자들을 이 상수에 종속되게 만든다.


이런 상수 인터페이스를 대체하는 방안으로 더 합당한 선택지들이 있다.

### 1. 특정 클래스나 인터페이스와 강하게 연결된 상수라면 클래스나 인터페이스 자체에 추가하기
- ex. `Integer.MIN_VALUE`, `Integer.MAX_VALUE` 등

### 2. Enum 타입으로 나타내기
- ex. `public enum Day{ MON, TUE, WED, THU, FRI, SAT, SUN};`

### 3. 인스턴스화를 막아놓은 유틸리티 클래스에 담아 공개하기
만약 상수를 공개할 목적이라면 특정 클래스나 인터페이스 자체에 추가해야한다. 열거 타입으로 나타내기 적합한 상수라면 열거 타입으로 만들어 공개하면 되고,


```java
public class PysicalConstants{
      private PysicalConstants(){}; // private 생성자로 인스턴스화 방지
      public static final double AVOGARDROS_NUMBER = 6.022_140_857e23;
      public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
      public static final double ELECTRON_MASS = 9.109_383_56e-3;
}
```


## Summary
- 인터페이스는 타입을 정의하는 용도로만 사용해야 한다.
- 상수 공개용 수단으로 사용하는 것은 안티패턴이므로 사용하지 말자.