# Item53: 가변인수는 신중히 사용하라

## 가변인수(varargs) 메서드

- 가변인수 메서드는 명시한 타입의 인수를 **0개 이상** 받을 수 있다.
- 가변인수 메서드를 호출하면, 가장 먼저 **인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장**하여 가변인수 메서드에 건네준다.

</br >

## 간단한 가변인 수 활용 예제

~~~java
    public static int sum(int... numbers) {
        int sum = 0;
        for (int number : numbers) {
            sum += number;
        }
        return sum;
    }
~~~

위와 같은 메서드는 인수가 0개 이상이면 된다.

하지만 인수가 1개 이상이어야 할 때도 있다. 다음 예를 봐보자.

</br >

## 인수가 1개 이상이어야 하는 가변인수 메서드

~~~java
    public static int min(int... numbers) {
        if (numbers.length == 0) {
            throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
        }
        int min = numbers[0];
        for (int i = 1; i < numbers.length; i++) {
            if (numbers[i] < min) {
                min = numbers[i];
            }
        }
        return min;
    }
~~~

위 코드는 최솟값을 찾는 멤서드로 인수가 1개이상 필요하다. 하지만 위 코드에는 몇가지 문제가있다.

### 문제점

1. 인수를 0개만 넣어 호출하면 **컴파일타임이 아닌 런타임에 실패한다.**
2. 가변인수(numbers)의 유효성 검사를 명시적으로 해야 한다.
3. min의 초기값을 `Integer.MAX_VALUE`로 설정하지 않고는 더 명료한 `for-each`문도 사용할 수 없다.

</br >

## 해결책

### 매개변수를 2개 받도록 변경

~~~java
    public static int min(int firstNumber, int... remainNumbers) {
        int min = firstNumber;
        for (int remainNumber : remainNumbers) {
            if (remainNumber < min) {
                min = remainNumber;
            }
        }
        return min;
    }
~~~

위와 같이 첫 번째로 평범한 매개변수를 받고, 가변인수는 두 번째로 받으면 위에서 얘기한 문제점이 전부 사라진다.

</br >

## 가변인수 성능 문제 해결책

처음에 얘기했듯이 **가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화**한다. 다행히 이 비용을 감당할 수는 없지만 가변인수의 유연성이 필요할 때 선택할 수 있는 패턴이 있다.

### 해결책

예를 들어 해당 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 할때 다음과 같이 정의하면 된다.

~~~java
    public void foo() {}
    public void foo(int a1) {}
    public void foo(int a1, int a2) {}
    public void foo(int a1, int a2, int a3) {}
    public void foo(int a1, int a2, int a3, int... rest) {}
~~~

인수가 0개인 것처럼 4새인 것까지 총 5개를 다중정의하면 마지막 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당하게된다.

위와 같은 패턴을 사용하는 예로 `EnumSet`의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화 한다.

~~~java
    public static <E extends Enum<E>> EnumSet<E> of(E e) {
        EnumSet<E> result = noneOf(e.getDeclaringClass());
        result.add(e);
        return result;
    }
    
    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2) {
        EnumSet<E> result = noneOf(e1.getDeclaringClass());
        result.add(e1);
        result.add(e2);
        return result;
    }
    
    // ...
    
    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4,
                                                    E e5)
    {
        EnumSet<E> result = noneOf(e1.getDeclaringClass());
        result.add(e1);
        result.add(e2);
        result.add(e3);
        result.add(e4);
        result.add(e5);
        return result;
    }

    @SafeVarargs
    public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest) {
        EnumSet<E> result = noneOf(first.getDeclaringClass());
        result.add(first);
        for (E e : rest)
            result.add(e);
        return result;
    }
~~~

</br >

## 정리

- 메서드를 정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자.