# Item26: 로 타입은 사용하지 말라

## 제네릭 클래스 or 제네릭 인터페이스

- 클래스와 인터페이스 선언에 타입 매개변수가 쓰이는 것을 말한다.

- 예로 List 인터페이스는 원소 타입을 나타내는 타입 매개변수 E를 받는다.

  ```java
  public interface List<E> extends Collection<E> {
  		...
  }
  ```

## 제네릭 타입

- 제네릭 클래스와 제네릭 인터페이스를 통틀어 제네릭 타입이라 한다.
- 제네릭 타입은 일력의 **매개변수화 타입(parameterized type)을 정의**한다.
- 제네릭 타입을 하나 정의하면 그에 딸린 **로 타입(raw type)**도 함께 정의된다.

</br >

## 로 타입(raw type)

- 로 타입은 타입 선언에서 제네릭 타입 정보가 전부 지워진 것처럼 동작한다.

- **로 타입은 타입 안전성이 확보되지 않는다.** 다음 예를 보자

  ~~~java
  Collection stamps = new ArrayList();
  stamps.add(new Coin());
  
  for(Iterator i = stamps.iterator(); i.hasNext();){
      Stamp stamp = (Stamp) i.next(); //여기서 문제 발생
      stamp.cancel();
  }
  ~~~

  - 이 코드는 Collection을 로 타입으로 선언했다. 도장(stamp)대신 동전(Coin)을 넣어도 아무 오류없이 컴파일되고 실행된다.
  - 오류는 컴파일할 때 발견하는 것이 좋다. 하지만 위 코드는 런타임에야 알아챌 수 있다.
    - 런타임에 문제를 겪는 코드와 원인을 제공한 코드가 물리적으로 상당히 떨어져 있을 경우 코드 전체를 훑어봐야 할 수도 있다.

- **로 타입을 쓰면 제네릭이 안겨주는 안전성과 표현력을 모두 잃게 된다.**

</br >

##  로 타입을 사용하지 말고 제네릭을 활용해라

~~~java
Collection<Stamp> stamps = new ArrayList();
~~~

- 이렇게 선언하면 stamps에 Stamp의 인스턴스만 넣어야 한다는 걸 컴파일러가 인지하게 된다.
  - 즉, 타입 안전성을 확보할 수 있다.

### `List`와 매개변수화 타입 `List<Object>`의 차이

- `List`는 제네릭 타입이에서 완전히 발을 뺀 것, `List<Object>`는 모든 타입을 허용한다는 의사를 컴파일러에게 전달

- 제네릭의 하위 타입 규칙으로

  매개변수에 `List`를 받는 메서드에 `List<String>`을 넘길수 있지만, `List<Object>`를 받는 메서드에는 넘길 수 없음

  - `List<String>`은 로 타입인 `List`의 하위 타입, `List<Object>`의 하위 타입은 아님

  ~~~java
  public class Main {
      public static void main(String[] args) {
          List<String> strings = new ArrayList<>();
  
          unsafeAdd(strings, Integer.valueOf(42));
          String s = strings.get(0); //문제 발생 지점
      }
  
      private static void unsafeAdd(final List list, final Object o) { //로 타입 사용
          list.add(o);
      }
  }
  ~~~

  - 위 코드는 컴파일은 되지만 Interger를 String으로 변화하려 할 때 ClassCastException 발생
  - 로 타입 `List`를 `List<Object>`로 바꾸면 오류 메시지가 출력되면 컴파일 조차 안됨

</br >

## 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않을때는 비한정적 와일드카드 타입을 사용하자

- 제네릭 타입은 `Set<E>`의 비한정적 와일드카드 타입은 `Set<?>`다.

### `Set<?>`과 로 타입 `Set`의 차이

- 와일드카드 타입은 안전하고, 로 타입은 안전하지 않다.

- Collection<?>에는 null 외에는 어떤 원소도 넣을 수 없다.

  ~~~java
  public class WildTest {
      public static void main(String[] args) {
          List<?> wild = new ArrayList<>();
  
          wild.add("Hello"); //컴파일 에러
          wild.add(null); //null 가능
          wild.size();
          wild.clear();
      }
  }
  ~~~

  - 위와 같이 null 외에는 어떤 원소도 넣을 수 없으며, 다른 원소를 넣으려 하면 컴파일할 때 오류를 알려준다.

- 결과적으로 컬렉션의 타입 불변식을 훼손하지 못하게 막았다.

</br >

## 로 타입 사용 예외

### class 리터럴은 로 타입을 사용해야 한다

- 자바 명세에 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다.
- List.class, String[].class, int.class는 허용
- `List<String>.class`, `List<?>.class`는 허용 안됨

### instaceof 연산자

- 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 외에 매개변수화 타입에는 적용할 수 없다.

- 로 타입이든 와일드카드 타입이든 instanceof는 똑같이 동작하므로 차라리 로 타입으로 깔끔하게 쓰는 편이 좋다.

- 다음은 instanceof를 사용한 올바른 예다.

  ~~~java
  if( o instanceof Set) { //로 타입
      Set<?> s = (Set<?>) o; //와일드카드 타입
  }
  ~~~

  

