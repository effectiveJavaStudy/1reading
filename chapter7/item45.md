# `Item45` 스트림은 주의해서 사용하라.

[**스트림 API와 함수형 프로그래밍.**](https://7942yongdae.tistory.com/160) 

[스트림이란?](https://incheol-jung.gitbook.io/docs/q-and-a/java/stream)

---

> **스트림 API의 등장.**
> 
- 다량의 데이터 처리 작업(순차적이든 병렬적이든)을 돕고자 **자바 8에 추가되었다.**

스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 의미하며, 스트림 파이프라인은 이 원소들을 수행하는 연산단계를 표현하는 개념이다. 

스트림 안의 데이터 원소들은 기본적으로 객체 참조타입이며, **int, long, double 세가지 기본타입을 지원하기도 한다.** 

---

> **[스트림 API 3가지 특징.](https://wildeveloperetrain.tistory.com/52)**
> 
- 원본 데이터를 변경하지 않는다.
- 일회용이다.
- 내부 반복으로 작업을 처리한다.

[참조](https://jongwoon.tistory.com/35)

---

> **스트림 연산**
> 

스트림 인터페이스 연산을 크게 두 가지(중간 연산, 최종 연산)로 구분할 수 있다. 

**중간 연산**

연산 결과를 스트림으로 반환하기 때문에 중간 연산을 연속해서 연결할 수 있다. 

| 연산 | 형 |
| --- | --- |
| filter | 중간 연산 |
| map | 중간 연산 |
| limit |  중간 연산 |
| sorted | 중간 연산 |
| distinct | 중간 연산 |

**최종 연산.** 

스트림의 요소를 소모하면서 연산을 수행하기 때문에 단 한번만 연산이 가능하다.

| 연산 | 형식 |
| --- | --- |
| forEach | 최종 연산 |
| count | 최종 연산 |
| collect |  최종 연산.  |

---

> **스트림 파이프라인**
> 

```java
public static void main(String[] args){
		List.of("apple", "banana", "car")
					.stream()
					.filter(s -> s.startsWith("b"))
				  .forEach(System.out::println);
}
```

소스 스트림에서 시작해 종단 연산(terminal operation) 으로 끝나며, 그 사이에 하나 이상의 중간 연산이 있을 수 있다. 

**각 중간 연산은 스트림을 다른 스트림으로 변환하는데, 변환된 스트림의 원소 타입은 변환 전 스트림의 원소 타입과 같을 수  있다.** 

---

> **스트림 지연 평가(Lazy evaluation)**
> 

**: 스트림은 종단 연산에 쓰이지 않는 데이터는 계산을 하지 않는다. 따라서 종단 연산이 없는 스트림은 아무 일도 하지 않는다.** 

***이러한 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠다. 종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않는 명령어인 no-op과 같으니, 종단 연산을 빼먹는 일이 절대 없도록 하자.** 

> **스트림 API는 메서드 연쇄를 지원하는 플루언트(fluent API)다.**
> 

즉, 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다. 파이프라인 여러 개를 연결해 표현식 하나로 만들 수도 있다. 

기본적으로 스트림 파이프라인은 순차적으로 수행된다. 

( 파이프 라인을 병렬로 실행하려면 파이프라인을 구성하는 스트림 중 하나에서 parallel  메서드를 호출해주기만 하면 되나, 효과를 볼 수 있는 상황은 많지 않다. 

스트림 API는 다재다능하여 사실상 어떠한 계산이라도 해낼 수 있다. 하지만 할 수 있다는 뜻이지, 해야 한다는 뜻은 아니다. 

( **스트림을 제대로 사용하면 프로그램이 짧고 깔**

**끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수도 힘들어 진다.)** 

---

**45-1 사전 하나를 훑어 원소 수가 많은 아나그램 그룹들을 출력한다.** 

```java
public clas Anagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        
        Map<String, Set<String>> groups = new HashMap<>();
        try(Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word),
                (unused) -> new TreeSet<>()).add(word);
            }
        }
        
        for (Set<String> group : groups.values())
            if(group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
    }
    
    private static String alphbetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

**45-2 스트림을 과하게 사용했다. - 따라 하지 말 것! ( 유지보수하기 어려워 진다. )** 

```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        // 스트림을 과하게 사용했다. - 따라 하지 말 것!
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new, (sb, c) -> sb.append((char) c), StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }

    private static String alphabetize(String word) {
        char[] a = word.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
```

**45-3 스트림을 적절히 활용하면 깔끔하고 명료해진다.** 

```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        // 스트림을 적절히 사용하면 깔끔하고 명료해진다.
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static String alphabetize(String word) {
        char[] a = word.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

---

char 값들을 스트림으로 처리하는 코드

```java
"Hello world!".chars().forEach(Sytem.out::println);
```

위의 결과는 72101108108113211911111410810033을 출력한다.

“Hello world!”.chars()가 반환하는 스트림의 원소는 char가 아닌 int 값이기 때문이다. 

```java
"Hello world!".chars().forEach(x -> System.out.print((char) x));
```

하지만 char 값들을 처리할 때는 스트림을 삼가는 편이 낫다. 

### 함수 객체로는 할 수 없지만 코드 블록으로는 할 수 있는 일

- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다.
- 코드 블록에서는 return 문을 사용해 메서드에서 빠져나가거나, break나 continue 문으로 블록 바깥의 반복문을 종료하거나 반복을 한 번  건너뛸 수 있다.
- 또한 메서드 선언에 명시된 검사 예외를 던질 수 있다.

### 스트림이 아주 안성맞춤인 일

- 원소들의 시퀀스를 일관되게 변환한다.
- 원소들의 시퀀스를 필터링한다.
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다. ( 더하기, 연결하기, 최소값 구하기 등)
- 원소들의 시퀀스를 컬렉션에 모은다.( 아무도 공통된 속성을 기준으로 묶어 가며)
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

### 스트림으로 처리하기 어려운 일.

- 한 데이터가 파이프라인의 여러 단계를 통과할 때 데이터의 각 단계에서의 값들에 동시에 접근하는 것은 처리하기 어렵다.
    - 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값을 잃는 구조이기 때문이다.
    

### 메르센 소수(Mersenne prmie)를 출력하는 프로그램

: 이 파이프라인의 첫 스트림으로는 모든 소수를 사용할 것이다. 다음 코드는 (무한) 스트림을 반환하는 메서드다.  BigInteger의 정적 멤버들은 정적 임포트하여 사용한다고 가정한다. 

```java
static Stream<BigInteger> primes(){
		return Stream.iterate(TWO, BigInteger::nextProbablePrime); 
}
```

처음 20개의 메르센 소수를 출력하는 프로그램

```java
public static void main(String[] args) {
			primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
						.filter(mersenne -> mersenne.isProbablePrime(50))
						.limit(20)
						.forEach(System.out::println);

}
```

지수는 단순히 숫자를 이진수로 표현한 다음 몇 비트인지를 세면 나오므로, 종단 연산을 다음처럼 작성하면 원하는 결과를 얻을 수 있다. 

```java
.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
```

**45-4 데카르트 곱 계산을 반복 방식으로 구현.** 

```java
private static List<Card> newDeck(){
		List<Card> result = new ArrayList<>();
		for(Suit suit : Suit.values())
				for(Rank rank : Rank.values())
						result.add(new Card(suit, rank));
		return result; 
}
```

스트림으로 구현한 코드다. 중간 연산으로 flatMap은 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음 그 스트림들을 다시 하나의 스트림으로 합친다. 이를 평탄화(flattening) 라고 한다. 

**45-5 데카르트 곱 계산을 스트림 방식으로 구현**

```java
private static List<Card> newDeck(){
			return Stream.of(Suit.values())
				.flatMap(suit -> 
								Stream.of(Rank.values())
										.map(rank -> new Card(suit, rank))
				.collect(toList());
}
```

---

### 정리

- **스트림을 사용해야 멋지게 처리할 수 있는 일이 있고, 반복 방식이 더 알맞는 일도 있다. 그리고 수많은 작업이 이 둘을 조합했을 때 가장 멋지게 해결된다.**
- **스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라.**