# Item43: 람다보다는 메서드 참조를 사용하라

## 서론

람다가 익명 클래스보다 나은 점 중 가장 큰 특징은 간결함이다.</br >그런데 **메서드 참조**를 사용하면 람다보다 더 간결하게 만들 수 있다.

</br >

## 람다 방식

~~~java
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        List<String> list = Arrays.asList("A", "B", "C", "A");
        
        for (String text : list) {
            map.merge(text, 1, (count, incr) -> count + incr);
        }

        System.out.println(map); // {A=4, B=2, C=2}
    }
~~~

위 코드는 람다를 사용한 방식이다. 매개변수인 count와 incr은 크게 하는 일 없이 공간만 차지한다. 이 람다는 두 인수의 합을 단순히 반환할 뿐이다.

`Integer`클래스는 이 람다와 기능이 같은 정적 메서드 `sum`을 제공한다. 메서드 참조로 바꿔보자.

## 메서드 참조 방식

~~~java
...
        for (String text : list) {
            map.merge(text, 1, Integer::sum);
        }
...
~~~

똑같은 결과지만 좀 더 간결하고 명확하다.

가끔 메서드 참조 보다 람다가 메서드 참조보다 간결할 때가 있다. 예를 들어 다음과 같은 코드가 있다고 해보자.

~~~java
public class GoshThisClassnameIsHumongous {

    public static void action(){
        System.out.println("action");
    }
}
~~~

~~~java
// 메서드 참조
service.execute(GoshThisClassnameIsHumongous::action)

// 람다
sevice.execute(() -> action());
~~~

두 방식을 비교해 봤을 때 메서드 참조 쪽은 더 짧지도, 명확하지도 않다. 따라서 람다 쪽이 낫다.

</br >

## 메서드 참조 유형 (5가지)

| 메서드 참조 유형    | 예                     | 람다 버전                                                |
| ------------------- | ---------------------- | -------------------------------------------------------- |
| 정적                | Integer::parseInt      | str -> Integet.parseInt(str)                             |
| 한정적 (인스턴스)   | Instant.now()::isAfter | Instant then = Instant.now();<br />t -> then.isAfter(t); |
| 비한정적 (인스턴스) | String::toLowerCase    | str -> str.toLowerCase()                                 |
| 클래스 생성자       | TreeMap<K, V>::new     | () -> new Treemap<K, V>()                                |
| 배열 생성자         | int[]::new             | len -> new int[len]                                      |

</br >

## 정리

- 메서드 참조는 람다의 간단명료한 대안이 될 수 있다.
- **메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하라.**

