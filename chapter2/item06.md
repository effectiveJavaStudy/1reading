# 아이템 6. 불필요한 객체 생성을 피하라



- 똑같은 기능의 객체를 매번 생성하기 보다는 객체 하나를 재사용하는 펀이 나을 때가 많다. 특히 불변 객체는 언제든 재사용할 수 있다.

```java
String s = new String("a");
```

- 실행될 때마다 String 인스턴스를 새로 만든다.

```java
String s = "a";
```

- 새로운 인스턴스를 만드는 대신 하나의 String 인스턴스를 사용한다.

# 방법

## 1. 정적 팩토리 메서드 사용

- 생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 그렇지 않다.

## 2. 캐싱 사용

- 생성 비용이 비싼 객체를 반복해서 사용한다면 캐싱해서 재사용을 한다.

```java
static boolean isRomanNumeral(String s) {
	return s.matches(~~);
}
```

- String.matches는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복적으로 사용하기엔 적합하지 않다.
- 이 메서드가 내부에서 만드는 정규표현식용 Pattern 인스턴스는, 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상이 된다.
- Pattern은 입력받은 정규표현식에 해당하는 유한 상태 머신(finite state machine)을 만들기 때문에 인스턴스 생성 비용이 높다.
- isRomanNumeral()메서드를 남발하게 된다면 Pattern 인스턴스 생성-파괴가 반복적으로 일어나 성능 저하에 큰 영향을 미칠 것이다.

```java
public class Roman {
	private static final String REGEX = "";
	private static final Pattern ROMAN = Pattern.compile(REGEX);

	static boolean isRomanNumeral2(String s){
		return ROMAN.matcher(s).matches();
	}
}
```

- Pattern 인스턴스 클래스 초기화 과정에서 직접 생성해 캐싱해두고 나중에 isRomanNumeral 메서드가 호출될 때마다 이 인스턴스를 재사용한다.

## 3. 어댑터 사용

- 객체가 불변이라면 재사용해도 안전함이 명백하다. 어댑터를 생각해보자.(어댑터를 뷰(View)라고도 한다.) 어댑터는 실제 작업은 뒷단 객체에 위임하고, 자신은 제 2의 인터페이스 역할을 해주는 객체다. 어댑터는 뒷단 객체만 관리하면 된다.
- Map 인터페이스의 keySet 메서드는 Map 객체 안의 키 전부를 담은 Set 뷰를 반환한다. keySet메서드는 호출할 때마다 매번 같은 Set 인스턴스를 반환한다.
- 반환한 객체 중 하나를 수정하면 다른 모든 객체가 따라서 바뀐다. 모두가 똑같은 Map 인터페이스를 대변하기 때문이다.
- 따라서 keySet이 어댑터(뷰) 객체를 여러 개 만들어도 상관없지만, 그럴 필요도 없고 이득도 없다.

## 4. 기본 타입 이용

- 불필요한 객체를 만들어내는 또다른 예로 오토 박싱(auto boxing)이 있다. 오토 박싱(auto boxing)은 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다. 오토 박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다.

```java
private static long sum(){
	long sum = 0L; // Long, long
	for(long i=0; i<=Integer.MAX_VALUE; i++){
		sum += i;
	}
	return sum;
}
```

- sum 변수를 long이 아닌 Long으로 선언해서 불필요한 Long 인스턴스가 2^31개나 만들어진 것이다.