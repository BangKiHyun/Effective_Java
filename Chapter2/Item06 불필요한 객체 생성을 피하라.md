# Item06: 불필요한 객체 생성을 피하라

똑같은 기능의 객체를 매번 생성하는 것 보다 객체 하나를 재사용하는 편이 나을 때가 많습니다.

만약 똑같은 기능의 객체를 매번 생성하게 된다면 쓸데없는 인스턴스가 수백만 개 만들어 질 수 있고, 이는 GC의 대상이 될 수 있습니다.

</br >

## 정적 팩토리 메서드

- `Boolean(String)` 생성자 대신 `Boolean.valueOf(String)` 팩토리 메서드를 사용하는 것이 좋습니다.
- 생성자는 호출할 때마다 새로운 객체를 만들지만, 팩토리 메서드는 그렇지 않습니다.

~~~java
boolean bad = new Boolean("TRUE"); // 생성자 호출
boolean good = Boolean.valueOf("TRUE"); // 팩토리 메서드 사용
~~~

</br >

## 생성비용이 비싼 객체

생성 비용이 '비싼 객체'가 반복해서 필요하다면 **캐싱하여 재사용**하는게 좋습니다. 

예를 들어 주어진 문자열이 유효한 로마 숫자인지를 확인하는 메서드를 작성한다고 해보겠습니다.

~~~java
static boolean isRomanNumberal(String text) {
        return text.matches("정규 표현식");
}
~~~

이 방식은 `String.matches` 메서드를 사용합니다. `matches` 메서드를 타고 들어가보면 다음과 같이 구현되어 있습니다.

![img](https://blog.kakaocdn.net/dn/m7j78/btq5GdueSpO/DWyQThFho4YuM0MUSpNOV1/img.png)

코드를 보면 `String.matches` 메서드는 `Pattern` 인스턴스를 생성하여 한 번 쓰고 버립니다. 이는 GC의 대상이 되기 때문에 성능저하가 발생합니다.

성능을 개선하려면 필요현 정규표현식을 표현하는 `Pattern` 인스턴스를 캐싱해서 사용하면 됩니다.

~~~java
private static final Pattern ROMAN = Pattern.compile("정규 표현식");

static boolean isRomanNumberal(String text){
     return;ROMAN.matcher(text);
}
~~~

이렇게 개션하면 `isRomanNumeral` 이 빈번히 호출되는 상황에서 성능을 올릴 수 있습니다.

</br >

## 오토박싱(Auto Boxing)

- 오토박싱은 또한 불필요한 객체를 생성합니다.
- 오토박싱이란 기본형타입을 그에 대응하는 박싱된 기본 타입으로 변환해줍니다.

~~~java
private static long sum() {
     Long sum = 0L;
     for (long i = 0; i <= Integer.MAX_VALUE; i++) {
         sum += i; // i의 long타입이 sum의 Long타입으로 Autoboxing됨
     }
   return sum;
}
~~~

위와 같은 상황에서 `sum` 변수를 `Long`으로 선언했습니다. 

`long` 타입인 `i`가 `Long` 타입인 `sum`에 더해질 때 마다 오토박싱이 되어 불필요한 `Long` 인스턴스가 계속 만들어집니다. 이는 큰 성능저하를 발생시킵니다.

박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의해야합니다.

