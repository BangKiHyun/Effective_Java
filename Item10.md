## Item10: equals는 일반 규약을 지켜 재정의하라

Object는 객체를 만들 수 있는 구체 클래스지만 기본적으로 상속해서 사용하도록 설계되었다.
Objcet에 final이 아닌 메서드(equals, hashcode, toString, clone, finalize)는 모두 재정의를 염두에 두고 설계된 것이다. 그러므로 Object를 상속하는 클래스는 이 메서들을 규약에 맞게 재정의해야 한다. 그러지 않을시 HashMap과 HashSet등을 오동작하게 만들 수 있다.

### equals 메서드

equals 메서드는 그냥 두면 클래스의 인스턴스는 오직 자기 자신과만 같게 된다.
다음과 같은 상황 중 하나에 해당된다면 **재정의 하지 않는게 최선**이다.

- 각 인스턴스가 본질적으로 고유하다. 값을 표현하는게 아니라 동작하는 것을 표현하는 클래스가 여기에 해당된다.(ex: Thread)
- 인스턴스의 논리적 동치성을 검사할 일이 없다.
  예로 java.util.regex.Pattern은 equals를 재정의해 두 Pattern이 같은지 재정의하지 않는다.
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 사용할 수 있다.
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.
- 값 클래스 여도, 같은 인스턴스가 둘 이상 만들어지지 않는다는 보장이 있다면 재정의하지 않아도 된다.
  예로 싱글턴, Enum이 있다.



### equals 메서드의 동치관계 만족

- 반사성: null아닌 아닌 모든 참조값 x에 대해, x.equals(x)는 true다.
- 대칭성: null이 아닌 모든 참조값 x,y에 대해 x.equals(y)가 true이면 y.eqauls(x)도 true
- 추이성: null이 아닌 모든 참조값 x,y,z 에 대해 x.equals(y)가 true이고, y.equals(z)도 true면, x.equals(z)도 true다
- 일관성: null이 아닌 모든 참조값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님: null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false다.

아래는 대칭성을 위반한 코드다.

~~~
public final class CaseInsensitiveString {
    private final String text;

    public CaseInsensitiveString(String text) {
        this.text = Objects.requireNonNull(text);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return text.equalsIgnoreCase(((CaseInsensitiveString) o).text);
        } else if (o instanceof String) {
            return text.equalsIgnoreCase((String) o);
        }
        return false;
    }

    public static void main(String[] args) {
        CaseInsensitiveString BANG = new CaseInsensitiveString("BANG");
        String bang = "bang";

        System.out.println(BANG.equals(bang)); //true
        System.out.println(bang.equals(BANG)); //false
        
        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(BANG);
        System.out.println(list.contains(bang)); //false
    }
}
~~~

CaseInsensitiveString의 equals는 일반 String을 알고 있지만 String의 equals는 CaseInsensitiveString의 존재를 모른다.

아래는 추이성을 위반한 코드다.

~~~
public class Point {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}

class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        } else if (!(o instanceof ColorPoint)) {
            return o.equals(this);
        }
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}

ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
System.out.println(p1.equals(p2)); //true
System.out.println(p2.equals(p3)); //true
System.out.println(p1.equals(p3)); //false

~~~

p1과 p2, p2와 p3가 같지만 p1과 p3는 다르다.



### null-아님 검사

~~~
@Oberride
public boolean equals(Object o){
	if(o == null){
		return false;
	}
}
~~~

위와 같이 명시적인 null검사 보다는 instanceof를 사용한 묵시적 null검사를 사용하자

~~~
@Oberride
public boolean equals(Object o){
	if(!(o instanceof MyType)){
		return false;
	}
	MyType mt = (MyType) o;
	...
}
~~~



### 좋은 equals 메서드 구현 방법

- == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
- instanceof 연산자로 입력이 올바른 타입인지 확인한다.
- 입력을 올바른 타입으로 형변환한다.
- 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
- 최상의 성능을 위해 다를 가능성이 크거나, 비교하는 비용이 싼 필드를 먼저 비교하라.

### 전형적인 equals 메서드의 예

~~~
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber (int areaCode, int prefix, int lineNum){
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNum = rangeCheck(lineNum, 999, "가입자번호");
    }

    private static short rangeCheck(int val, int max, String arg){
        if(val < 0 || val > max){
            throw new IllegalArgumentException(arg + ":" + val);
        }
        return (short) val;
    }

    @Override
    public boolean equals(Object o){
        if (o == this){
            return true;
        }
        if (!(o instanceof PhoneNumber)){
            return false;
        }
        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode = areaCode;
    }
}

~~~



### 주의 사항

- equals를 재정의할 때 hashCode도 반드시 재정의하자.
- 필드들의 동치성만 검사해도 equals 규약을 지킬 수 있다.
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
  (ex: public boolean equals(MyClass o))

