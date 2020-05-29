## Item6 : 불필요한 객체 생성을 피하라

- 똑같은 기능의 객체를 매번 생성하는 것 보다 객체 하나를 재사용하는 편이 나을 때가 많다.
- 특히 불변 객체는 언제든 재사용할 수 있다.



### 정적 팩토리 메서드

- Boolean(String) 생성자 대신 Boolean.valueOf(String) 팩토리 메서드를 사용하는 것이 좋다.
- 생성자는 호출할 때마다 새로운 객체를 만들지만, 팩토리 메서드는 그렇지 않다.

~~~
boolean bad = new Boolean("TRUE"); // 생성자 호출
boolean good = Boolean.valueOf("TRUE"); // 팩토리 메서드 사용
~~~



### 생성비용이 비싼 객체

생성 비용이 ''비싼 객체'가 반복해서 필요하다면 캐싱하여 재사용하는게 좋다. 
예를 들어 주어진 문자열이 유효한 로마 숫자인지를 확인하는 메서드를 작성한다고 해보자.

~~~
static boolean isRomanNumberal(String text) {
        return text.matches("정규 표현식");
}
~~~

이 방식은 String.matches 메서드를 사용한다. String.matches 메서드는 내부에서 Pattern 인스턴스를 생성하여 한 번 쓰고 버려져서 GC의 대상이 된다.

성능을 개선하려면 필요현 정규표현식을 표현하는 Pattern 인스턴스를 캐싱해서 사용하면 된다.

~~~
private static final Pattern ROMAN = Pattern.compile("정규 표현식");

static boolean isRomanNumberal(String text){
     return;ROMAN.matcher(text);
}
~~~

이렇게 개션하면 isRomanNumeral 이 빈번히 호출되는 상황에서 성능을 올릴 수 있다.



### 오토박싱(Auto Boxing)

- 오토박싱은 또한 불필요한 객체를 생성한다.
- 오토박싱이란 기본형타입을 그에 대응하는 박싱된 기본 타입으로 변환해준다.

~~~
private static long sum() {
     Long sum = 0L;
     for (long i = 0; i <= Integer.MAX_VALUE; i++) {
         sum += i;
     }
   return sum;
}
~~~

위와 같은 상황에서 sum 변수를 Long으로 선언했다.
long 타입인 i가 Long 타입인 sum에 더해질 때 마다 오토방식이 되어 불필요한 Long 인스턴스가 계속 만들어진다.

박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의해야한다.