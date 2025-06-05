# Item 02. 생성자에 매개변수가 많다면 빌더를 고려하라
- 정적 팩토리 메서드와 생성자 방식에는 공통적인 불편함이 있다.
- 선택적으로 들어가는 매개변수가 많을 때 적절히 대응하기 어렵다는 것인데, 이를 빌더 패턴을 통해 극복할 수 있다.


## 빌더 패턴을 사용하지 않았을 때 불편함
### 1. 점층적 생성자 패턴
- 필수적인 매개변수를 받는 생성자를 정의한 후, 선택적인 매개변수를 하나씩 추가한 생성자를 정의하는 것이다.


```java

public class NutritionFacts {
	private final int servingSize;  // 필수
	private final int servings;     // 필수
	private final int calories;     // 선택
	private final int fat;          // 선택
	private final int sodium;       // 선택
	private final int carbohydrate; // 선택

	public NutritionFacts(int servingSize, int servings) {
		this(servingSize, servings, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories) {
		this(servingSize, servings, calories, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat) {
		this(servingSize, servings, calories, fat, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
		this(servingSize, servings, calories, fat, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
	    this.servingSize = servingSize;
		this.servings = servings;
		this.calories = calories;
		this.fat = fat;
		this.sodium = sodium;
		this.carbohydrate = carbohydrate;
	}
}

```


- 매개변수의 개수가 적을 때는 비교적 간단하게 유지할 수 있으나, 매개변수가 늘어날수록 코드 작성과 관리가 어려워진다.
- 특히, 개발자가 직접 매개변수의 개수나 타입을 신경써야하므로 Human Error가 발생하기 쉽다.
- 또한, 확장이 필요할 때 계속 수동으로 생성자를 만들어주는 비용과, 매개변수 개수만 다른 비슷한 생성자가 계속해서 생겨난다는 단점이 있다. (확장의 어려움)


### 2. JavaBeans 패턴
- 이 패턴은, 매개변수가 없는 생성자로 객체를 먼저 만든 후, `setter` 메서드를 호출하여 원하는 매개변수의 값을 설정하는 패턴이다.
- 수많은 `setter` 호출로 일관성 깨지는 문제가 발생하고, 불변 객체를 만드는게 불가능하다.


```java

public class NutritionFacts {
	private int servingSize = -1;  // 필수
	private int servings = -1;     // 필수
	private int calories = 0;
	private int fat = 0;
	private int sodium = 0;
	private int carbohydrate = 0;

	public NutritionFacts() {}

	public void setServingSize(int val) { servingSize = val; }
	public void setServings(int val) { servings = val; }
	public void setCalories(int val) { calories = val; }
	public void setFat(int val) { fat = val; }
	public void setSodium(int val) { sodium = val; }
	public void setCarbohydrate(int val) {carbohydrate = val; }
}

// 사용할 때
NutritinFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);

```


- JavaBeans 패턴은 이전의 점층적 생성자 패턴의 단점을 보완한다.
- 코드가 쉬워지고 깔끔해져서 가독성이 좋아진다.
- 그러나, 위에서 설명한 몇 가지 단점이 존재한다.
    - 1) 원하는 정보를 가진 객체가 만들어질 때까지 계속 메서드를 호출해야 한다는 것
    - 2) 이 패턴에서 객체를 만드려면, 객체가 `setter`로 매개변수 값이 완전히 주입될 때까지 객체 일관성이 무너진 상태가 된다는 것
        이 문제 때문에 클래스를 불변 클래스로 만들 수 없으며, set하는 동안 스레드 안전성을 얻으려면 개발자의 추가적인 처리가 필요하다.
    - 3) 위 단점을 해결하려고 `freeze` 메서드를 통해 객체를 얼리는 작업을 하기도 하지만, 이를 잘 호출했는지 컴파일러가 보증하기 어렵기 때문에 런타임 에러에 취약하다.


## 빌더 패턴은 위 단점들을 어떻게 극복하는가?
- 빌더 패턴은 점층적 생성자 패턴의 안전성과 JavaBeans 패턴의 가독성, 편리함을 겸비한 생성자 패턴이다.
- 여기서 설명하는 코드의 빌더 패턴은 수동 빌더 패턴이다.


```java
public class NutritionFacts {
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;

	public static class Builder {
        // 필수 매개변수
		private final int servingSize;
		private final int servings;

        // 선택적 매개변수 - default 값으로 초기화
		private int calories = 0;
		private int fat = 0;
		private int sodium = 0;
		private int carbohydrate = 0;

        // 필수 매개변수만으로 Builder의 생성자 or 정적 팩토리를 호출해 빌더 객체를 얻는다.
		public Builder(int servingSize, int servings) {
			this,servingSize = serginsSize;
			this.servings = servings;
		}

		public Builder fat(int val) {
			fat = val;
			return this;
		}

		public Builder sodium(int val) {
			sodium = val;
			return this;
		}

		public Builder carbohydrate(int val) {
			carbohydrate = val;
			return this;
		}

		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}

	private NutirionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
		sodium = builder.fat;
		carbohydrate = builder.carbohydrate;
	}
}

// 사용할 때
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                                .calories(100)
                                .sodium(35)
                                .carbohydrate(27)
                                .build();

```

