# 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라



# 1. 문제점

```java
public class Fruit {
	private final Apple apple = new Apple();
}
```

- Apple이라는 객체를 Orange, Mango 등으로 변경하고 싶을 때 코드를 수정 해야 한다.
- 싱글톤이나 정적유틸리티 클래스로 구현이 되어 있다면 사용하는 자원에 따라 동작이 달라지는 클래스에는 적합하지 않다.

# 2. 의존 객체 주입

```java
public class Fruit {
	private final Eat eat;
	
	public Fruit(Eat eat) {
		this.eat = Objects.requireNonNull(eat);
	}
}
```

- 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식
- 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.

# 3. 팩터리

- 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체
- 생성자에 자원 팩터리를 넘겨주는 방식이 있다.
- [팩토리 메서드 패턴](https://ko.wikipedia.org/wiki/팩토리_메서드_패턴) : 부모 클래스에 알려지지 않은 구체 클래스를 생성하는 패턴이며 자식 클래스가 어떤 객체를 생성할지 결정하도록 하는 패턴

```java
public abstract class Pizza {
	public abstract int getPrice();

	public enum PizzaType{
		HamMushroom, Deluxe, Seafood
	}

	public static final Pizza PizzaFactory(PizzaType pizzaType) {
		switch (pizzaType){
			case HamMushroom :
				return new HamMushroomPizza();
			case Deluxe :
				return new DeluxePizza();
			case Seafood :
				return new SeafoodPizza();
		}
		throw new RuntimeException("The pizza type " +
			pizzaType.toString() + "  is not Recognized.");
	}

}

class HamMushroomPizza extends Pizza {
	private int price = 15_000;
	@Override public int getPrice() {
		return price;
	}
}
class DeluxePizza extends Pizza {
	private int price = 20_000;
	@Override public int getPrice() {
		return price;
	}
}
class SeafoodPizza extends Pizza {
	private int price = 23_000;
	@Override public int getPrice() {
		return price;
	}
}
```