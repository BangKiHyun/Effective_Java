## Item16: public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

~~~
public class Point {
    public double x;
    public double y;
}
~~~

위와 같은 클래스는 이를 사용하는 클라이언트가 생겨 내부 표현 방식을 마음대로 바꿀수 없으며, API를 수정하지 않고는 내부 표현을 바꿀 수 없고, 불변식을 보장할 수 없다. 또한 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.



### 해결 방법

public 필드를 private으로 바꾸고 public 접근자(getter)를 추가하면 된다.

~~~
public class Point {
    private double x;
    private double y;

    public Point(final double x, final double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }
}
~~~

패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있다.

하지만 **package-private 클래스 혹은 private 중첩 클래스라면 public 필드를 사용해도 무방**하다. 이 방식은 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서 접근자 방식보다 깔끔하다.

- package-private 클래스는 패키지 바깥 코드는 전혀 손대지 않고 데이터 표현 방식을 바꿀 수 있다.
- private 중첩 클래스는 수정 범위가 더 좁아져서 이 클래스를 포함하는 외부 클래스까지 제한할 수 있다.

</br >

### public final 필드

pulbic 클래스의 필드가 불변이라면 직접 노출되더라도 불변식을 보장할 수 있다.
그러나 API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점이 여전하다.

</br >

### 정리

- public 클래스는 절대 가변 필드를 직접 노출하지 말자.
- 불변 필드라면 노출해도 덜 위험하지만 안심할 수 없다.
- package-private 클래스나 private 중첩 클래스는 종종 필드를 노출하는 편이 나을 수 있다.(?)

