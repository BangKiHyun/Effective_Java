# Item39: 명명 패턴보다 애너테이션을 사용하라

## 서론

전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다. 테스트 프레임워크은 JUnit은 버전 3까지 테스트 메서드 이름을 `test`로 시작하게했다.

## 명명 패턴 단점

1. 오타가 나면 안 된다.

   실수로 이름을 `testSafety`로 지으면 JUnit3은 이 메서드를 무시하고 지나치기 때문에 개발자는 이 테스트가 실패하지 않았으니 통과했다고 오해할 수 있다.

2. 올바른 프로그램 요소에서만 사용되라라 보증할 방법이 없다.

   메서드가 아닌 클래스 이름을 `TestSafetyMechanisms`로 지어 JUnit에 던져줬다고 해보자. 개발자는 잉 클래스에 정의된 테스트 메서드들을 수행해주길 기대하겠지만 JUnit은 클래스 이름에 관심이 없다. 경고 메시지조차 출력하지 않지만 개발자가 의도한 테스트는 전혀 수행되지 않는다.

3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.

   특정 예외를 던져야만 성공하는 테스트가 있다고 해보자. 기대하는 예외 타입을 테스트에 매개변수로 전달해야 하는 상황이다. 예외의 이름을 테스트 메서드 이름에 덧붙이는 방법도 있지만, 보기도 나쁘고 꺠지기도 쉽다.

위의 모든 문제를 해결해주는 애너테이션이 JUnit4부터 나왔다.

</br >

## 마커 애너테이션

### 마커 애너테이션 타입

~~~java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
~~~

`Test`라는 이름의 애너테이션을 정의했다. 자동으로 수행되는 간단한 테스트용 애너테이션으로, 예외가 발생하면 해당 테스트를 실패로 처리한다.

- `@Retention(RetentionPolicy.RUNTIME)`: `@Test`가 런타임에도 유지되어야 한다는 표시
- `@Target(ElementType.METHOD)`: `@Test`가 반드시 메서드 선언에만 사용돼야 한다는 표시. 따라서 클래스 선언, 필드 선언 등 다른 프로그램 요소에는 달 수 없음

### 마커 애너테이션 사용 예제

~~~java
public class Sample {
    @Test
    public static void m1() { // 성공해야 한다.
    }

    public static void m2() {
    }

    @Test
    public static void m3() { // 실패해야 한다.
        throw new RuntimeException("실패");
    }

    public static void m4() {
    }

    @Test
    public void m5() { //잘못 사용한 예: 정적 메서드 아님
    }

    public static void m6() {
    }

    @Test
    public static void m7() { // 실패해야 한다.
        throw new RuntimeException("실패");
    }

    public static void m8() {
    }
}
~~~

위 코드는 `@Test` 애너테이션을 실제 적용한 모습이다. 이와 같은 애너테이션을 "아무 매개변수 없이 단순히 대상에 마킹한다"는 뜻에서 마커 애너테이션이라 한다.

- `Sample` 클래스에는 정ㅈ거 메서드가 7개고, 그중 4개에 `@Test`를 달았다.
- `m3`와 `m7` 메서드는 예외를 던지고 `m1`과 `m5`는 그렇지 않다. 하지만 `m5`는 인스턴스 메서드이므로 `@Test`를 잘못 사용한 경우다.
- 총 4개의 테스트 메서드 중 
  - 1개 성공
  - 2개 실패
  - 1개 잘못 사용
- `@Test`를 붙이지 않은 나머지 4개의 메서드는 무시

### 마커 애너테이션 처리 프로그램

~~~java
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName("Sample");
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}

~~~

- `isAnnotationPresent()`: 실행할 메서드를 찾아주는 메서드
- `InvocationTargetException`: 테스트 메서드가 예외를 던지면 리플렉션 메커니즘이 `InvocationTargetException`으로 감싸거 다시 던진다. 그래서 이 프로그램은 `InvocationTargetException`을 잡아 원래 예외에 담긴 실패 정보를 추출해(getCause) 출력한다.
- 두 번째 catch 블록: `InvocationTargetException`외의 예외가 발생한다면 발생한 예외를 붙잡아 적절한 오류 메시지를 출력한다.

</br >

## 매개변수 하나를 받는 애너테이션

### 매개변수 하나를 받는 애너테이션 타입

~~~java
import java.lang.annotation.*;