- 필수 매개변수를 무조건 받아야하는 것 외에, 유효성 검사를 추가한다면 안전함이 추가된다.
- 빌더에 유효성 검사 코드를 추가하면 잘못된 매개변수를 최대한 일찍 발견하여 "객체 생성 시점"부터 막을 수 있다.
- 또한, 빌더의 생성자와 메서드에서 입력 매개변수를 검사한 후, 매개변수가 잘못되었을 때 적절한 Exception을 던질 수 있다.


### 불변(immuatable, immutability) & 불변식(invariant) 의 개념
1) 불변성 immutability
- 불변성은 객체의 상태가 한 번 설정되면 절대 바뀌지 않음을 보장하는 성질이다.
- 불변 객체는 내부 필드가 모두 `final`이며, `setter`가 존재하지 않고, 외부에서 상태를 바꿀 수 없도록 설계된다.
- 대표적인 예시: `String`, `Integer`, `LocalDateTime` 등은 모두 불변 객체이다.
    - `String s = "hi";` `s += "there";` → 실제로는 "hi"가 변경되는 게 아니라 "hithere"라는 새로운 String 객체 생성


2) 불변식 invariant
- 불변식은 객체가 언제나 만족해야 하는 조건 또는 상태 불변 조건을 뜻한다.
- 객체의 필드나 로직이 변경되더라도, 클래스 설계자가 정한 "절대 깨지면 안 되는 규칙" 을 의미한다.
    - ex. `은행 계좌 클래스의 잔고는 항상 0 이상이어야 한다.`


### 빌더 패턴 계층적으로 사용하기
- 빌더 패턴은 계층적으로 설계된 클래스와 함께 사용하기 좋다.
    - 유연하고, 상속받은 필드까지도 깔끔하게 Builder로 처리할 수 있기 때문이다.
- 각 계층의 클래스에 관련 빌더를 멤버로 정의할 수 있다.
    - 추상 클래스는 추상 빌더를, 구체 클래스는 구체적인 빌더를 갖게 한다.


```java
// 책보다 간단한 예시 코드를 통해 알아보자
// 추상 클래스 Pizza
public abstract class Pizza {
    final Set<String> toppings;

    abstract static class Builder<T extends Builder<T>> {
        Set<String> toppings = new HashSet<>();
        public T addTopping(String topping) {
            toppings.add(topping);
            return self();
        }
        abstract Pizza build();

        // self() + 재귀적 제네릭
        // 하위 타입 스스로를 반환하게 하여 타입 안정성을 유지하는 핵심 메서드
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        this.toppings = builder.toppings;
    }
}
```


```java
// 하위 클래스 NyPizza
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = size;
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        // 부모의 self()를 오버라이딩하여 정확한 자신의 타입 반환
        // → 부모의 제네릭 타입 T에 자신의 타입을 정확하게 전달해 체이닝 시 타입 안정성 확보
        @Override protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        this.size = builder.size;
    }
}
```



### 빌더 패턴이 가지는 계층 구조에서의 이점
1) 상속 관계에서도 가독성과 타입 안정성을 유지한다.
    - 부모 클래스의 필드도 빌더 체이닝으로 설정할 수 있고, 자식 클래스의 필드도 연속적으로 설정 가능하다.


2) Builder 제네릭 `self()` 패턴으로 하위 타입의 빌더를 반환함을 보장한다.
    - 부모 클래스에서 `Builder<T extends Builder<T>>` 를 정의하고, `self()`를 추상으로 강제해서 하위 빌더가 자신을 반환한다.
    - 이걸 통해 타입 안전한 체이닝을 구현한다.


3) 객체 생성을 하위 클래스에서 커스터마이징 할 수 있다.
    - 각 하위 클래스에서 자신만의 `build()` 로직을 정의하여, 복잡한 제약이나 로직이 있을 때도 유연하게 객체를 생성한다.



## 빌더 패턴의 단점
- 객체를 만들기 전에 먼저 빌더부터 만들어야 하기 때문에 성능에 민감한 상황에서는 주의해야 한다.
- 또한, 코드가 장황하기 때문에 매개변수의 수가 많거나, 필수와 선택 매개변수가 나뉘어져서 복잡한 상황에 사용하는 것이 좋다.
- 그러나 개발 시, 시간이 지날수록 매개변수가 많아지는 경향이 있으므로 처음부터 빌더를 도입할 지 잘 고민해야 한다.



