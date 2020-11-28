# Item55: 옵셔널 반환은 신중히 하라

## 메서드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 방법(자바 8 이전)

1. 예외 던지기
   - 예외는 진짜 예외적인 상황에서만 사용해야 함
   - 예외를 생성할 떄 스택 추적 전체를 캡처하므로 비용도 만만치 않음
2. null 반환
   - null을 반환할 수 있는 메서드를 호출할 때, 별도의 null 처리 코드를 추가해야함
   - null처리를 무시하면 언젠가 `NullPointerException`이 발생할 수 있음

</br >

## Optional (자바 8)

자바 버전이 8로 올라가면서 `Optional`이라는 선택지가 생겼다. `Optional<T>`는 null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다.

옵셔널은 원소를 최대 1개 가질 수 있는 '불변'컬렉션이다.

### 장점

- 특정 조건에서 아무것도 반환하지 않아야 할 때 T 대신 `Optional<T>`를 반환하면 된다.
- 옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉽다.
- null을 반환하는 메서드보다 오류 가능성이 작다.

</br >

### 컬렉션에서 최댓값을 구하는 예제

~~~java
    public static <E extends Comparable<E>> E max(Collection<E> c) {
        if(c.isEmpty()){
            throw new IllegalArgumentException("빈 컬렉션");
        }
        
        E result = null;
        for(E e : c){
            if(result == null || e.compareTo(result) > 0){
                result = Objects.requireNonNull(e);
            }
        }
        
        return result;
    }
~~~

위 메서드는 빈 컬렉션을 반환하는 `IllegalArgumentException`을 던진다. `Optional<E>`를 반환하도록 수정하면 다음과 같다.

### `Optional<E>`로 반환

~~~java
    public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
        if (c.isEmpty()) {
            return Optional.empty();
        }

        E result = null;
        for (E e : c) {
            if (result == null || e.compareTo(result) > 0) {
                result = Objects.requireNonNull(e);
            }
        }

        return Optional.of(result);
    }
~~~

- 빈 옵셔널: `Optional.empty()`
- 값은 든 옵셔널: `Optional.of(value)`
  - value에 null을 넣으면 `NullPointerException` 던짐
  - null 값 허용하는 옵셔널을 만들고 싶으면 `Optional.ofNullable(value)` 사용

**옵셔널을 반환하는 메서드에서 절대 null을 반환하지 말자.** 옵셔널을 도입한 취지를 무시하는 행위다.

</br >

## 옵셔널 활용

반환값이 없을 수도 있음을 API 사용자에게 명확히 알려줘야 할 때 옵셔널을 사용하자.

비검사 예외를 던지거나 null을 반환하면 API 사용자가 그 사실을 인지하지 못할 수 있다.

### 1. 기본값을 정해둘 수 있다.

~~~java
String lastWordInLexicon = max(words).orElse("단어 없음");
~~~

- 클라이언트가 값을 받지 못했을 때 기본값을 설정해 줄 수 있다.
- `orElse` 메서드에 기본값을 설정

### orElseGet 메서드

- `orElse` 메서드는 `Optional`과 상관없이 무조건 실행된다.

- `orElseGet` 메서드는 `Optional`의 값이 비어있을 때만 실행된다.

- `orElseGet`을 사용하면, 초기 설정 비용을 낮출 수 있다.

  ~~~java
      public static void main(String[] args) {
          final Optional<String> noneEmpty = Optional.of("noneEmpty");
          // 메서드 실행 O
          noneEmpty.orElse(noneEmptyOptional());
          // 메서드 실행 X
          noneEmpty.orElseGet(OptionalExam::noneEmptyOptional);
  
          final Optional<Object> emptyOptional = Optional.empty();
          // 메서드 실행 O
          emptyOptional.orElse(emptyOptional());
          // 메서드 실행 O
          emptyOptional.orElseGet(OptionalExam::emptyOptional);
      }
  
      private static String noneEmptyOptional() {
          System.out.println("noneEmpty optional");
          return "noneEmpty";
      }
  
      private static String emptyOptional() {
          System.out.println("empty optional");
          return "empty";
      }
  
  // 출력 결과
  // noneEmpty optional
  // empty optional
  // empty optional
  ~~~

</br >

### 2. 원하는 예외를 던질 수 있다.

~~~java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
~~~

- 위 코드는 실제 예외가 아니라 예외 팩터리를 건낸다. 이렇게 하면 예외가 실제로 발생하지 않는한 예외 생성 비용은 들지 않는다.
- orElseThrow() 메서드로 원하는 예외를 던짐

</br >

### 3. 항상 값이 채워져 있다고 가정한다.

~~~java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
~~~

- 옵셔널에 항상 값이 채워져 있다고 확신할 때 사용한다.
- 잘못 판단하면 `NoSuchElementException`이 발생한다.

</br >

## isPresent 메서드 사용은 지양해라

isPresent는 Optional의 안전밸브 역할의 메서드로, 값이 있으면 true, 없으면 false를 반환한다.

이 메서드로 원하는 모든 작업을 수행할 수 있지만 신중히 사용해야 한다. 실제로 `isPresent`를 쓴 코드 중 상당수는 앞에 언급한 메서드들로 대체 가능하다.

### 예제

~~~java
        Optional<ProcessHandle> parentProcess = ph.parent();
        System.out.println(parentProcess.isPresent() ?
                String.valueOf(parentProcess.get().pid()) : "N/A");
~~~

위 코드는 `Optional`의 `map`을 사용하여 수정할 수 있다.

~~~java
        Optional<ProcessHandle> parentProcess = ph.parent();
        System.out.println(parentProcess
                .map(processHandle -> String.valueOf(processHandle.pid()))
                .orElse("N/A"));
~~~

</br >

## 주의점

### 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸지 말자

빈 `Optional<List<T>>`를 반환하는 것 보다 빈 `List<T>`를 반환하자. 빈 컨테이너를 그대로 반환하면 클라이언트에서 옵셔널 처리 코드를 넣지 않아도 된다.

**옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 게 적절한 상황은 거의 없다.**

</br >

### 박싱된 기본 타입을 담는 옵셔널을 반환하는 일은 없도록 하자

박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무겁다. int, long, double 전용 옵셔널 클래스를 사용하자.

- `OptionalInt`,` OptionalLong`, `OptionalDouble`

</br >

## 정리

- 값을 반환하지 못할 가능성이 크고, 호출할 때마다 반환값이 없을 가능성이 있는 메서드라면 옵셔널을 반환해야 할 상황일 수 있다.
- **옵셔널 반환에는 성능 저하**기 뒤따르니, 성능에 민감한 메서드라면 null을 반환하거나 예외를 던지는 편이 나을 수 있다.
  - Optional도 새로 할당하고 초기화해야 하는 객체이고, 그 안에서 값을 꺼내려면 메서드를 호출해야 하기 때문에 한 단계를 더 거치게된다.
- 옵셔널을 반환값 이외의 용도로 쓰는 경우는 매우 드물다.

