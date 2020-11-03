# Item40: @Override 애너테이션을 일관되게 사용하라

## 서론

`@Override`는 메서드 선언에만 달 수 있으며, 이 애너테이션이 달렸다는 것은 상위 타입의 메서드를 재정의했음을 뜻한다. 이 애너테이션을 일관되게 사용하면 여러 버그들을 예방해준다.

</br >

## 버그 발생 코드

~~~java
import java.util.HashSet;
import java.util.Set;

public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram o) {
        return first == o.first && second == o.second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
            System.out.println(s.size()); // 260
        }
    }
}
~~~

위 코드는 똑같은 소문자 2개로 구성된 바이그램 26개를 10번 반복해 집합에 추가한 다음, 그 집합의 크기를 출력한다.</br > `Set`은 중복을 허용하지 않으니 **26**이 출력될 거 같지만, 실제로 **260**이 출력된다.

**분명히 `equals` 메서드를 재정의 한 것처럼 보이고, `hasCode`도 함께 재정의 했다.**

하지만 `equals`를 **재정의(overrideng)한 게 아니라 다중정의(overloading) 해버렸다.**</br >`Object`의 `equals`를 재정의하려면 매개변수 타입을 `Object`로 해야만 한다. 그렇게 하지 않았기 때문에 `equals`를 새로 정의한 꼴이 되었다,

~~~java
// Object의 equals 메서드
public boolean equals(Object obj) {
    return (this == obj);
}
~~~

위와 같이 `Object`의 `equals`메서드는 **객체 식별성(identity)만을 확인**하기 때문에 Bigram 10개 각각이 서로 다른 객체로 인식되어 결국 260을 출력한 것이다.

</br >

## @Override 애너테이션 추가

~~~java
@Override
public boolean equals(Bigram o) {
    return first == o.first && second == o.second;
}
~~~

위와 같이 @Override 애너테이션으로 재정의한다는 의도를 명시하고, 다시 컴파일하면 다음의 컴파일 오류가 발생한다.

~~~java
Method does not override method from its superclass
~~~

**이처럼 잘못된 부분을 명확히 알려주므로 곧장 올바르게 수정할 수 있다.**

~~~java

public class Bigram {
    //...

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Bigram)) return false;
        Bigram bigram = (Bigram) o;
        return first == bigram.first &&
                second == bigram.second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }
        System.out.println(s.size()); // 26
    }
}
~~~

**상위 클래스의 메서드를 재정의하려는 모든 메서드에 `@Override` 애너테이션을 달자.**

예외 케이스가 한 가지 존재한다. 구체 클래스에서 상위 클래스의 추상 메서를 재정의할 때는 굳이 `@Override`를 달지 않아도 된다. 컴파일러가 알아서 알려주기 때문이다.



