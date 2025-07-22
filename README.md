# ☕ Effective-Java
『이펙티브 자바 3판』기준으로 정리, 다양한 내용을 추가하여 재구성한 학습용 레포지토리 입니다.

## 📑 Rules
1. 주 1회 오프라인 모임
2. 해당 주차 정해진 분량의 책을 읽고 정리한다. (분량은 약 40page 내외)
    - Item 별로 구분해서 준비한다.
        - 이전에 몰랐던 것 중에 새로 알게된 부분
        - 본인이 중요하다고 생각한 부분 (정리할 필요가 있거나 강조할 만한 부분)
        - 관련 내용 중에서 코드에 적용해본 경험이 있거나 새로 적용해본 부분 (실습 코드)
3. 정리 플랫폼 자유
    - Github
    - Notion
    - 기타 기술 블로그나 코드, 내용 공유할 수 있는 플랫폼 가능


## 🗂️ Contents
### Chapter 02. 객체 생성과 파괴
- [Item 01. 생성자 대신 정적 팩토리 메서드를 고려하라](/chapter02/Item-01.md)
- [Item 02. 생성자에 매개변수가 많다면 빌더를 고려하라](/chapter02/Item-02.md)
- [Item 03. private 생성자나 enum 타입으로 싱글턴임을 보증하라](/chapter02/Item-03.md)
- [Item 04. 인스턴스화를 막으려거든 private 생성자를 사용하라](/chapter02/Item-04.md)
- [Item 05. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라](/chapter02/Item-05.md)
- [Item 06. 불필요한 객체 생성을 피하라](/chapter02/Item-06.md)
- [Item 07. 다 쓴 객체 참조를 해제하라](/chapter02/Item-07.md)
- [Item 08. finalizer와 cleaner 사용을 피하라](/chapter02/Item-08.md)
- [Item 09. try-finally 보다는 try-with-resources 를 사용하라](/chapter02/Item-09.md)

---

### Chapter 03. 모든 객체의 공통 메서드
- 추후 학습

---

### Chapter 04. 클래스와 인터페이스
- [Item 15. 클래스와 멤버의 접근 권한을 최소화하라](/chapter04/Item-15.md)
- [Item 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라](/chapter04/Item-16.md)
- [Item 17. 변경 가능성을 최소화하라](/chapter04/Item-17.md)
- [Item 18. 상속보다는 컴포지션을 사용하라](/chapter04/Item-18.md)
- [Item 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라](/chapter04/Item-19.md)
- [Item 20. 추상 클래스보다는 인터페이스를 우선하라](/chapter04/Item-20.md)
- [Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라](/chapter04/Item-21.md)
- [Item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라](/chapter04/Item-22.md)
- [Item 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라](/chapter04/Item-23.md)
- [Item 24. 멤버 클래스는 되도록 static으로 만들라](/chapter04/Item-24.md)
- [Item 25. 톱레벨 클래스는 한 파일에 하나만 담으라](/chapter04/Item-25.md)


---
### Chapter 05. 제네릭
- [Item 26. 로 타입은 사용하지 말라](/chapter05/Item-26.md)
- [Item 27. 비검사 경고를 제거하라](/chapter05/Item-27.md)
- [Item 28. 배열보다는 리스트를 사용하라](/chapter05/Item-28.md)
- [Item 29. 이왕이면 제네릭 타입으로 만들라](/chapter05/Item-29.md)
- [Item 30. 이왕이면 제네릭 메서드로 만들라](/chapter05/Item-30.md)
- [Item 31. 한정적 와일드카드를 사용해 API 유연성을 높이라](/chapter05/Item-31.md)
- [Item 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라](/chapter05/Item-32.md)
- [Item 33. 타입 안전 이종 컨테이너를 고려하라](/chapter05/Item-33.md)


---
### Chapter 06. 열거 타입과 애너테이션
- 추후 학습


---
### Chapter 07. 람다와 스트림
- [Item 42. 익명 클래스보다는 람다를 사용하라](/chapter07/Item-42.md)
- [Item 43. 람다보다는 메서드 참조를 사용하라](/chapter07/Item-43.md)
- [Item 44. 표준 함수형 인터페이스를 사용하라](/chapter07/Item-44.md)
- [Item 45. 스트림은 주의해서 사용하라](/chapter07/Item-45.md)
- [Item 46. 스트림에서는 부작용 없는 함수를 사용하라](/chapter07/Item-46.md)
- [Item 47. 반환 타입으로는 스트림보다 컬렉션이 낫다](/chapter07/Item-47.md)
- [Item 48. 스트림 병렬화는 주의해서 적용하라](/chapter07/Item-48.md)


---
### Chapter 08. 메서드
- [Item 49. 매개변수가 유효한지 검사하라](/chapter08/Item-49.md)
- [Item 50. 적시에 방어적 복사본을 만들라](/chapter08/Item-50.md)
- [Item 51. 메서드 시그니처를 신중히 설계하라](/chapter08/Item-51.md)
- [Item 52. 다중정의는 신중히 사용하라](/chapter08/Item-52.md)
- [Item 53. 가변인수는 신중히 사용하라](/chapter08/Item-53.md)
- [Item 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라](/chapter08/Item-54.md)
- [Item 55. 옵셔널 반환은 신중히 하라](/chapter08/Item-55.md)
- [Item 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라](/chapter08/Item-56.md)


---
### Chapter 09. 일반적인 프로그래밍 원칙
- 추후 학습


---
### Chapter 10. 예외
- [Item 69. 예외는 진짜 예외 상황에만 사용하라](/chapter10/Item-69.md)
- [Item 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라](/chapter10/Item-70.md)
- [Item 71. 필요 없는 검사 예외 사용은 피하라](/chapter10/Item-71.md)
- [Item 72. 표준 예외를 사용하라](/chapter10/Item-72.md)
- [Item 73. 추상화 수준에 맞는 예외를 던져라](/chapter10/Item-73.md)
- [Item 74. 메서드가 던지는 모든 예외를 문서화하라](/chapter10/Item-74.md)
- [Item 75. 예외의 상세 메시지에 실패 관련 정보를 담으라](/chapter10/Item-75.md)
- [Item 76. 가능한 한 실패 원자적으로 만들라](/chapter10/Item-76.md)
- [Item 77. 예외를 무시하지 말라](/chapter10/Item-77.md)


---
### Chapter 11. 동시성
- [Item 78. 공유 중인 가변 데이터는 동기화해 사용하라](/chapter11/Item-78.md)
- [Item 79. 과도한 동기화는 피하라](/chapter11/Item-79.md)


---
## 📌 Update History
- 1주차: Item 01-05 (25.06.04)
- 2주차: Item 06-09 (25.06.10)
- 3주차: Item 15-25 (25.06.17)
- 4주차: Item 26-30 (25.06.24)
- 5주차: Item 31-33 & 42-45 (25.07.01)
- 6주차: Item 46-52 (25.07.08)
- 7주차: Item 53-56 & 69-72 (25.07.15)
- 8주차: Item 73-79 (25.07.22)