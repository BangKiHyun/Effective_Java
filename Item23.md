# Item23: 태그 달린 클래스보다는 클래스 계층구조를 활용하라

## 태그 달린 클래스

### 예제 코드

~~~java
public class Figure {
    enum Shape {RECTANGLE, CIRCLE}

    //태그 필드
    final Shape shape;

    //사격형 필드
    double length;
    double width;

    //원 필드
    double radius;

    // 원용 생성자
    public Figure(final Shape shape, final double radius) {
        this.shape = shape;
        this.radius = radius;
    }

    // 사각형용 생성자
    public Figure(final Shape shape, final double length, final double width) {
        this.shape = shape;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
~~~

### 태그 달린 클래스 단점

- 열거 타입 선언(enum), 태그 필드(enum 타입), switch 문 등 **쓸데없는 코드가 많다.**
- 여러 구현이 한 클래스에 혼합돼 있어 **가독성이 나쁘다.**
  - switch문에 여러 구현 혼합
- 필드들은 final로 선언하려면 **쓰이지 않는 필드들까지 생성자에서 초기화해야 한다.**
  - 즉, 불필요한 코드가 늘어난다.
- 또 다른 의미를 추가하려면 코드를 수정해야 한다.

**정리하자면, 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.**

</br >

## 계층구로조 바꾸는 방법

1. 계층구조의 루트(root)가 될 추상 클래스를 정의하자.
   - 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.(area 메서드)
2. 태그와 상관없이 동작이 일정한 메서드들은 루트 클래스에 일반 메서드로 추가한다.
3. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다.
4. 루트 클래스를 확장한 구체 클래스를 정의한다.(Circle, Rectangle)
5. 각 하위 클래스에 각자 의미에 해당하는 데이터 필드들을 넣는다.
   - Circle(radius), Rectangel(length, width)
6. 루트 클래스에 정의한 추상 메서드를 각자의 의미에 맞게 구현한다.

### 계층구조로 바꾼 코드

~~~java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    public Rectangle(final double length, final double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
~~~

위 코드는 태그 달린 클래스의 단점을 모두 날렸다.

- 각 클래스의 생성자가 모든 필드를 초기화

  - 관련 없는 데이터 필드 제거
  - 모두 final 필드

- 독립적으로 계층구조를 확장가능

- 타입 사이의 계층 관계를 반영할 수 있어 유연성 및 컴파일타임 타입 검사를 높여줌

  ~~~java
  class Square extends Rectangle {
  	Square(double side) {
  		super(side, side);
  	}
  }
  ~~~

</br >

## 정리

- 새로운 클래스를 작성하는 태그 필드가 등장한다면 게층구조로 대체하는 방법을 생각해보자.
- 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩터링하는 방법을 고민해보자.