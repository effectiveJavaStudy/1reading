# 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라

## 1. 개요

- 문제 : 정적 팩터리, 생성자 등을 만들 때 매개변수가 많을 경우 적절히 대응하기 어렵다.

- 대안 1) 점층적 생성자 패턴 사용

  ```java
  public class NutritionFacts {
      private final int servingSize;  // (mL, 1회 제공량)     필수
      private final int servings;     // (회, 총 n회 제공량)  필수
      private final int calories;     // (1회 제공량당)       선택
      private final int fat;          // (g/1회 제공량)       선택
      private final int sodium;       // (mg/1회 제공량)      선택
      private final int carbohydrate; // (g/1회 제공량)       선택
  
      public NutritionFacts(int servingSize, int servings) {
          this(servingSize, servings, 0);
      }
  
      public NutritionFacts(int servingSize, int servings,
                            int calories) {
          this(servingSize, servings, calories, 0);
      }
  
      public NutritionFacts(int servingSize, int servings,
                            int calories, int fat) {
          this(servingSize, servings, calories, fat, 0);
      }
  
      public NutritionFacts(int servingSize, int servings,
                            int calories, int fat, int sodium) {
          this(servingSize, servings, calories, fat, sodium, 0);
      }
      public NutritionFacts(int servingSize, int servings,
                            int calories, int fat, int sodium, int carbohydrate) {
          this.servingSize  = servingSize;
          this.servings     = servings;
          this.calories     = calories;
          this.fat          = fat;
          this.sodium       = sodium;
          this.carbohydrate = carbohydrate;
      }
  }
  ```

  ⇒ 점층적 생성자 패턴으로 주로 사용하면서 **가독성**이 좋지 않다는 단점이 있다.

- 대안 2) 자바빈즈 패턴 사용 : 세터 메서드를 호출해서 매개변수 값 설정

  ```java
  public class NutritionFacts {
      // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
      private int servingSize  = -1; // 필수; 기본값 없음
      private int servings     = -1; // 필수; 기본값 없음
      private int calories     = 0;
      private int fat          = 0;
      private int sodium       = 0;
      private int carbohydrate = 0;
  
      public NutritionFacts() { }
  
      // Setters
      public void setServingSize(int val)  { servingSize = val; }
      public void setServings(int val)     { servings = val; }
      public void setCalories(int val)     { calories = val; }
      public void setFat(int val)          { fat = val; }
      public void setSodium(int val)       { sodium = val; }
      public void setCarbohydrate(int val) { carbohydrate = val; }
  
      public static void main(String[] args) {
          NutritionFacts cocaCola = new NutritionFacts();
          cocaCola.setServingSize(240);
          cocaCola.setServings(8);
          cocaCola.setCalories(100);
          cocaCola.setSodium(35);
          cocaCola.setCarbohydrate(27);
      }
  }
  ```

  ⇒ 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지 일관성이 무너진다.

  ⇒ 어디서나 Setter가 호출될 수 있기 때문에 불안정한 상태이다.

## 2. 빌더 패턴

- 정의 : 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다. → 이후 일종의 세터 메서드를 호출해 필요한 객체를 얻는다.

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
          private int calories      = 0;
          private int fat           = 0;
          private int sodium        = 0;
          private int carbohydrate  = 0;
  
          public Builder(int servingSize, int servings) {
              this.servingSize = servingSize;
              this.servings    = servings;
          }
  
          public Builder calories(int val) { 
              calories = val;      
              return this; 
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
  
      private NutritionFacts(Builder builder) {
          servingSize  = builder.servingSize;
          servings     = builder.servings;
          calories     = builder.calories;
          fat          = builder.fat;
          sodium       = builder.sodium;
          carbohydrate = builder.carbohydrate;
      }
  
      public static void main(String[] args) {
          NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                  .calories(100).sodium(35).carbohydrate(27).build();
      }
  }
  ```

## 3. 빌더 패턴 사용

- 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.

  - Pizza 클래스

    ```java
    public abstract class Pizza {
        public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
        final Set<Topping> toppings;
    		
    		// T와 그 자손들을 구현한 클래스들만 매개변수로 가능
        abstract static class Builder<T extends Builder<T>> {
            EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
    
            public T addTopping(Topping topping) {
                toppings.add(Objects.requireNonNull(topping));
                return self();
            }
    
            abstract Pizza build();
    
            // 하위 클래스는 이 메서드를 재정의(overriding)하여
            // "this"를 반환하도록 해야 한다.
            protected abstract T self();
        }
    
        Pizza(Builder<?> builder) {
            toppings = builder.toppings.clone(); // 아이템 50 참조
        }
    }
    ```

  - 뉴욕 피자

    ```java
    // 코드 2-5 뉴욕 피자 - 계층적 빌더를 활용한 하위 클래스 (20쪽)
    public class NyPizza extends Pizza {
        public enum Size { SMALL, MEDIUM, LARGE }
        private final Size size;
    
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
            protected Builder self() {
                return this;
            }
        }
    
        private NyPizza(Builder builder) {
            super(builder);
            size = builder.size;
        }
    }
    ```

  - 칼초네 피자

    ```java
    public class Calzone extends Pizza {
        private final boolean sauceInside;
    
        public static class Builder extends Pizza.Builder<Builder> {
            private boolean sauceInside = false; // 기본값
    
            public Builder sauceInside() {
                sauceInside = true;
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
            sauceInside = builder.sauceInside;
        }
    }
    ```

- 클라이언트는 형변환에 신경쓰지 않고 빌더를 사용할 수 있다.

- 단점

  1. 객체를 만들 때 Builder 부터 만들어야 한다. ⇒ 성능에 민감한 상황에서는 문제가 될 수 있다.
  2. 점층적 생성자 패턴보다 코드가 길어서 매개변수가 4개 이상은 되어야 값어치를 한다.