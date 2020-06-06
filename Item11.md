## Item11: equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스에서 hashCode도 재정의해야 한다.
그렇지 않으면 HashMap이나 HashSet같은 컬렉션의 원소로 사용할 때 문제를 일으킨다.



### Object 명세 규약

1. equals 비교에서 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode메서드는 몇번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
   단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
2. equals(Object)가 두 객체가 같다고 판단했다면, hashCode는 똑같은 값을 반환해야 한다.
3. euqals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.
   단, 다른 객체에 대해서는 다른 값을 반환해야 해시 테이블의 성능이 좋아진다.

hashCode 재정의를 잘못했을 때 2번째 규약을 어기게 된다.
hashCode는 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

~~~
Map<PhoneNumber, String> map = new HashMap<>();
map.put(new PhoneNumber(123,456,789), "Bang");

System.out.println(map.get(new PhoneNumber(123,456,789))); //결과: null 
~~~

위 코드의 출력값은 "Bang"이 나와야 할 것 같지만 실제로는 null을 반환한다.
PhoneNumber클래스의 hashCode를 재정의 하지 않았기 때문에 논리적인 동치인 두 객체가 서로 다른 해시코드를 반환한다.
그 결과 get메서드는 다른 객체를 반환하게 된다.

>참고로 HashMap은 해시코드가 다른 값이라면 동치성 비교(equals)를 시도조차 하지 않도록 최적화 되어있다.

### hashCode의 안좋은 예

~~~
    @Override
    public int hashCode() {
        return 42;
    }
~~~

위 코드는 모든 객체에서 똑같은 해시코드를 반환하지만,  모든 객체에게 똑같은 값만 내어주므로 모든 객체가 해시테이블의 버킷하나에 담겨 연결 리스트처럼 동작한다.
결과적으로 평균 수행시간이 O(1)에서 O(n)으로 느려지게 된다.



### 좋은 hashCode 작성 방법

1. Int변수 result를 선언한 후 2번 방식으로 계산한 해시코드 값(c)으로 초기화한다.
2. 기본 타입필드라면 **Type.hashCode(value)**를, 필드가 배열이라면 **Arrays.hashCode**를 사용한다.
   이 떄 equals비교에 사용되지 않는 필드는 해시코드 계산 로직에서 반드시 제외한다.
3. 처음에 선언한 result값을 **result = 31 * result + c;**  와 같이 갱신한 후, result를 반환한다.

~~~
@Override public int hashCode() {
    int result = Short.hasCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
~~~



### 한 줄짜리 hashCode

Objcets 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 hash를 제공한다.
이 메서드를 활용하면 앞에서 구현한 코드와 비슷한 수준의 hashCode 함수를 한 줄로 작성할 수 있다.

~~~
@Override public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
~~~

위 코드는 상대적으로 속도가 느리다. 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱을 거쳐야 하기 때문이다. 그러므로 hash메서드는 성능에 민감하지 않는 상황에서 사용하는게 좋다.



### 해시코드의 *지연 초기화(lazy initialization)

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다 캐싱하는 방식을 고려해야 한다.
이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해둬야 한다.
해시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산하는 지연 초기화 전략을 고려하면 좋다.
다만 지연 초기화하려면 그 클래스를 스레드 안전하게 만들어야 한다.

~~~
private int hashCode; //자동으로 0으로 초기화된다.

@Override public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
~~~

#### 지연 초기화(lazy initialization) : 필드 초기화를 실제로 그 값이 쓰일 때까지 미루는 것



### 주의사항

- 성능을 높이기 위해 해시코드를 계산할 때 핵심 필드를 생략하면 안된다.
  해시 품질이 나빠져 해시테이블 성능을 떨어뜨릴 수 있다.
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 알려주지 말자.
  그래야 클라이언트가 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.