/**
 * 명시한 예외를 던져야만, 성공하는 테스트케이스 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> values();
}
~~~

이 애너테이션의 매개변수 타입은 `Class<? extends Throwable>`이다. "Throwable을 확장한 클래스의 Class 객체"라는 뜻이다. 따라서 모든 예외와 오류 타입을 다 수용한다.

### 매개변수 하나를 받는 애너테이션 사용 예제

~~~java
public class Sample2 {
    @ExceptionTest(values = ArithmeticException.class)
    public static void m1() { // 성공해야 한다.
        int i = 0;
        i = i / i;
    }

    @ExceptionTest(values = ArithmeticException.class)
    public static void m2() { // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }

    @ExceptionTest(values = ArithmeticException.class)
    public static void m3() { // 실패해야 한다. (예외가 발생하지 않음)
    }
}
~~~

### 매개변수 하나를 받는 애너테이션 처리 프로그램

~~~java
if(m.isAnnotaionPresent(ExceptionTest.class)) {
    test++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (InvocationTargetException wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
        if(excType.isInstance(exc)) {
            passed++;
        } else {
            System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc);
        }
    } catch (Exception e) {
        System.out.println("잘못 사용한 @ExceptionTest: " + m);
    }
}
~~~

이 코드는 애너테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인하는데 사용한다.</br >
테스트 프로그램이 문제없이 컴파일되면 애너테이션 매개변수가 가리키는 예외가 올바른 타입이라는 뜻이다.

</br >

## 배열 매개변수를 받는 애너테이션

예외를 여러 개 명시하고 그중 하나가 발생하면 성공하게 만들 수도 있다. `@ExceptionTest` 애너테이션의 매개변수 타입을 `Class` 객체의 배열로 수정해보자.

### 배열 매개변수를 받는 애너테이션 타입

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable>[] values();
}
```

단일 원소 배열에 최적화했지만, 앞서의 `@ExceptionTest`들도 모두 수정 없이 수용한다.

### 배열 매개변수를 받는 애너테이션 사용 예제

~~~java
import java.util.ArrayList;
import java.util.List;

public class Sample3 {
    @ExceptionTest(values = {IndexOutOfBoundsException.class, NullPointerException.class})
    public static void doubleyBad() {
        List<String> list = new ArrayList<>();
        // 자바 명세에 따르면, 다음 메서드는 IndexOutOfBoundsException이나,
        // NullPointerException을 던질 수 있다.
        list.add(5, null);
    }
}
~~~

원소가 여럿인 배열을 지정할 때 위과 같이 원소들을 중괄호로 감싸고 쉼표로 구분해주기만 하면 된다.

### 배열 매개변수를 받는 애너테이션 처리 프로그램

~~~java
if(m.isAnnotaionPresent(ExceptionTest.class)) {
    test++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (Throwable wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        int oldPassed = passed;
        Class<? extends Throwable>[] excTypes = m.getAnnotation(ExceptionTest.class).value();
        for(Class<? extends Throwable> excType : excTypes) {
            if(excType.isInstance(exc)) {
            	passed++;
              break;
            }
        }
        if(passed == oldPassed) {
            System.out.println("테스트 %s 실패: %s %n", m, exc);
        }
    }
}
~~~

`Class<? extends Throwable>[]`를 배열로 받아 이미 지정한 `Exception`이면 passed를 더하는 코드로 바뀌었다.

</br >

## 반복 가능한 애너테이션

- 자바 8에서는 여러 개의 값을 받는 애너테이션을 다른 방식으로도 만들 수 있다.
- 배열 매개변수를 사용하는 대신 애너테이션에 `@Repeatable` 메타애너테이션을 다는 방식이다.

### 주의할 점

1. `@Repeatable`을 단 애너테이션을 반환하는 '컨테이너 애너테이션'을 하나 더 정의한다.
2. `@Repeatable`에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
3. 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 `value` 메서드를 정의해야 한다.
4. 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)과 적용 대상(@Target)을 명시해야 한다. 그렇지 않으면 컴파일X

### 반복 가능한 애너테이션 타입

~~~java
// 반복 가능한 애너테이션 타입
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

// 컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
~~~

### 반복 가능한 애너테이션 사용 예제

~~~java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {...}
~~~

반복 가능 애너테이션을 여러 개 달면 하나만 달았을 때와 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용된다. 

- `getAnnotationByType` 메서드: 이 둘을 구분하지 않아서 반복 애너테이션과 그 컨테이너 애너테이션을 모두 가져온다.
- `isAnnotationPresent` 메서드: 이 둘을 명확히 구분한다.
  - 반복 가능 애너테이션을 여러 번 단 다음 `isAnnotationPresent`로 반복 가능 애너테이션이 달렸는지 검사한다면 "그렇지 않다"라 알려준다. </br >
    그 결과 애너테이션을 여러 번 단 메서드들을 모두 무시하고 지나친다.
  - 같은 이유로, `isAnnotationPresent`로 컨테이너 애너테이션이 달렸는지 검사한다면 반복 가능 애너테이션을 무시하고 지나친다.
  - 그래서 달려 있는 수와 상관없이 모두 검사하려면 둘을 따로따로 확인해야 한다. 

### 반복 가능 애너테이션 처리 프로그램

~~~java
if (m.isAnnotationPresent(ExceptionTest.class) || m.isAnnotationPresent(ExceptionTestContainer.class)) {
  tests++;
}
try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (InvocationTargetException wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        int oldPassed = passed;
    
        ExceptionTest[] excTests = m.getAnnotationByType(ExceptionTest.class);
        for(ExceptionTest excType : excTypes) {
            if(excType.isInstance(exc)) {
            	passed++;
                break;
            }
        }
        
        if(passed == oldPassed) {
            System.out.println("테스트 %s 실패: %s %n", m, exc);
        }
}
~~~

</br >

## 정리

- 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.
- 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다.