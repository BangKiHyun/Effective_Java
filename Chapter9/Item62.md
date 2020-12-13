# Item62: 다른 타입이 적절하다면 문자열 사용을 피하라

문자열(String)은 텍스트를 표현하도록 설계되었다. 하지만 문자열이 원래 의도하지 않은 용도로 쓰이는 경향이 있다.

</br >

## 문자열은 다른 값 타입을 대신하기 적합하지 않다

- 입력 받은 데이터가 수치형이라면 `int`, `float` 등 수치타입으로 변환하자.
- 예/아니오 질문의 답이라면 적절한 `enum`이나 `boolean`으로 변환하자.

기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고, 없다면 새로 하나 작성하자.

</br >

## 문자열은 열거 타입을 대신하기 적합하지 않다

- 상수를 열거할 때 문자열보다 열거 타입이 월등히 낫다.
- [Item34: int 상수 대신 열거 타입을 사용하라](https://github.com/BangKiHyun/Effective_Java/blob/master/Chapter6/Item34.md)

</br >

## 문자열은 혼합 타입을 대신하기 적합하지 않다

여러 요소가 혼합된 데이터를 하나의 문자열로  표현하는 것은 대체로 좋지 않다.

### 부적절한 예

~~~java
String compoundKey = className + "#" + i.next();
~~~

두 요소를 구분해주는 문자 "#"이 두 요소중 하나에서 쓰였다면 혼란스런 결과를 초래할 수 있다. ex): Class#name#next

### 각 요소를 개별로 접근하려면 문자열을 파싱해야 한다

- 결과적으로 느리고, 귀찮고, 오류 가능성도 커진다.

또한, 적절한 `equals`, `toString`, `compareTo` 메서드를 제공할 수 없다. **차리리 private 정적 클래스로 새로 만드는 편이 낫다.**

</br >

## 문자열은 권한을 표현하기에 적합하지 않다

### 잘못된 예 - 문자열을 사용해 권한을 구분

~~~java
public class ThreadLocal {
    private ThreadLocal() {}
 
    // 현 스레드의 값을 키로 구분해서 저장
    public static void set(String key, Object value);
    
    // 키가 가리키는 현 스레드의 값을 반환
    public static Object get(String key);
}
~~~

위 방식의 문제점으로 스레드 구분용 문자열 키가 전역 이름 공간에서 공유된다는 점이다.

만약 각 두 클라이언트가 같은 키를 쓰기로 결정한다면, 의도치 않게 같은 변수를 공유하게 된다.

</br >

### Key 클래스로 권한 구분

~~~java
public class ThreadLocal {
    private ThreadLocal() {}
    
    public static class Key { // 권한
        Key() {}
    }
    
    // 위조 불가능한 고유 키 생성
    public static Key getKey() {
        return new Key();
    }

    public static void set(Key key, Object value);
    public static Object get(Key key);
}
~~~

위조할 수 없는 고유 `Key`를 생성해서 첫 번째 코드의 문제점을 해결했다.

개선할 점으로 `set`과 `get` 메서드는 정적인 이유가 없으므로 `Key` 클래스의 인스턴스 메서드로 바꾸자. 이렇게 되면 `Key`는 더 이상 스레드 지역변수를 구분하기 위한 키가 아니라. 그 자체가 스레드 지역변수가 된다.

결과적으로 톱레벨 클래스인 `Threadlocal`의 하는 일이 없어지므로, `Key`의 이름을 `ThreadLocal`로 바꾸면 다음과 같이 된다.

~~~java
public class ThreadLocal {
    public ThreadLocal();
    public void set(Object value);
    public Object get();
}
~~~

위 코드에서 한 가지 더 개선하자면 `get`으로 얻은 `Object`를 실제 타입으로 형변환해야 하기 때문에 타입 안전하지 않다. 제네릭을 사용하여 다음과 같이 바꾸면 된다.

~~~java
public class ThreadLocal <T> {
    public ThreadLocal();
    public void set(T value);
    public T get();
}
~~~

