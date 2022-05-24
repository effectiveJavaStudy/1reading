# 추상클래스보다는 인터페이스를 우선하라.

기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.

인터페이스가 요구하는 메서드를 추가하기만 하면 된다.

인터페이스는 믹스인 정의에도 사용할수 있다.

믹스인이란 클래스가 구현할 수 있는 타입으로 믹스인을 구현한 클래스으ㅔ 원래의 주된 타입외에도 특정 선택적 행위를 제공하는 효과를 준다.

예를 들어 Comparable은 자신을 구현한 인스턴스 끼리는 순서를 정할 수 있다.

인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

```java

public interface Singer {
    void sing();
}



public interface SongWriter {
    void compose();
}


public interface SingerSongwriter extends Singer, SongWriter {

    void actSensitive();

}

public class People implements SingerSongwriter {

    @Override
    public void actSensitive() {

    }

    @Override
    public void sing() {

    }

    @Override
    public void compose() {

    }

}

```

인터페이스로 정의하면  조합을 쉽게 조합해서 클래스를 만들어 낼 수 있다는 편리함이 있다.

래퍼클래스와 같이 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.

인터페으소아 추상 골격 구현 클래스를 함계 젝ㅇ하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법도 있다.

인터페이스는 타입을 정의하고 필요하면 디폴트 메서드 몇 개도 함께 정의해서 제공한다.
골격 구현 클래스는 나머지 메서드들까지 구현한다.
이렇게 해두면 단순히 골격 구현을 확장하는 것만으로 이 인터페이스를 구현하는 데 필요한 일이 대부분 완료된다.


```java
//추상 골격 구현 클래스 사용 하지 않는 버전.
public interface Character {
  public void move();
  public void seat();
  public void attack();
}

public class Thief implements Character{
  @Override
  public void move() {
    System.out.println("걷다");
  }

  @Override
  public void seat() {
    System.out.println("앉다");
  }

  @Override
  public void attack() {
    System.out.println("표창을 던진다");
  }    
}

public class Wizard implements Character{
  @Override
  public void move() {
    System.out.println("걷다");
  }

  @Override
  public void seat() {
    System.out.println("앉다");
  }

  @Override
  public void attack() {
    System.out.println("마법봉을 휘두르다");
  }
}

public static void main(String[] args) {
  Thief thief = new Thief();
  Wizard wizard = new Wizard();
  thief.process();
  wizard.process();
}//추상 골격 구현 클래스 사용 하지 않는 버전.
public interface Character {
    public void move();
    public void seat();
    public void attack();
}

public class Thief implements Character{
    @Override
    public void move() {
        System.out.println("걷다");
    }

    @Override
    public void seat() {
        System.out.println("앉다");
    }

    @Override
    public void attack() {
        System.out.println("표창을 던진다");
    }
}

public class Wizard implements Character{
    @Override
    public void move() {
        System.out.println("걷다");
    }

    @Override
    public void seat() {
        System.out.println("앉다");
    }

    @Override
    public void attack() {
        System.out.println("마법봉을 휘두르다");
    }
}

    public static void main(String[] args) {
        Thief thief = new Thief();
        Wizard wizard = new Wizard();
        thief.process();
        wizard.process();
    }
```
```java
//추상 골격 구현 클래스 사용하는 버전
public abstract class AbstractCharacter implements Character{
    @Override
    public void move() {
        System.out.println("걷다");
    }

    @Override
    public void seat() {
        System.out.println("앉다");
    }

    @Override
    public void process() {
        move();
        seat();
        attack();
    }
}

public class Thief extends AbstractCharacter implements Character{
    @Override
    public void attack() {
        System.out.println("표창을 던진다");
    }
}

public class Wizard extends AbstractCharacter implements Character{
    @Override
    public void attack() {
        System.out.println("마법봉을 휘두르다");
    }
}

```

골격 구현은 기본적으로 상속해서 사용하므로 아이템 19에서 이야기한 설계 및 문서화 지침을 모두 따라야 한다.

단순 구현은 골격 구현의 작은 변종으로 AbstractMap.SimpleEntry가 좋은 예다. 단순 구현도 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상클래스가 아니다.

단순 구현은 동작하는 가장 단순한 구현으로 그대로 써도 되고 필요에 맞게 확장해도 된다.
