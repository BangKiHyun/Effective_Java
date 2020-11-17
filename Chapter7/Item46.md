# Item46: 스트림에서는 부작용 없는 함수를 사용하라

스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다. 이때 각 변환 단계는 가능한 한 이전 단계외 결과를 받아 처리하는 순수 함수여야 한다.

순수 함수란 오직 입력만이 결과에 영향을 주는 함수를 말한다. 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다. 이렇게 하려면 스트림 연산에 건네는 함수 객체는 모두 **부작용이 없어야 한다.**

## 코드 예제

### 스트림 패러다임을 이해하지 못한 채 API만 사용한 코드

~~~java
    public static void main(String[] args) {
        Map<String, Long> freq = new HashMap<>();
        try (Stream<String> words = new Scanner(file).tokens()) {
            words.forEach(word -> {
                freq.merge(word.toLowerCase(), 1L, Long::sum);
            });
        }
    }
~~~

- 위 코드는 스트림 코드를 가장한 반복적 코드다. 모든 작업이 종단 연산인 `forEach`에서 일어난다.
- `forEach`가 그저 스트림이 수행한 연산 결과를 보여주는 일 이상을 하는 모습이 보인다.

`forEach` 연산은 종단 연산 중 기능이 가장 적고 덜 스트림 답다. 대놓고 반복적이라 병렬화할 수도 없다. **`forEach`연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자.**

### 스트림을 제대로 활용한 코드

~~~java
    public static void main(String[] args) {
        Map<String, Long> freq;
        try (Stream<String> words = new Scanner(file).tokens()) {
            freq = words
                    .collect(groupingBy(String::toLowerCase, counting()));
        }
    }
~~~

첫 번째 코드와 같은 일을 하지만, 스트림 API를 제대로 사용했다.

</br >

## Collector

스트림을 사용하기위해 꼭 배워야 하는 새로운 개념이다. `java.util.stream.Collectors` 클래스는 39개의 메서드를 갖고 있다.

**collector는 축소 전략을 캡슐화한 블랙박스 객체**라고 생각하자. 축소란 스트림의 원소들을 객체 하나에 취합한다는 뜻이다.

### 빈도표에서 가증 흔한 단어 10개를 뽑아내는 스트림 파이프라인(toList)

```java
freq.keySet().stream()
        .sorted(comparing(freq::get).reversed())
        .limit(10)
        .collect(toList());
```

> 마지막 toList는 Colletors 메서드로, **정적 임포트하여 쓰면 스트림 파이프라인 가독성이 좋아져, 흔히 이렇게 사용한다.**

</br >

### toMap

`toMap(keyMapper, valueMapper)`로, 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다.

~~~java
private static final Map<String, Operation> stringToEnum = 
        Stream.of(Operation.values())
        .collect(toMap(Objects::toString, e -> e));
~~~

위 코드는 열거 타입 상수의 문자열 표현을 열거 타입 자체에 매핑한다.

- **`toMap` 형태는 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합하다.**
- 스트림 원소 다수가 같은 키를 사용하면 파이프라인이 `IllegalStateException`을 던지면 종료될 것이다.

### 인수 3개를 받는 toMap

어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때 유용하다.

~~~java
Map<Artist, Album> topHits = albums.collect(
            toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
~~~

위 코드를 말로 풀어보자면 "앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것이다"는 이야기다.

</br >

### groupingBy

이 메서드는 입력으로 분류 함수를 받고 출력으로 원소들을 카테고리별로 모아 놓은 맵을 반환한다.

다중정의된 `groupingBy` 중 형태가 가장 간단한 것은 분류 함수 하나를 인수로 받아 맵을 반환한다.

~~~java
public class GroupingByExam {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("a", "b", "c", "d");

        final Map<Integer, List<String>> collect = words.stream()
                .collect(groupingBy(word -> word.length()));
    }
}
~~~

리스트 외의 값을 갖는 맵을 생성하게 하려면, 분류 함수와 함께 다운스트림 수집기도 명시해야 한다. **다운스트림 수집기의 역할은 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성하는 일이다.** 

~~~java
        final Map<String, Long> freq = words.stream()
                .collect(groupingBy(String::toLowerCase, counting()));
~~~

</br >

### joining

이 메서드는 문자열 등의 CharSequence 인스턴스의 스트림에만 적용할 수 있다.

- 매개변수가 없는 `joining`은 단순히 원소들을 연결하는 수집기를 반환한다.
- 인수 하나짜리 `joining`은 구분문자(delimiter)를 매개변수로 받는다.
- 인수 3개짜리 `joining`은 구분문자에 더해 접두문자(prefix)와 접미문자(suffix)도 받는다.

```java
public class JoiningExam {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("Hello", " ", "word", "!");

        final String connect = words.stream()
                .collect(joining()); // Hello word!

        final String delimiter = words.stream()
                .collect(joining(",")); // Hello, ,word,!

        final String three = words.stream()
                .collect(joining(",", "[", "]")); // [Hello, ,word,!]
    }
}
```

