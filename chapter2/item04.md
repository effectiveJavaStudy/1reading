# 아이템4  - 인스턴스화를 막으려거든 private 생성자를 사용하라

## 개요

이따금 단순히 정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때가 있다. 객체 지향적 관점으로는 맞지 않은 방식이지만 분명 나름의 쓰임새가 있다. 예를 들어, java.lang.Math 나, java.util.Arrays 가 있다.

## Private Constructor

Class를 구현할 때 생성자가 없으면 자동으로 매개변수를 받지않는 public 생성자를 만든다. 이때 클라이언트는 생성자가 자동으로 생성된지를 알 수 없기 때문에 의도치 않게 인스턴스화 할 수 있다.

```
// Util이지만 생성자가 없음
public class ImageUtility {
    private static String IMAGE_DATE_FORMAT = "yyyyMMddHHmm";

    public static String makeImageFileNm(String imgFileNm) {
        return imgFileNm + "_" + new SimpleDateFormat(IMAGE_DATE_FORMAT).format(new Date());
    }
}
```

```
// 개발자의 의도이나,
ImageUtility.makeImageFileNm("test", ".png");

// 이를 모르는 클라이언트는 아래와 같이 사용할 수 있다. 
ImageUtility imageUtility = new ImageUtility();
String imageFileNm = imageUtility.makeImageFileNm("test", ".png");
```

또한, 추상 클래로 만드는 것으로 인스턴스화를 막을 수 없다. 아래 코드를 보면 알 수 있다.

```
// Util이지만 생성자가 없음
abstract class ImageUtility {
    private static String IMAGE_DATE_FORMAT = "yyyyMMddHHmm";

    public static String makeImageFileNm(String imgFileNm) {
        return imgFileNm + "_" + new SimpleDateFormat(IMAGE_DATE_FORMAT).format(new Date());
    }
}
```

```
public class ItemImageUtility extends ImageUtility {
  // ...
}
```

```
// 추상클래스이기 때문에 생성자 생성 불가
// ImageUtility imageUtility = new ImageUtility();

// 상속받은 클래스에서 생성자 호출 가능
ItemImageUtility itemImageUtility = new ItemImageUtility();
```

하지만 다행히도 인스턴스화를 막는 방법은 아주 간단하다. **private 생성자를 추가하면 된다.**
명시적 생성자가 private이니 클래스 바깥에서는 접근 할 수 없다. 꼭 Assertion Error를 던질 필요는 없지만 실수를 방지할 수 있다.

```
public class ImageUtility {
    // 기본 생성자가 만들어지는 것을 방어(인스턴스화 방지용)
    private ImageUtility(){
        throw new AssertionError();
    }
}
```

이 방식은 상속을 불가능하게 하는 효과도 있다. 모든 생성자는 상위 클래스의 생성자를 호출하게 되는데, 이를 private으로 선언했으니 하위 클래스가 상위 클래스의 생성자에 접근할 길이 막힌다.


## 참고

### java.util.Arrays

```
public class Arrays {

    /**
     * The minimum array length below which a parallel sorting
     * algorithm will not further partition the sorting task. Using
     * smaller sizes typically results in memory contention across
     * tasks that makes parallel speedups unlikely.
     */
    private static final int MIN_ARRAY_SORT_GRAN = 1 << 13;

    // Suppresses default constructor, ensuring non-instantiability.
    private Arrays() {}

    /**
     * A comparator that implements the natural ordering of a group of
     * mutually comparable elements. May be used when a supplied
     * comparator is null. To simplify code-sharing within underlying
     * implementations, the compare method only declares type Object
     * for its second argument.
     *
     * Arrays class implementor's note: It is an empirical matter
     * whether ComparableTimSort offers any performance benefit over
     * TimSort used with this comparator.  If not, you are better off
     * deleting or bypassing ComparableTimSort.  There is currently no
     * empirical case for separating them for parallel sorting, so all
     * public Object parallelSort methods use the same comparator
     * based implementation.
     */
    static final class NaturalOrder implements Comparator<Object> {
        @SuppressWarnings("unchecked")
        public int compare(Object first, Object second) {
            return ((Comparable<Object>)first).compareTo(second);
        }
        static final NaturalOrder INSTANCE = new NaturalOrder();
    }
   ...
```


### java.util.Math

```
public final class Math {

    /**
     * Don't let anyone instantiate this class.
     */
    private Math() {}

    /**
     * The {@code double} value that is closer than any other to
     * <i>e</i>, the base of the natural logarithms.
     */
    public static final double E = 2.7182818284590452354;

    /**
     * The {@code double} value that is closer than any other to
     * <i>pi</i>, the ratio of the circumference of a circle to its
     * diameter.
     */
    public static final double PI = 3.14159265358979323846;

    /**
     * Returns the trigonometric sine of an angle.  Special cases:
     * <ul><li>If the argument is NaN or an infinity, then the
     * result is NaN.
     * <li>If the argument is zero, then the result is a zero with the
     * same sign as the argument.</ul>
     *
     * <p>The computed result must be within 1 ulp of the exact result.
     * Results must be semi-monotonic.
     *
     * @param   a   an angle, in radians.
     * @return  the sine of the argument.
     */
    public static double sin(double a) {
        return StrictMath.sin(a); // default impl. delegates to StrictMath
    }
```

## 결론

**Private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.**

## 출처
https://dahye-jeong.gitbook.io/java/java/effective_java/2021-01-16-private-constructor
혀래이의 개발이야기 - https://h-coding.tistory.com/36







