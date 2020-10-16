# 한정적 와일드카드를 사용해 API 유연성을 높이라

## 서론

- 매개변수화 타입은 불공변하기 떄문에 `List<String>`은 `List<Object>`의 하위 타입이 아니다.
- 따져보면 `List<Object>`에는 어떤 객체든 넣을 수 있지만 `List<String>`은 문자열만 넣을 수 있다. 즉, 리스코프 치환 원칙에 어긋난다.

하지만 불공변 방식보다 유연한 무언가가 필요할 때 한정적 와일드카드를 사용해보자.

</br >

## 한정적 와일드카드 사용 전(Stack 예제)

~~~java
public class StackCustom<E> extends Stack<E> {
    public void pushAll(Iterable<E> src) {
        for (E e : src) {
            push(e);
        }
    }
}
~~~

위 메서드는 **Interable src의 매개변수 타입이 Stack의 매개변수 타입과 일치하면 잘 동작**한다. 그렇지 않으면 오류를 발생시킨다.

다음 예는 `Stack<Number>`로 선언한 후 pushAll(Integer Type)을 넣은 예다. 즉, 매개변수 타입이 다르다.

~~~java
public static void main(String[] args) {
    StackCustom<Number> stack = new StackCustom<>();
    Iterable<Integer> integers = Arrays.asList(10, 11, 12);
    stack.pushAll(integers);
}
~~~

Integer는 Number의 하위 타입이니 잘 동작해야 겠지만 **불공변**이기 때문에 다음과 같은 오류 메시지가 뜬다.

~~~java
java: incompatible types: java.lang.Iterable<java.lang.Integer> cannot be converted to java.lang.Iterable<java.lang.Number>
~~~

</br >

## 해결책 (한정적 와일드카드 사용)

- 해결책으로는 **한정적 와일드카드 타입을 사용**하면 된다.
- pushAll의 입력 매개변수 타입은 '`E`의 `Iterable`'이 아니라 '`E`의 하위 타입의 `Iterable`'이어야 한다.
  - `Iterable<? Extends E>`

~~~java
public class StackCustom<E> extends Stack<E> {
    public void pushAll(Iterable<? extends E> src) {
        for (E e : src) {
            push(e);
        }
    }
}
~~~

</br >

## popAll 메서드

popAll메서드는 Stack 안의 모든 원소를 주어진 컬렉션으로 옮겨 담는다.

~~~java
public void popAll(Collection<E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
}
~~~

위 코드 또한 컬렉션의 매개변수 타입과 스택의 매개변수 타입이 일치한다면 컴파일된다. 그렇지 않으면 오류를 발생시킨다.

와일드카드 타입으로 해결해 보자.

~~~java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
}
~~~

위 코드는 '`E`의 `Collection`이 아니라 '`E`의 상위 타입의 `Collection`'이다. 즉, 모든 타입은 자기 자신의 상위 타입이라는 뜻이다.

</br >

## 결론

유연성을 극대화하려면 원소의 생상자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하자

</br >

## 참고

- 매개변수화 타입 T가 생산자라면 `<? extends T>`를 사용하고, 소비자라면 `<? super T>`를 사용해라. (**producer-extends, consumer-super**)
  - `pushAll`의 src 매개변수는 `Stack`이 사용할 `E` 인스턴스를 생산하므로 `extends`
  - `popAll`의 dst 매개변수는 `Stack`으로부터 `E` 인스턴스를 소비하므로 `super`
- 한정적 와일드카드 타입 기능은 자바 8부터 제대로 컴파일된다.
- `Comparable`과 `Comparator`는 모두 소비자이다.