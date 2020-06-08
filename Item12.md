## Item12: toString을 항상 재정의하라 

Object의 기본 toString 메서드는 보통 **'클래스이름@16진수로 표시한 해시코드'** 를 반환한다.
toString의 일반 규약에 따르면 '간결하면서 사람이 읽기 쉬운 형태의 유익한 정보'를 반환해야 한다.
그러므로 toString은 모든 하위 클래스에서 재정의 하자.



### 재정의 하는 이유

~~~
   PhoneNumber me = new PhoneNumber(123,456,7890);
   System.out.println(me);
~~~

위와 같은 코드가 있을 때 toString을 재정의 하지 않았을 시 'PhoneNumber@5b2133b1' 과 같은 값을 반환한다.
디버깅시 이와 같은 값이 나온다면 그 클래스를 사용한 시스템은 디버깅하기 어려워 질 것이다.

아래와 같이 재정의 해보자.

~~~
    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }
~~~

재정의를 했을시 출력값은 '123-456-7890' 의 값을 반환한다.
이 처럼 toString을 재정의 했을 시 사용하기 편하고, 디버깅 하기 쉬워진다.



### 좋은 toString 작성 방법

- toString은 그 겍체가 가진 주요 정보 모두를 반환하는게 좋다.
- 주석으로 포맷을 명시하든 아니든 메시지의 의도는 명확히 밝혀야 한다.
- 반환값에 포함된 정보를 제공하자.
  포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.
  - PhoneNumber 클래스의 지역 코드, 프리픽스, 가입자 번호용 접근자를 제공해야 한다.
    그렇지 않으면 이 정보가 필요한 프로그래머는 toString의 반환값을 파싱할 수 밖에 없다.

~~~
    /**
     * 이 전화번호의 문자열 표현을 반환한다.
     * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
     * XXX는 지역코드, YYY는 프리픽스, ZZZ는 가입자 번호다.
     */
    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }
~~~

PhoneNumber는 위와 같이 포맷 정보를 제공해 줄 수 있다.



### 재정의하지 않아도 되는 경우

- 정적 유틸리티 클래스(상태를 갖고 있는 클래스가 아니다.)
- 열거타입(자바가 제공하는 기본 toString으로 사용하면 된다.)
- 상위 클래스에서 이미 알맞게 재정의한 경우



### 정리

모든 구체 클래스에서 Object의 toString을 재정의하자.
toString을 재정의한 클래스는 시스템을 디버깅하기 쉽게 도와준다.
toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.

