# Item61: 박싱된 기본 타입보다는 기본 타입을 사용하라

## 기본 타입 vs 박싱된 기본 타입

### 1. 기본 타입은 값만 가지고 있고, 박싱된 기본 타입은 값에 더해 식별성(identity)을 갖는다.

- 박싱된 기본 타입의 두 인스턴스는 값이 같아도 서로 다르다고 식별될 수 있다.

### 2. 기본 타입의 값은 언제나 유효하나, 박싱된 기본 타입은 유효하지 않은 값을 가질 수 있다.

- 박싱된 기본 타입을 `null`을 가질 수 있다.

### 3. 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.(Item6)

</br >

## 박싱된 기본 타입에 == 연산자를 사용하면 오류가 일어난다

### 코드 예제

~~~java
    public static void main(String[] args) {
        Comparator<Integer> naturalOrder =
                (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);

        Integer firstInteger = new Integer(42);
        Integer secondInteger = new Integer(42);

        System.out.println(naturalOrder.compare(firstInteger, secondInteger));
    }

// 출력 결과
// 1
~~~

위 코드의 `Integer`인스턴스의 값이 42로 같으므로 0을 출력해야 하지만 1을 출력하는 것을 볼 수 있다. 즉, 첫 번째가 두 번째보다 크다고 말한다.

**위에서 언급했듯이 박싱된 기본 타입을 식별성을 갖고 있는다. 즉, 동일성(identity, ==)을 비교할 때 값이 다르게 될 수 있다.**

위 코드를 수정하기 위해 박싱된 `Integer` 값을 기본 타입 정수로 저장한 다음, 비교를 기본 타입 변수로 수행하면 된다.

### 수정된 코드

~~~java
    public static void main(String[] args) {
        Comparator<Integer> naturalOrder =
                (iBoxed, jBoxed) -> {
                    int i = iBoxed, j = jBoxed; // 오토박싱
                    return i < j ? -1 : (i == j ? 0 : 1);
                };

        final Integer firstInteger = new Integer(42);
        final Integer secondInteger = new Integer(42);

        System.out.println(naturalOrder.compare(firstInteger, secondInteger));
    }

// 출력 결과
// 0
~~~

</br >

## 기본 타입과 박싱된 기본 타입을 혼용한 연산에서 박싱된 기본 타입의 박싱이 자동으로 풀린다

### 코드 예제

~~~java
public class Unbelievable {
    static Integer i;

    public static void main(String[] args) {
        if (i == 42) // NullPointerException 발생
            System.out.println("믿을 수 없군!");
    }
}
~~~

위 코드를 실행하면 `NullPointerException`이 발생한다. 원인은 i가 `int`가 아닌 `Integer`이기 때문이다.

**박싱된 기본 타입(`Integer`)이 `null` 참조를 언박싱 하면서 `NullPointerException`이 발생하게 된다.**

간단한 해결 방법으로 i를 `int`로 선언해주면 된다.

</br >

## 박싱된 기본 타입은 언제 써야 되나?

- 컬렉션의 원소, 키, 값으로 쓴다. 컬렉션은 기본 타입을 담을 수 없으므로 어쩔 수 없이 박싱된 기본 타입을 써야 한다.
- 매개변수화 타입이나 매개변수화 메서드의 타입 매개변수로 박싱된 기본타입을 써야 한다.

