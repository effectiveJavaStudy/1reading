# Item47. 반환 타입으로는 스트림보다 컬렉션이 낫다.

[**for-each 와 Iterable ?**](https://soyeondev.tistory.com/60)

[Iterable 과 Iterator의 차이점?](https://devlog-wjdrbs96.tistory.com/84)

---

**자바 7까지는메서드 반환타입으로 Collection, Set, List 같은 컬렉션 인터페이스 혹은 Iterable 이나 배열을 썼다.**

이 중 가장 적합한 타입을 선택하기라 그다지 어렵지 않았다. 

for-each 문에서만 쓰이거나 반환된 원소 시퀀스가 일부 Collection 메서드를 구현할 수 없을 때는 Iterable 인터페이스를 썼다. 

반환 원소들이 기본 타입이거나 성능에 민감한 상황이라면 배열을 썼다. 그런데 

**자바8이 스트림이라는 개념을 들고 오면서 이 선택이 아주 복잡한 일이 되어 버렸다.** 

원소 시퀀스를 반환할 때는 당연히 스트림을 사용해야 한다는 이야기를 들어봤을지 모르겠지만, 

**아이템45에서 이야기했듯이 스트림은 반복(iteration)을 지원하지 않는다. 따라서 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다.** 

사실 Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함할 뿐만 아니라, 

Iterable 인터페이스가 정의한 방식대로 동작한다. 그럼에도 **for-each로 스트림을 반복할 수 없는 까닭은 바로 Stream 이 Iterable 을 확장(extend)하지 않아서다.** 

---

**Iterable은 collection의 상위 인터페이스이다.**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7aa03975-d9f7-4999-883c-7305b9bc93c3/Untitled.png)

Iterator 인터페이스는 Collection과는 별개로 존재하는 인터페이스이다. 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b2e77d3c-466f-4305-af9e-3936bcbe67da/Untitled.png)

**47-1. 자바 타입 추론의 한계로 컴파일되지 않는다.**  

```java
for(ProcessHandle ph : ProcessHandle.allProcesses()::iterator){
	// 프로세스를 처리한다. 
}
```

위 코드는 오류가 난다. 

오류를 바로 잡으려면 메서드 참조를 매개변수화된 Iterable 로 적절히 형변환 해주어야 한다. 

**47-2. 스트림을 반복하기 위한 ‘끔직한’ 우회 방법.**

```java
for(ProcessHandle ph : (Iterable<ProcessHandle>)
												ProcessHandle.allProcesses()::iterator){
		// 프로세스를 처리한다.
}
```

다행이 어댑터 메서드를 사용하면 상황이 나아진다. 

자바는 이런 메서드를 제공하지 않지만 다음 코드와 같이 쉽게 만들어낼 수 있다. 

**47.3 Stream<E> 를 Iterable<E>로 중개해주는 어댑터**

```java
public static <E> Iterable<E> iterableOf(Stream<E> stream){
		return stream::iterator;
}
```

어댑터를 사용하면 어떤 스트림도 for-each 문으로 반복할 수 있다. 

```java
for(ProcessHandle p : iterableOf(ProcessHandle.allProcesses())){
		// 프로세스를 처리한다.
}
```

반대로, API가 Iterable만 반환하면 이를 스트림 파이프라인에서 처리하려는 프로그래머가 성을 낼 것 이다. 자바는 이를 위한 어댑터도 제공하지 않지만, 역시 손쉽게 구현할 수 있다. 

**47.4 Iterable<E>를 Stream<E>로 중개해주는 어댑터**

```java
public static <E> Stream<E> streamOf(Iterable<E> iterable)P
		return StreamSupport.stream(iterable.spliterator(), false);
}
```

**객체 시퀀스를 반환하는 메서드를 작성하는데,** 

이 메서드가 오직 스트림 파이프라인에서만 쓰인 걸 안다면 마음 놓고 스트림을 반환하게 해주자. **반대로 반환된 객체들이 반복문에서만 쓰일 걸 안다면 Iterable을 반환하자.** 

하지만 공개 API를 작성할 때는 스트림 파이프라인을 사용하는 사람과 반복문에서 쓰려는 사람 모두를 배려해야 한다. 사용자 대부분이 한 방식만 사용할 거라는 그럴싸한 근거가 없다면 말이다. 

