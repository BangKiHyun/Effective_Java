# Item32: 제네릭과 가변인수를 함께 쓸 때는 신중하라

## 가변인수(varargs)

- 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해준다.
- 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어진다.

</br >

## 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.

### 힙 오염(heap pollution)

- 매개변수화 타입의 변수가 타입이 다른 객체를 참조할 때 발생

~~~java
    public static void dangerous(List<String>... stringLists) {
        List<Integer> intList = List.of(42);
        Object[] objects = stringLists;
        objects[0] = intList; // 힙 오염 발생
        String s = stringLists[0].get(0); // ClassCastException
       }
~~~

- 위와 같이 `List<String>`에 `List< Integer>`가 참조(다른 타입 객체를 참조)하는 상황을 말한다.

`dangerous`메서드의 마지막 줄을 보면 형변환하는 곳이 보이지 않는데도 인수를 건네 호출하면 `ClassCastException`을 던진다.(보이지 않는 형변환)

**이처럼 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 타입 안전성을 깨뜨리기 때문에 안전하지 않다.**

</br >

## 제네릭 varargs 매개변수를 받는 메서드의 경고 숨김 및 안전하게 사용 하는 방법

제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 매우 유용하기 때문이라고 한다. 예로 `Arrays.asList(T... a)`, `EnumSet.of(E first, E... rest)`가 있다.

</br >

### @SafeVarargs

- `@SafeVarargs`어노테이션은 제네릭 가변인수 메서드가 작성자 **클라이언트 측에서 발생하는 경고를 숨길 수 있게 해준다.**
- 즉, 메서드 작성자가 그 메서드가 **타입 안전함을 보장하는 장치**다.
- 확실하지 않다면 절대 `@SafeVarargs`어노테이션을 달면 안 된다.

</br >

### 메서드가 안전한지 확인하는 방법

- 가변인수 메서드를 호출할 때 varargs 매개변수를 담는 제네릭 배열이 만들어진다.
- **메서드가 이 배열에 아무것도 저장하지 않고(그 매개변수들을 덮어쓰지 않고), 배열의 참조가 밖으로 노출되지 않는다면 타입 안전하다.**
- 즉, varargs 매개변수 배열이 호출자로부터 그 메서드로 **순수하게 인수들을 전달하는 일만 한다면** 그 메서드는 안전하다.

</br >

### 제네릭 매개변수 배열의 참조를 노출하는 코드

~~~java
    static <T> T[] toArray(T... args) { 
        return args; //외부로 노출
    }
~~~

- 이 메서드가 반환하는 타입은 메서드에 인수를 넘기는 **컴파일타임에 결정**된다.
- 그 시점에 컴파일러에게 충분한 정보가 주어지지 않아 타입을 잘못 판단할 수 있다.

</br >

### toArray를 사용하는 메서드

~~~java
    public static <T> T[] pickTwoV1(T a, T b, T c) {
        switch (ThreadLocalRandom.current().nextInt(3)) {
            case 0:
                return toArray(a, b);
            case 1:
                return toArray(b, c);
            case 2:
                return toArray(c, a);
        }
        throw new AssertionError();
    }
~~~

- 위 메서드를 본 컴파일러는 `toArray`에 넘길 T 인스턴스 2개를 담을 varargs 매개변수 배열을 만드는 코드를 생성한다.
- 이 때 만들어지는 배열의 타입은 `Object[]`인데, 그 이유는 `pickTwoV1`에 어떤 타입의 객체를 넘기더라도 담을 수 있는 가장 구체적인 타입이기 때문이다.

</br >

### main 메서드

~~~java
    public static void main(String[] args) {
        String[] pickTwoV1 = GenericVarargsTest.pickTwoV1("a", "b", "c"); // ClassCastException
    }
~~~

- 위 메서드는 별다른 경고 없이 컴파일되지만 실행하면 ClassCastException을 던진다.
- 왜냐하면 `Object[]`는 `String[]`의 하위 타입이 아니기 때문에 형변환이 실패하기 때문이다.(묵시적 다운 캐스팅 불가)

</br >

## 결론

- 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하고, 배열과 제네릭의 타입 규칙이 서로 다르기 때문에 서로 궁합이 좋지 않다.
- 제네릭 varargs 매개변수는 타입 안전하지는 않지만 허용된다.
- 메서드에 제네릭 varargs 매개변수를 사용하고자 한다면, 그 메서드가 타입 안전한지 확인 후 `@SafeVarargs`어노테이션을 달자.