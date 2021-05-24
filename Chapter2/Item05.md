# Item05: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

많은 클래스는 하나 이상의 자원에 의존합니다.

- 하나의 자원에 의존한다면 정적 유틸리티 클래스나, 싱글톤 방식을 써도 괜찮을 수 있습니다.
- 하지만 **사용하는 자원에 따라 동작이 달라지는 클래스라면 정적 유틸리티 클래스나 싱글톤 방식이 적합하지 않습니다.**

## 정적 유틸리티를 잘못 사용한 예

~~~java
public class SpellChecker {
    private static final Lexicon dictionary = new Lexicon(); // 특정 자원 명시

    private SpellChecker() {} // 인스턴스화 방지

    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
~~~

</br >

## 싱글톤을 잘못 사용한 예

~~~java
public class SpellChecker {
    private final Lexicon dictionary = new Lexicon(); // 특정 자원 명시

    private SpellChecker() {} // 인스턴스화 방지

    public static SpellChecker2 INSTANCE = new SpellChecker2();

    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
~~~

위의 두 방식 모두 특정 사전(Lexicon)을 명시했기 때문에 단 하나의 사전(Lexicon)만 사용할 수 있습니다. 하지만 이 방법은 좋지 않습니다. 이유는 다음과 같습니다.

- 실제 사전은 언어별로 따로 있고, 특수 어휘용 사전이나 테스트용 사전도 필요할 수 있습니다.
- 사전 하나로 이 모든 케이스를 대응하기에는 힘듭니다.

### final 한정자 제거 및 setter를 추가한다면?

그렇다면 final 한정자를 제거하고 사전(Lexicon)을 설정할 수 있는 setter를 추가한다면 괜찮을까요?

결론은 좋지 않습니다. 이 방식은 **불변성이 보장되지 않아** 오류를 내기 쉽고, 멀티스레드 환경에서는 사용할 수 없기 때문입니다.

</br >

## 해결 방법

위와 같은 상황에서는 인스턴스를 생성할 때 **생성자에 필요한 자원을 넘겨주는 의존 객체 주입 방식을 사용하면 됩니다.**

~~~java
public class SpellChecker {
    private final Lexicon dictionary; // 특정 자원 명시하지 않음
    
    public SpellChecker (Lexicon dictionary) { // 생성자를 통해 원하는 객체 주입
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
~~~

위 클래스는 맞춤법 검사기(SpellChecker)를 생성할 때 의존 객체인 사전(Lexicon)을 주입해 줍니다.

- 자원이 몇 개든 의존 관계가 어떻든 상관없이 잘 동작하며, 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있습니다. (final로 불변성을 보장해 주기 때문에)
- 의존 객체 주입 패턴의 변형으로, **생성자에 자원 팩토리를 넘겨주는 방식**이 있습니다.
  - 즉, *팩토리 메서드 패턴을 구현한 것입니다.

### 팩토리 메서드 패턴(Factory Method pattern) 

- 팩토리 메서드 패턴이란 **객체 생성 처리를 서브 클래스로 분리 해 처리하도록 캡슐화하는 패턴입니다.**
  - 즉, 객체의 생성 코드를 별도의 클래스/메서드로 분리함으로써 객체 생성의 변화에 대비하는데 유용합니다.
- 특정 기능의 구현은 개별 클래스를 통해 제공되는 것이 좋습니다.
  - 기능의 변경이나 상황에 따른 기능의 선택은 해당 객체를 생성하는 코드의 변경을 초래합니다.
  - 상황에 따라 적절한 객체를 생성하는 코드는 자주 중복될 수 있습니다.
  - 객체 생성 방식의 변화는 해당되는 모든 코드 부분을 변경해야 하는 문제가 발생합니다.

</br >

다음 코드는 클라이언트가 제공한 팩토리가 생성한 타일(Tile)로 구성된 모자이크(Mosaic)를 만드는 메서드입니다.

~~~java
Mosaic create(Supplier<? extends Tile> tileFactory) {...}
~~~

클라이언트는 타일(Tile)을 생성하는 Supplier를 정의한 후 인자로 넘겨주면, `create` 메서드에서 넘겨받은 Supplier로부터 생성되는 타일(Tile)을 가지고 모자이크(Mosaic)를 만들 수 있게 됩니다.

</br >

## 정리

- 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 정적 유틸리티 클래스와 싱글턴은 사용하지 않는 것이 좋습니다. 또한, 이 자원들을 클래스가 직접 만들게 해서도 안됩니다.
- 필요한 자원을 생성자에 넘겨주어(의존 객체 주입) 클래스의 유연성, 재사용성, 테스트 용이성을 개선할 수 있습니다.

