과도한 동기화

- 성능 저하
- 교착 상태
- 예측 불가능한 상황.

> 동기화 메서드나 동기화 블럭에서 제어를 클라이언트에 양도하면 안된다.
> 

- **observer Pattern이란?**
    
    **객체의 상태 변화를 관찰하는 관찰자들**, 즉 옵저버들의 목록을 객체에 등록하여 상태 변화가 있을 때마다 메서드 등을 통해 객체가 직접 목록의 각 옵저버에게 통지하도록 하는 디자인 패턴이다.  
    
    클라이언트는 Set 집합에 원소가 추가되면 알림을 받을 수 있다. 
    

- **공통**
    
    ```java
    package effectivejava.chapter11.item79;
    import java.util.*;
    
    // 재사용할 수 있는 전달 클래스 (118쪽의 코드 18-3 재사용)
    public class ForwardingSet<E> implements Set<E> {
        private final Set<E> s;
        public ForwardingSet(Set<E> s) { this.s = s; }
    
        public void clear()               { s.clear();            }
        public boolean contains(Object o) { return s.contains(o); }
        public boolean isEmpty()          { return s.isEmpty();   }
        public int size()                 { return s.size();      }
        public Iterator<E> iterator()     { return s.iterator();  }
        public boolean add(E e)           { return s.add(e);      }
        public boolean remove(Object o)   { return s.remove(o);   }
        public boolean containsAll(Collection<?> c)
        { return s.containsAll(c); }
        public boolean addAll(Collection<? extends E> c)
        { return s.addAll(c);      }
        public boolean removeAll(Collection<?> c)
        { return s.removeAll(c);   }
        public boolean retainAll(Collection<?> c)
        { return s.retainAll(c);   }
        public Object[] toArray()          { return s.toArray();  }
        public <T> T[] toArray(T[] a)      { return s.toArray(a); }
        @Override public boolean equals(Object o)
        { return s.equals(o);  }
        @Override public int hashCode()    { return s.hashCode(); }
        @Override public String toString() { return s.toString(); }
    }
    ```
    
    ```java
    package effectivejava.chapter11.item79;
    
    // 집합 관찰자 콜백 인터페이스 (421쪽)
    public interface SetObserver<E> {
        // ObservableSet에 원소가 더해지면 호출된다.
        void added(ObservableSet<E> set, E element);
    }
    ```
    

### 동기화 블럭에서 제어를 클라이언트에 양도한 예시와 문제점.

ObservableSet에 관찰자를 SetObserver를 추가하면(addObserver) ObservableSet.add 메서드가 호출될 때마다 notifyElementAdded가 호출되고, 

추가된 관찰자의 added 메서드가 호출된다(observer.added())

```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>(); // 관찰자리스트 보관

    public void addObserver(SetObserver<E> observer) { // 관찰자 추가
        synchronized (observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) { // 관찰자제거
        synchronized (observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) { // Set에 add하면 관찰자의 added 메서드를 호출한다.
        synchronized (observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);  // notifyElementAdded를 호출한다.
        return result;
    }
}
```

```java
public class Test1 {
    public static void main(String[] args) {
         ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

        set.addObserver((set1, element) -> System.out.println(element)); // add 된 원소를 출력한다.
    // 제어로직이 람다식으로 밖에있다

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```

oberservers는 한번에 한 스레드에서만 접근할 수 있도록 synchronized 블록으로 감쌌다.

언뜻보면 문제가 없는 코드인 것 같지만, 위에서 주위하라고 언급되었던 제어권은 이 코드 밖에 있다. ( 람다식 ) 

### 문제를 일으킬 수 있는 익명함수 콜백 - concurrentModificationException

```java
public class Test2 {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
            new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<>() {
            @Override
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) // 값이 23이면 자신을 구독해지한다.
                    s.removeObserver(this); //this 를 넘겨주어야하기 때문에 람다로는 안됨.
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```

메인에서 정의한 added 메서드는 removeObserver 메서드를 호출하여 자기 자신을 넘겨주고 있는데 이미 for -each 문에서 observer를 synchronized로 감싸고 있기 때문에 removeObserver 호출시 ConcurrentModificationException이 발생한다. 

