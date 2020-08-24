# Item29: 이왕이면 제네릭 타입으로 만들어라

## 단순한 스택 코드 예제

~~~java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack(){
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e){
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop(){
        if(size == 0){
            throw new EmptyStackException();
        }
        
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
    
    public boolean isEmpty(){
        return size == 0;
    }
    
    private void ensureCapacity(){
        if(elements.length == size){
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}

~~~

### 위 클래스의 문제점

- 스택에서 꺼낼 때마다 객체를 형변환 해야 한다. 이때 런타입 오류가 날 위험이 있다.

### 제네릭으로 바꾼다면?

- 이 클래스를 제네릭으로 바꾼다고 해도 현재 버전을 사용하는 클라이언트에는 아무런 해가 없다.
- 일반 클래스를 제네릭 클래스로 만들기 위한 첫 단계는 **클래스 선언에 타입 매개변수를 추가**하는 일이다.
- **타입 이름으로 보통 E를 사용**한다.

</br >

## 제네릭 스택 첫 단계

~~~java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(){
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e){
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop(){
        if(size == 0){
            throw new EmptyStackException();
        }

        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    public boolean isEmpty(){
        return size == 0;
    }

    private void ensureCapacity(){
        if(elements.length == size){
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
~~~

- 문제가 되는 Object를 타입 매개변수(E)로 바꿨다.

### 문제점

- 위 코드에서 다음과 같은 오류가 하나 발생한다.

  ~~~
  java: generic array creation
  ~~~

- Item28에서 언급했었던 것처럼, E와 같은 실체화 불가 타입은 배열을 만들 수 없다.
- 배열을 사용하는 코드를 제네릭으로 만들 때 이 문제가 많이 발생한다.

</br >

### 해결책

1. ### 제네릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법

   ~~~java
   public class Stack<E> {
       ...
   
       public Stack() {
           elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
       }
       
       ...
   }
   ~~~

   - 다음과 같은 **경고**를 보냄

     ~~~
     Unchecked cast: 'java.lang.Object[]' to 'E[]'
     ~~~

   - 이 방법은 일반적으로 타입 안전하지 않음

   - 컴파일러는 이 프로그램이 타입 안전한지 증명할 방법이 없기 떄문에 이 비검사 형변환이 타입 안전성을 해치지 않음을 우리 스스로 확인해야 함

     - 배열 elements는 private 필드에 저장
     - 클라이언트로 반환되거나 다른 메서드에 전달되는 일 없음
     - push 메서드를 통해 배열에 저장되는 원소 타입을 항상 E
     - 따라서 이 비검사 형변환은 타입 안전함
     - 범위를 최소로 좁혀 @SuppressWarnings 애너테이션으로 해당 경고를 숨김(Item27)

     ~~~java
     public class Stack<E> {
         ...
     
         @SuppressWarnings("unchecked")
         public Stack() {
             elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY]; //경고 발생 부분
         }
         
         ...
     }
     ~~~

   </br >

2. ### elements 필드의 타입을 E[] 에서 Object[]로 바꾸는 방법

   ~~~java
   public class Stack<E> {
       private Object[] elements;
       ...
   
       public Stack() {
           elements = new Object[DEFAULT_INITIAL_CAPACITY];
       }
       ...
   
       public E pop() {
           if (size == 0) {
               throw new EmptyStackException();
           }
   
           E result = elements[--size]; //오류 발생 부분
           elements[size] = null;
           return result;
       }
       ...
   }
   ~~~

   - 다음과 같은 **오류**를 보냄

     ~~~
     incompatible types: java.lang.Object cannot be converted to E
     ~~~

   - 배열이 반환할 원소를 E로 형변환하면 오류 대신 경고가 뜸

     ~~~java
     E result = (E) elements[--size];
     
     // 경고문
     // Unchecked cast: 'java.lang.Object' to 'E' 
     ~~~

     - E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없음
     - 첫 번째 방법과 마찬가지로 @SuppressWarnings("unchecked")를 사용하여 숨길 수 있음

     ~~~java
     public class Stack_v2<E> {
         ...
     
         public E pop() {
             if (size == 0) {
                 throw new EmptyStackException();
             }
     
             @SuppressWarnings("unchecked")
             E result = (E) elements[--size];
           
             elements[size] = null;
             return result;
         }
         ...
     }
     ~~~

   </br >

   ## 첫 번째 방법 vs 두 번째 방법

   ### 첫 번째 방법 장점

   - 가독성이 더 좋다.
     - 배열의 타입을 E[]로 선언하여 오직 E 타입의 인스턴스만 받음을 확실히 보여준다.
   - 코드가 더 짧다.
     - 보통의 제네릭 클래스는 다양한 곳에서 배열을 사용한다.
     - 첫 번째 방식은 형변환을 배열 생성 시 단 한 번만 해주면 되지만, 두 번째 방식에서는 배열에서 원소를 읽을 때마다 해줘야 한다.

   보통 첫 번째 방법을 더 선호하면 자주 사용된다.

   </br >

   ### 첫 번째 방법 단점

   - 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염(heap pollution)을 일으킨다.

   > 힙 오염(heap pollution)
   >
   > 매개변수화 타입의 변수(ex: List<String>)가 타입이 다른 객체를 참조할때 발생

   - 두 번째 방법은 힙 오염이 발생하지 않아 이를 더 선호하는 프로그래머들이 있다.

</br >

## 타입 매개변수와 한정적 타입 매개변수

### 타입 매개변수

- 대다수의 제네릭 타입은 타입 매개변수에 아무런 제약을 두지 않음

  `Stack<Object>, Stack<int[]>, Stack<List<String>>, Stack`등 어떤 참조타입도 가능

- 단 기본타입은 사용할 수 없음

  - `Stack<int>, Stack<double> ` 등 컴파일 오류 발생

### 한정적 타입 매개변수

- 타입 매개변수에 제약을 두는 제네릭 타입

- 대표적인 예로 java.util.concurrent.DelayQueue가 있음

  ~~~java
  public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
      implements BlockingQueue<E> {
      
      ...
  }
  ~~~

- 타입 매개변수 목록인 `<E extends Delayed>`는 Delayed의 **하위 타입만 받는다는 뜻**

- 즉, DelayQueue 자신과 DelayQueue를 사용하는 클라이언트는 DelayQueue의 원소에서 형변환 없이 곧바로 Delayed 클래스의 메서드를 호출할 수 있음

