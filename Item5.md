## Item5 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

- 많은 클래스는 하나 이상의 자원에 의존한다. 하나의 자원에 의존한다면 적적 유틸리티 클래스나, 싱글톤 방식을 써도 괜찮을 수 있지만, **사용하는 자원에 따라 동작이 달라지는 클래스라면 정적 유틸리티 클래스나 싱글톤 방식이 적합하지 않다.**

### 정적 유틸리티를 잘못 사용한 예

~~~
public class SpellChecker {
    private static final Lexicon dictionary = new Lexicon();

    private SpellChecker() {} // 인스턴스화 방지

    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
~~~

### 싱글톤을 잘못 사용한 예

~~~
public class SpellChecker {
    private final Lexicon dictionary = new Lexicon();

    private SpellChecker() {} //인스턴스화 방지

    public static SpellChecker2 INSTANCE = new SpellChecker2();

    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
~~~

위의 두 방식 모두 단 하나의 사전만 사용한다고 가정한다는 점에서 좋지 않다. 실제 사전은 언어별로 따로 있고, 특수 어휘용 사전이나 테스트용 사전도 필요할 수 있다. 사전 하나로 이 모든 케이스를 대응하기에는 힘들다.

필드에서 final 한정자를 제거하고 다른 사전으로 교체하는 메서드를 추가할 수 있지만, 이 방식은 어색하고 오류를 내기 쉬우며 멀티스레드 환경에서는 사용할 수 없다.



### 해결 방법

위와 같은 상황에서는 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 의존 객체 주입 방식을 사용하면 된다.

~~~
public class SpellChecker {
    private final Lexicon dictionary; //특정 자원 명시하지 않음
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
~~~

위 클래스는 맞춤법 검사기를 생성할 때 의존 객체인 사전을 주입해준다.

- 자원이 몇 개든 의존 관계가 어떻든 상관없이 잘 동작하며, 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있다.
- 의존 객체 주입 패턴의 변형으로, 생성자에 자원 팩토리를 넘겨주는 방식이 있다.
  즉, *팩토리 메서드 패턴을 구현한 것이다.

### 팩토리 메서드 패턴(Factory Method pattern) 

- 객체 생성 처리를 서브 클래스로 분리 해 처리하도록 캡슐화하는 패턴
  즉, 객체의 생성 코드를 별도의 클래스/메서드로 분리함으로써 객체 생성의 변화에 대비하는데 유용하다.
- 특정 기능의 구현은 개별 클래스를 통해 제공되는 것이 좋다.
  - 기능이의 변경이나 상황에 따른 기능의 선택은 해당 객체를 생성하는 코드의 변경을 초래한다.
  - 상황에 따라 적절한 객체를 생성하는 코드는 자주 중복될 수 있다.
  - 객체 생성 방식의 변화는 해당되는 모든 코드 부분을 변경해야 하는 문제가 발생한다.



다음 코드는 클라이언트가 제공한 팩토리가 생성한 타일(Tile)로 구성된 모자이크(Mosaic)를 만드는 메서드다.

~~~
Mosaic create(Supplier<? extends Tile> tileFactory) {...}
~~~

클라이언트는 Tile을 생성하는 Supplier를 정의한 후 인자로 넘겨주면, create 메서드에서 넘겨받은 Supplier로부터 생성되는 Tile을 가지고 Mosaic를 만들 수 있게 된다.



### 정리

- 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 정적 유틸리티 클래스와 싱글턴은 사용하지 않는 것이 좋다.
  이 자원들을 클래스가 직접 만들게 해서도 안된다.
- 필요한 자원을 생성자에 넘겨주어(의존 객체 주입) 클래스의 유연성, 재사용성, 테스트 용이성을 개선할 수 있다.

