- `**ITEM 78` 공유 중인 가변 데이터는 동기화를 사용하라.**
    
    [가변 객체와 불변 객체](https://steady-coding.tistory.com/559)?
    
    - 가변 객체
        
        가변 객체는 Java에서 Class의 인스턴스가 생성된 이후에 내부 상태 변경 가능한 객체이다. 
        
        - 가변 객체는 멀티 스레드 환경에서 별도의 동기화 처리가 필요하다.
        
        ex) ArrayList, Hash, Map, StringBuilder, StringBuffer 
        
    - 불변 객체
        
        불변 객체는 가변 객체와 반대로 Java에서 Class의 인스턴스가 생성된 이후에 내부 상태를 변경할 수 없는 객체이다. 
        
        - 멀티 스레드 환경에서도 안전하게 사용할 수 있다.
        
        ex) String
        
    
    ---
    
    ### 동기화
    
    - 동기화는 배타적 실행 뿐만 아니라 한 스레드씩 수행하도록 보장.
    - 프로그램의 일관성 제공.
    - 스레드간 통신 역할.
    
    **스레드를 멈추기 위해 사용하는 방법.**
    
    - Thread.stop 사용금지
    
    ```java
    package effectivejava.chapter11.item78.brokenstopthread;
    import java.util.concurrent.*;
    
    // 코드 78-1 잘못된 코드 - 이 프로그램은 얼마나 오래 실행될까? (415쪽)
    public class StopThread {
        private static boolean stopRequested;
    
        public static void main(String[] args)
                throws InterruptedException {
            Thread backgroundThread = new Thread(() -> {
                int i = 0;
                while (!stopRequested)
                    i++;
            });
            backgroundThread.start();
    
            TimeUnit.SECONDS.sleep(1);
            stopRequested = true;
        }
    }
    ```
    
    - 다른 스레드를 멈출 때는 boolean 값을 이용하자.
        - boolean 사용할 떄는 그냥하면 최적화 되어 동작하지 않을 수도 있다.
    
    ```java
    package effectivejava.chapter11.item78.fixedstopthread1;
    import java.util.concurrent.*;
    
    // 코드 78-2 적절히 동기화해 스레드가 정상 종료한다. (416쪽)
    public class StopThread {
        private static boolean stopRequested;
    
        private static synchronized void requestStop() {
            stopRequested = true;
        }
    
        private static synchronized boolean stopRequested() {
            return stopRequested;
        }
    
        public static void main(String[] args)
                throws InterruptedException {
            Thread backgroundThread = new Thread(() -> {
                int i = 0;
                while (!stopRequested())
                    i++;
            });
            backgroundThread.start();
    
            TimeUnit.SECONDS.sleep(1);
            requestStop();
        }
    }
    ```
    
    **volatile 필드 사용** 
    
    ```java
    package effectivejava.chapter11.item78.fixedstopthread2;
    import java.util.concurrent.*;
    
    // 코드 78-3 volatile 필드를 사용해 스레드가 정상 종료한다. (417쪽)
    public class StopThread {
        private static volatile boolean stopRequested;
    
        public static void main(String[] args)
                throws InterruptedException {
            Thread backgroundThread = new Thread(() -> {
                int i = 0;
                while (!stopRequested)
                    i++;
            });
            backgroundThread.start();
    
            TimeUnit.SECONDS.sleep(1);
            stopRequested = true;
        }
    }
    ```
    
    - **동기화가 필요한 이유?**
        - 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.
        - 동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤이나 보게 될지 알 수 없다.
        - 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.
        
    - **동기화와 스레드**
        - 한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화 해도 된다. 그러면 그 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭게 값을 읽어갈 수 있다.
        
    - 클래스 초기화 과정에서 객체를 정적 필드, volatile 필드, final 필드, 혹은 보통의 락을 통해 접근하는 필들에 저장해도 된다.
    
    <aside>
    💡 여러 쓰레드에서 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화 해야 한다. 
    
    동기화하지 않으면 한 스데르다 수행한 변경을 다른 스레드에서 보지 못할 수 도 있다. 
    
    공유되는 가변 데이터를 동기화하는데 실패하면 응답 불가 상태에 빠지거나 안전 실패로 이어질 수 있다. 이는 디버깅 난이도가 가장 높은 문제에 속한다.
    
    </aside>
    
    ### 정리
    
    - 가변 데이터를 공유하지 않는다.
    - 가변 데이터는 단일 스레들에서만 쓰도록 하자.
    
    > 여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화 해야 한다.
    >