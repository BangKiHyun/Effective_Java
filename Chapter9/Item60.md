# Item60: 정확한 답이 필요하다면 float와 double은 피하라

## float와 double 타입은 과학과 공학 계산용으로 설계되었다

- 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 세심하게 설계되었다.
- 따라서 **정확한 결과가 필요료 할 때 사용하면 안된다.**

</br >

## float와 double 타입은 금융 관련 계산과 맞지 않는다

0.1 혹은 10의 음의 거듭제곱 수(10^-1)를 표현할 수 없다.

### 코드 예제

~~~java
public static void main(String[] args) {
    System.out.println(1.03 - 0.42);
}
    
// 출력 결과
// 0.6100000000000001
~~~

예상한 출력 결과(0.61)와 실제 출력 결과가 다르다.

결괏값을 출력하기 전에 반올림하면 해결되리라 생각할 수 있지만, 반올림을 해도 틀린 답이 나올 수 있다.

### 오류 발생 코드 예제

~~~java
    public static void main(String[] args) {
        double fudns = 1.00;
        int itemsBought = 0;
        for (double price = 0.10; fudns >= price; price += 0.10) {
            fudns -= price;
            itemsBought++;
        }
        System.out.println(itemsBought + "개 구입");
        System.out.println("잔돈(달러): " + fudns);
    }

// 출력 결과
// 3개 구입
// 잔돈(달러): 0.3999999999999999
~~~

예상한 출력 결과(4개 구입, 잔돈 0)와 실제 출력 결과가 다르다.

</br >

## 금융 계산에는 BigDecimal, int 혹은 long을 사용하자

### BigDecimal로 수정

~~~java
    public static void main(String[] args) {
        final BigDecimal TEN_CENTS = new BigDecimal("0.10"); // 문자열 사용

        int itemsBought = 0;
        BigDecimal funds = new BigDecimal("1.00"); // 문자열 사용
        for (BigDecimal price = TEN_CENTS;
             funds.compareTo(price) >= 0;
             price = price.add(TEN_CENTS)) {
            funds = funds.subtract(price);
            itemsBought++;
        }
        System.out.println(itemsBought + "개 구입");
        System.out.println("잔돈(달러): " + funds);
    }

// 촐력 결과
// 4개 구입
// 잔돈(달러): 0.00
~~~

double 타입을 BigDecimal로 교체했다. 출력 결과도 올바르게 나온 걸 확인할 수 있다.

BigDecimal의 생성자 중 **문자열을 받는 생성자를 사용**한 이유는 계산 시 부정확한 값이 사용되는 걸 막기 위해서다.

</br >

### BigDecimal 단점

- 기본 타입보다 쓰기가 불편하고, 성능 저하를 일으킨다.
- 소수점 추적은 시스템에 맡기고, **코딩 시 불편함이나 성능 저하를 신경 쓰지 않겠다면 사용**해도 된다.

</br >

### int혹은 long 타입

- BigDecimal의 대안으로 int 혹은 long 타입을 쓸 수 있다.
- 하지만 다룰 수 있는 크기가 제한되고, 소수점을 직접 관리해야된다는 단점이 있다.
- **성능**이 중요하고 **소수점을 직접 추적**할 수 있고 **숫자가 너무 크지 않다면** int나 long을 사용하자

