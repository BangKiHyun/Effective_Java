# Item37: ordinal 인덱싱 대신 EnumMap을 사용하라

배열이나 리스트에서 원소를 꺼낼 때 `oridinal` 메서드로 인덱스를 얻는 코드가 있다. 다음 코드를 봐보자.

~~~java
public class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL}

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
~~~

위 코드는 식물을 간단히 나타낸 클래스이다.

다음으로 정원에 심은 식물들을 배열 하나로 관리하고, 이들을 생애주기별로 묶어보자.

~~~java
import java.util.HashSet;
import java.util.Set;

public class Main {
    public static void main(String[] args) {
        Set<Plant>[] plantsByLifeCycle =
                (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

        for (int i = 0; i < plantsByLifeCycle.length; i++) { // 생에주기멸 총 3개의 집합 만듦
            plantsByLifeCycle[i] = new HashSet<>();
        }

        Plant[] garden = {
                new Plant("annual plant", Plant.LifeCycle.ANNUAL),
                new Plant("perennial plant", Plant.LifeCycle.PERENNIAL),
                new Plant("biennial plant", Plant.LifeCycle.BIENNIAL),
        };

        for (Plant p : garden) {
            plantsByLifeCycle[p.lifeCycle.ordinal()].add(p); // 정원을 돌며 각 식물을 해당 집합에 넣음
        }

        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            System.out.println(String.format("%s: %s",
                    Plant.LifeCycle.values()[i], plantsByLifeCycle[i]));
        }
    }
}
~~~

- 생애주기별로 총 3개의 집합을 만들고 정원을 한 바퀴 돌면 각 식물을 해당 집합에 넣는다.
- 생애주기의 ordinal 값을 그 배열의 인덱스로 사용

### 문제점

위 코드는 동작 하지만 문제가 많다.

1. 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않는다.
2. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
3. 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다. 정수는 열거 타입과 달리 타입 안전하지 않기 때문이다.

</br >

## EnumMap

해결책으로 **열거 타입을 키로 사용하도록 설계한 `EnumMap`을 사용**하면 된다.

~~~java
public class Main {
    public static void main(String[] args) {

        Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
                new EnumMap<>(Plant.LifeCycle.class);

        for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
            plantsByLifeCycle.put(lc, new HashSet<>());
        }

        Plant[] garden = {
                new Plant("annual plant", Plant.LifeCycle.ANNUAL),
                new Plant("perennial plant", Plant.LifeCycle.PERENNIAL),
                new Plant("biennial plant", Plant.LifeCycle.BIENNIAL),
        };
        for (Plant p : garden) {
            plantsByLifeCycle.get(p.lifeCycle).add(p);
        }
        System.out.println(plantsByLifeCycle);
    }
}
~~~

- 안전하지 않은 형변환은 쓰지 않는다.
- 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 일도 없다.
- 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 없다.
- `EnumMap`의 성능이 `ordinal`을 쓴 배열에 비견된다. 그 이유는 그 내부에서 배열을 사용하기 때문이다.
  - 내부 구현 방식을 안으로 숨겨서 `Map`의 타입 안전성과 배열의 성능을 모두 얻어냈다.

여기서 `EnumMap`의 생성자가 받는 키 타입의 `Class`객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다.

</br >

## Stream 사용

Stream을 사용해 맵을 관리하면 코드를 더 줄일 수 있다.

### Stream Version1

~~~java
System.out.println(Arrays.stream(garden)
            .collect(groupingBy(p -> p.lifeCycle)));
~~~

위 코드는 `EnumMap`이 아닌 **고유한 맵 구현체를 사용**했기 때문에 `EnumMap`을 써서 얻은 **공간과 성능 이점이 사라진다는 문제**가 있다.

### Steam Version2

~~~java
System.out.println(Arrays.stream(garden)
            .collect(groupingBy(p -> p.lifeCycle,
                    () -> new EnumMap<>(Plant.LifeCycle.class), toSet())));
~~~

위 코드는 매개변수 3개짜리 `Collectors.groupingBy` 메서드를 사용하여 `mapFactory` 매개변수에 원하는 맵 구현체를 명시해 호출했다.

</br >

### EnumMap Version vs Stream Version

스트림을 사용하면 EnumMap만 사용했을 때 살짝 다르게 동작한다.

- `EnumMap` 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만든다.
- `Stream` 버전에서는 해당 생애주기가 없다면, 없는 생애주기는 만들지 않는다.
- 예를들어 `garden`에 한해살이와 여러해살이 식물만 살고 두해살이는 없다면, `EnumMap` 버전에서는 맵을 3개 만들고 `Stream` 버전에는 2개만 만든다.

</br >

## 중첩 EnumMap

두 가지 열거 타입을 매핑해야 한다고 가정해보자.

다음은 두 가지 상태(Phase)를 전이(Transition)와 매핑하도록 구현한 프로그램이다. 예컨대 액체(LIQUID)에서 고체(SOLID)로의 전이는 응고(FREEZE)가 되고, 액체에서 기체(GAS)로의 전이는 기화(BOLD)가 된다.

~~~java
import java.util.EnumMap;
import java.util.Map;
import java.util.stream.Stream;

import static java.util.stream.Collectors.groupingBy;
import static java.util.stream.Collectors.toMap;

public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
~~~

- `Map<Phase, Map<Phase, Transition>>`: 이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵
- 첫 번째 `groupingBy`: 전이를 이전 상태를 기준으로 묶음
- 두 번째 `toMap`: 이후 상태를 전이에 대응시키는 `EnumMap` 생성
  - `toMap`의 병합 함수인 `(x, y) -> y`는 선언만 하고 실제로 쓰이지는 않는다. 

위 코드에서 새로운 Phase인 플라즈마(PLASMA)를 추가해보자. 간단히 Phase에 PLASMA를 추가하고 TRANSITION에 전이 상태를 추가하면 끝이다.

~~~java
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);
        IONIZE(GAS, PLASMA)
        DEIONIZE(PLASMA, GAS);
        //...
    }
}
~~~

