# Item49: 매개변수가 유효한지 검사하라

메서드 몸체가 실행되기 전에 매개변수를 확인한다면 잘못된 값이 넘어왔을때 즉각적이고 깔끔한 방식으로 예외를 던질 수 있다. 오류를 발생한 즉시 잡지 못하면 해당 오류를 감지하기 어려워지고, 감지하더라도 오류의 발생 지점을 찾기 어려워진다.

## 매개변수 검사를 제대로 하지 못했을때 문제점

1. 매서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다.
2. 메서드가 잘 수행되지만 잘못된 결과를 반환할 수 있다.
3. 메서드는 문제없이 수행됐지만, 어떤 객체를 이상한 상태로 만들어 미래의 알 수 없는 시점에 이 메서드와 관련없는 오류를 낼 수 있다.(실패 원자성을 어긴다.)

</br >

## public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다

1. @thows 자바독 태그를 사용하라.
2. 매개변수의 제약을 문서화한다면 그 제약을 어겼을 때 발생하는 예외도 함께 기술하라.

### 코드 예제 (BigInteger mod 메서드)

~~~java
    /**
     * Returns a BigInteger whose value is {@code (this mod m}).  This method
     * differs from {@code remainder} in that it always returns a
     * <i>non-negative</i> BigInteger.
     *
     * @param  m the modulus.
     * @return {@code this mod m}
     * @throws ArithmeticException {@code m} &le; 0
     * @see    #remainder
     */
    public BigInteger mod(BigInteger m) {
        if (m.signum <= 0)
            throw new ArithmeticException("BigInteger: modulus not positive");

        BigInteger result = this.remainder(m);
        return (result.signum >= 0 ? result : result.add(m));
    }
~~~

이 메서드는 `m`이 `null`일 때 `NullPointerException`을 던진다는 말은 메서드 설명에 없다. 이유는 설명을 개별 메서드가 아닌 `BigInteger` 클래스 수준에 기술했기 때문이다.

**클래스 수준 주석은 그 클래스의 모든 public 메서드에 적용되므로 각 메서드에 일일이 기술하는 것보다 훨씬 깔끔하다.** 

**자바 7에 추가된 `java.util.Object.requireNonNull` 메서드는 유연하고 사용하기 편하니, null 검사를 수동으로 하지 않아도 된다.**

</br >

## 공개되지 않은 메서드(public이 아닌)는 단언문(assert)을 사용해 매개변수 유효성을 검증하자

### 재귀 정렬 private 도우미 함수

~~~java
    private static void sord(long a[], int offset, int length){
        assert a != null;
        assert offset >=0 && offset <= a.length;
        assert length >=0 && length <= a.length - offset;
        //...
    }
~~~

여기서 핵심은 **단언문들은 자신이 단언한 조건이 무조건 참이라고 선언하는 것이다.**

단언문은 몇 가지 면에서 일반적인 유효성 검사와 다르다.

1. 실패하면 `AssertionError`를 던진다.
2. 런타임에 아무런 효과도, 성능 저하도 없다.
   - 단, java를 실행할 때 명령줄에서 `-ea` 혹인 `--enableassertions` 플래그를 설정하면 런타임에 영향을 준다.

</br >

## 메서드가 직접 사용하지 않으나 나중에 쓰기 위해 저장하는 매개변수는 더욱 조심하자

### int 배열의 List 뷰(view) 반환 메서드

~~~java
    static List<Integer> intArrayAsList(int[] a){
        Objects.requireNonNull(a); // null 검사
        
        return new AbstractList<Integer>() {
            @Override
            public Integer get(int i) {
                return a[i];
            }

            @Override
            public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;
                return oldVal;
            }

            @Override
            public int size() {
                return a.length;
            }
        };
    }
~~~

이 메서드에서 `Objects.requireNonNull(a)`를 이용해 null 검사를 실행한다. 

**만약 이 검사를 생략했다면 새로 생성한 List 인스턴스를 반환하는데, 클라이언트가 돌려받은 `List`를 사용하려 할 때 `NullPointerException`이 발생한다.**

그렇게 되면 `List`를 어디서 가져왔는지 추적하기 어려워 디버깅이 괴로워질 수 있다.

</br >

## 예외 케이스

- 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때
- 계산 과정에서 암묵적으로 검사가 수행될 때

예로 `Collections.sort(List)` 처럼 객체 리스트를 정렬하는 메서드를 보면, 리스트 안의 객체들은 모두 상호 비교될 수 있어야 하며, 정렬 과정에서 이 비교가 이뤄진다.

상호 비교될 수 없는 타입의 객체가 들어 있다면 그 객체와 비교할 때 `ClassCastException`을 던질 것이다.

따라서 리스트 안의 모든 객체가 상호 비교될 수 있는지 검사해봐야 별다른 실익이 없다.

