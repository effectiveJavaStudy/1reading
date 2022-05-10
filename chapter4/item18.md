# 상속보다는 컴포지션을 사용하라.

## 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.

상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스이 동작에 이상이 생길 수 있다.

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll (Collection< ? extends E > c){
        addCount += c.size();
        return super.addAll(c);

    }

    public int getAddCount () {
        return addCount;
    }
}
```

```java
InstrumentedHashset<String> s  = enw InstrumenetedHashSet<>();
s.addAll(List.of("틱","틱틱","틱틱틱"));
```
getAddCount 메서드를 호출하면 3을 반환하리라 기대했겠지만, 실제로는 6을 반환한다.

원인은 addAll 메서드가 add메서드를 사용한 것이다.

HashSet은 addAll은 add 메서드를 통해서 추가하는데 이때 불리는 add가 InstrumentedHashSet의 add이다. 따라서 총 6이 되는 것이다.

addAll 메서드를 다른 식으로 재정의 할 수도 있다. 상위 클래스의 메서드 동작을 다시 구현하는 방식은 어렵고 , 시간도 더들고 오류를 내거나 성능을 떨어뜨릴수도 있다.
하위 클래스에서는 접근할수 없는 private 필드일 경우 구현 자체가 불가능하다.

상위 클래스에서 새로운 메서드를 추가하면 보안이 깨질 수가 있다.

그리고 다음에 상위 클래스에 새 메서드가 추가가 됐는데 운 없게도 하필 하위클래스의 메서드와 시그니처가 같고 반환 타입은 다르다면
하위 클래스는 컴파일 조차 되지 않는다.

이 문제를 피하려면 새로운 클래스를 만들고 기존 클래스가 새로운 클래스의 인스턴스를 참조하게 하는 것 이다.

기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 컴포지션이라 한다.

새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 ㅔㅁ서드를 호출해 그 결과를 반환한다. 이 방식을 forwarding이라 한다.
```java
public class ForwardingSet<E> implements Set<E> {

    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s;}

    public void clear() { s.clear(); }
    public boolean contains(Object o) {return s.contains(o); }
    public boolean isEmpty() { return s.isEmpty(); }
    public int size() { return s.size(); }
    public Iterator<E> iterator() { return s. iterator(); }
    public boolean add(E e) { return s.add(e); }
    public boolean remove(Object o) {return s.remove(o); }
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
    public boolean removeAll(Collection<?> c) { return s.removeAll(c); }
    public boolean retainAll(Collection<?> c) { return s.retainAll(c); }
    public Object[] toArray() { return s.toArray(); }
    public <T> T[] toArray(T[] a) { return s.toArray(a); }
    @Override public boolean equals(Object o) { return s.equals(o); }
    @Override public int hashCode() {return s.hashCode(); }
    @Override public String toString() { return s.toString(); }

}
```

```java

public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll (Collection< ? extends E > c){
        addCount += c.size();
        return super.addAll(c);

    }

    public int getAddCount () {
        return addCount;
    }
}
```
FowardingSet이 set을 가지고 있고 상속받고 있다.

다른 Set 클래스를 감싸고 있다는 뜻에서 InstrumentedSet같은 클래스를 래퍼 클래스라 하며 Decorator 패턴이라 부른다.
컴포지션과 전달의 조합은 넓은 의미로 위임이라고 부른다.

래퍼 클래스는 단점이 거의 없으나 래퍼 클래스가 콜백 프레임워크와는 어울리지 않다는 단점이 있다.

콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출때 사용하도록  한다.
내 부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신의 참조를 넘기고 콜백때는 내부 객체를 호출하게 된다.

상속은 하위 클래스가 상위 클래스의 진짜 하위 타입인 상황에서만 쓰여야한다.

클래스 B가 클래스 A의 is-a 관계 일때만 클래스 A를 상속해야 한다.
