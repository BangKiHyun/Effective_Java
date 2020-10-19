# Item33: 타입 안전 이종 컨테이너를 고려하라

## 타입 안전 이종 컨테이너

- `Set<E>`, `Map<K, V>`처럼 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수는 제한된다.
  - Set에는 원소의 타입을 뜻하는 하나의 타입 매개변수
  - Map에는 키와 값의 타입을 뜻하는 2개 매개변수

- 하지만 타입의 수에 제약없이 유연한 수단이 필요한 경우,</br >
  예를들어 데이터베이스의 행은 임의 개수의 열을 가질 수 있는데, 모두 열을 타입 안전하게 이용하고 싶을 때이다.

- 이 때 **컨테이너 대신 키를 매개변수화**한 다음, **값을 넣거나 뺄 때 매개변수화한 키를 함께 제공**하면 된다.
  - 다양한 값을 타입 안전하게 이용할 수 있다.
  - 이러한 설계 방식을 **타입 안전 이종 컨테이너 패턴**이라 한다.

</br >

### 타입 안전 이종 컨테이너 예제 코드

~~~java
import java.util.*;

public class TypeToken {

    static class TypesafeMap {
        Map<Class<?>, Object> map = new HashMap<>();

        <T> void put(Class<T> type, T value) {
            map.put(type, value);
        }

        <T> T get(Class<T> type) {
            return type.cast(map.get(type)); //cast 메서드 사용
        }
    }

    // TYPE TOKEN
    public static void main(String[] args) {

        TypesafeMap m = new TypesafeMap();
        m.put(Integer.class, 1);
        m.put(String.class, "String");
        m.put(List.class, Arrays.asList(1, 2, 3));

        System.out.println(m.get(Integer.class));
        System.out.println(m.get(String.class));
        System.out.println(m.get(List.class));
    }
}

~~~

- `TypesafeMap.class`는 각 타입의 **Class 객체를 매개변수화한 키 역할로 사용**한다.
- 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 **메서드들이 주고받는 class 리터럴을 타입 토큰**이라 한다.
  - class  리터럴: `Class<T>`, ex) `Class<String>`, `Class<Integer>`
- `TypesafeMap.class`에서 `String`을 요청하면 `Integer`를 반환하는 일은 절대 없다. 또한 **모든 키의 타입이 제각각**이라, 일반적인 맵과 달리 **여러 가지 타입의 원소를 담을 수 있다.**
- 따라서 `TypesafeMap.class`는 타입 안전 이종 컨테이너라 할 수 있다.

</br >

### cast 메서드

~~~java
public class Class<T> {
    T cast(Object obj);
}
~~~

- cast 메서드는 형변환 연산자의 동적 버전이다.
- 이 메서드는 주어진 인수가 Class 객체가 알려주는 타입의 인스턴스인지를 검사한 다음, 맞다면 그 인수를 그대로 반환하고, 아니면 `ClassCastException`을 던진다.

### cast 메서드 사용 이유

- cast 메서드 코드를 보면 cast의 반환 타입은 Class 객체의 타입 매개변수와 같다.

- 그러므로 T로 비검사 형변환하는 손실 없이 `TypesafeMap.class`을 타입 안전하게 만들어 준다.

  - map의 value값이 Object이기 때문에 반환타입 T로 형변환을 시켜줘야 한다.

  ~~~java
  return (T) map.get(type); // T로 비검사 형변환  
  return type.cast(map.get(type)); // 비검사 형변환 없이 타입 안전하게 바꿔줌
  ~~~

  

</br >

## 두 가지 제약

### 첫 번째, Class 객체를 제네릭이 아닌 로 타입으로 넘기면 TypesafeMap 인스턴스의 타입 안전성이 쉽게 깨진다.

예를들어 `HashSet`의 로 타입을 사용하면 `HashSet<Integer>`에 `String`을 넣을 수 있다. 

~~~java
HashSet<Integer> set = new HashSet<>();
((HashSet)set).add("String");
~~~

</br >

### 해결책(동적 형변환 사용)

~~~java
        <T> void put(Class<T> type, T value) { // type safe
            map.put(type, type.cast(value));
        }
~~~

put 메서드에서 인수로 주어진 instance의 타입의 type으로 명시한 타입과 같은지 확인하면 된다.

</br >

### 두 번째, 실체화 불가 타입에는 사용할 수 없다.

- 실체화 불가 타입 `List<String>`, `List<Integer>`는 저장할 수 없다.
-  `List<String>`, `List<Integer>`는 `List.class`라는 같은 Class 객체를 공유하기 때문이다.

</br >

### 해결책(슈퍼 타입 토큰)

슈퍼 타입 토큰을 사용한다면 어느 정도 완화시킬 수 있다.

~~~java
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.*;

public class TypeToken {

    static class TypesafeMap {

        // SUPER TYPE TOKEN
        Map<TypeReference<?>, Object> map = new HashMap<>();

        <T> void put(TypeReference<T> tr, T value) {
            map.put(tr, value);
        }

        <T> T get(TypeReference<T> tr) {
            if (tr.type instanceof Class<?>) {
                return ((Class<T>) tr.type)
                        .cast(map.get(tr));
            } else {
                return ((Class<T>) ((ParameterizedType) tr.type)
                        .getRawType())
                        .cast(map.get(tr));
            }
        }
    }

    static class TypeReference<T> {
        Type type;

        public TypeReference() {
            Type superType = getClass().getGenericSuperclass();
            if (superType instanceof ParameterizedType) {
                this.type = ((ParameterizedType) superType).getActualTypeArguments()[0];
            } else throw new RuntimeException();
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (!(o instanceof TypeReference)) return false;
            TypeReference<?> that = (TypeReference<?>) o;
            return Objects.equals(type, that.type);
        }

        @Override
        public int hashCode() {
            return Objects.hash(type);
        }
    }

    public static void main(String[] args) {
      TypesafeMap m = new TypesafeMap();
        m.put(new TypeReference<Integer>() {}, 1);
        m.put(new TypeReference<String>() {}, "String");
        m.put(new TypeReference<List<Integer>>() {}, Arrays.asList(1, 2, 3));
        m.put(new TypeReference<List<String>>() {}, Arrays.asList("a", "b", "c"));

        System.out.println(m.get(new TypeReference<Integer>() {}));
        System.out.println(m.get(new TypeReference<String>() {}));
        System.out.println(m.get(new TypeReference<List<Integer>>() {}));
        System.out.println(m.get(new TypeReference<List<String>>() {}));
    }
}

~~~

