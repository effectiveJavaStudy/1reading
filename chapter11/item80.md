[Java에서 외부 프로세스를 실행할 때](https://d2.naver.com/helloworld/1113548) 

[How to execute shell command from Java](https://mkyong.com/java/how-to-execute-shell-command-from-java/)

[[자바] Executor 프레임워크](https://12bme.tistory.com/359)

[Executor 프레임워크 예제](https://xzio.tistory.com/1131)

[newFixedTheadPool 은 어떻게 동작하는가?](https://hamait.tistory.com/937)

[FixedTheadPool 예제](https://codechacha.com/ko/fixed-size-thread-pool-executor/)

[newCachedThreadPool( )    vs  newFixedThreadPool( )](https://www.baeldung.com/java-executors-cached-fixed-threadpool)

[스레드풀을 사용하는 이유, 종료 및 작업 처리](https://cornswrold.tistory.com/197)

[ExecutorService 와 ThreadPoolExecutor에 대해](https://bepoz-study-diary.tistory.com/392) 

[자바 병령 프로그래밍 volatile과 synchronized](https://wiserloner.tistory.com/548) 

---

- **Executors framework 가 하는 일 3가지**
    
    > Executor Framework는 Java Application 에서 Thread를 생성하고 관리하기 쉽게 다양한 method 와 interface를 제공합니다.
    > 
    1. Thread 생성 **: thread를 생성하거나 thread pool을 만드는 mehod를 제공한다.** 
    2. Thread 관리 : **thread의 생명주기를 관리합니다. t**hread pool이 활성화되어 있는지 죽은 상태인지에 대해 고려하지 않아도 되게끔 threa를 관리합니다.
    3. Task 제출 및 실행 : Runnable 혹은 Callable method를 제출하고 그것을 원하는 때에 실행할 수 있게 해준다. 

### 실행자 프레임워크

실행자 프레임워크는 java.util.concurrent 패키지에 속해 있으며 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다. 

(java.util.concurrent 패키지를 사용하여 스레드 풀을 생성할 수 있다.)

과거에는 단순한 작업 큐(work queue)를 만들기 위해서 수많은 코드를 작성해야 했는데, 이제는 아래와 같이 간단하게 작업 큐를 생성할 수 있다. 

```jsx
// 큐를 생성한다. 
ExecutorService exec = Executors.newSingleThreadExecutor();

// 태스크 실행
exec.execute(runnable);

// 실행자 종료
exec.shutdown();
```

> 큐를 둘 이상의 스레드가 처리하게 하고 싶다면 간단히 다른 정적 팩터리를 이용하여 다른 종류의 실행자 서비스(스레드 풀)를 생성하면 된다.
> 

### 실행자 프레임워크의 기능

- submit().get() : get 메서드를 이용해 특정 태스크가 완료되기를 기다림.
- invokeAny() / invokeAll : 태스크 모음중에서 하나가 완료되는 것을 기다림 / 태스크 중에서 모든 태스크가 완료되는 것을 기다림.
- awaitTermination : 실행자 서비스가 종료하기를 기다림.
- ExecutorCompletionService : 완료된 태스크들의 결과를 차례로 받음.
- ScheduledTheadPoolExecutor : 태스크를 특정 시간 혹은 주기적으로 실행하게 한다.

### 실행자 서비스 생성

> Executor 클래스
ExecutorService 인터페이스
> 

Executors는 ExecutorService를 생성하여 다음 메소드를 제공하여 스레드 풀 개수 및 종류를 정할 수 있다. 

- **newFixed TheadPool(int)** : 인수 개수만큼 고정된 스레드 풀을 만든다.
- **newCachedTheadPool**() : 필요할 때 필요한 만큼 스레드 풀을 생성. 이미 생성된 스레드를 재활용 할 수 있어서 성능이 용이하다.
- **newScheduledTheadPool(int)** : 일정 시간 뒤에 실행되는 작업이나, 주기적으로 수행되는 작업이 있다면 스케줄스레드 풀을 활용할 수 있다.
- **newSIngleTheadExecutor()** : 스레드 1개인 ExecutorService를 리턴한다. 싱글 스레드에서 동작하는 작업 처리시 사용한다.

### 실행자 서비스를 사용하면 안되는 경우

가벼운 애플리케이션 서버의 경우 Executors.**newCachedTheadPool**을 사용하는 것이 적합하지만 **무거운 프로덕션 서버에서는 사용하는 것은 권장하지 않는다.** 

( CachedTheadPool 에서는 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행함. ) 

무거운 프로덕션 서버에서는 스레드 개수를 고정한 Executors.**newFixedTheadPool**을 선택하거나 완전히 통제할 수 있는 TheadPoolExecutor를 직접사용하는 편이 좋다. 

### 작업 생성과 처리 요청.

**1. 작성 생성 ( 추상 태스크)**

> 작업의 단위를 나타내는 핵**심 추상 개념은 태스크이다.**
> 

이 태스크는 Runnable과 Callable로 나눌 수 있다. ( 두 클래스의 차이는 작업 처리 완료 후 리턴값 유무이다. ) 

- Runnable
    
    ```jsx
    Runnable task = new Runnable(){
    		@Override
    		public void run(){
    			// 작업 내용
    		}
    }
    ```
    
- Callable
    
    ```jsx
    Callable<T> task = new Callable<T>(){
    		@Override
    		public T call() throws Exception{
    			// 작업 내용
    			return T; 
    		}
    }
    ```
    

1. **작업처리 요청**

> 작업 처리 요청이란 ExecutorService의 작업 큐에 Runnable 또는 Callable 객체를 넣는 행위이다.
> 
- execute() : 작업 처리 도중 예외가 발생하면 스레드가 종료되고 스레드 풀에서 제거된다.
- submit() : 작업 처리 도중 예외가 발생하여도 종료되지 않고 재사용된다. ( 따라서 오버헤드를 줄이기 위해서는 submit()을 사용하는 것이 좋다. )

*실행자 프레임워크를 좀 더 알고 싶다면 자바 병렬 프로그래밍.