# Item 27. 비검사 경고를 제거하라
- 제네릭을 사용하기 시작하면 다양한 컴파일러 광고를 볼 수 있다.
- ex. 비검사 형변환 경고, 비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고 등
- 컴파일러가 체크해주는 경고를 체크하고 수정하자.


## 모든 비검사 경고를 제거해야하는 이유
- 타입 안전성이 보장된 코드가 된다. (런타임에도 안전, 의도한대로 동작)
-` @SuppressWarining("unchecked")` 어노테이션으로 경고를 숨길 수도 있다.
    - 단, 이는 타입 안전함을 본인이 직접 따져보고 검증한 후에, 정말 안전하다고 생각할 때 추가해야 한다.
    - 또한, 가능한 좁은 범위에 달아야 한다. (변수, 짧은 메서드, 생성자 등)
    - 절대 클래스 전체에 달면 안된다. (범위가 너무 크면 더 중요한 세부 경고를 놓칠 수 있다.)
    - 해당 어노테이션을 사용했다면, 이 코드가 안전한 이유를 주석으로 남겨야한다.


    ```java
    public <T> T[] toArray(T[] a) {
        if (a.length < size) {
              // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환
            @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
            return result;
        }
        System.arraycopy(elements, 0, a, 0, size);
        if (a.length > size) {
            a[size] = null;
        }
        return a;
    }
    ```


## Summary
- 비검사 경고는 중요하니 무시하지 말고 모두 제거해야 한다.
- 경고를 없앨 수 없다면 `@SuppressWarining("unchecked")` 어노테이션과 함께 해당 코드가 안전한 이유를 남기자.