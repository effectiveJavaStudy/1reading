# [아이템 16] public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라.

public 클래스에서는 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공해서 내부 표현 방식을
언제든 바꿀 수 있는 유연성을 얻을 수 있다.

필드를 공개하면 사용하는 클라이언트가 생겨서 마음대로 바꿀수가 없게 된다.

### package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다!

```
public class TopPoint {

    private static class Point {
        public double x;
        public double y;
    }

    public Point getPoint() {
        Point point = new Point();  // TopPoint 외부에선
        point.x = 3.5;              // Point 클래스 내부 조작이
        point.y = 4.5;              // 불가능 하다.
        return point;
    }

}
```

public 클래스에 접근을 해도 private 클래스는 접근할수 없으니 문제가 되지않는다.

클래스의 필드가 불변이라면 단점이 조금은 줄지만 표현 방식을 바꿀수도 없고 부수 작업을 수행할수 없다는 단점이 아직 남아있다.
```
public final class Time {

    public final int hour;
    public final int minute;
```
## 핵심 정리

> public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-p
> rivate나 private 중첩 클래스는 노출하는 편이 나을때도 있다.
