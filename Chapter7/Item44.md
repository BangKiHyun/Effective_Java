# Item44: 표준 함수형 인터페이스를 사용하라

## 서론

`fava.util.funtion` 패키지를 보면 다양한 용도의 표준 함수형 인터페이스가 담겨있다.

**필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하자.**

</br >

## 표준 함수형 인터페이스 종류

`fava.util.funtion` 패키지에 총 43개의 인터페이스가 담겨 있다. 다 알 필요는 어렵겠지만, 기본 인터페이스 6개만 기억하면 나머지를 충분히 유추해 낼 수 있다.

1. UnaryOperator
   - 인수값 1개
   - 반환값과 인수의 타입이 같은 함수
2. BinaryOperator
   - 인수값 2개
   - 반환값과 인수의 타입이 같은 함수
3. Predicate
   - 인수값 1개
   - boolean을 반환하는 함수
4. Function
   - 인수값 1개
   - 인수와 반환 타입이 다른 함수
5. Supplier
   - 인수값 없음
   - 값을 반환(혹은 제공)하는 함수
6. Consumer
   - 인수값 1개
   - 반환값이 없는(특히 인수를 소비하는) 함수

| 인터페이스          | 함수 시그니처       | 예                  |
| ------------------- | ------------------- | ------------------- |
| `UnaryOperator<T>`  | T apply(T t)        | String::ToLowerCase |
| `BinaryOperator<T>` | T apply(T t1, T t2) | BigInteger::add     |
| `Predicate<T>`      | boolean test(T t)   | Collection::isEmpty |
| `Function<T, R>`    | R apply(T t)        | Arrays::asList      |
| `Supplier<T>`       | T get()             | Instant::now        |
| `Consumer<T>`       | void accept(T t)    | System.out::println |

- 기본 인터페이스는 기본 타입인 int, long, double용으로 각 3개씩 변형이 생겨난다.
  - `IntPredicate`, `LongBinaryOperator`
- 기본 타입을 반환하는 변형이 있다.
  - `LongToIntFunction`, `ToLongFunction<int[] >`
- 인수를 2개씩 받는 변형이 있다.
  - `BiPredicate<T, U>`, `BiFunction<T, U, R>`

**주의할 점으로 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지 말자.** 동작은 하지만 성능이 느려질 수 있다.

</br >

## 전용 함수형 인터페이스

다음 중 하나 이상을 만족한다면 함수형 인터페이스를 직접 구현해야 하는 건 아닌지 고민해보자.

- 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
- 반드시 따라야 하는 규약이 있다.
- 유용한 디폴트 메서드를 제공할 수 있다.

예로 `Comparator<T>`인터페이스가 있다.

</br >

### 전용 함수형 인터페이스 설계시 주의점

1. **전용 함수형 인터페이스 설계시 @FuntionalInterface 애너테이션을 사용하자.** 이유는 다음과 같다.
   - 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.
   - 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
   - 유지보수 과정에서 누군가 실수로 메서드를 추가힞 못하게 막아준다.

2. **서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의하지 말자.**

