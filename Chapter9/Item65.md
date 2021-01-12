# Item65: 리플렉션보다는 인터페이스를 사용하라

## 리플렉션 기능

- 자바의 리플렉션 기능을 이용하면 프로그램에서 임의의 클래스에 접근할 수 있다.
- Class 객체가 주어지면 그 클래스의 생성자, 메서드, 필드 인스턴스들을 가져 올 수 있다.
- 각 인스턴스를 이용해 각각에 연결된 실제 생성자, 메섣, 필드를 조작할 수 있다.

즉, 리플렉션을 이용하면 컴파일 당시에 존재하지 않던 클래스도 이용할 수 있다는 장점이 있다.

</br >

## 리플렉션 기능의 단점

- 컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없다.
- 리플렉션을 이용하면 코드가 지저분하고 장황해진다.
- 성능이 떨어진다.

</br >

## 리플렉션 사용 방법

**리플렉션은 아주 제한된 형태로만 사용하자.** 컴파일타임에 이용할 수 없는 클래스를 사용해야만 하는 프로그램은 비록 컴파일타임이라도 적절한 인터페이스나 상위 클래스를 이용할 수 있을 것이다.

이와 같은 경우 **리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자.**

</br >

### 예제

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import java.util.Arrays;
import java.util.Set;

public class ReflectionExam {
    public static void main(String[] args) {
        String hashSet = "java.util.HashSet";

        // 클래스 이름을 Class 객체로 변화
        Class<? extends Set<String>> cl = null;
        try {
            cl = (Class<? extends Set<String>>)
                    Class.forName(hashSet);
        } catch (ClassNotFoundException e) {
            fatalError("클래스를 찾을 수 없습니다.");
        }

        // 생성자를 얻는다.
        Constructor<? extends Set<String>> cons = null;
        try {
            cons = cl.getDeclaredConstructor();
        } catch (NoSuchMethodException e) {
            fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
        }

        // 집합의 인스턴스를 만든다.
        Set<String> s = null;
        try {
            s = cons.newInstance();
        } catch (IllegalAccessException e) {
            fatalError("생성자에 접근할 수 없습니다.");
        } catch (InstantiationException e) {
            fatalError("클래스를 인스턴스화할 수 없습니다.");
        } catch (InvocationTargetException e) {
            fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
        } catch (ClassCastException e) {
            fatalError("Set을 구현하지 않은 클래스입니다.");
        }

        // 생성한 집합을 사용한다.
        s.addAll(Arrays.asList("a", "b", "c").subList(1, 3));
        System.out.println(s);
    }

    private static void fatalError(String msg) {
        System.err.println(msg);
        System.exit(1);
    }
}
```

이 예는 리플렉션의 단점 두 가지를 보여준다.

- 런타임에 총 여섯 가지나 되는 예외를 던질 수 있다.
- 클래스 이름만으로 인스턴스를 생성해내기 위해 25줄이나 되는 코드를 작성했다.

