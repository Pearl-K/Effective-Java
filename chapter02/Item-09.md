# Item 09. try-finally 보다는 try-with-resources 를 사용하라
## 직접 닫아줘야 하는 자원들
- Java 라이브러리에는 `close` 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.
- 예를 들어, `InputStream`, `OutputStream`, `java.sql.Connection` 등이 있다.
- 클라이언트가 자원을 명시적으로 닫는 것을 놓치기 쉽기 때문에 안전망이 필요하다.


## 전통적으로 자원을 닫는 방법: `try-finally`
- 해당 방식은 한계가 존재한다.
  - 자원을 여러 개 닫아야 할 때 코드가 복잡해진다.
  - try 블록과 finally 블록에서 일어나는 예외 상황이 여러 개라면, 안전하게 처리되지 않음 (특정 예외 묻힘)


## 안전한 방법 (Java7+): `try-with-resources`
- 이 구조를 사용하려면, 해당 자원이 `AutoCloseable` 인터페이스를 구현해야 한다.
- 자바 라이브러리 및 third-party 라이브러리들의 수많은 클래스와 인터페이스가 이미 이를 구현하거나 확장했다.
- 만약, 개발자가 수동으로 닫아야 하는 자원을 만든다면 `AutoCloseable`을 반드시 구현하는게 안전하다.



## Summary
- 꼭 회수해야 하는 자원을 다룰 때, `try-finally` 대신 `try-with-resources`를 사용하자.
- 코드가 더 짧고 분명해지며, 추적되는 예외 정보도 훨씬 유용하며, 정확하고 쉬운 자원 회수가 가능해진다.