### 스레드 안전성을 문서화 해야 하는 이유

- 한 메서드를 여러 스레드가 동시에 호출할 때, 그 메서드가 어떻게 동작하느 지는 해당 클래스와 사용자와의 중요한 계약이다.
- API 문서에서 아무런 언급도 없으면 그 클래스 사용자는 나름의 가정을 해야 한다.

( 만약 그 가정이 틀리면 클라이언트 프로그램은 동기화를 충분히 하지 못하거나 지나치게 한 상태일 것이며, 두 경우 모두 심각한 오유로 이어질 수 있다. ) 

### 스레드 안전성 수준의 종류.

스레드 안정성에서 수준이 있다. 

- 멀티스레드 환경에서도 API를 안전하게 사용하게 하려면 클래스가 지원하는 스레드 안전성 수준을 명시해야 함.

**스레드 안전성이 높은 순으로 나열한 것이다.** 

- **불변( immutable) :** 이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화도 필요 없다.
    
    ex) String , Long, BigInteger가 대표적이다. 
    
- • **무조건적 스레드 안전(unconditionally thread-safe)** : 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화 없이 동시에 사용해도 안전하다. `AtomicLong` , `ConcurrentHashMap` 이 여기에 속한다.

- • **조건부 스레드 안전(conditionally thread-safe)** : 무조건적 스레드 안전과 같으나, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다. `Collections.synchronized` 래퍼 메서드가 반환한 컬렉션들이 여기 속한다.(Collections.synchronized로 만든 컬렉션들을 반복해서 돌릴 때 외부에서 동기화 해야 한다.)
    - 조건부 스레드 안전 클래스는 주의해서 문서화 해야 한다.
    - 어떤 순서로 호출할 때 외부 동기화가 필요한지, 그리고 그 순서로 호출하려면 어떤 락 혹은 락들을 얻어야 하는지 알려줘야 한다.

- • **스레드 안전하지 않음(not thread-safe)** : 이 클래스의 인스턴스는 수정될 수 있다. 동시에 사용하려면 각각의 메서드 호출을 클라이언트가 선택한 외부 동기화 메커니즘으로 감싸야 한다. `ArrayList`, `HashMap` 같은 기본 컬렉션이 여기 속한다.

- **스레드 적대적(thread-hostile)**
 : **이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티 스레드 환경에서 안전하지 않다.** 이 수준의 클래스는 일반적으로 정적 데이터를 아무 동기화 없이 수정한다. 이런 클래스를 고의로 만드는 사람은 없겠지만, 동시성을 고려하지 않고 작성하다 보면 우연히 만들어질 수 있다. 스레드 적대적으로 밝혀진 클래스나 메서드는 일반적으로 문제를 고쳐 재배포하거나 deprecated API로 지정한다.

### 스레드 안전성 annotation

- **@Immutable**
- **@ThreadSafe** : 무조건적 스레드 안전, 조건부 스레드 안전
- **@NotThreadSafe**

### 스레드 안전성 수준 문서화

- 조건부 스레드 안전한 클래스는 어떤 순서로 외부 동기화가 필요한지, 그 순서를 호출하려면 어떤 락/ 락들을 얻어야 하는지 주의해서 문서화
- 클래스의 스레드 안전성은 보통 클래스의 문서화 주석에 기재, 독특한 메서드라면 해당 메서드의 주석에 기재
- 반환 타입만으로 알 수 없는 정적 팩터리라면 반환하는 객체의 스레드 안전성을 반드시 문서화

[Collection.ysnchronizedMap API 문서](https://www.baeldung.com/java-synchronizedmap-vs-concurrenthashmap)

### 락 제공

클래스가 외부에서 사용할 수 있는 락을 제공하면 클라이언트에서 일련의 메서드 호출을 원자적으로 수행할 수 있다. 

하지만 이 유연성에는 대가가 따른다. 내부에서 

.

내부에서 처리하는 고성능 동시성 제어 메커니즘과 혼용할 수 없게 되는 것이다.

그래서 `ConcurrentHashMap` 과 같은 동시성 컬렉션과는 함께 사용하지 못한다.

또한, 클라이언트가 공개된 락을 오래 쥐고 놓지 않는 서비스 거부 공격(denial-of-service attack) 을 수행할 수도 있다.

```java
// 비공개 락 객체 관용구 - 서비스 거부 공격을 막아준다. 
private final Object lock = new Ojbect();

public void foo(){
		Synchronized(lock){
				...
		}
}
```

**비공개 락 객체**는 클래스 바깥에서는 볼 수 없으니 클라이언트가 그 객체의 동기화에 관여할 수 없다. 

※ 위 예제에서 lock 필드를 final 로 선언했는데

이는 우연히 락 객체가 교체되는 일을 예방해준다. 락이 교체되면 끔찌한 결과로 이어진다. 

> **락 필드는 항상 final로 선언하자.**
> 

- 비공개 락 객체 관용구는 무조건적 스레드 안전 클래스에서만 사용할 수 있다.
- **조건부 스레드 안전 클래스**에서는 특정 호출 순서에 필요한 락이 무엇인지를 클라이언트에게 알려줘야 하므로 이 관용구를 사용할 수 없다
- 비용개 락 객체 관용구는 상속용으로 설계한 킄래스에 특히 잘 맞는다.
- 

### 🤘🏻정리

- 모든 클래스가 자신의 스레드 안전성 정보를 명확히 문서화 해야 한다.

- 조건부 스레드 안전 클래스는 메서드를 어떤 순서로 호출할 때 외부 동기화가 요구되고, 그 때 어떤 락을 얻어야 하는지도 알려줘야 한다.
- 무조건적 스레드 안전 클래스를 작성할 때는 synchronized 메서드가 아닌 비공개 락 객체를 사용하자.( 이렇게 해야 클래스인트나 하위 클래스에서 동기화 매커니즘을 깨뜨리는걸 예방할 수 있고, 필요하다면 다음에 더 정교한 동시성 메커니즘을 재구현할 여지가 생긴다. )