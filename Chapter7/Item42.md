# Item42: 익명 클래스보다는 람다를 사용하라

## 서론

예전에 자바에서 함수 타입을 표현할 때 **추상 메서드를 하나만 담은 인터페이스를 사용**했다. 이런 인터페이스의 인스턴스를 함수 객체라고 하여, 특정 함수나 동작을 나타내는 데 썼다.

</br >

## 익명 클래스(낡은 기법)

다음은 문자열을 길이순으로 정렬하는데, 정렬을 위한 함수로 익명 클래스사용한다.

~~~java
Collections.sort(words, new Comparator<String>() {
        @Override
        public int compare(String s1, String s2) {
            return Integer.compare(s1.length(), s2.length());
        }
    });
~~~

익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않다.

</br >

## 람다(익명 클래스 대체)

**자바 8부터 추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 람다식(lambda expression)을 사용해 만들 수 있게 됐다.** 람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다.

다음은 익명 클래스를 사용한 앞의 코드를 람다 방식으로 바꾼 모습이다.

~~~java
Collections.sort(words,
        (s1, s2) -> Integer.compare(s1.length(), s2.length()));
~~~

- 람다 타입: `Comparator<String>`
- 매개변수(s1, s2) 타입: `String`
- 반환값 타입: `int`

이처럼 각각의 타입이 다르지만 코드에서는 언급이 없다. 컴파일러가 문맥을 살펴 타입을 추론해주기 때문에 가능하다.</br >
컴파일러가 타입을 결정하지 못할 수도 있는데, 그럴 때는 프로그래머가 직접 명시해야 한다.

**타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자.**

참고로 **컴파일러가 타입 추론하는 데 필요한 타입 정보 대부분을 제네릭에서 얻는다.** 이 정보를 제공하지 않으면 컴파일러는 람다의 타입을 추론할 수 없게 된다. 위 코드에서 `words`가 `List<String>`이 아니라 `List`였다면 오류가 난다.

비교자 생성 메서드와 List 인터페이스에 추가된 sort 메서드를 사용하면 더 간결하게 만들 수 있다.

~~~java
// 비교자 생성 메서드 사용
Collections.sort(words, Comparator.comparingInt(String::length));

// 자바 8 List 인터페이스에 추가된 sort 메서드 사용
words.sort(Comparator.comparingInt(String::length));
~~~

</br >

## Operation 열거 타입 예제

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

`apply` 메서드의 동작이 상수마다 달라야 해서 상수별 클래스 몸체를 사용해 각 상수에서 `apply` 메서드를 재정의 했다.

아이템 34에서 상수별 클래스 몸체를 구현하는 방식보다 **열거 타입에 인스턴스 필드를 두는 편이 낫다**고 했다. 람다를 사용하여 바꾸면 다음과 같다.

~~~java
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
~~~

각 열거 타입 상수의 동작을 람다로 구현해 생성자에 넘기고, 생성자는 이 람다를 인스턴스 필드로 저장해둔다.

그 다음 `apply`메서드에서 필드에 저장된 람다를 호출하기만 하면 된다.

</br >

## 람다 단점

1. 람다는 이름이 없고 문서화를 못한다.
   - 그러므로 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.
2. 세 줄을 넘어가면 가독성이 심하게 나빠진다.
   - 람다는 한 줄일 때 가장 좋고, 세 줄 안에 끝내는게 좋다.
3. 람다는 자신을 참조할 수 없다.
   - 람다에서 `this`키워드는 바깥 인스턴스를 가리킨다. 반면 익명클래스에서의 `this`는 익명 클래스의 인스턴스 자신을 가리킨다.
   - 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다.
4. 직렬화 형태가 구현별로 다를 수 있다.
   - **람다를 직렬화하는 일은 극히 삼가야 한다.**
   - 직렬화해야만 하는 함수 객체가 있다면 private 중첩 클래스의 인스턴스를 사용하자.

