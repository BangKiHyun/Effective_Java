# Item04: 인스턴스화를 막으려거든 private 생성자를 사용하라

정적 메서드와 정적 필드만을 담은 클래스를 쓸 때가 있습니다.

예를들어 `java.lang.Math`와 `java.util.Arrays` 처럼 기본 타입 값이나 배열 관련 메서드들을 모아놓거나, `java.util.Collections`처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아놓을 수도 있습니다.

</br >

## 정적 멤버만 담은 유틸리티 클래스의 인스턴스화?

정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한게 아닙니다. 하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어줍니다.

~~~java
public class RandomNumberGenerateUtil {
    private static final Random random = new Random();
    private static final int MAX_NUMBER = 10;

    // 생성자 없음

    public int getRandomNumber() {
        return random.nextInt(MAX_NUMBER);
    }
}
~~~

위와 같은 유틸리티성 클래스가 있다고 가정할 때, 인스턴스화가 필요하지 않다고 생각하여 생성자를 정의하지 않았습니다.

하지만 컴파일러가 자동으로 기본 생성자를 만들어 주기 때문에 아래와 같이 인스턴스화가 가능해집니다.

~~~java
public void someone(){
        RandomNumberGenerateUtil randomNumberGenerateUtil = new RandomNumberGenerateUtil();
        System.out.println("Random Number : " + randomNumberGenerateUtil.getRandomNumber());
    }
~~~

</br >

**추상 클래스로 만드는 것으로도 인스턴스화를 막을 수 없습니다.** 클래스를 상속해서 하위 클래스를 인스턴스화하면 되기 때문입니다.

~~~java
public abstract class RandomNumberGenerateUtil {
    private static final Random random = new Random();
    private static final int MAX_NUMBER = 10;

    public int getRandomNumber() {
        return random.nextInt(MAX_NUMBER);
    }
 }
 
class SubClass extends RandomNumberGenerateUtil{}

public void someone(){
    SubClass subClass = new SubClass();
}
~~~

</br >

## 인스턴스화를 막는 방법

컴파일러가 기본 생성자를 만드는 경우는 오직 명시된 생성자가 없을 때입니다. 그러므로 `private` 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있습니다.

~~~java
public class RandomNumberGenerateUtil {
    private static final Random random = new Random();
    private static final int MAX_NUMBER = 10;
    
    // 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용)
    private RandomNumberGenerateUtil(){
        throw new AssertionError();
    }

    public int getRandomNumber() {
        return random.nextInt(MAX_NUMBER);
    }
}
~~~

- AssertionError를 던질 필요는 없지만 클래스 안에서 실수로라도 생성자를 호출하지 않도록 해줍니다.
- 생성자가 존재하지만 호출할 수 없다는게 직관적이지 않기 때문에, 위 코드처럼 적절한 주석을 달아주는게 좋습니다.
- 모든 생성자는 상위 클래스의 생성자를 호출하게 되는데, **위 코드처럼 외부에 공개된 생성자가 없는 경우 상위 클래스의 생성자에 접근할 길이 막혀버립니다. 즉, 상속이 불가능합니다.**

