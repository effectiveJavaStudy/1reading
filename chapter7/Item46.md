# Item46. 스트림에서는 부작용 없는 함수를 사용하라.

[Java Collectors 알아보기](https://daddyprogrammer.org/post/1163/java-collectors/)

[Java 에서 스트림 병합](https://www.baeldung.com/java-merge-streams)

---

> **스트림은 그저 또 하나의 API가 아닌 함수형 프로그래밍에 기초한 패러다임이다.**
> 

“스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다. 

이때 각 변환 단계는 가능한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다.”

**순수함수란?**

: 오직 입력만이 결과에 영향을 주는 함수를 말한다. 

다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다. 

이렇게 하려면 (중간 단계든 종단 단계든) 스트림 연산에 건네는 함수 객체는 모두 부작용이 없어야 한다 .

**46-1. 스트림 패러다임을 이해하지 못한 채 API만 사용했다.** 

```java
Map<String, Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()){
		words.forEach(word -> { 
				freq.merge(word.toLowerCase(), 1L, long::sum);
		});
}
```

위 코드는 스트림, 람다, 메서드 참조를 사용했고, 결과도 올바르다. 하지만 **절대 스트림 코드라고 할 수 없다.** 

**스트림 코드를 가장한 반복적 코드다.**  스트림 API의 이점을 살리지 못하여 같은 기능의 반복적 코드보다 길고, 읽기 어렵고, 유지보수에도 좋지 않다. 

**46-2 스트림을 제대로 활용해 빈도표를 초기화한다 .**

```java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()){
		freq = words
			.collect(groupingBy(String::toLowerCase, counting()));

}
```

자바 프로그래머는 for-each 반복문을 사용할 줄 알 텐데, **for-each 반복문**은 **forEach 종단 연산**과 비슷하게 생겼다. 하지만 **forEach 연산은 종단 연산 중 기능이 가장 적고 가장 ‘덜’ 스트림답다.** 

> forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자.
> 

위 소스는 collector(수집기)를 사용하는데, 스트림을 사용하려면 꼭 배워야 하는 새로운 개념이다. 

java.util.stream.Collectors 클래스는 무려 39개나 가지고 있고, 그 중에는 타입 매개변수가 5개나 되는 것도 있다. 

> **Collector(수집기)를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다**
> 

수집기는 총 세가지로, toList(), toSet(), toCollection(collectionFactory)가 그 주인 공이다. 

( 이들은 차례로 리스트, 집합, 프로그래머가 지정한 컬렉션 타입을 반환한다. ) 

**46-3 빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인**

```java
List<String> topTen = freq.keySet().stream()
			.sorted(comparing(freq::get).reversed())
			.limit(10)
			.collect(toList());
```

> 마지막 toList는 Collectors의 메서드다. 이처럼 Collectors의 멤버를 정적 임포트하여 쓰면 스트림 파이프라인 가독성이 좋아져, 흔히들 이렇게 사용한다.
> 

가장 간단한 맵 수집기는 toMap(keyMapper, valueMapper)로, 보다시피 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다. 

**46-4 toMap 수집기를 사용하여 문자열을 열거 타입 상수에 매핑한다.** 

```java
private static final Map<String, Operation> stringToEnum = 
		Stream.of(values()).collect(
				toMap(Object::toString, e -> e));
```

이 간단한 toMap 형태를 스트림의 각 원소가 고유한 키에 매핑되어 있을 떄 적합하다. 

스트림 원소 다수가 같은 키를 사용한다면 파이프라인이 `IllegalStateException`을 던지며 종료될 것이다.

**46-5 각 키와 해당 키의 특정 원소를 연관 짓는 맵을 새성하는 수집기.**

```java
Map<Artist, Album> topHits = albums.collect(
		toMap<Album::artist, a->a, maxBy(comparing(Album::sales))));

```

여기서 비교자로는 BinaryOperator에서 정적 임포트한 maxBy라는 정적 팩터리 메서드를 사용했다. 

maxBy는 Comparator<T>를 입력받아 BinaryOperator<T>를 돌려준다. 

> **“앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것이다. “**
> 

**46-7 마지막에 쓴 값을 취하는 수집기.**

```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

세 번째이자 마지막 toMap은 네 번째 인수로 맵 팩터리를 받는다. 이 인수로는 **EnumMap이나 TreeMap처럼 원하는 특정 맵 구현체를 직접 지정할 수 있다.** 

이상의 세 가지 toMap에는 변종이 있다. 그중 toConcurrentMap은 병렬 실행된 후 결과로 ConcurrentHashMap 인스턴스를 생성한다. 

`words.collect(groupingBy(word → alphabetize(word))`

groupingBy가 반환하는 수집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면, 분류 함수와 함께 다운스트림(downstream) 수집기도 명시해야 한다. 

다운스트림 수집기의 역할은 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성하는 일이다. 

이 매개변수를 사용하는 가장 간단한 방법은 toSet()을 넘기는 것이다. 그러면 groupingBy는 원소들의 리스트가 아닌 집합(Set)을 값으로 갖는 맵을 만들어 낸다. 

다운스트림 수집기로 counting()을 건네는 방법.

```java
map<String, Long> req = words.
		collect(groupingBy(String::toLowerCase, couning()));
```

**GroupingByConcurrent 메서드**

: 메서드의 동시 수행 버전으로, ConcurrentHashMap 인스턴스를 만들어준다. 

**partitioningBy**

: 분류 함수 자리에 프레디키트(predicate)를 받고 키가 boolean인 맵을 반환한다. 

( 프레디키드에 더해 다운스트림 수집기까지 입력받는 버전도 다중정의되어 있다. ) 

counting 메서드가 반환하는 수집기는 다운스트림 수집기 전용이다. Stream의 count 메서드를 직접 사용하여 같은 기능을 수행할 수 있으니 collect 형태로 사용할 일은 전혀 없다.

Collections에는 이런 속성의 메서드가 16개나 더 있다. 

그중 9개는 이름이 summing, averaging, summarizing으로 시작하며, 각각 int, long, double 스트림용으로 하나식 존재한다.

( 다중정의된 reducing 메서드들, filtering, mapping, flatMapping, collectingAndThen 메서드가 있는데, 대부분 프로그래머는 이들의 존재를 모르고 있어도 상관 없다.)

**minBy와 maxBy**

: 인수로 받은 비교자를 이용해 스트림에서 가장 가장작은 혹은 가장 큰 원소를 찾아 반환한다. Stream 인터페이스의 min과 max 메서드를 살짝 일반화한 것이자, java.util.function.BinaryOperator의 minBy와maxBy 메서드가 반환하는 이진 연산자의 수집기 버전이다 .

**joining 메서드.** 

: 이 메서드는 CharSequence 인스턴스의 스트림에만 적용할 수 있다. 이중 매개변수가 없는 joining 은 단순히 원소들을 연결(concatenate)하는 수집기를 반환한다. 

- **스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다**.
- **스트림 뿐만 아니라 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다.**
- **종단 연산 중 forEach는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다. 계산 자체에는 이용하지말자.**
- **스트림을 올바로 사용하려면 수집기를 알아둬야 한다. 가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining이다.**