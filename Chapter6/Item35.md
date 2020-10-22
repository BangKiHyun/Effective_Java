# Item35: ordinal 메서드 대신 인스턴스 필드를 사용하라

## 서론

대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응된다. 그리고 모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 `ordinal` 이라는 메서드를 제공한다.

이런 이유로 열거 타입 상수와 연결된 정숫값이 필요하면 `ordinal` 메서드를 이용하고 싶어진다.

</br >

## ordinal을 잘못 사용한 예

다음 코드는 합주단의 종류를 연주자가 1명인 솔로부터 10명인 디텍트까지 정의한 열거 타입이다.

~~~java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() {
        return ordinal() + 1;
    }
}

~~~

### 단점

- 동작은 하지만 **상수 선언 순서를 바꾸는 순간 `numberOfMusicians`가 오동작**하며, 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다.
- 값을 중간에 비워둘 수도 없다.
  - 예로 12명이 연주하는 상수를 추가한다고 가정할 때, 중간에 11명짜리 상수도 채워야 한다. **즉, dummy 상수를 같이 추가해야만 한다.**

</br >

## 해결책

해결책은 간단하다. 열거 타입 상수에 연결된 값은 `ordinal` 메서드로 얻지 말고, **인스턴스 필드에 저장하자.**

~~~java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), NONET(9), DECTET(10);

    private final int numberOfMusicians;

    Ensemble(int size) {
        this.numberOfMusicians = size;
    }

    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}
~~~

>Enum의 API 문서 ordinal 메서드 : 대부분 프로그래머는 이 메서드를 쓸 일이 없다. 이 메서드는 EnumSet과 EnumMap 같이 열거 타입기반의 범용 자료구조에 쓸 목적으로 설계되었다.

위와같은 용도가 아니라면 `oridinal`메서드는 절대 사용하지 말자.