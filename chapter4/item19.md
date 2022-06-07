# 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라.

## 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다.

클래스의 API로 공개된 메서드는 자신의 또 다른 메서드를 호출할 수도 있다. 
만약 재정의 가능한 메서드를 호출 한다면 API 설명에 적시해야 한다.

재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야 한다.

메서듸 설명 끝에서 가끔 "Implementation Requirements"로 시작하는 절ㅇ ㅣ있는데
 그 메서드의 내부 동작을 설명하는 곳이다.


>public boolean remove(Object o) 
> 주어진 원소가 이 컬렉션 안에 있다면 그 인스턴스를 하나 제거한다(선택적 동의). 
> 더 정확하게 말하면, 이 컬렉션 안에 'Object.equals(o, e)가 참인 원소' e가 하나 이상 있다면 그중 하나를 제거한다. 주어진 원소가 컬렉션 안에 있었다면(즉, 호출 결과 이 컬렉션이 변경됐다면) true를 반환한다.
Implementation Requirements: 이 메서드는 컬렉션을 순회하며 주어진 원소를 찾도록 구현되었다. 주어진 원소를 찾으면 반복자의 remove 메서드를 사용해 컬렉션에서 제거한다. 이 컬렉션이 주어진 객체를 갖고 있으나, 이 컬렉션의 iterator 메서드가 반환한 반복자가 remove 메서드를 구현하지 않았다면 UnsupportedOperationException을 던지니 주의하자.

위 예시에서는 iterator 메서드를 재정의하면 remove 메서드의 동작에 영향이 갈수 있다.

좋은 API 문서는 어떻게가 아닌 무엇을 하는지를 설명해야 한다는 원칙과 위배 된다.

클래스를 안전하게 상속할 수 있도록 하려면 내부 구현 방식을 설명해야만 한다.

상속을 잘 하려면 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(Hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.

java.util.AbstractList의 removeRange메서드를 예로 들 수 있다.

이 메서드는 하위 클래스에서 부분리스트의 clear 메서드를 고성능으로 만들기 쉽게 하기 위해서 정의되었다.

상속용 클래스를 시험하는 방법은 직접 클래스를 만들어보는 방법밖에 없다.

상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다.

상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 호출되기 때문이다.

```java
public class Super {
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
        System.out.println("super method");
    }
}
```

```java
public class Super {
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
        System.out.println("super method");
    }
}
```
결과는
처음에 null이 나온다.
상위 클래스의 생성자가 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 overrideme를 호출하기 때문이다.

Cloneable과 Serializable 인터페이스를 상속한 클래스를 상속할수 있게 설계하는거는 일반적으로 좋지않은 생각이다.

clone과 readObject 메서드는 생성자와 비슷한 효과를 낸다( 새로운 객체 생성)

clone과 readObject 모두 재정의 가능 메서드를 호출해서는 안된다.

일반적인 구체 클래스는 변화가 생길 때마다 하위 클래스를 오작동 시킬수도 있다.

이 문제를 해결하는 가장 좋은 방법은 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이다.

상속을 금지하는 방법은 두 가지다.
1. 클래스를 final로 선언하는 방법
2. 모든 생성자를 private 이나 package-private로 선언하고 정적 팩터리를 만들어주는 방법이다.
3. 

