# Item47: 반환 타입으로는 스트림보다 컬렉션이 낫다

## 서론

원소 시퀀스를 반환할 때 당연히 스트림을 사용한다는 이야기를 들어봤을지 모르겠지만, **스트림은 반복을 지원하지 않는다.**

API를 스트림만 반환하도록 짜놓으면 반환된 스트림을 for-each로 반복하길 원하는 사용자는 불만은 토로할 것이다.

참고로 `Steam` 인터페이스는 `Iterable` 인터페이스가 정의한 추상 메서드를 전부 포함하지만, for-each로 스트림을 반복할 수 없는 까닭은 `Steam`이 `Iterable`을 확장하지 않아서다.

</br >

## 스트림을 반복하기 위한 우회 방법

```java
for (ProcessHandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator) {
  // 프로세스 처리
}
```

위와 같이 `Steam`의 `Iterator` 메서드에 메서드 참조를 건네줬다. 작동은 하지만 실전에 쓰기에 너무 난잡하고 직관성이 떨어진다.

어댑터 메서드를 사용해 바꿔보자.

### `Stream<E>`를 `Iterable<E>`로 중개해주는 어댑터

~~~java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

for (ProcessHandle ph : iterableOf(ProcessHandle.allProcesses())) {
  // 프로세스 처리
}
~~~

위와 같이 어댑터 메서드를 만들 수 있다. 이 경우 자바의 타입 추론이 문맥을 잘 파악하여 어댑터 메서드 안에서 따로 형변환하지 않아도 된다.

`Iterable`을 `Stream`으로 제공하는 경우에도 다음과 같은 어댑터를 사용하면 된다.

### `Iterable<E>`를 `Stream<E>`로 중개해주는 어댑터

```java
public static <E> Stream<E> streamOf(Iterable<E> iterable){
    return StreamSupport.stream(iterable.spliterator(), false);
}
```

하지만 공개 API를 작성할 때는 둘다 사용할 수 있게 배려해야 한다.

`Collection` 인터페이스는 `Iterable`의 하위 타입이고 `stream` 메서드도 제공한다. 따라서 **원소 시퀀스를 반환하는 공개 API의 반환 타입에는 `Collection`이나 그 하위 타입을 쓰는 게 일반적으로 최선이다.**

</br >

## 전용 컬렉션

반환하는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 `ArrayList`나 `HashSet` 같은 표준 컬렉션 구현체를 반환하는 게 최선일 수 있다.

**하지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다.**

반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현해 보자.

### 코드 예제 (멱집합)

멱집합은 원소의 갯수가 n개일 때, 원소의 갯수는 2^n이 된다.

~~~java
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
       List<E> src = new ArrayList<>(s);
       if(src.size() > 30) {
           throw new IllegalArgumentException("집합에 원소가 너무 많습니다(최대 30개).: " + s);
       }

       return new AbstractList<Set<E>>() {
           @Override
           public int size() {
               return 1 << src.size();
           }

           @Override
           public boolean contains(Object o) {
               return o instanceof Set && src.containsAll((Set) o);
           }

           @Override
           public Set<E> get(int index) {
               Set<E> result = new HashSet<>();
               for (int i = 0; index != 0; i++, index >>=1) {
                   if((index & 1) == 1) {
                       result.add(src.get(i));
                   }
               }
               return result;
           }
       };
    }
}
~~~

위와 같이 `AbstractList`를 이용해 전용 컬렉션을 손쉽게 구현할 수 있다.

단점으로는 `Stream`이나 `Iterable`은 size에 고민이 없지만, `Collection`은 스퀀스의 최대 길이가 2^31 - 1로 제한된다는 점이다. (size 메서드가 int 값을 반환하기 때문)

위에서 보았듯이 `AbstractCollection`을 활용해 `Collection` 구현체를 작성할 때는 반복 메서드 외에 `contains`와 `size`만 구현하면 된다.

**하지만 반복이 시작되기 전에는 시퀀스의 내용을 확정할 수 없는 등의 사유로 `contains`와 `size`를 구현하는 게 불가능할 때는 `Collection` 보다 `Stream`이나 `Iterable`을 반환하는 편이 낫다.**

</br >

## 어댑터를 사용 예제

~~~java
public class SubList {

    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()),
                prefixes(list).flatMap(SubList::suffixes));
    }

    public static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                .mapToObj(end -> list.subList(0, end));
    }

    public static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.rangeClosed(0, list.size())
                .mapToObj(start -> list.subList(start, list.size()));
    }
}
~~~

위 코드는 어댑터를 사용한 코드다. 이러한 어댑터는 클라이언트 코드를 어수선하고, 느리게 만든다.

다음과 같이 for 반복문을 중첩해 만들 수 있다.

~~~java
for (int start = 0; start < src.size(); start++) {
    for (int end = start + 1; end <= src.size(); end++) {
        System.out.println(src.subList(start, end));
    }
}
~~~

</br >

## 정리

- 원소 시퀀스를 반환하는 메서드를 작성할 때는, 컬렉션을 반환할 수 있다면 그렇게 하라.
- 반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 `ArraysList` 같은 표준 컬렉션에 담아 반환하라.
- 컬렉션을 반환하는 게 불가능하면 `Stream`과 `Iterable` 중 더 자연스러운 것을 반환하라.