# Item50: 적시에 방어적 복사본을 만들라

자바로 작성한 클래스는 시스템의 다른 부분에서 무슨 짓을 하든 그 불변식이 지켜진다. 하지만 아무리 자바라 해도 다른 클래스로부터의 침범을 아무런 노력 없이 다 막을 수 있는 건 아니다.

**클라이언트가 불변식을 깨뜨리려 혈안이 되어 있다고 가정하고 방어적으로 프로그래밍해야 한다.**

</br >

## 불변식을 지키지 못한 예제

기간(period)를 표현하는 다음 클래스는 한번 값이 정해지면 변하지 않도록 할 생각이다.

### 기간을 표현하는 클래스

~~~java
import java.util.Date;

public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param start 시작 시각
     * @param end   종료 시각; 시작 시각보다 뒤어야 한다.
     * @throws IllegalArgumentException 시작 시간이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException     start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(
                    start + "가" + end + " 보다 늦다.");
        }
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
}
~~~

위 코드는 불변처럼 보이지만 `Date`가 가변이기 때문에 그 불변식을 꺠뜨릴 수 있다.

</br >

### Period 인스턴스 내부 공격 1

~~~java
    public static void main(String[] args) {
        Date start = new Date();
        Date end = new Date();
        final Period period = new Period(start, end);
        end.setYear(78); // period의 내부 수정
    }
~~~

간단한 해결 방안으로 자바 8 이후라면 `LocaDateTime`이나 `ZonedDateTime`을 사용하면 된다.

**`Date`는 낡은 API이니 새로운 코드를 작성할 때는 더 이상 사용하면 안된다.**

`Date`처럼 가변 값 타입을 안전하게 사용하려면 **생성자에서 받은 가변 매개변수 각각을 방어적으로 복사해야 한다.**

</br >

### Period 인스턴스 내부 공격 2

~~~java
    public static void main(String[] args) {
        Date start = new Date();
        Date end = new Date();
        Period p = new Period(start, end);
        p.end().setTime(78); // period의 내부 변경
    }
~~~

접근자 메서드가 내부의 가변 정보를 직접 드러내기 때문에 발생한다.

해결책으로 단순히 접근자가 **가변 필드의 방어적 복사본을 반환하면 된다.**

</br >

## 매개변수의 방어적 복사본

### 방어적 복사본 1 (매개변수 방어적 복사본)

~~~java
    public Period(Date start, Date end) {
        // 유효성 검사 전 방어적 복사본 생성
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        
        // 복사본을 통한 유효성 검증
        if (this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(
                    this.start + "가" + this.end + " 보다 늦다.");
        }
    }
~~~

`Period`인스턴스 안에서 원본이 아닌 복사본을 사용한다. 이렇게 되면 앞서의 공격은 더 이상 `Period`에 위협이 되지 않는다.

</br >

### TOCTOU 공격 (time-of-check/time-of-use)

TOCTOU은 **검사시점/사용시점** 공격이라 한다. **멀티스레드 환경에서 원본 객체의 유효성을 검사한 후 복사본을 만드는 그 찰나의 취약한 순간에 다른 스레드가 원본 객체를 수정하는 위험**이 있다.

**`Period` 생성자를 보면 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사한 점에 주목하자.** 순서가 부자연 스러워 보일 수 있지만 TOCTOU 공격을 예방할 수 있기 때문에 반드시 이렇게 작성해야 한다.

</br >

### 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해선 안된다.

방어적 복사에 `Date`는 `clone` 메서드를 사용하지 않았다. `clone` 메서드는 매개변수가 final 클래스가 아니기 때문에 상속이 가능한 타입이라면 사용해선 안된다.

예로 `Date`를` clone` 메서드를 통해 복사를 할 수도 있으나 `Date` 클래스를 상속한 클래스가 재정의하여 하위 클래스의 인스턴스를 반활 수도 있다.

</br >

### 방어적 복사본 2 (필드의 방어적 복사본)

~~~java
    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }
~~~

접근자가 가변 필드의 방어적 복사본을 반환하게 리팩터링했다.

이제 `Period` 자신 말고는 가변 필드에 접근할 방법이 없으므로 완벽한 불변으로 거듭났다.

</br >

## 정리

- 클래스가 클라이언트로부터 받은 혹은 클라이언트로 반환하는 구성요소가 가변이라면 그 요소는 반드시 방어적으로 복사하자.
- 복사 비용이 너무 크거나 클라이언트가 그 요소를 잘못 수정할 일이 없음을 신뢰한다면 방어적 복사본을 수행하는 대신</br >해당 구성요소를 수정했을때의 책임이 클라이언트에게 있음을 문서에 명시하도록 하자.