# Item48: 스트림 병렬화는 주의해서 적용하라

동시성 프로그래밍을 할 때는 안전성과 응답 가능 상태를 유지해야 한다. 병렬 스트림 파이프라인 프로그래밍에서도 마찬가지다.

## 메르센 소수 예제

~~~java
    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }

    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
~~~

아이템 45에서 다뤘던 코드다. 위 코드의 속도를 높이고 싶어 `parallel()`를 호출하면 어떻게 될까? 성능 향상을 기대했겠지만 이 프로그램은 아무것도 출력하지 않는다. 왜그럴까?

이유는 스트림 라이브러리가 이 파이프라인을 병렬화하는 방법을 찾아내지 못했기 때문이다.

</br >

## 병렬화로 성능 개선이 안될 때

1. 데이터 소스가 `Stream.iterate` 일때
   - `iterate`가 박싱된 객체를 생성하므로 이를 다시 언박싱하는 과정이 필요하다.
   - `iterate`는 병렬로 실행될 수 있도록 독립적인 청크로 분할하기 어렵다.
     - 이전 연산의 결과에 따라 다음 함수의 입력이 달라지기 때문에 `iterate`연산을 청크로 분할하기 어렵다.
2. 중간 연산으로 `limit`을 사용할 때
   -  `limit`이나 `findFirst`같은 **요소의 순서에 의존하는 연산**은 병렬 스트림에서 사용하려면 비싼 비용을 치뤄야 한다.
   - 반대의 예로 `findAny`, `allMatch`, `noneMatch`는 순서에 의존하지 않기 때문에 성능 개선을 기대할 수 있다.

## 병렬화를 사용하면 좋을 때

- ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스
- 배열, int 범위, long 범위일 때

**위 자료구조들은 모두 데이터를 원하는 크기로 정확하게 나눌 수 있어서 일을 다수의 스레드에 분배하기 좋다는 특정이 있다.**

또한, 원소들을 순차적으로 실행할 때의 **참조 지역성**이 뛰어나다. 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 뜻이다. 참조들이 가리키는 실제 객체가 메모리에서 서로 떨어져 있을 수 있는데, 그러면 참조 지역성이 나빠진다.

참조 지역성이 가장 뛰어난 잘구조는 기본 타입의 배열로, 데이터 자체가 메모리에 연속해서 저장되기 때문이다.

</br >

## 종단 연산이 병렬화에 미치는 영향

종단 연산에서 수행하는 작업은 파이프라인 전체 작업에서 상당 비중을 차지한다. **순차적인 연산이 아니라면 파이프라인 병렬 수행의 효과는 제한된다.**

### 축소(reduction)

- 병렬화에 가장 적합한 것은 축소다. `Stream`의 `reduce` 메서드 중 하나, min, max, count, sum 같이 완성된 형태로 제공되는 메서드 중 하나를 선택해 수행한다.
- `anyMatch`, `allMatch`,` noneMatch` 처럼 조건에 맞으면 바로 반환하는 메서드도 병렬화에 적합하다.

### 가변 축소(mutable reduction)

- `Stream`의 `collect` 메서드는 병렬화에 적합하지 않다. 컬렉션을 합치는 부담이 크기 때문이다.

</br >

## 병렬화 시 Stream규약

**스트림을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다.**

Stream의 연산에 건네지는 accumulator와 combiner 함수는 다음을 만족해야 한다.

1. 결합법칙(associative)
   - (a op b) op c == a op (b op c)
2. 간섭받으면 안된다.(non-interfering)
   - 파이프라인이 수행되는 동안 데이터 소스가 변경되지 않아야 한다.
3. 상태를 같지 않아야 한다.(stateless)
   - 상태를 같고 있다면 SideEffect가 발생할 수 있다.

</br >

스트림 병렬화는 오직 성능 최적화 수단이다. 다른 최적화와 마찬가지로 변경 전후로 반드시 성능을 테스트하여 병렬화를 사용할 가치가 있는지 확인해야 한다.

또한, 스트림 병렬화가 효과를 보는 경우가 많지 않다. 하지만 **조건이 잘 갖춰지면 `parallel` 메서드 호출로 프로세서 코어 수에 비례하는 성능 향상을 만끽할 수 있다.**

</br >

## 병렬화의 좋은 예

~~~java
    public static long pi(long n) {
        return LongStream.rangeClosed(2, n)
                .parallel()
                .mapToObj(BigInteger::valueOf)
                .filter(i -> i.isProbablePrime(50))
                .count();
    }
~~~

위 코드는 n보다 작거나 같은 소수의 개수를 계산하는 함수다. `parallel()`을 호출 했을때와 안했을 때의 성능차이가 크다.