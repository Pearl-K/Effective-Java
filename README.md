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
- 개별 학습

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


---
## 📌 Update History
- 1주차: Item 01~05 (25.06.04)
- 2주차: Item 06~09 (25.06.10)
- 3주차: Item 15~25 (25.06.17)
- 4주차: Item 26~30 (25.06.24)