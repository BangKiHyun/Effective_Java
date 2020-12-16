# Item63: 문자열 연결은 느리니 주의하라

## 문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례한다

문자열은 불변 아이템이라서 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야 한다.

즉, 새로운 문자열을 더할 때마다 새로운 인스턴스를 생성하기 때문에 성능 상이나 속도 면에서 비효율적이다.

~~~java
    public String statement() {
        String result = "";
        for (int i = 0; i< numItems(); i++) {
            result += lineForItem(i); // 문자열 연결(비효율적)
        }
        return result;
    }
~~~

</br >

## 성능을 포기하고 싶지 않다면 String대신 StringBuilder를 사용하자

### StringBuilder로 개선

~~~java
    public String statement() {
        StringBuilder sb = new StringBuilder(numItems() * LINE_WIDTH); // 크기 초기화
        for (int i = 0; i< numItems(); i++) {
            sb.append(lineForItem(i));
        }
        return sb.toString();
    }
~~~

`String`과 `StringBulider`는 문자를 연결하는 갯수가 많을 수록 성능 차이가 심해진다.

참고로 `StringBuilder`의 결과를 담기에 충분한 크기로 초기화 한다면 조금 더 성능 개선을 할 수 있다.

[String vs StringBuilder vs StringBuffer 간단 정리](https://github.com/BangKiHyun/collect-knowledge/blob/master/Java/String%20vs%20StringBuffer%20vs%20StringBuilder.md)

</br >

## 정리

- 성능에 신경 써야 한다면 많은 문자열을 연결할 때 문자열 연결 연산자(+)를 피하자.
  - 대신 StringBuilder의 append 메서드를 사용하자.

