# 변경 가능성을 최소화하라

클래스를 불변으로 만드려면 다섯가지 규칙이 있다.

- 객체의 상태를 변경하는 메서드를 제공하지 않는다.
- 클래스를 확장할 수 없도록 한다.
- 모든 필드를 final로 선언한다.
- 모든 필드를 private로 선언한다.
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.


```java
public final class Complex {
    private final double realNumber;
    private final double imaginaryNumber;

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE = new Complex(1, 0);
    public static final Complex I = new Complex(0, 1);

    public Complex(double realNumber, double imaginaryNumber) {
        this.realNumber = realNumber;
        this.imaginaryNumber = imaginaryNumber;
    }

    public double realPart() {
        return realNumber;
    }

    public double imaginaryPart() {
        return imaginaryNumber;
    }

    public Complex plus(Complex c) {
        return new Complex(realNumber + c.realNumber, imaginaryNumber + c.imaginaryNumber);
    }

    // 코드 17-2 정적 팩터리(private 생성자와 함께 사용해야 한다.) (110-111쪽)
    public static Complex valueOf(double realNumber, double imaginaryNumber) {
        return new Complex(realNumber, imaginaryNumber);
    }

    public Complex minus(Complex c) {
        return new Complex(realNumber - c.realNumber, imaginaryNumber - c.imaginaryNumber);
    }

    public Complex times(Complex c) {
        return new Complex(realNumber * c.realNumber - imaginaryNumber * c.imaginaryNumber,
                realNumber * c.imaginaryNumber + imaginaryNumber * c.realNumber);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.realNumber * c.realNumber + c.imaginaryNumber * c.imaginaryNumber;
        return new Complex((realNumber * c.realNumber + imaginaryNumber * c.imaginaryNumber) / tmp,
                (imaginaryNumber * c.realNumber - realNumber * c.imaginaryNumber) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.realNumber, realNumber) == 0
                && Double.compare(c.imaginaryNumber, imaginaryNumber) == 0;
    }

    @Override
    public int hashCode() {
        return 31 * Double.hashCode(realNumber) + Double.hashCode(imaginaryNumber);
    }

    @Override
    public String toString() {
        return "(" + realNumber + " + " + imaginaryNumber + "i)";
    }
}
```

여기서 사칙연산 메서드는 수정하지 않고 새로운 인스턴스를 만들어 반환한다.
결과는 같지만 피연산자 자체는 그대로인 프로그래밍 패턴을 함수형 프로그래밍이라 한다.

함수형 프로그래밍 방식으로 구성하면 코드에서 불변이 되는 영역의 비율이 높아지는 장점을 누릴 수 있다.

### 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다.

여러 스레드가 동시에 사용해도 절대 훼손되지 않는다. 따라서 안심하고 공유 할 수가 있다.
불변 클래스는 인스턴스를 만들었다면 최대한 재활용해서 사용하기를 권한다.
가장 쉬운 재활용 방법은 상수로 사용 할 수 있다.

```java
public static final Complex ZERO = new Complex(0,0);
public static final Complex ONE = new Complex(1,0);
public static final Complex I = new Complex(0,1);
```

### 불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유 할 수 있다.

### 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.

구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하다.

### 불변 객체는 그 자체로 실패 원자성을 제공한다(아이템 76)
절대 변하지 않으니 잠깐이라도 불일치 상태에 빠질 가능성이 없다.

### 불변 클래스에도 단점이 있다. 값이 다르면 반드시 독립된 객체로 만들어야 한다는 것이다.

만약 백만비트짜리 BigInterger에서 비트 하나를 바꿔야한다고 해보자.

```
Bitinterger moby= ...
moby = moby.flipbit(0) 
```

flipbit 메서드는 새로운 인스턴스를 생성한다. 따라서 크기에 비례해 공간과 시간을 잡아먹는다.
Bitset 클래스는 원하는 비트하나만 상수 시간안에 바꿔주는 메서드를 제공한다.

원하는 객체를 완성하기 까지 단계가 많고 그 중간 단계에서 만들어진 객체들이 모두 버려지면 성능 문제가 있다.

이 문제 해결 방법은 두가지가 있다.

- 흔히 쓰일 다단계 연산들을 예측하여 기본 기능으로 제공하는 방법이다. 이러한 연산을 기본으로 제공한다면 더 이상 각 단계마다 객체를 생성하지 않아도 된다.
- 정확히 예측할 수 있다면 가변 동반 클래스 만으로도 충분하다. 좋은 예시가 String과 StringBuilder이다.

모든 생성자를 private 혹은 package-private로 만들고 public 정적 팩터리를 제공하는 방법이다(아이템 1) 이렇게 하면 더 유연하게 불변 인스턴스를 만들 수 있다.

``` java
public static Complex valueOf(double re ,double im){
    return new Complex(re,im);
    
}
private Complex(double re, double im){
    this.re = re;
    this.im = im;
}
```

생성자가 없으니 다른 패키지에서는 이 클래스를 확장하는게 불가능하다.

만약 클라이언트로부터 BigInterger이나 BigDecimal 인스턴스를 인수로 받는다면 주의해야 한다.
신뢰할수 없는 하위 클래스의 인스턴스라고 확인 되면 , 이 인수들은 가변이라 가정하고 방어적으로 복사해 사용해야 한다. (아이템 50)

어떤 불변 객체는 계산 비용이 큰 값을 나중에 계싼하여 final이 아닌 필드에 캐시 해놓기도 한다.

지연 초기화(아이템 83)의 예이기도 한 이 기법을 String도 사용한다.

무조건 Setter를 만들지는 말자

클래스는 꼭 필요한 경우가 아니면 불변이어야한다.

불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.

모든 필드는 private final이어야 한다.
