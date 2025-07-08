# Item 50. 적시에 방어적 복사본을 만들라
자바는 충분히 안전한 언어이지만, 클라이언트가 개발자의 불변식을 깨뜨릴 수 있어서 방어적으로 프로그래밍해야 한다.


## 🚨 1. 의도치 않게 내부가 수정되는 코드
- 어떤 객체든 해당 객체의 허락 없이 외부에서 내부를 수정할 수 없다.
- 하지만, 코드 상의 문제가 있으면 의도치 않게 내부 수정을 허락할 수 있다.


```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException();
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }

    // ...
}
```

- 위 클래스는 언뜻 보면 불변으로 보이나, `Date` 가 가변이라는 사실을 알면 불변식을 깨뜨릴 수 있다.
- ex. `end.setYear(2000);` 등으로 내부를 setter로 수정할 수 있다.
- 물론, Java8+ 부터 Date 대신 불변 객체를 사용하면 되지만, 과거 코드에 이런 잔재가 남아있을 수 있으므로 주의해서 불변식을 지켜야한다.



## 💚 2. 방어적 복사 수행을 통해 공격 막기
- 위 코드를 수정하여 불변식을 깨려는 공격을 막을 수 있다.
- 적시에 방어적 복사를 수행해서 클래스 내부에서만 수정하고, 유효성 검사를 반드시 거치게 하는 방법이다.


```java
// 아래 코드로 적시에 방어적 복사를 수행
// 이후, 복사본으로 유효성을 검사한다.
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());

        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦습니다");
        }

        // 새로운 접근자 메서드에 방어적 복사로 직접 수정을 불가능하게 만든다.
        public Date start() {
            return new Date(start.getTime());
        }

        public Date end() {
            return new Date(end.getTime());
        }
    }
```


## 3. 클라이언트와 상호작용하는 상황에서 가변 객체로 인한 문제가 발생할 수 있을 때
1. 클라이언트에서 제공받은 객체
    - ex. 클라에서 받은 객체의 참조를 내부 자료구조에 보관해야 할 때, 이 객체가 변경 가능성이 있는지 생각해야 한다. (변경되어도 다른 곳에 문제 없는지 파악 필수)
2. 클라이언트에게 반환할 객체
    - ex. 가변 객체를 클라이언트에 반환할 때, 클라이언트가 가변 객체를 바꾸어서 클래스 내부 상태를 직접 수정하려고 할 수 있다. (위 `Date` 예시 생각)


---
## 방어적 복사 줄이기
- 되도록 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어든다.
- 위 Date 코드와 같은 사례를 통해 얻을 수 있는 교훈이다.
- 방어적 복사는 복사 비용으로 인한 성능 저하가 있고, 항상 쓸 수 있는 것도 아니기 때문에 주의해야 한다.
- 방어적 복사를 생략해도 되는 상황은 다음과 같다.
    - 해당 클래스와 사용하는 클라이언트가 상호 신뢰할 수 있을 때 (공격이나 불변식 깨뜨릴 일이 없을 때)
    - 불변식이 혹시 깨지더라도 영향이 오직 호출한 클라이언트로 국한될 때 (다른 곳에 나쁜 영향 확장 없을 때)


---
## Summary
- 클래스가 클라이언트로 받는 or 클라이언트에게 반환하는 구성요소가 가변 요소라면, 반드시 방어적으로 복사해야 한다.
- 복사 비용이 너무 크거나 요소가 잘못 수정될 일이 없음을 강력하게 신뢰한다면, 방어적 복사를 수행하는 대신 해당 요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명시하자.