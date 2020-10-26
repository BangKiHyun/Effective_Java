# Item38: 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

## 서론

대부분 상황에서 열거 타입을 확장하는건 좋지 않은 생각이다.

- 확장한 타입의 원소는 기반 타입의 원소로 취급하지만 그 반대는 성립하지 않으면 이상하다.
- 기반 타입과 확장된 타입들의 원소 모두를 순회할 방법도 마땅치 않다.
- 화장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 더 복잡해진다.

</br >

## 확장할 수 있는 열거 타입에 어울리는 쓰임

확장할 수 있는 열거 타입에 어울리는 쓰임이 하나 존재한다. 바로 연산 코드다. 연산 코드의 각 원소는 특정 기계가 수행하는 연산을 뜻한다.

### 인터페이스를 이용한 확장 가능 열거 타입

확장 가능한 열거 타입을 사용하려면 열거 타입이 임의의 인터페이스를 구현할 수 있다는 사실을 이용하면 된다. 연산 코드용 인터페이스를 정의하고 열거 타입이 이 인터페이스를 구현하게 하면 된다.

열거 타입이 이 인터페이스를 구현하게 하면 된다.

~~~java
public interface Operation {
    double apply(double x, double y);
}
~~~

~~~java
public enum BasicOperation implements Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}

~~~

열거 타입인 `BasicOperation`은 확장할 수 없지만 인터페이스인 `Operation`은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다.

이렇게 하면 `Operation`을 구현한 또 다른 열거 타입을 정의해 기본 타입인 `BasciOperation`을 대체할 수 있다.

~~~java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        @Override
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAIDER("%") {
        @Override
        public double apply(double x, double y) {
            return x % y;
        }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}

~~~

위와 같이 앞의 연산 타입을 확장해 지수 연산(EXP)과 나머지 연산(REMAINDER)을 추가했다.

- 새로 작성한 연산은 기존 연산을 쓰던 곳이면 어디든 쓸 수 있다.
- 열거 타입에 따로 추상 메서드로 선언하지 않아도 된다.

</br >

## 한정적 타입 토큰을 사용한 테스트 방법

~~~java
public class Main {

    public static void main(String[] args) {
        double x = 4;
        double y = 2;
        test(ExtendedOperation.class, x, y);
    }

    private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType,
                                                             double x, double y) {
        for (Operation op : opEnumType.getEnumConstants()) {
            System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
        }
    }
}
~~~

- `ExtendedOperation`의 `class 리터럴`을 넘겨 확장된 연산들이 무엇인지 알려준다. `class 리터럴`은 한정적 타입 토큰 역할을 한다.
- `<T extends Enum<T> & Operation>`: `Class` 객체가 열거 타입인 동시에 `Operation`의 하위 타입이여야 한다.

</br >

## 한정적 와일드카드 타입을 사용한 테스트 방법

~~~java
import java.util.Arrays;
import java.util.Collection;

public class Main {

    public static void main(String[] args) {
        double x = 4;
        double y = 2;
        text(Arrays.asList(ExtendedOperation.values()), x, y);
    }

    private static void text(Collection<? extends Operation> opSet,
                              double x, double y) {
        for (Operation op : opSet) {
            System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
        }
    }
}

~~~

`Collection<? extends Operation>` 한정적 와일드카드 타입을 넘겼다.

여러 구현 타입의 연산을 조합해 호출할 수 있게 되어 좀 더 유연해 졌다.

</br >

### 문제점

인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식에 한 가지 문제점이 있다. **열거 타입이끼리 구현을 상속할 수 없다.**

아무 상태에도 의존하지 않는 디폴드 구현을 이용해 인터페이스에 추가하는 방법이 있다. 다만 공유하는 기능이 많아진다면 중복량이 많아지기 떄문에, 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 줄일 수 있다.

