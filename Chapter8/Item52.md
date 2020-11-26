# Item52: 다중정의는 신중히 사용하라

## 재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택된다

다음은 다중정의를 잘못한 예제다.

### 다중정의한 메서드

~~~java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> list) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<>(),
                new ArrayList<>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections) {
            System.out.println(classify(c)); // 그 외 그 외 그 외
        }
    }
}
~~~

위 코드는 "집합", "리스트", "그 외"를 차례로 출력할 것 같지만, 실제로는 "그 외"를 세 번 출력한다.

왜냐하면 **다중정의(overloading)된 메서드는 어느 메서드를 호출할 지 컴파일타임에 정해지기 때문이다.**

컴파일타임에는 for 문 안의 c 는 항상 `Collection<?>` 타입이다. 런타임에는 타입이 매번 달라지지만, 호출할 메서드를 선택하는데 영향을 주지 못한다.

이처럼 **다중정의한 메서드는 정적으로 선택**된다. 그렇다면 재정의한 메서드는 어떻게 될까?

### 재정의한 메서드

~~~java
class Wine {
    String name() {
        return "포도주";
    }
}

class SparklingWine extends Wine {
    @Override
    String name() {
        return "발포성 포도주";
    }
}

class Champagne extends SparklingWine {
    @Override
    String name() {
        return "샴페인";
    }
}


public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
                new Wine(), new SparklingWine(), new Champagne());

        for (Wine wine : wineList) {
            System.out.println(wine.name()); // 포도주 발포성 포도주 샴페인
        }
    }
}
~~~

위 코드는 "포도주", "발포성 포도주", "샴페인"을 차례대로 출력한다.

for 문에서의 컴파일타임 타입이 모두 `Wine`인 것에 무관하게 **항상 '가장 하위에서 정의한' 재정의 메서드가 실행**되는 것이다.

즉, **런타임 시점에 타입이 정해져 정적으로 선택**된다.

</br >

## 다중정의 메서드 주의점

재정의한 메서드는 기대한 대로 동작하지만, 다중정의한 메서드는 그렇지 않을 수 있다.

특히 공개 API라면 사용자가 매개변수를 넘기면서 어떤 다중정의 메서드가 호출될지 모른다면 프로그램이 오동작하기 쉽다. 그러니 **다중정의가 혼동을 일으키는 상황은 피해야 한다.**

- 안전하고 보수적으로 가려면 **매개변수 수가 같은 다중정의는 만들지 말자.**
- **가변인수를 사용하는 메서드라면 다중정의를 아예 사용하지 말자.**

</br >

## 회피 방법

### 1. 다중정의하는 대신 메서드 이름을 다르게 지어주자

예로 `ObjectOutputStream`클래스의 `wirteBoolean(boolean)`, `writeInt(int)`, `writeLong(long)` 메서드가 있다.

### 2. 매개변수 중 하나 이상을 근본적으로 다르게 하자

매개변수가 같은 다중정의 메서드가 많더라도, 매개변수 중 하나 이상이 "근본적으로 다르다"면 런타임 타입만으로 결정할 수 있다. 근본적으로 다르다는 건 **두 타입의 값을 서로 어느 쪽으로도 형변환할 수 없다는 뜻**이다.

예로 `ArrayList`에 `int`를 받는 생성자와 `Collection`을 받는 생성자가 있다. 두 생성자 중 어느 것이 호출될지 헷갈릴 일이 없다.

</br >

## 다중정의 방해요소

### 오토박싱

```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list); // [-3, -2, -1] [-2, 0, 2]
    }
}
```

위 코드는 음이 아닌 값, 0, 1, 2 를 제거한 후 "[-3, -2, -1] [-3, -2, -1]"을 출력하리라 예상할 것이다. 하지만 리스트에서는 홀수를 제거한 후 ""[-3, -2, -1] [-2, 0, 2]" 를 출력한다.

이유는 `List`에 `remove(int index)`와 `remove(Object)`로 다중정의 되어있다. 그 중 `remove(int index)`를 선택하기 때문이다.

이 문제는 `Integer`로 형변환하여 올바른 다중정의 메서드를 선택하게 하면 된다.

</br >

### 람다와 메서드 참조

자바 8에 도입한 람다와 메서드 참조 역시 다중정의 시 혼란을 줄 수 있다.

~~~java
    public static void main(String[] args) {
        // 1번
        new Thread(System.out::println).start();

        // 2번
        ExecutorService es = Executors.newCachedThreadPool();
        es.submit(System.out::println);
    }
~~~

1번과 2번 모두 `Runnable`을 받는 형제 메서드를 다중정의하고 있다. 하지만 2번은 컴파일 오류가 난다.

이유는 `submit` 다중정의 메서드 중 `Callable<T>`를 받는 메서드도 있다. 만약 `println`이 다중정의 없이 단 하나만 존재했다면 이 `submit` 메서드도 호출이 제대로 컴파일됐을 것이다.

**참조된 메서드(println)와 호출한 메서드(submit) 양쪽 다 다중정의**되어, 다중정의 해서 알고리즘(적절한 다중정의 메서드를 찾는 알고리즘)이 기대처럼 동작하지 않는 상황이다.

**메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다.**

</br >

## 인수 포워드

다음은 다중정의 메서드로 일을 넘겨버린(forward) 코드다.

```java
public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence) sb);
}
```

인수를 포워드하여 두 메서드가 동일한 일을 하도록 보장한다. **동일한 일을 보장하므로 해로울 건 전혀 없다.**

하지만 다음은 같은 객체를 건네더라도 전혀 다른 일을 수행한다.

~~~java
public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}

  
public static String valueOf(char data[]) {
    return new String(data);
}
~~~

이렇게 해야 할 이유가 없었음에도, 혼란을 불러올 수 있는 잦ㄹ못된 사례다.