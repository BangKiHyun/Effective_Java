# Item27: 비검사 경고를 제거하라

제네릭을 사용하면 수많은 컴파일러 경고를 보게 된다.

대부분의 비검사 경고는 컴파일러가 알려주기 떄문에 쉽게 제거할 수 있다. 가능한 모든 비검사 경고를 제거하자.

비검사 경고를 제거하면 **타입 안정성이 보장**된다. 즉, **런타임에 오류가 발생할 일이 없이 의도한 대로 동작하리라 확신**할 수 있다.

</br >

## @SuppressWarnings("unchcked")

- 이 어노테이션을 사용하여 경고를 숨길수 있다.
- 경고를 제거할 수는 없지만 타입 안전하다고 확신한다면 이 어노테이션을 달자.
- 단, 타입 안전함을 검증하지 않은 채 경고를 숨기면 잘못된 보안 인식을 심어주게 된다.

</br >

### @SuppressWarnings 어노테이션은 항상 가능한 한 좁은 범위에 적용하자

- 보통 변수 선언, 아주 짧은 메서드, 생성자에 선언한다. 절대로 클래스 전체에 적용하지 말자
- 한 줄이 넘는 메서드나 생성자에 달린 @SuppressWarnings 어노테이션을 발견하면 지역변수 선언 쪽으로 옮기자.

### 코드 예제

- ArrayList에 있는 toArray 메서드

~~~java
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
~~~

- 어노테이션은 선언에만 달 수 있기 때문에 return 문에는 @SuppressWarnings를 다는게 불가능하다. 그렇기 때문에 메서드 전체에 어노테이션을 달아준 것을 볼 수 있다.
- 이렇게 메서드 전체에 달면 범위가 필요 이상으로 넓어진다. 반환값을 담을 지역변수를 하나 선언하여 그 변수에 어노테이션을 달아주면 다음과 같이 수정할 수 있다.
- 수정된 toArray

~~~java
    public <T> T[] toArray(T[] a) {
        if (a.length < size) {
            // Make a new array of a's runtime type, but my contents:
          @SuppressWarnings("unchecked") T[] result =
              (T[]) Arrays.copyOf(elementData, size, a.getClass());
        return reslt;
        }
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
~~~

</br >

### @SuppressWarnings("unchecked") 어노테이션을 사용할 때 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨주자

위에서 사용한 코드 예제를 보면 경고를 무시해도 되는 이유를 주석으로 남겨준 것을 볼 수 있다.

- 다른 사람이 그 코드를 이해하는 데 도움이 된다.
- 다른 사람이 그 코드를 잘못 수정하여 타입 안전성을 잃는 상황을 줄여준다.

</br >

## 정리

- 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻하니 최대한 제거하자.
- 경고를 없앨 방법을 못찾았으면, 그 코드가 타입 안전함을 주석으로 달아주고 @SuppressWarnings("unchecked") 어노테이션으로 경고를 숨기자.