### 문제를 일으키는 스레드 - 교착 상태

```java
// ObservableSet 동작 확인 #3
public class Test3 {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

// 코드 79-2 쓸데없이 백그라운드 스레드를 사용하는 관찰자 (423쪽)
        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) {
                    ExecutorService exec =
                            Executors.newSingleThreadExecutor();
                    try {
                        exec.submit(() -> s.removeObserver(this)).get();
                    } catch (ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    }
                }
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```

위의 exec는 새로운 쓰레드에 removeObserver를 수행하는 람다함수를 넘긴다. 

 이 메서드는 실행하면 23까지 출력하고 계속 멈춰있다. 

removeObserver를 수행하기 위해서는 observer(락)를 획득해야 하는데 이 observer는 메인스레드에서 실행중인 notifyElementAdded 메서드에 의해 획득될 수 없다. 

- 불변식이 임시로 깨진 경우
    
    ### 불변식이 임시로 깨진 경우
    
    위의 경우는 문제가 생길 수는 있지만 observers의 일관성이 꺠지니는 않는다. 
    
     락의 재진입 가능성 : 이미 락을 획득한 스레드는 다른 synchronized 블록을 만났을 때 락을 다시 검사하지 않고 진입 가능하다. 
    
    재진입 가능한 락은 다음과 같이 교착상태를 회피할 수는 있게 하지만, 안전실패로 변모시킬 수 있다. 
    
    ```java
    public class Test {
      public synchronized void a() {
        b(); // 이론적으로라면 여기서 교착상태여야하지만 같은 스레드에 한해 재진입을 허용하기 때문에 
      }
      public synchronized void b() { // 진입 가능하다
      }
      public static void main(String[] args) {
        new Test().a();
      }
    }
    ```
    
    동기화 영역 밖에서 오는 외계인 메서드(바깥에서 오는 메서드)는 열린호출(open call )이라고 한다. 
    
    이 메서드는 언제 끝날지 내부에서는 알 수 없기 때문에 동기화블럭 안에서 실행하면 다른 스레드는 이 Lock 을 잡지 못하고 계속해서 대기해야 한다 .
    
    동기화의 기본 법칙은 동기화 영역 내에서 가능한 일을 적게 하는 것이다.
    

**가변 클래스를 작성할 떄 따라야 할 선택지** 

1. 동기화를 고려하지 말고 사용하는 측에서 고려하도록 하라. 
2. 동기화를 내부에서 수행해 스레드안전하게 만들자 ( 외부에서 전체 락을 거는 것보다 더 효율이 높을 시에만 ) 

- 락 분할 (lock splitting) - 하나의 클래스에서 기능적으로 락을 분리해서 사용하는 것 ( 두개 이상의 락 - ex : 읽기 전용 락, 쓰기 전용 락 )
- 락 스트라이핑(lock striping) - 자료 구조 관점에서 한 자료구조 전체가 아닌 일부분(buckets or stripes 단위)에 락을 적용하는 것
- 비 차단 동시성 제어 (nonblocking concurrentcy control )

### 성능 측면

과도한 동기화는 병렬로 실행할 기회를 잃고 모든 코어가 메모리를 일관되게 보기 위한 지연 시간이 진짜 비용, 가상 머신 코드 최적화를 제한한다는 점도 과도한 동기화의 또 다른 비용이다. 

- 동기호를 전혀 하지말고 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 하자.
- 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들다. ( 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 )
    - 락 분할, 락 스트라이핑, 비차단 동시성 제어 등 다양한 기법을 동원해 동시성을 높일 수 있음.

> 동기화 블록 안에 제어를 절대 클라이언트에 양도하면 안됨.  외부에서 컨트롤 가능한 요소들이 들어오는 건 절대 안됨.
> 

### 결론

교착상태와 데이터 훼손을 피하기 위해선 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자. 

동기화 영역 안에서 작업을 최소한으로 줄이자.

가변 클래스를 설계할 떄는 스스로 동기화해야 할지 고민하자. 

합당한 이유가 있을 때만 내부에서 동기화 하고, 그 여부를 문서화 하자.