# Item28: 배열보다는 리스트를 사용하라

## 배열과 제네릭 타입 차이점

### 배열은 공변(covariant), 제네릭은 불공변(무공변성, invariant)이다

- 공변이란?
  - **자기 자신과 자식 객체를 허용한다.**
    - Sub가 Super의 하위 타입이라면 배열 Sub[]는 배열 Super[]의 하위 타입이 된다.
    - 즉, 함께 변한다.
  - Java에서 `<? extends T>`와 같다.
- 불공변이란?
  - 상속 관계에 상관없이 **자신의 타입만 허용한다.**
    - 서로 다른 타입 Type1과 Type2가 있을 때, `List<Type1>`은 `List<Type2>`의 하위 타입도 아니가 상위 타입도 아니다.
  - Java에서 `<T>`와 같다.

### 코드 예제

- 배열(convariant)

  ~~~java
  public static void main(String[] args) {
      Object[] covariant = new Long[1]; //Object가 상위 타입
      covariant[0] = "타입이 달라 넣을 수 없다.";
  }
  ~~~

  - 위 코드는 **런타임시 다음과 같은 오류를 발생**시킨다.
    - Exception in thread "main" java.lang.ArrayStoreException: java.lang.String
  - 배열에서는 이런 **오류를 런타임에야 알 수 있다.**

- 리스트(invariant)

  ~~~java
  public static void main(String[] args) {
      List<Object> invariant = new ArrayList<Long>();
      invariant.add("타입이 달라 넣을 수 없다.");
  }
  ~~~

  - 위 코드는 **컴파일타임에 오류**를 알려준다.
  - 즉, 리스트를 사용하면 **컴파일시점에 오류를 바로 알 수 있다.**

</br >

### 배열은 실체화, 제네릭은 소거된다

- 실체화
  - 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.
  - 위 코드에서 보았듯이 런타임시 Long 배열에 String을 넣을 때 ArrayStoreException을 발생시킨다.
- 소거
  - 타입 정보가 런타임에 소거된다. 이 말은 원소 타입을 컴파일타임에만 검사하면 런타임에는 알수 없다.
  - 타입 정보가 소거되면(로 타입) 제네릭이 지원되기 전의 레거시 코드와 제네릭 타입을 함께 사용할 수 있게 해준다.

이런 차이로 인해 배열과 제네릭은 잘 어우러지지 못한다. 참고로 **배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.** `new List<E>[], new List<String>[], new E[]` 와 같은 식을 말한다.

</br >

## 배열로 형변환할 때 오류가 발생한다면 리스트를 활용하자

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열인 `E[]` 대신 컬렉션인 `List<E>`를 사용하면 해결된다.

### 코드 예제

~~~java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(final Collection choices) {
        this.choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
~~~

- 위 코드는 choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 한다.
- 만약 다른 타입의 원소가 들어 있었다면 런타임에 형변환 오류가 난다.

</br >

### 제네릭으로 수정

~~~java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(final Collection<T> choices) {
        this.choiceArray = (T[]) choices.toArray(); //경고
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
~~~

- 제네릭으로 수정했지만 다음과 같은 경고가 뜬다.
  - `Unchecked cast: 'java.lang.Object[]' to 'T[]' `
- 위와같은 경고는 **T가 무슨 타입인지 알 수 없으니 컴파일러는 이 형변환이 런타임에도 안전한지 보장할 수 없다는 메시지다.**
  - 제네릭에서 원소의 타입 정보가 소거되어 런타임에 무슨 타입이 알 수 없기 때문이다.

</br >

### **Unchecked cast같은 비검사 형변환 경고를 제거하려면 배열 대신 리스트를 사용하자**

~~~java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(final Collection<T> choices) {
        this.choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
~~~

- 배열대신 리스트를 사용하면 위 코드는 오류나 경고 없이 컴파일된다.
  - 타입 안정성을 확보할 수 있다.

