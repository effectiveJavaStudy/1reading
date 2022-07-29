# Item 42 익명 클래스보다는 람다를 사용하라.

### 참조.

[익명 클래스와 람다.](https://www.codelatte.io/courses/java_programming_basic/O2PZAC2T82LKBXAY)

[익명 클래스](https://sgcomputer.tistory.com/82)

[전략 패턴](https://steady-coding.tistory.com/381)

---

> **내부클래스의 종류**
> 
1. 정적 클래스(static class)
2. 인스턴스 클래스 ( instance class)
3. 지역 클래스 ( local class)
4. **익명 클래스 ( anonymous class)** 

**익명 클래스란?**

> **클래스 선언과 객체 생성과 동시에 단 한번 사용할 수 있게 만든 클래스를 말한다.** 그래서 단 한번 사용할 수 있게 만든 클래스를 말한다.  그래서 단 한번만 사용되고 오직 하나의 객체만 생성가능한 일회용 클래스이다.
> 

( 이름이 없어서 생성자를 가질 수 없고 단 하나의 클래스를 상속받거나 하나의 인터페이스만 구현 가능하다. ) 

**익명 클래스는 왜 존재하는가?**

> 이유는 추상 클래스나, 인터페이스 때문입니다. 추상 클래스나 인터페이스를 Java 파일을 별도로 만들어서 구현할 수도 있지만 코드 흐름이나 가독성을 위해 가벼운 내용의 경우는 익명 클래스로 작성하는 것이 더 나을 때도 있기 때문이다.
> 

**익명 클래스와 일반 클래스의 차이?**

> 일반 클래스는 인터페이스를 제한 없이 상속받을 수 있지만, 익명 클래스는 단 하나의 인터페이스/클래스만을 구현할 수 있다.
> 

---

1. **익명 클래스의 인스턴스를 함수 객체로 사용 - 낡은 기법이다.** 

```java
Collections.sort(words, new Comparator<String>(){
		public int compare(String s1, String s2){
				return Integer.compare(s1.length(), s2.length());
		}
});
```

전략 패턴처럼 ? 함수 객체를 사용하는 과거 객체 지향 디자인 패턴에는 익명 클래스면 충분 했다. 

이 코드에서 Comparator 인터페이스가 정렬을 담당하는 추상 전략을 뜻하며, 문자열을 정렬하는 구체적인 전략을 익명 클래스로 구현했다. 

하지만 익명 클래스 방식은 코드가 너무 길기 때문에 자바는 **함수형 프로그래밍에 적합하지 않았다.** 

**자바 8에 와서…**

추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 특별한 대우를 받게 되었다. 지금은 함수형 인터페이스라 부르는 이 인터페이스들의 인스턴스를 람다식을 사용해 만들 수 있게 된 것이다. 

람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다. 

다음은 익명 클래스를 사용한 앞의 코드를 람다 방식으로 바꾼 모습이다. 

1. **람다식을 함수 객체로 사용 → 익명 클래스 대체** 

```java
Collections.sort(words, 
				(s1, s2) -> Integer.compare(s1.length(), s2.length());
```

여기서 람다, 매개변수(s1, s2), 반환값의 타입은 각각 (Comparator<String>), String, int 지만 코드에서는 언급이 없다. 우리 대신 컴파일러가 문맥을 살펴 타입을 추론해준 것이다. 

> 타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자.
> 

람다 자리에 비교자 생성 메서드를 사용하면 더 간결하게 만들 수 있다.

```java
Collections.sort(words, comparingInt(String::length));
```

더 나아가 자바 8 때 List 인터페이스에 추가된 sort 메서드를 이용하면 더욱 짧아진다. 

```java
words.sort(comparingInt(String::length));
```

**또 다른 예제** 

람다를 언어 차원에서 지원하면서 기존에는 적합하지 않았던 곳에서도 객체를 실용적으로 사용할 수 있게 되었다. 

1. **상수별 클래스 몸체와 데이터를 사용한 열거 타입.**

```java
public enum Operation {
		PLUS("+"){
					public double apply(double x, double y){ return x + y; }
		},
		MINUS("-"){
					public double apply(double x, double y){ return x - y; }
		},
		TIMES("*"){
					public double apply(double x, double y){ return x * y; }
		},
		DIVIDE("/"){
					public double apply(double x, double y){ return x / y; }
		};

		private final String symbol;

		Operation(String symbol) { this.symbol = symbol; }

		@Override public String toString() { return  symbol; }
		public abstract double apply(double x, double y);

}
```

1. **함수 객체(람다)를 인스턴스 필드에 저장해 상수별 동작을 구현한 열거 타입.** 

```java
public enum Operation {
			PLUS ("+", (x, y) -> x + y), 
			MINUS ("-", (x, y) -> x - y),
			TIMES ("*", (x, y) -> x * y),
			DIVIDE("/", (x, y) -> x / y);

			private final String symbol;
			private final DoubleBinaryOperator op;

			Operation(String symbol, DoubleBinaryOperator op){
					this.symbol = symbol;
					this.op = op;
			}

			@Overrdie public String toString() { return symbol; }

			public double apply(double x, double y){
					return op.applyAsDouble(x, y);

			}
}
```

람다 기반 Operation 열거 타입을 보면서 상수별 클래스 몸체는 더 이상 사용할 이유가 없다고 느낄지 모르지만, 꼭 그렇지는 않다. 메서드나 클래스와 달리, 람다는 이름이 없고 문서화도 못 한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다. 

> 람다는 한 줄 일 때 가장 좋고 길어야 세 줄 안에 끝내는 게 좋다. 세줄을 넘어가면 가독성이 심하게 나빠진다. 람다가 길거나 읽기 어렵다면 더 간단히 줄여보거나 람다를 쓰지 않는 쪽으로 리팩터링하길 바란다.
> 

열거 타입 생성자에 넘겨지는 인수들의 타빙도 컴파일타임에 추론된다. 따라서 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다. 따라서 상수별 동작을 몇 줄로 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야만 하는 상황이라면 상수별 클래스 문체를 사용해야 한다.

---

### 정리

- 람다의 시대가 열리면서 익명 클래스가 설 자리가 크게 좁아졌디만, 람다로 대체할 수 없는 분도 있다.
- 람다는 함수형 인터페이스에서만 쓰인다.
- 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니, 익명 클래스를 써야 한다.
- 비슷하게 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때도 익명 클래스를 쓸 수 있다.
- 람다는 자기 자신을 참조할 수 없다. 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다.
- 반면 익명 클래스에서의 this 는 익명 클래스의 인스턴스 자신을 가리킨다. 그래서 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다.

- 람다도 익명 클래스처럼 직렬화 형태가 구현별로 다를 수 있다. 따라서 람다를 직렬화하는 일을 극히 삼가야 한다.
- 직렬화해야만 하는 함수 객체가 있다면 private 정적 중첩 클래스의 인스턴스를 사용하자.

> 익명 클래스는 (함수형 인터페이스가 아닌) 타입의 인스턴스를 만들 때만 사용하라. 람다는 작은 함수 객체를 아주 쉽게 표현할 수 있어 함수형 프로그래밍의 지평을 열었다 .
>