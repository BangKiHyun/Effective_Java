# Item45: 스트림은 주의해서 사용하라

스트림 API는 다량의 데이터 처리 작업을 돕고자 자바 8에 추가되었다. 이 API가 제공하는 추상 개념 중 핵심은 두 가지다.

1. 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다.
2. 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.

## 스트림 파이프라인

- 스트림 파이프라인은 소스 스트림에서 시작해서 **종단 연산**으로 끝나며, 그 사이에 하나 이상의 **중간 연산**이 있을 수 있다.
- 스트림 파이프라인은 **지연 평가**된다. 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다
- 종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않기 때문에, 종단 연산을 빼먹지 말자.
- 기본적으로 스트림 파이프라인은 순차적으로 수행된다. 파이프라인을 병령로 실행하려면 parallel 메서드를 호출하면 되기는 하나 효과를 볼 수 있는 상황을 많지 않다.

</br >

## 코드 예제

다음은 사전 파일에서 단어를 읽어 사용자가 지정한 값보다 원소 수가 많은 아나그램 그룹을 출력한다. 아나그램이란 철자를 구성하는 알파벡이 같고 순서만 다른 단어를 말한다.

~~~java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word),
                        (unused) -> new TreeSet<>()).add(word);

            }
        }
        for (Set<String> group : groups.values())
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
~~~

- computeIfAbsent(Java 8에 추가): 맵 안에 키가 있는지 찾은 다음, 있으면 단순히 그 키에 매핑된 값을 반환. 키가 없으면 건네진 함수 객체를 키에 적용하여 값을 계산한 다음 그 키와 값을 매핑한 후, 계산된 값을 반환.

</br >

### 스트림(따라 하지 말 것)

~~~java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new, (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
~~~

위 코드는 확실히 짧지만 읽기는 어렵다. 이처럼 **스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.**

### 적적힐 사용한 스트림

~~~java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));

        }
    }
  
    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
~~~

> 람다 매개변수 이름은 주의해서 정해야 한다
>
> **람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.**
>
> **도우미 메서드를 적절히 활용하는 일의 중요성은 일반 반복코드에서보다 스트림 파이프라인에서 훨씬 크다.**
>
> 위 코드에서 매개변수 g는 사실 group이다. 또한 단어의 철자를 알파벳순으로 정렬하는 일은 alphabetize에서 수행했다. 연산에 적절한 이름을 주어 세부 구현을 주 프로그램 로직 밖으로 빼내 전체적 가독성을 높인 것이다.

</br >

### char 값들을 처리할 때 스트림을 삼가는 편이 낫다

자바는 기본 타입인 char용 스트림을 지원하지 않는다. 다음 코드를 봐보자.

~~~java
public class streamCharExam {

    public static void main(String[] args) {
        "Hello world!".chars().forEach(System.out::print);
    }
}

// 출력결과
// 721011081081113211911111410810033
~~~

String.chars()가 반환하는 스트림의 원소는 char가 아니라 int 값이다. 그러므로 정숫값을 출력하게 된 것이다.

다음처럼 명시적 형변환을 해줘야 한다.

~~~java
"Hello world!".chars().forEach(x -> System.out.print((char) x));
~~~

**그러니 기존 코드는 스트림을 사용하도록 리팩터링하된, 새 코드가 나아 보일 때문 반영하자**

</br >

## 스트림 적용하기 좋은 후보

다음 일들에는 스트림이 아주 안성맞춤이다.

- 원소들의 시퀀스를 일관되게 변환한다.
- 원소들의 시퀀스를 필터링한다.
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다.(더하기, 연결하기, 최솟값 구하기 등)
- 원소들의 시퀀스를 컬렉션에 모은다.
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

이러한 일 중 하나를 수행하는 로직이라면 스트림을 적용하기에 좋은 후보다.

</br >

## 메르센 소수 예제

메르센 수는 `2^p - 1`형태의 수로, 여기서 p가 소수이면 해당 메르센 수도 소수일 수 있는데, 이를 메르센 소수라 한다.

~~~java
public class MersennePrime {
    // 스트림을 반환하는 메서드 이름은 복수 명사로 쓰자!
    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }

    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }
}
~~~

위 코드는 소수들을 사용해 메르센 수를 계산하고, 결괏값이 소수인 경우만 남긴 다음, 결과 스트림의 원소 수를 20개로 제한해놓고, 작업이 끝나면 결과를 출력한다.

만약 메르센 소수의 앞에 지수(p)를 출력하길 원한다고 해보자. 이 값은 **초기 스트림에만 나타나므로 결과를 출력하는 종단 연산에서는 접근할 수 없다.**

다행히 지수는 단순히 숫자를 이진수로 표현한 다음 몇 비트인지를 세면 나온다. 다음과 같이 작성하면 된다.

~~~java
public class MersennePrime {
    // 스트림을 반환하는 메서드 이름은 복수 명사로 쓰자!
    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }

    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
    }
}
~~~

</br >

## 데카르트 곱 예제

### 반복 예제

스트림과 반복 중 어느 쪽을 써야 할지 바로 알기 어려운 작업도 많다. 다음 코드를 봐보자.

~~~java
public class Deck {
    private static List<Card> newDeck() {
        List<Card> result = new ArrayList<>();
        for(Suit suit : Suit.values()){
            for(Rank rank : Rank.values())
                result.add(new Card(suit, rank));
        }
        return result;
    }
}
~~~

위 코드는 두 집합(Suit, Rank)의 원소들로 만들 수 있는 가능한 모든 조합을 계산한다.

### 스트림 예제

~~~java
public class Deck {
    private static List<Card> newDeck() {
        return Stream.of(Suit.values())
                .flatMap(suit ->
                        Stream.of(Rank.values())
                                .map(rank -> new Card(suit, rank)))
                .collect(toList());
    }
}
~~~

- flatMap: 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음 그 스트림을 다시 하나의 스트림으로 합침.

두 방식 중 어느 방법이 더 좋아 보이는가? 결국 개인 취향과 프로그래밍 환경의 문제다.

**스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 선택해라.**

