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
- `try-with-resources` 에도 `try-finally` 처럼 `catch`를 사용하여 다수의 예외를 처리하게 만들 수도 있다.


## `try-finally` 가 적용된 코드를 `try-with-resources`로 수정하기
### 1. 자원이 단수


```java
static String firstLineOfFile(String path) throws IOException {
  BufferedReader br = new BufferedReader(new FileReader(path));
  try {
    return br.readLine();
  } finally {
    br.close();
  }
}
```



```java
static String firstLineOfFile(String path) throws IOException {
  try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readLine();
  }
}
```


- 코드가 훨씬 간편해지고, 자원을 회수하기 좋다.
- 리소스를 닫는 부분은 `try-with-resources` 문법을 통해 자동으로 처리된다.
- `BufferedReader`가 `AutoCloseable` 인터페이스를 구현하고 있어서, 컴파일러가 자동으로 `br.close()` 를 호출하는 finally 블록을 만들어준다.


### 2. 자원이 복수


```java
static void copy(String src, String dst) throws IOException {
  InputStream in = new FileInputStream(src);
  try {
    OuputStream out = new FileOutputStream(dst);
    try {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0)
        out.write(buf, 0, n);
    } finally {
      out.close();
    }
  } finally {
    in.close();
  }
}
```


- 이렇게 Input과 Output 자원 두 가지를 사용하니 `try-finally` 가 중첩되면서 코드가 복잡해진다.
- 예외 처리 디버깅에도 좋지 못하다.
  - 특정 문제로 연쇄적으로 메서드가 실패하고 예외가 터지면 나중 예외가 초기 예외를 삼켜서 스택 추적이 불편해진다.


```java
Static void copy(String src, String dst) throws IOException {
  try (InputStream in = new FileInputStream(src); OutputStream out = new FileOutputStream(dst)) {
    byte[] buf = new byte[BUFFER_SIZE];
    int n;
    while ((n = in.read(buf)) >= 0)
      out.write(buf, 0, n);
  }
}
```


- 이렇게 try-with-resource 문에 여러 리소스를 선언할 수도 있다.
- 여러 리소스를 선언하면, 선언된 순서대로 리소스를 열고, try 블록이 끝나면 선언 역순으로 리소스를 닫는다.
- 역순으로 닫는 이유는, 예외 발생 시 리소스 누수나 의존 관계 꼬임 등을 방지하기 위함이다.



## Summary
- 꼭 회수해야 하는 자원을 다룰 때, `try-finally` 대신 `try-with-resources`를 사용하자.
- 코드가 더 짧고 분명해지고, 추적되는 예외 정보도 훨씬 유용하며, 정확하고 쉬운 자원 회수가 가능해진다.
