# Item 28. 배열보다는 리스트를 사용하라
- 배열과 제네릭 타입에 중요한 차이 두 가지가 있다. (공변성 관련)


1. 배열은 공변이다.
2. 제네릭은 불공변이다.

> 공변(covariant)이란?
> 
> 함께 변한다는 의미로, `Sub` 가 `Super` 의 하위 타입이면, 배열 `Sub[]`도 `Super[]` 배열의 하위 타입이 된다.
> 
> 제네릭은 불공변이므로, `List<Sub>`가 `List<Super>`의 하위타입이 되지 않는다. (서로 별개)
>

그러나 배열이 공변성이 있기 때문에 생기는 문제가 있다.

---
## 1. 예상치 못한 런타임 오류 가능성이 생긴다.
- ex. 상위 타입으로 Array를 선언하고, 특정한 하위 타입으로 구체화한 배열이 있을 때, 상위 타입을 상속한 다른 하위 타입으로 값을 변경하려는 코드를 짤 수 있다.
- 이런 코드가 컴파일 타임에 잡히지 않아서 위험해진다.


```java
Object[] objectArray = new Long[1];
objectArray[0] = "sample string"; // ArrayStoreException 발생
```


그러나 제네릭을 사용하면 컴파일 타임에 이런 오류가 잡히기 때문에 더 안전하다.


```java
List<Object> o = new ArrayList<Long>();
o.add("sample string");
```


- `List<T>`는 무공변이라 `ArrayList<Long>`을 `List<Object>` 변수에 대입할 수 없다.
- 즉, 컴파일러가 첫번째 줄에서 멈추기 때문에 다음 줄은 애초에 컴파일 단계까지 가지 않는다.


---
## 2. 배열의 실체화
- 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.
- 그러나, 제네릭은 타입 정보가 런타임에 소거된다.
  - 원소 타입을 컴파일 타임에만 검사하며, 런타임에는 모른다는 의미이다.
  - 타입 소거는 제네릭이 지원되기 전에 레거시 코드(이전 자바 버전)와 제네릭을 함께 사용할 수 있게 해주는 메커니즘이다.
  - 정리하면, 타입 정보를 인지하고 있는 배열과, 타입 정보가 소거된 제네릭이 서로 달라 잘 어우러지지 못한다.


---
## 제네릭 배열 생성을 막은 이유
- 제네릭 배열은 컴파일 타임에 막힌다.
- 그 이유는, 제네릭 배열 생성이 허용되면 배열의 공변성으로 인해 런타임 오류가 발생할 수 있기 때문이다.


```java
List<String>[] stringLists = new List<String>[1]; // (1) 제네릭 배열이 허용된다고 가정하자.
List<Integer> intList = List.of(42);
Object[] objects = stringLists; // 배열의 공변성 때문에 가능하다.
objects[0] = intList; // 제네릭이 타입 소거 방식으로 동작하기 때문에, 런타임에 object 밑에 List[] 할당이 가능해진다.
String s = stringLists[0].get(0); // 결과적으로 intList를 String으로 casting 하려고 시도
// 이런 말도안되는 코드가 배열의 공변성으로 인해 컴파일 타임에 잡히지 않고 런타임 오류를 낼 수 있다.
// 따라서, 제네릭 배열 생성을 막아서 타입 안전성을 지킨 것이다.
```


## 제네릭 - 실체화 불가 타입
- 실체화되지 않아서 런타임에는 컴파일 타임보다 타입 정보를 적게 가지는 타입이라는 의미이다. 
- 컴파일러는 컴파일 시점에 제네릭에 대해 원소 타입 소거(type erasure)를 한다.
  - 즉, 컴파일 타임에만 타입 제약 조건을 정의하고, 런타임에는 타입을 제거한다는 뜻이다.
  - unbounded Type(`<?>`, `<T>`)은 `Object` 로 변환 
  - bound type(`<E extends Comparable>`)의 경우는 `Object` 가 아닌 `Comprarable` 으로 변환 
  - 제네릭 타입을 사용할 수 있는 일반 클래스, 인터페이스, 메소드에만 소거 규칙을 적용 
  - 타입 안정성 보존을 위해 필요시 type casting 
  - 확장된 제네릭 타입에서 다형성을 보존하기위해 bridge method 생성


## 배열보다 리스트를 사용해야 하는 경우
- 배열로 형변환 할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우
- 배열보다 리스트를 사용하여 컴파일 오류를 막을 수 있다.


```java
public class ChooserArray {

    private final Object[] choiceArray;
    
    // 생성자에 어떤 컬렉션을 넘기느냐에 따라 주사위판, 매직8볼, 몬테카를로 시뮬레이션용으로 활용 가능
    public ChooserArray(Collection choices) {
        this.choiceArray = choices.toArray();
    }

    // 컬렉션안의 원소 중 하나를 무작위로 선택해 반환
    // 반환된 Object를 원하는 타입으로 형변환 필요 -> 타입이 다를 경우 런타임 오류 발생
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

위 클래스의 경우 choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환 해야 한다.
이 때, 다른 타입의 원소가 들어 있다면 런타임 오류가 발생 하므로 List로 변경하는 것이 좋다.


```java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        this.choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```


최종적으로 형변환 오류 메시지와 런타임 오류를(`ClassCastExcpetion`) 막을 수 있는 코드이다.


List를 사용해 코드 성능이 조금 더 느릴 수는 있어도 오류를 막는 것이 중요하기 때문에 감수할 만한 비용이다.


---
## Summary
- 배열과 제네릭에는 매우 다른 타입 규칙이 적용된다.
- 배열은 공변이고 실체화되는 반면에, 제네릭은 불공변이고 타입 정보가 소거된다.
  - 그 결과, 배열은 런타임에는 타입 안전하지만 컴파일 타임에는 안전하지 않다.
  - 제네릭은 반대이기 때문에 둘을 섞어서 사용하기 어렵다. (서로 차이점으로 인해 어울리지 않음)
- 둘을 섞어서 사용하다가 컴파일 오류나 경고가 있을 때, 배열을 리스트로 대체하는 방법을 적용해보자.
