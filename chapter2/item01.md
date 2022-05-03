# 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라

## 1. 개요

- 클래스의 인스턴스를 얻는 전통적인 수단은 2가지가 있다.

  1. public 생성자

     ```java
     public Node(){}
     ```

  2. 정적팩터리 메서드

     ```java
     public static Node newInstance() {
     	return instance();
     }
     ```

## 2. 장점

### 1. 이름을 가질 수 있다.

- 생성자의 매개변수 이름만으로는 반환 객체의 특성을 제대로 묘사할 수 없지만 정적 팩토리 메소드는 이름만 잘 지으면 객체의 특성을 쉽게 표현할 수 있다.

  ```java
  // 생성자
  public BigInteger(int, int, Random)
  
  // 정적 팩토리 메소드
  public static BigInteger probablePrime(int, int, Random)
  ```

### 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

- 불변 클래스는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.

  - 인스턴스 캐싱

    1. 캐싱 : 캐시(값을 미리 복사해 놓는 임시 장소)라는 작업을 하는 행위
    2. 불필요한 객체 생성을 피할 수 있어서 메모리 낭비를 막을 수 있다.

    - 참고 - https://velog.io/@lxxjn0/반복적으로-사용되는-인스턴스-캐싱하기

  - 인스턴스 통제

    - 통제 이유
      1. 클래스를 싱글톤으로 만들 수 있다.
      2. 인스턴스화 불가로 만들 수 있다.
      3. 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다.
      4. 플라이웨이트 패턴의 근간이 되며, 열거 타음은 인스턴스가 하나만 만들어짐을 보장한다.
         - 플라이웨이트 패턴
           - 데이터를 공유하여 메모리를 절약하는 패턴, 공통으로 사용되는 객체는 한번만 사용되고 Pool에 의해서 관리, 사용된다. (JVM의 String Pool에서 같은 String이 잇는지 먼저 찾는다. [불변객체 String])

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

- 반환 객체의 클래스를 자유롭게 선택할 수 있게 하는 엄청난 유연성이 있다.

- 구현 클래스를  공개하지 않고도 객체를 반환할 수 있다.

  ```java
  public interface Type {
      static Type getAType() {
          return new AType();
      }
  
      static Type getBType() {
          return new BType();
      }
  }
  
  Type.getAType()
  ```

- 자바 8부터 인터페이스에 정적 메소드를 사용할 수 있다.

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

- 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.

  ```java
  public abstract class EnumSet<E extends Enum<E>> extends AbstractSet<E>
      implements Cloneable, java.io.Serializable
  {
      public static <E extends Enum<E>> EnumSet<E> allOf(Class<E> elementType) {
          EnumSet<E> result = noneOf(elementType);
          result.addAll();
          return result;
      }
  
      public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
          Enum<?>[] universe = getUniverse(elementType);
          if (universe == null)
              throw new ClassCastException(elementType + " not an enum");
  
          if (universe.length <= 64)
              return new RegularEnumSet<>(elementType, universe);
          else
              return new JumboEnumSet<>(elementType, universe);
      }
  
      public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2) {
          EnumSet<E> result = noneOf(e1.getDeclaringClass());
          result.add(e1);
          result.add(e2);
          return result;
      }
  }
  ```

  - public 생성자 없이 오직 정적 팩터리만 제공

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

- 이러한 유연함은 서비스 제공자 프레임워크를 만드는 근간이 된다.

- 대표적인 서비스 제공자 프레임워크로는 JDBC가 있다.

- 서비스 제공자 프레임워크의 컴포넌트

  1. 서비스 인터페이스

     - 구현체의 동작 정의
     - ex. Connection

  2. 제공자 등록 API

     - 제공자가 구현체를 등록할 때 사용
     - ex. DriverManager.registerDriver

  3. 서비스 접근 API

     - 클라이언트가 서비스의 인스턴스를 얻을 때 사용

     - ex. DriverManger.getConnection

     - 변형 → 더 풍부한 서비스 인터페이스를 클라이언트에 반환 가능(브리지 패턴)

     - 브리지 패턴

       구현부에서 추상층을 분리하여 각자 독립적으로 변형이 가능하고 확장이 가능하도록 합니다. 즉 기능과 구현에 대해서 두 개를 별도의 클래스로 구현을 합니다.

  4. 서비스 제공자 인터페이스

     - 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체 설명
     - ex. Driver

## 2. 단점

### 1. 상속을 하려면 public이나 protected생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

- 컬렉션 프레임워크의 유틸리티 구현 클래스들은 상속할 수 없다.(접근제어자가 private이기 때문)

### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

- API Docs 등을 볼 때 생성자는 바로 찾을 수 있지만 정적 팩토리 메서드는 다른 메서드와 구분 없이 함께 보여진다.
- 명명 방식들
  1. from : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
  2. of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
  3. valueOf : from과 of의 더 자세한 버전
  4. instance / getInstance : 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않음
  5. create / newInstance : 매번 새로운 인스턴스를 생성해 반환함을 보장
  6. getType : 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용
  7. newType : 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용
  8. type : getType과 newType의 간결한 버전