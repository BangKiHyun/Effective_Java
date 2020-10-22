# Item36: 비트 필드 대신 EnumSet을 사용하라

## 비트 필드 열거 상수

~~~java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값
    public void applyStyles(int styles) {
        System.out.println(String.format("styles : %s", styles));
    }
}


public class Main {
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(Text.STYLE_BOLD | Text.STYLE_ITALIC); // styles : 3
    }
}

~~~

다음과 같은식으로 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 비트 필드(bit field)라 한다.

### 문제점

비트 필드는 정수 열거 상수의 단점을 그대로 지니며, 추가로 다음과 같은 문제까지 안고 있다.

1. 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다.
2. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다.
3. 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입을 선택해야 한다.

</br >

## EnumSet 클래스

비트 필드의 대안으로 `EnumSet` 클래스가 있다. `EnumSet` 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.

### 장점

- Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 `Set` 구현체와도 함께 사용할 수 있다.
- 내부가 비트 벡터로 구현되어 있어 원소가 총 64개 이하라면 `EnumSet` 전체를 `long` 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.
- `removeAll`과` retainAll` 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다.

</br >

## EnumSet을 사용한 코드

~~~java
import java.util.Set;

public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    public void applyStyles(Set<Style> styles) {
        System.out.println(String.format("styples : %s", styles));
    }
}

import java.util.EnumSet;

public class Main {
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Text.Style.BOLD, Text.Style.ITALIC)); // styles : [BOLD, ITALIC]
    }
}
~~~

`EnusSet`은 집합 생성 등 다양한 기능의 정적 팩터리를 제공한다. 여기서는 `of` 메서드를 사용했다.

applyStyles 메서드가 `EnumSet<Style>`이 아닌 `Set<Style>`을 받음으로써 어떤 `Set` 구현체를 넘기더라도 처리할 수 있게 했다.

`EnumSet`의 유일한 단점이라면 자바 9까지는 아직 불변 `EnumSet`을 만들 수 없다는 것이다. 대안으로 `Collections.unmodifiableSet`으로 `EnumSet`을 감싸 사용할 수 있다.