# Item 75. 예외의 상세 메시지에 실패 관련 정보를 담으라

예외가 터지면, 자바 시스템은 예외의 StrackTrace 정보를 자동으로 출력한다. StackTrace는 예외 객체의 toString 메서드를 호출해 얻는 문자열로, 보통 예외 클래스 이름 뒤에 상세 메시지가 붙는 형태이다.


이는 실패 원인을 분석하고 복구해야하는 프로그래머가 얻을 수 있는 유일한 정보이기 때문에, 해당 toString 메서드에 실패 원인에 대한 정확하고 많은 정보를(가능한 많이) 담아 반환하는 것이 중요하다.


## 실패 순간 포착
- 발생한 예외에 관여된 모든 매개변수와 필드값을 실패 메세지에 담기
- 예를 들면, IndexOutOfBoundsException 의 상세 메시지에 범위의 최소/최대/벗어난 인덱스 값 등을 모두 담아야 한다.


이런 정보는 실패에 대한 많은 것을 제공한다. 구체적으로 어떤 값이 잘못되었는지 파악하고 그걸 바탕으로 소스 코드까지 개선해야 한다.


보통은 StackTrace에 예외가 발생한 파일, 줄번호, 호출 메서드 등이 기록되기 때문에 위치 정보 외에 의미있는 값이 있을 때 전달하도록 하자.


## 예외 상세 메시지와 최종 사용자에게 보여줄 오류 메시지는 다르다.
최종 사용자에게는 친절한 안내 메시지를 보여줘야한다. (불친절하거나 너무 전문적인 메시지는 불필요함)


이에 반해, 예외 메시지의 주 소비층은 프로그래머이므로 정확하고 상세한 내용을 전달하는 것이 중요하다.


위 제안사항을 기반으로, `IndexOutOfBoundsException`의 생성자 구현을 아래처럼 개선해볼 수 있다.


```java
/**
 * IndexOutOfBoundsException 생성한다. (생성자)
 * 
 * @param lowerBound 인덱스의 최솟값
 * @param upperBound 인덱스의 최댓값
 * @param index 인덱스 실제 값 
*/
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
    // 실패를 포착하는 상세 메시지 생성
    super(String.format(
        "최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index
    ));

    // 프로그램에서 이용할 수 있도록 실패 정보 저장
    this.lowerBound = lowerBound;
    this.upperBound = upperBound;
    this.index = index;
}
```


또한, 이전 아이템([Item70](/chapter10/Item-70.md))에서 제안한 것 처럼, 예외에서 실패와 관련된 정보를 얻을 수 있는 적절한 접근자 메서드를 제공하는 것이 좋다. 
(앞 예시로 따지면, `lowerBound`, `upperBound`, `index` 정도가 필요할 것)