**Collection 인터페이스는 Iterable 의 하위 타입이고, stream 메서드도 제공하니 반복과 스트림을 동시에 지원한다.** 

따라서 원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는게 일반적으로 최선이다. 

**Arrays 역시 Arrays.asList와 Stream.of 메서드로 손쉽게 반본과 스트림을 지원할 수 있다.** 반환하는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList나 HashSet 같은 표준 컬렉션 구현체를 반환하는 게 최선일 수 있다. 하지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다. 

**47.5 입력 집합의 역집합을 전용 컬렉션에 담아 반환한다.**

```java
public class PowerSet {

		public static final <E> Collection<Set<E>> of(Set<E> s){
				List<E> src = new ArrayList<>(s);
				if(src.size() > 30)
						throw new IllegalArgumentException(
									"집합에 원소가 너무 많습니다.(최대 30개). : " + s);

				return new AbstractList<Set<E>>(){
						@Override public int size(){
									// 역집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱 것과 같다. 
									return 1 << src.size();
						}
						@Override public boolean contains(Object o){
									return o instanceof Set && src.containsAll((Set)o);
						}
						@Override public Set<E> get(int index){
									Set<E> result = new HashSet<>();
									for(int i=0; index != 0; i++, index >>= 1)
											if((index & 1) == 1)
													result.add(src.get(i));
									return result;
						}
				};

		}

}
```

- 입력 집합의 원소 수가 30을 넘으면 PowerSet.of가 예외를 던진다. 이는 Stream이나 Iterable이 아닌 Collection 을 반환타입으로 쓸 때의 단점을 잘 보여준다.
- 다시 말해, Collection의 size 메서드가 int 값을 반환하므로 PowerSet.of가 반환되는 시퀀스의 최대 길이는 Integer.MAX_VALUE 혹은 2(31)-1로 제한된다.
- Collection 명세에 따르면 컬렉션이 더 크거나 심지어 무한대일 때 size가 2(31)-1 을 반환해도 되지만 완전히 만족시러운 해법은 아니다.

AbstractCollection을 활용해서 Collection 구현체를 작성할 때는 Iterable 용 메서드 외에 2개만 더 구현하면 된다. 

바로 contains와 size다. 

contains와 size를 구현하는게 불가능할 때는 컬렉션보다는 스트림이나 Iterable 을 반환하는 편이 낫다. 원한다면 별도의 메서드를 두어 두 방식을 모두 제공해도 된다. 

**47.6 입력 리스트의 모든 부분리스트를 스트림으로 반환한다.**

```java
public class SubLists {
		public static <E> Stream<List<E>> of(List<E> list){
				return Stream.concat(Stream.of(Collections.emptyList()),
						perfixes(list).flatMap(SubLists::suffixes));
		}

		private static <E> Stream<List<E>> prefixes(List<E> list){
				return IntStream.rangeClosed(1, list.size())
						.mapToObj(end -> list.subList(0, end));
		}

		private static <E> Stream<List<E>> subffixes(List<E> list){
				return IntStream.range(0, list.size())
						.mapToObj(start -> list.subList(start, list.size()));

		}

}
```

**47.7 입력 리스트의 모든 부분리스트를 스트림으로 반환한다.**

```java
public static <E> Stream<List<E>> of(List<E> list){
			return IntStream.range(0, list.size())
				.mapToObj(start -> 
						IntStream.rangeClosed(start + 1, list.size())
									.mapToObj(end -> list.subList(start, end))
					.flatMap(x -> x);
}
```

---

### 정리

- **원소 시퀀스를 반환하는 메서드를 작성할 때는 이를 스트림으로 처리하기를 원하는 사용자와 반복으로 처리하길 원하는 사용자가 모두 있음을 떠올리고, 양쪽을 다 만족시키려고 노력하자.**
- **컬렉션을 반환할 수있다면 그렇게 하라.**
- **반환 전부터 이미 원소드을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList 같은 표준 컬렉션에 담아 반환하라. 그렇지 않으면 앞서의 멱집합 예처럼 적용 컬렉션을 구현할지 고민하라.**
- **컬렉션을 반환하는게 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환하라. 만약 나중에 Stream 인터페이스가 Iterable을 지원하도록 자바가 수정된다면 , 그때는 안심하고 스트림을 반환하면 될 것이다. (스트림 처리와 반복 모두에 사용할 수 있으니)**