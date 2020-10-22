# Item34: int 상수 대신 열거 타입을 사용하라

## 정수 열거 패턴

자바에서 열거 타입을 지원하기 전에는 다음 코드처럼 정수 상수를 한 묶음 선언해서 사용했다.

~~~java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
~~~

위와 같은 정수 열거 패턴 기법에는 단점이 많다.

</br >

## 정수 열거 패턴 단점

1. 타입 안전을 보장할 방법이 없으면 표현력도 좋지 않다.

   - 오렌지를 건네야 할 메서드에 사과를 보내고 동등 연산자(==)로 비교하더라도 컴파일러는 아무런 경고 메시지를 출력하지 않는다.

   - ~~~java
     APPLE_FUJI == ORANGE_NAVEL
     ~~~

2. 정수 열거 패턴을 사용한 프로그램은 깨지기 쉽다.

   - 평범한 상수를 나열한 것뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다.
   - 따라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다.

3. 정수 상수 문자열로 출력하기가 다소 까다롭다.

   - 그 값을 출력하거나 디버거로 살펴보면 의미가 아닌 단지 숫자로만 보여서 도움이 되지 않는다.

다행히 자바는 열거 패턴의 단점을 보완하고 여러 장점을 안겨주는 **열거 타입(enum type)**을 제공한다.

</br >

## 열거 타입(enum type)

~~~java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH};
public enum Orange {NAVEL, TEMPLE, BLOOD};
~~~

위 코드는 처음 봣던 정수 열거 패턴을 열거 타입으로 바꾼 코드이다.

</br >

## 특징

- 열거 타입 자체는 클래스이다.
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개한다.
- 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 `final`이다.
  - 따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으므로 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다.

</br >

## 장점

1. 컴파일타임 타입 안전성을 제공한다
   - 열거 타입을 매개변수로 받는 메서드가 있다면, 건네받은 참조는 열거 타입의 값중 하나임이 확실하다.
   - 다른 타입의 값을 넘기려 하면 컴파일 오류난다.

~~~java
public enum Apple {
    FUJI, PIPPIN, GRANNY_SMITH;

    public static void appleType(Apple appleType) {
    }

    public static void test() {
        appleType(0); // complie error
        appleType(Apple.FUJI);
    }
}
~~~

</br >

2. 열거 타입에는 각자의 이름 공간이 있어서 이름이 같은 상수도 평화롭게 공존한다.

3. 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다.

   - 공개되는 것이 오직 필드의 이름뿐이라, 정수 열거 패턴과 달리 상수 값이 클라이언트로 컴파일되어 각인되지 않기 때문이다.

4. 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.

5. 열거 타입에 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현할 수 있다.

   ~~~java
   public enum Planet {
       MERCURY(3.302e+23, 2.439e6),
       VENUS(4.869e+24, 6.052e6),
       EARTH(5.975e+24, 6.378e6),
       MARS(6.419e+23, 3.393e6),
       JUPITER(1.899e+27, 7.149e7),
       SATURN(5.685e+26, 6.027e7),
       URAUS(8.683e+25, 2.556e7),
       NEPTUNE(1.024e+26, 2.477e7);
   
       private final double mass;
       private final double radius;
       private final double surfaceGravity;
   
       private static final double G = 6.67300E-11;
   
       Planet(double mass, double radius) {
           this.mass = mass;
           this.radius = radius;
           this.surfaceGravity = G * mass / (radius * radius);
       }
   
       public double mass() {
           return mass;
       }
   
       public double radius() {
           return radius;
       }
   
       public double surfaceGravity() {
           return surfaceGravity;
       }
   
       public double surfaceWeight(double mass) {
           return mass * surfaceGravity;
       }
   }
   ~~~

   - 각 열거 타입 상수 오른쪽 괄호 안 숫자는 생성자에 넘겨지는 매개변수로, 질량과 반지름을 뜻한다.
   - **열거 타입 상숫 각각을 특정 데이터와 연결지으려면 생성자에 데이터를 받아 인스턴스 필드에 저장하면 된다.**
   - 참고로 열거 타입은 근본적으로 불변이라 **모든 필드는 final**이어야 한다.
   - 필드를 public으로 선언해도 되지만, p**rivate으로 두고 별도의 public 접근자 메서드**를 두는 게 낫다.

   </br >

   열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 `values()`를 제공한다.

   ~~~java
   public static void main(String[] args) {
       for (Planet p : Planet.values()) {
       System.out.println(String.format("Planet name : %s", p));
       }
   }
       
   //출력 결과
   //Planet name : MERCURY
   //Planet name : VENUS
   //Planet name : EARTH
   //Planet name : MARS
   //Planet name : JUPITER
   //Planet name : SATURN
   //Planet name : URAUS
   //Planet name : NEPTUNE
   ~~~

</br >

