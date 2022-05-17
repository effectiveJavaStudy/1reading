[SynchronizedMap과 ConcurrentHashMap의 차이점.](https://ooz.co.kr/71)

[CountDownLatch 사용 방법](https://codechacha.com/ko/java-countdownlatch/)

[병행성을 위한 CountDownLatch](https://imasoftwareengineer.tistory.com/100)

[CountDownLatch를 이용해 thread 대기하기](https://1minute-before6pm.tistory.com/30) 

[스레드 동기화 Semaphore](https://lordofkangs.tistory.com/27)

[뮤텍스, 세마포어, 모니터](https://steady-coding.tistory.com/557)

---

### 정리

wait와 notify 대신 동시성 유틸리티를 사용하자. 

Legacy에서 wait는 항상 while  안에서 호출하자

notify보다는 notifyAll을 사용하는 것이 안전하다. 

notify를 사용한다면 응답 불가 상태에 빠지지 않게 각별히 조심하자.

---

### **java.util.concurrent 의 고수준 유틸리티 세 범주로 구분.**

- 동시성 컬렉션 (concurrent collection)
- 실행자 프레임워크
- 동기화 장치 (synchronizer)

---

### 동시성 컬렉션 (Concurrent Collection )

> **List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다.**
> 
- 높은 동기성에 도달하기 위해 동기화를 각자의 내부에서 수행한다.
- 동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.
- 동시성 무력화가 불가능하므로 여러 메서드를 원자적으로 묶어 호출하는 것도 불가능 → 여러 동작을 하나의 원자적 동작으로 묶는 상태 의존적 수정 메서드 추가.
    - 유용하여 자바 8에서 일반 컬렉션 인터페이스들에도 디폴트 메서드로 추가됨. ex) Map 의 putlfAbsent(key, value)
- 동기화한 컬렉션이 아닌 동시성이 뛰어나며 속도도 빠른 동시성 컬렉션을 사용하자. 즉, Collections.synchronizedMap보다는 ConcurrentHashMap을 사용하는 것이 훨씬 좋다.
- 컬렉션 인터페이스 중 작업이 성공적으로 완료 될 때까지 기다리도록 확장한 것들도 있다.
    - Queue를 확장한 BlockingQueue → 대부분 실행자 서비스 구현자에서 작업 큐로 확장됨.

### 동기화 장치

> 다른 스레드를 기다릴 수 있게하여 서로 작업을 조율할 수 있게 한다.
> 
- CountDownLatch :  어떤 스레드가 다른 스레드에서 작업이 완료될 때까지 기다릴 수 있도록 해주는 클래스이다.
- Semaphore : 멀티 프로그래밍 환경에서 다수의 프로세스나 스레드가 n개의 공유 자원에 대한 접근을 제한하는 방법으로 사용되는 동기화 기법이다.

***synchronized의 경우 오직 하나의 스레드에서만 수행** 가능하다면, [세마포어는 동시에 실행할 수 있는 스레드의 수를 제어할 수 있다.](https://steady-coding.tistory.com/557)

### **wait와 notify사용.**

- wait : 갖고 있던 고유 락을 해제하고, 스레드를 잠들게 하는 거.
- notify : 스레드 중 임의로 하나를 골라 깨우는 거.

*notify는 synchronized 블록이나 메서드에서 호출되어야 하고, 올바르게 사용하기 까다로우니 고수준 동시성 유틸리티를 사용하자.

### wait와 notify를 다루는 경우 In Legacy

새로운 코드라면 항상 동시성 유틸리티를 써야 하지만, 레거시 코드의 경우 어쩔 수 없이 wait와 notify를 다뤄야 하는 상황이 생길 수 있는데,

이 때 wait와 notify 사용 시 주의사항. 

1. wait 메서드
    
    ```jsx
    // wait 메서드를 사용하는 표준 방식
    synchronized(obj){
    		while(조건이 충족되지 않는다){
    				obj.wait(); // 락을 놓고, 꺠어나면 다시 잠근다.
    		}
    	
    		.. // 조건이 충족됐을 때 동작을 수행한다. 
    }
    ```
    
    - wait 메서드를 사용할 때는 반드시 대기 반복문 관용구를 사용하고 반복문 밖에서는 절대로 호출하지 말자.
        - 대기전 조건 검사 : 조건에 만족한다면 wait를 건너뛴다. 이것은 응답 불가 상태를 예방하는 조치이다.  만약 조건이 충족되었는데 스레드가 notify 혹은 notifyAll 메서드를 호출한 후 대기 상태로 빠지면, 스레드를 다시 깨우는 것을 보장할 수 없다.
        - 대기후 조건 검사 : 조건이 만족되지 않았다면 다시 대기하게 한다. 이는 안전 실패를 막는 조치이다. 만약 조건이 충족되지 않았는데 스레드가 동작을 이어가면 Lock 보호하는 불변식이 깨질 위험이 있다.

### notify와 notifyAll

- notify : 스레드 하나만 깨운다.
- notifyAll : 모든 스레드를 깨운다. ( notifyAll을 사용하는게 안전하고 합리적이다. )

**why?**

- notifyAll을 사용하면 깨어나야할 **모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 수 있다.  다른 스레드까지 꺠어날 수 있지만, 프로그램의 정확성에 영향을 주지는 않는다.**
    
    깨어난 스레드들은 조건이 충족되었는지 확인하고, 충족되지 않았다면 다시 대시 상태로 변경될 것이기 때문이다. 
    
    (다만, 모든 스레드가 같은 조건을 기다리는 상황이고 조건이 충족될 때 하나의 스레드만 혜택을 받는 상황이라면 notify로 최적화 할 수 있다.)
    
    그럼에도 불구하고 notifyAll을 사용해야 하는 이유는 관련 없는 스레드가 wait 를 호출하는 공격으로부터 꼭 깨어나야 하는 스레드를 보호할 수 있기 때문이다. 
    
    ( 깨어나야 할 스레드가 notify 를 삼켜버린다면 영원히 대기하게 될 수 있다. )