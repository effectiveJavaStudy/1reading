### 참조

[Optional 이란?](https://mangkyu.tistory.com/70)

[방어적 복사 ( 불변 객체에 대해 자주 나오는 키워드 )](https://tecoble.techcourse.co.kr/post/2021-04-26-defensive-copy-vs-unmodifiable/) 

[TCP School Optional > Optional 사용방법.](http://www.tcpschool.com/java/java_stream_optional)

---

**자바 8전에 메서드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 선택지 두가지**

- **예외 던지기**
- **null 반환.**

**자바 8 이후 또 하나의 선택지가 생겼다.**

 그 주인공은 Optional<T> 는 null 이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다. 

아무것도 담지 않은 옵셔널은 비었다고 말한다. 반대로 어떤 값을 담은 옵셔널은 비지 않았다고 한다 

- 옵셔널은 원소를 최대 1개 가질 수 있는 불변 컬렉션이다.

(Optional<T> 가 Collection<T> 를 구현하지 않았지만, 원칙적으로는 그렇다는 말이다. )

클라이언트에서 어떻게 Optional을 활용하나?

1. **기본값을 정해둘 수 있다.**

```jsx
String lastWordLexicon = max(words).orElse("단어없다.");
```

1. **원하는 예외를 던질 수 있다.** 

```jsx
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

1. **항상 값이 채워져 있다고 가정한다.** 

( NoSuchElementException을 주의한다.)

```jsx
Element lastNodeGas = max(Elements.NOBLE_GASES).get();
```

1. **기본 값 설정 비용이 클 경우.** 

```jsx
public T orElse(T other)

public static String orElseBechmark(){
		return Optional.of("baedung").orElse(getRandomName());
}
```

```jsx
public T orElseGet(Supplier<? extends T> other)ㄱpublic T orElseGet(Supplier<? extends T> other)ㄱ

public static String orElseGetBenchmark(){
		return Optinal.of("baeldung").orElseGet(() -> getRandomName());
}
```

Supplier<T> 를 인수로 같은 orElseGet을 사용하면 초기 설정 비용을 낮출 수 있다. 

isPresent 메서드

- 옵셔널이 채워져 있으면 true, 비어있으면 false 를 반환.
- 원하는 모든 메서드를 수행할 수 있으나 신중히 사용하자. isPresent를 사용하기 전에 앞의 옵셔널 활용 1~4로 표현할 수 있는지 면밀히 검토하자.

```java
public class ParentPid {

		public static void main(String[] args){
				ProcessHandle ph = ProcessHandle.cureent();

				// Inappropriate use of isPresent
				Optional<ProcessHandle> parentProcess = ph.parent();

				System.out.println("Prant PID:" + (parentProcess.isPresent() ?
								String.valueOf(parentProcess.get().pid()) : "N/A"));

				// Equivalent (and superior) code using orElse
				System.out.println("Praent PID: " + ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));		

		}

}
```

**스트림을 사용할 경우.** 

Java8 의 stream을 이용하여 아래와 같이 표현할 수 있다. 

옵셔널에 값이 있다면 (Optional::isPresent) 그 값을 꺼내 (Optional::get) 스트림에 매핑한다. 

```java
streamOfOptionals
	.filter(Optional::isPresent)
	.map(Optional::get)
```

자바9 부터는 Optional에 stream() 메서드가 추가되었다. 

이 메서드는 Optional을 Stream 으로 변환해주는 어댑터다. 

옵셔널에 값이 있으면 그 값을 원소로 담은 스트림으로 값이 없다면 빈 스트림으로 변환한다. 이를 Stream 의 flatMap 메서드 와 조합하면 앞의 코드를 다음처럼 명료하게 바꿀 수 있다. 

```java
streamOfOptionals.flatMap(Optinal::stream)
```

**반환형으로 옵셔널을 사용한다고 해서 무조건 득이 되는 건 아니다.** 

### Optional을 사용하면 안되는 경우.

- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다.
- 빈 Optional<List<T>> 를 반환하기보다는 빈 List<T> 를 반환하는게 좋다.
    
    ( 빈 컨테이너를 그대로 반환하면 클라이언트에 옵셔널 처리 코드를 넣지 않아도 된다. ) 
    

### Optional을 맵의 값으로 사용하면 절대 안된다.

- 맵 안에 키가 없다는 사실을 나타내는 방법이 두가지가 된다.
- 키는 있지만 그 키가 속이 빈 옵셔널인 경우.

- 복잡성을 높여서 오류 가능성을 키우기 때문에 사용하지 말자. 더 **일반화하여 이야기하면 옵셔널을 컬렉션의 키, 값, 값, 원소나 배열의 원소로 사용하는게 적절한 상황은 거의 없다.**

---

### T 대신 Optional을사용하면 좋은 경우

- 결과를 얻을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional<T> 를 반환한다.
- Optional을 반환하는데는 대가가 따른다.
    - Optional도 엄연히 새로 할당하고 초기화해야 하는 객체이고 , 그 안에서 값을 꺼내려면 메서드를 호출해야 하니 한 단계를 더 거치는 셈이다. 그래서 성능이 중요한 상황에서는 옵셔널이 맞지 않을 수 있다.
    

> 박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무거울 수밖에 없다. 값을 두겹이나 감싸기 때문이다.
> 

그래서 자바 API 설계자는 int, long, double 전용 옵셔널 클래스들을 준비해놨다. 

바로 OptionalInt, OptionalLong, OptionalDouble 이다.  

이 옵셔널들도 Optional<T> 가 제공하는 메서드를 거의 다 제공한다 .

이렇게 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자. 

---

값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야 하는 메서드라면 옵셔널을 반환해야 할 상황일 수 있다. 하지만 옵셔널 반환에는 성능 저하가 뒤따르니, 성능에 민감한 메서드라면 null 을 반환하거나 예외를 던지는 편이 나을 수 있다. 그리고 옵셔널을 반환값 이외의 용도로 쓰는 경우는 매우 드물다.