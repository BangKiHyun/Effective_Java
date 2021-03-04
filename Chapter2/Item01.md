# Item01 : 생성자 대신 정적 팩토리 메서드를 고려하라

- 클래스는 클라이언트에게 public 생성자 대신 static 맥토리 메서드를 제공할 수 있다.
- 여기서 말하는 팩토리 메서드는 디자인 패턴에서 나오는 팩토리 메서드 패턴과 다른 의미다.

</br >

## 장점

1. ### 이름을 가질 수 있다.

~~~java
public class Foo1 {

    String name;

    public Foo1(String name) {
        this.name = name;
    }

    public static Foo1 withName(String name) {
        return new Foo1(name);
    }
}
~~~

- 위와 같이 withName이라는 이름을 갖게 되어 가독성이 좋아진다.

  ~~~java
  Foo1 foo1 = new Foo1("bang"); // 클라이언트는 "bang"이 무엇을 의미하는지 이해하기 힘들다. 
  Foo1 foo1 = Foo1.withName("bang"); //클라이언트는 "bang"이 이름을 의미하는지 명시적으로 알 수 있다.
  ~~~

</br >

2. ### 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다. 

~~~java
public class Foo2 {

    private static final Foo2 GOOD_FOO2 = new Foo2();

    private Foo2() {
    }

    public static Foo2 getFoo2() {
        return GOOD_FOO2;
    }
}
~~~

- 위와 같이 매번 인스턴스를 생성하지 않고 미리 캐싱해둔 인스턴스를 반환할 수 있다.
- 싱글톤 객체를 만들 때 이 방법을 사용할 수 있다,

</br >

3. ### 반환 타입의 하위 타입 객체를 반활할 수 있다.

~~~java
public class Foo3 implements Foo3Interface {
    private String name;

    public Foo3(String name) {
        this.name = name;
    }

    @Override
    public void print() {
        System.out.println(name);
    }
}

interface Foo3Interface {
    void print();

    static Foo3Interface createFoo3(String name) {
        return new Foo3(name);
    }
}
~~~

~~~java
    public static void main(String[] args) {
        Foo3Interface foo3Interface = Foo3Interface.createFoo3("bang");
        foo3Interface.print();
    }
~~~

-  같이 구현 클래스를 공개하지 않고 원하는 객체를 반환할 수 있게 하여 API를 작게 유지할 수 있다. (유연함이 있다.)

</br >

4. ### 리턴하는 객체의 클래스가 입력 매개변수에 따라 매번 다를 수 있다.

~~~java
public interface Foo4 {
    
    static Foo4 createFoo(boolean win){
        if(win){
            return new WinnerFoo();
        }
        return new LoserFoo();
    }
}

class WinnerFoo implements Foo4{
}

class LoserFoo implements Foo4{
}
~~~

- 위와 같이 매개 변수의 값이 true면 WinnerFoo를 false면 LoserFoo를 반환해줄 수 있다.
- 객체 타입은 노출하지 않고 감춰져 있기 때문에 유연함이 있다.

</br >

5. ### 리턴하는 객체의 클래스가 public static 팩토리 메서드를 작성할 시점에 반드시 존재하지 않아도 된다.

- 인터페이스나 클래스가 만들어 지는 시점에는 하위 타입의 클래스가 존재 하지 않아도 나중에 만들 클래스가 기존의 인터페이스나 클래스를 상속 받는 상황이라면 언제든지 의존성 주입 받아서 사용이 가능하다.

- 반환값이 인터페이스여도 되며, 정적 팩토리 메서드의 변경 없이 구현체를 바꿔 끼울 수 있다.

- JDBC를 예로들면

  - 프로바이더 등록 API : DirverManager.registerDriver()
  - 서비스 엑세스 API : DriverManager.getConnection()
  - 서비스 프로바이더 인터페이스 : Driver
  - 서비스 인터페이스 : Connection

  이를 홀용하여 JDBC라는 서비스의 규칙을 지킨 (Connection 인터페이스를 구현한 DB회사의 라이브라러리 등) 여러 DB를 사용 가능 하게된다.

</br >

## 단점

1. pulbic 또는 protected 생성자 없이 static public 메서드만 제공하는 클래스는 상속할 수 없다,
   - 이 제약은 상속보다 합성을 사용하도록 유도하고, 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서
     오히려 장점으로 받아들일 수 있다.

2. 프로그래머가 static 팩토리 메서드를 찾는게 어렵다. 
   - static 맥토리 메서드는 API 문서에서 특별히 다뤄주지 않는다.
   - 따라서 클래스나 인터페이스 문서 상단에 팩토리 메서드에 대한 문서를 제공하는 것이 좋겠다.