---


## 비교: 수동 빌더 패턴 vs Lombok `@Builder`

* 수동 빌더 패턴은 객체 생성을 안전하고 유연하게 만들기 위한 디자인 패턴이다.
* Lombok의 `@Builder`는 빌더 패턴을 어노테이션으로 자동 생성해주는 도구로, 생산성을 크게 향상시킨다.
* 둘은 개념적으로 동일하지만, 사용 목적과 세부 제어권, 코드 가시성, 유효성 검사 등에서 차이가 있다.

---

## 어떤 상황에서 무엇을 택해야 하는가? (선택 기준)
### ✅ Lombok `@Builder`를 사용할 때

| 기준                                                    | 설명                                            |
| ----------------------------------------------------- | --------------------------------------------- |
| **간단한 DTO / Request 객체**                              | 주로 컨트롤러 입력 값이나 중간 전달 객체를 구성할 때                |
| **필드 수가 많지만 유효성은 Spring Validation 만으로 해결할 때** | ex. `@NotNull`, `@Min`, `@Size` 등으로 유효성 검사 위임 |
| **명시적 생성 로직이 필요 없는 경우**             | 생성자가 단순히 필드 할당만 한다면 `@Builder`가 충분            |
| **테스트 코드 / 임시 객체 생성**                 | 빠르게 임시 객체를 만들고 싶은 경우                          |



### ✅ 수동 빌더 패턴을 사용할 때

| 케이스                          | 설명                                           |
| ---------------------------- | -------------------------------------------- |
| **핵심 도메인 모델의 불변성 유지**        | 필드 변경을 막고, 생성 시점에 필수값을 강제해야 할 때              |
| **필드 간 제약조건이 복잡한 경우**        | `A가 있으면 B는 없어야 함`, `X와 Y는 함께 들어와야 함` 등       |
| **생성 시점에 부작용이 필요한 경우**       | ex. 생성 중 값 계산, 다른 객체 의존 등 로직이 복잡할 때          |
| **API 문서화 / 체이닝 가독성 강조**     | `.withName().withEmail()` 등 명시적 체이닝으로 사용자 유도 |
| **외부 라이브러리 / SDK로 제공되는 API** | 사용자가 직접 조작하는 클래스라면, 명시적 빌더가 더 안전        |


---

## 유효성 검사 시점에 따른 차이

### 1. Lombok `@Builder` + Spring Validation

* Spring MVC 환경에서는 `@Valid` 어노테이션과 함께 필드 단위 유효성 검사를 자주 사용한다.
* 이는 컨트롤러 계층에서 DTO를 위한 것이며, **JPA 엔티티의 생성 유효성 보장에는 부족하다**


```java
@PostMapping("/register")
public ResponseEntity<?> register(@Valid @RequestBody MemberDto dto) {
    // builder로 생성 + Spring Validation
}
```


### 2. 수동 빌더 패턴 + 생성자 내부 유효성

* 생성자나 `build()` 내부에서 **명시적 유효성 검사 수행 가능**
* 단순히 값만 세팅하는 것이 아니라, 복잡한 로직을 처리할 수 있는 유연성이 있음


```java
// 이전에 복잡한 NutritionFacts 코드에 추가할 수 있는 유효성 검사
public NutritionFacts build() {
    if (calories < 0 || fat < 0) {
        throw new IllegalArgumentException("칼로리와 지방은 음수가 될 수 없습니다.");
    }
    return new NutritionFacts(this);
}
```


---

## 정리: 선택 기준 요약

| 구분              | Lombok `@Builder` | 수동 빌더 패턴            |
| --------------- | ----------------- | ------------------- |
| 코드 작성 속도        | 빠름 ✅              | 느림 ❌                |
| 유효성 검사 제어       | 제한적 ❌             | 자유롭고 정밀 ✅           |
| 객체 생성 로직        | 단순한 필드 세팅 ✅       | 복잡한 생성 조건 대응 가능 ✅   |
| 유지보수 / 추적성      | IDE 의존적 ❌         | 명시적 클래스 구조로 추적 쉬움 ✅ |
| 가독성 / 체이닝       | 높음 ✅              | 높음 ✅                |
| 외부 사용자 대상 API   | 위험할 수 있음 ❌        | 안정적 ✅               |
| DTO, 테스트, 임시 객체 | 적합 ✅              | 과함 ❌                |
| 도메인 객체, 핵심 클래스  | 제한적 ❌             | 적합 ✅                |


- 이전 기업 연계 프로젝트에서, 영양소 관련 Entity의 유효성 검사를 많이 진행해야 했다.
- 이를 Dto 수준의 Spring Validation으로 처리했는데, 도메인에서 수동 빌더 패턴을 통해 제대로 검사하는 방향으로 고칠 수 있을 것 같다.