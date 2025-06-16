# Item 25. 톱레벨 클래스는 한 파일에 하나만 담으라
- 소스 파일 하나에 톱레벨 클래스를 여러 개 선언해도 컴파일러가 컴파일을 해주지만, 이는 매우 위험하다.
- 한 파일에 여러 개의 톱레벨 클래스를 담으면 위험한 점은 다음과 같다.
  1. 한 클래스를 여러 가지로 정의할 수 있어서 위험함
  2. 여러 가지로 정의된 클래스 중, 실제 무엇을 사용할 지는 컴파일러가 어느 소스 파일을 먼저 컴파일하느냐에 따라 달라짐


## 간단한 해결책
- 이는 단순히 톱레벨 클래스들을 서로 다른 소스 파일로 분리하여 해결할 수 있다.
- 여러 톱레벨 클래스를 꼭 한 파일에 담고 싶다면, 정적 멤버 클래스를 사용할 수도 있다.
    ```java
  public class Test{
      public static void main(String[] args){
        System.out.println(Utensil.NAME + Dessert.NAME);
      }
      private static class Utensil {
        static final String NAME = "pan";
      }
      private static class Dessert {
        static final String NAME = "cake";
      }
  }
  ```

## Summary
- 소스 파일 하나에는 반드시 톱레벨 클래스(or 톱레벨 인터페이스)를 하나만 담자.
- 컴파일러가 한 클래스에 대한 정의를 여러 개 만들어 내는 일을 없애야 한다.
- 소스 파일을 어떤 순서로 컴파일하든, binary 파일이나 프로그램 동작이 달라지는 일이 일어나지 않게 만들기 위함이다.