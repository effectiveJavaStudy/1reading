# 아이템3  - private 생성자나 여거 타입으로 싱글턴임을 보증하라


## 주요 용어 정리

- 인스턴스 : 생성된 객체, **어떤 클래스에 속하는 각각의 객체**
객체를 생성하여 JVM(자바 가상 머신)이 관리하는 메모리에 적재된 것을 의미하고,   
어떤 클래스로부터 만들어진 객체를 그 클래스의 인스턴스라 한다.

    instance ins = new instance(); //인스턴스 만드는 과정

- 인스턴스화 : 클래스로부터 객체를 만드는 과정

- 붕어빵에 비유하자면
붕어빵 틀 = Class  
붕어빵 = 객체(Object)  
붕어빵을 굽다 = 인스턴스(Instance)화 하다
만들어진 각각의 붕어빵 = 인스턴스


## 개요

singleton이란 인스턴스를 오직 하나만 생성할 수 있는 클래스이다.
인터페이스를 구현해서 만든 싱글턴이 아니라면 mock 구현으로 대체 할수 없기 때문에 **클라이언트를 테스트하기가 어려워질 수 있다.**
> **Mock 객체란?**
> 
> 실제 객체를 다양한 조건으로 인해 제대로 구현하기 어려울 경우 **가짜 객체를 만들어 사용**하는데, 이를 Mock 객체라 한다.
> 
> **Mock 객체가 필요한 경우.**
> -   테스트 작성을 위한 **환경 구축이 어려운 경우.**
> -   테스트가 특정 경우나 **순간에 의존적인 경우.**
> -   **시간이 걸리는 경우**

싱글턴임을 보증하는 방법은 크게 3가지가 있다.

## public static final 필드 방식의 싱글턴

private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화 할 때 딱 한번만 호출된다.
```
public class Elvis {  
    public static final Elvis INSTANCE = new Elvis();  
  
 private Elvis() { }  
  
    public void leaveTheBuilding() {  
        System.out.println("Whoa baby, I'm outta here!");  
  }  
  
    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!  
  public static void main(String[] args) {  
        Elvis elvis = Elvis.INSTANCE;  
  elvis.leaveTheBuilding();  
  }  
}
```

### 장점
- 해당 클래스가 싱글턴임이 API에 명백히 보여진다.
- 코드의 간결함


##  정적 팩터리 방식의 싱글턴
정적 팩터리 메서드를 public static 멤버로 제공한다.
```
public class Elvis {  
    private static final Elvis INSTANCE = new Elvis();  
 private Elvis() { }  
    public static Elvis getInstance() { return INSTANCE; }  
  
    public void leaveTheBuilding() {  
        System.out.println("Whoa baby, I'm outta here!");  
  }  
  
    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!  
  public static void main(String[] args) {  
        Elvis elvis = Elvis.getInstance();  
  elvis.leaveTheBuilding();  
  }  
}
```

### 장점
- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
- 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.(아이템 30)
- 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다는 점이다.(아이템 43,44)

## 리플렉션 공격 방어

위 방법들은 인스턴스가 전체 시스템에서 하나뿐임을 보장하는데, 권한이 있는 클라이언트가 있는 경우에 리플렉션 API를 통해 private 생성자를 호출할 수 있다. 이를 방어하기 위한 코드는 아래와 같다.

```
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { 
          if( INSTANCE != null) {
            throw new RuntimeException("Can't create Constructor");
        }    
          //... 
    }
}
```


## 싱글턴 클래스 직렬화

싱글턴 클래스를 직렬화 하려면 단순히 Serializable을 구현한다고 선언하는 것만으로는 부족하다. 모든 인스턴스 필드를 일시적(transient)이라고 선언하고, readResolve 매서드를 제공해야한다. 이렇게 하지 않으면, 역직렬화시 새로운 인스턴스가 만들어진다. 아래코드는 readResolve 예시이다.

```
public class Elvis implements Serializable{
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

      public static Elvis getInstance() { return INSTANCE; }

      // singleton임을 보장
      private Object readResolve() {
          // 역직렬화가 되어 새로운 인스턴스가 생성되더라도 INSTANCE를 반환하여 싱글턴 보장
           // 새로운 인스턴스는 GC에 의해 UnReachable 형태로 판별되어 제거
          return INSTANCE;
    }
}
```


## 열거 타입 방식의 싱글턴

원소가 하나인 열거타입을 선언하는 것이다.

```
public enum Elvis {  
    INSTANCE;  
  
 public void leaveTheBuilding() {  
        System.out.println("지금 나갈께!");  
  }  
  
    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!  
  public static void main(String[] args) {  
        Elvis elvis = Elvis.INSTANCE;  
  elvis.leaveTheBuilding();  
  }  
}
```

### 장점
- public 필드 방식보다 더 간결하고, 추가 노력 없이 직렬화 할 수 있다.
- 아주 복잡한 직렬화 상황, 리플렉션 공격에도 제 2의 인스턴스가 생기는 일을 완벽히 막아준다.



**대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.**
단, 만들려는 싱글턴이 Enum외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.

## 출처
[자바 기초(클래스, 객체, 인스턴스, 인스턴스화)](https://blog.naver.com/hhw1990)
[dahye-jeong.gitbook](https://dahye-jeong.gitbook.io/java/java/effective_java/2021-01-14-singleton)








