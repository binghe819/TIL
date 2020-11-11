> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라

> 생성자나 static 팩토리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다.

객체를 생성할 때 매개변수가 많이 필요하다면 [아이템 1]()에서 말한 static 팩토리 메서드와 public 생성는 매우 불편해진다.

이 불편함을 해결할 수 있는 방법은 3 가지이다.

* 점층적 생성자 패턴
* 자바빈즈 패턴
* 빌더 패턴

이중에서 빌더 패턴이 제일 효율적이라 볼 수 있다. 왜 효율적인지 아래 코드를 살펴보자.



## 점층적 생성자 패턴

```java
public class NutritionFacts {

    private final int servingSize; // (ml, 1회 제공량)
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int cabohydrate;
    
 		// 점층적 생성자 패턴
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
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int cabohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.cabohydrate = cabohydrate;
    }
}
```

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27); // 각각의 숫자가 무엇을 의미하는지 모른다.
```

* 점층적 생성자 패턴
  * 점층적 생성자 패턴을 이용하면 원하는 매개변수의 개수에 따라 호출할 수 있다.
* 단점
  * **매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다. 매개변수 값이 무슨 의미인지도 모른다.**



## 자바빈즈 패턴

```java
public class NutritionFacts {

    private int servingSize = -1; // 필수
    private int servings = -1; // 필수
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int cabohydrate = 0;

    public NutritionFacts() {}

    public void setServingSize(int servingSize) {this.servingSize = servingSize;}

    public void setServings(int servings) {this.servings = servings;}

    public void setCalories(int calories) {this.calories = calories;}

    public void setFat(int fat) {this.fat = fat;}

    public void setSodium(int sodium) {this.sodium = sodium;}

    public void setCabohydrate(int cabohydrate) {this.cabohydrate = cabohydrate;}
}
```

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

* 자바빈즈 패턴
  * 매개변수가 없는 생성자로 객체를 만든 후, setter 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식
* 단점
  * **객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다.**
  * 불변 객체를 만들 수 없다.
  * 스레드 안전을 보장하려면 추가적인 작업이 필요하다. (locking, freezing)



## 빌더 패턴

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
        
        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        
        public Builder calories(int val) {calories = val; return this;}
        
        public Builder fat(int val) {fat = val; return this;}
        
        public Builder sodium(int val) {sodium = val; return this;}
        
        public Builder carbohydrate(int val) {carbohydrate = val; return this;}
        
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
    
    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 0)
  .calories(100).sodium(35).carbohydrate(27).build();
```

* 빌더 패턴
  * **필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다. 그다음 빌더 객체가 제공하는 일종의 setter 메서드들로 원하는 선택 매개변수들을 설정하면 된다.**
  * 마지막으로 매개변수가 없는 build 메서드를 호출해 필요한 (불변)객체를 얻을 수 있다.
* 상태 초기화
  * 필수 - 생성자
  * 부가 - setter 체이닝

* 장점
  * 점층적 생성자 패턴의 안정성과 자바 빈즈 패턴의 가독성을 겸비한 패턴이다.
  * 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.
  * 각 속성에 대한 유효성을 확인할 수 있다.
* 단점
  * 객체를 만들려면, 그에 앞서 빌더부터 만들어야 한다. 성능에 민감한 상황이 올 수 도 있다.



### 빌더 패턴의 계층적 사용

```java
public abstract class Pizza {

    public enum Topping {
        HAM, MUSHROOM, ONION, PEEPER, SAUSAGE
    }

    final Set<Topping> toppings;

    abstract static class Builder<T extends  Builder<T>> { // 재귀적인 타입 매개변수
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        // 피자에 토핑을 추가하는 메서드 
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self(); // 하위 Builder 자기 자신을 반환받을 수 있도록 self()를 사용한다.
        }

        abstract Pizza build(); // Convariant 리턴 타입을 위한 준비작업 (Pizza 하위 타입을 리턴)

        protected abstract T self(); // self-type 개념을 사용해서 메소드 체이닝이 가능케 함
    }

  	// 생성자
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }

}
```

```java
public class NyPizza extends Pizza {

    public enum Size {
        SMALL, MEDIUM, LARGE
    }

    private final Size size;

    // Pizza의 Builder를 상속받아 Builder를 만들어준다. 
    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() { // Builder 자기 자신을 반환하는 메서드 
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder); // Pizza의 상태(topping)를 초기화
        size = builder.size;
    }
}
```

```java
public class Calzone extends Pizza {

    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauseInside = false;

        public Builder sauceInde() {
            sauseInside = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauseInside;
    }

}
```

* 빌더 패턴은 클래스 계층 구조를 잘 활용할 수 있다.
  * 추상 빌더를 가지고 있는 추상 클래스를 만들고 하위 클래스에서는 추상 클래스를 상속받으며 각 하위 클래스용 빌더도 추상 빌더를 상속받아 만들 수 있다.
* **추상 빌더는 재귀적인 타입 매개변수를 사용하고 `self`라는 메서드를 사용해 `self-type` 개념을 모방할 수 있다.**
  * 하위 클래스에서는 `build` 메서드의 리턴 타입으로 해당 하위 클래스 타입을 ㅣ턴하는 `Convariant 리턴 타이핑` 을 사용하면 클라이언트 코드에서 타입 캐스팅을 할 필요가 없어진다.
  * 만약, `addToping` 메서드에서 `return this`를 한다면 하위 클래스에서 하위 클래스만의 setter를 사용할 수 없게 된다. 그러므로 `self`를 통해 이 문제를 해결한다.

```java
NyPizza nyPizza = new NyPizza.Builder(SMALL)
    .addTopping(Pizza.Topping.SAUSAGE)
    .addTopping(Pizza.Topping.ONION)
    .build();

Calzone calzone = new Calzone.Builder()
    .addTopping(Pizza.Topping.HAM)
    .sauceInde()
    .build();
```



## lombok

> [롬복 @Builder](https://projectlombok.org/features/Builder)