## 열거 타입 상수마다 동작이 달라져야 한다면?

Planet 상수들은 서로 다른 데이터와 연결되는 데 그쳤지만, 상수마다 동작이 달려져야 하는 상황도 있을 것이다. 예를들어 사칙연산 계산기의 연산 종류를 열거 타입으로 선언하고, 실제 연산까지 열거 타입 상수가 직접 수행했으면 한다고 가정해보자.

</br >

### switch문 사용

~~~java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(double x, double y) {
        switch (this) {
            case PLUS:
                return x + y;
            case MINUS:
                return x - y;
            case TIMES:
                return x * y;
            case DIVIDE:
                return x / y;
        }
        throw new AssertionError(String.format("알 수 없는 연산: %s", this));
    }
}
~~~

- 위 코드는 동작은 하지만 깨지기 쉬운 코드이다.
- 예를들어 새로운 상수를 추가하면 해당 case 문도 추가해야 한다. 혹시라도 깜빡한다면, 컴파일은 되지만 새로 추가한 연산을 수행하려 할 떄 "알 수 없는 연산"이라는 런타임 오류를 낼 것이다.

</br >

### 상수별 메서드 구현

다행히 열거 타입은 더 나은 수단을 제공한다. 열거 타입에 apply라는 추상 메서드를 선언하고 각 상수별 자신에 맞게 재정의하는 방법이다.

~~~java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    }, MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    }, TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    }, DIVIDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    public abstract double apply(double x, double y);
}
~~~

- 상수별 메서드 구현을 사용하면 새로운 상수를 추가할 때 **apply가 추상 메서드이므로 재정의하지 않았다면 컴파일 오류로 알려준다.**

- 열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 `valueOf(String)` 메서드가 자동 생성된다.

  ~~~java
  Operation.valueoOf("PLUS");
  ~~~

</br >

## 전략 열거 타입 패턴

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다. 예를들어 급여명세서에서 쓸 요일을 표현하는 열거 타입을 예로 생각해 보자.

~~~java
public enum PayrollDay {
    MONDAY, TUESDAY, WENDESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch (this) {
            case SATURDAY:
            case SUNDAY: // 주말
                overtimePay = basePay / 2;
                break;
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }

        return basePay + overtimePay;
    }
}
~~~

위 코드는 switch 문을 이용하였다. 분명 간결하지만, 앞에서 말했듯이 관리 관점에서는 위험한 코드다.

</br >

## 해결책

상수별 메서드 구현으로 급여를 정확히 계산하는 방법은 두 가지다.

1. 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣기.
2. 계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메서드로 작성한 다음 각 상수가 자신에게 필요한 메서드를 적절히 호출하기.

위 두 방식 모두 코드가 장황해져 가독성이 크게 떨어지고 오류 발생 가능성이 높아진다.

**전략 열거 타입 패턴**을 사용하면 위 두 가지 방법보다 더 안전하고 유연하게 해결할 수 있다.

~~~java
import static effertiveJava.Item34.PayrollDay.PayType.*;

public enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WENDESDAY(WEEKDAY), THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;
    
    PayrollDay(PayType payType){
        this.payType = payType;
    }
    
    int pay(int minutesWorked, int payRate){
        return payType.pay(minutesWorked, payRate);
    }
    
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate){
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate){
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minsWorked, int payRate){
            int basePay = minsWorked * payRate;
            return basePay - overtimePay(minsWorked, payRate);
        }
    }
}
~~~

PayrollDay 열거 타입은 잔업수당 계산을 전략 열거 타입에 위임하여, switch 문이나 상수별 메서드 구현이 필요 없게 된다.

</br >

## switch 문은 언제 사용할 수 있을까?

switch 문은 열거 타입의 상수별 동작을 구현하는 데 적합하지 않다. 하지만 **기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있다.**

예로 Operation 열거 타입을 생각해 보았을 때, 각 연산의 반대 연산을 반환하는 메서드가 필요하다고 해보자. 이럴 때 switch 문을 이용해 원래 열거 타입에 없는 기능을 만들 수 있다.

~~~java
public static Operation inverse(Operation op) {
        switch (op) {
            case PLUS:
                return Operation.MINUS;
            case MINUS:
                return Operation.PLUS;
            case TIMES:
                return Operation.DIVDE;
            case DIVDE:
                return Operation.TIMES;
            
            default: throw new AssertionError(String.format("알 수 없는 연산 : %s" , op);
        }
}
~~~

</br >

## 정리

- 필요한 원소를 **컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용**하자.
- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.
- 하나의 메서드가 상수별로 다르게 동작해야 할 때가 있다. 이런 경우 switch 문 대신 **상수별 메서드 구현**을 사용하자.
- 열거 타입 상수 일부가 같은 동작을 공유한다면 **전략 열거 타입 패턴**을 사용하자.