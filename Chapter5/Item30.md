# Item30: 이왕이면 제네릭 메서드로 만들라

매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다.

### 두 집합의 합집합 반환 코드(로 타입)

~~~java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
~~~

- 위 코드에 발생하는 두 가지 경고
  - Unchecked call to 'HashSet(Collection<? extends E>)' as a member of raw type 'java.util.HashSet'
  - Unchecked call to 'addAll(Collection<? extends E>)' as a member of raw type 'java.util.Set

### 해결책

- 메서드 선언에서 세 집합(입력 2개, 반환 1개)의 원소 타입을 타입 매개변수로 명시
- 메서드 안에서도 이 타입 매개변수만 사용하게 수정
- 타입 매개변수들을 선언은 **메서드의 제한자와 반환 타입 사이에 들어감**

타입 매개변수 목록: `<E>`, 반환 타입: `Set<E>`

### 제네릭 메서드 코드

~~~java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
~~~

**제네릭 메서드 코드의 장점**

- 경고 없이 컴파일 됨
- 타입 안전하며 쓰기도 쉬움

</br >

## 제네릭 싱글턴 팩터리

제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화할 수 있게 만들어야함

- 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 것을 제네릭 싱글턴 팩터리라 함

### Collection.emptySet 예제

~~~java
public class Collecitons{

    public static final Set EMPTY_SET = new EmptySet<>();
    ...
    @SuppressWarnings("unchecked")
    public static final <T> Set<T> emptySet() {
        return (Set<T>) EMPTY_SET;
    }
    ...
}
~~~

- 위 코드로 보았을 때 Collections에 있는 emptySet 메서드는 제네릭 메서드로 되어있다.
- 또한 emptySet은 EMPTY_SET 이라는 싱글턴 객체를 반환하는데 반환할 때  `(Set<T>)`을 통해 요청한 타입 매개변수에 맞게 바꿔주기 때문에 신경쓰지 않고 사용할 수 있다.

</br >

## 정리

- 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기 쉽다.
- 형변환을 해줘야 하는 메서드가 있다면 제네릭 메서드로 만들자.