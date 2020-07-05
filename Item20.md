# Item20: 추상 클래스보다는 인터페이스를 우선하라

자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상 클래스 두 가지다. 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다.(인터페이스는 자바 8부터 default 메서드를 제공 할 수 있게 됨)

## 추상 클래스 vs 인터페이스

추상 클래스: 추상 클래스를 정의한 타입을 구현한 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다.

인터페이스: 인터페이스가 선언한 메서드를 모두 정의하고 그 일반규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급된다.

</br >

## 인터페이스의 장점

### 1. 기존 클래스에 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다.

- 인터페이스가 요구하는 메서드를 추가하고, 클래스에 선언에 implements 구문만 추가하면 끝이다.
- 기존 클래스위에 추상 클래스를 확장하려면 추상 클래스는 계층구조상 두 클래스의 공통 조상이어야 한다.
  - 새로 추가된 추상 클래스의 모든 자손이 이를 상속하게 되는 것(강제성)

</br >

### 2. 믹스인 정의에 안성맞춤이다.

- 믹스인이란 클래스가 구현할 수 있는 타입으로 믹스인을 구현한 크래스에 원래의 '주된 타입'외 에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.
- 예를들어 Comparable은 자신을 구현한 클래스들의 순서를 정의할 수 있다고 선언하는 믹스인 인터페이스이다.
- 이처럼 **대상 타입의 주된 기능에 선택적 기능을 '혼합'한다고 해서 믹스인이라 부른다.**
- 추상 클래스는 기존 클래스에 덧씌울 수 없기 때문에 믹스인을 정의할 수 없다.

</br >

### 3. 계층구조가 없는 타입 프레임워크를 만들 수 있다.

타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있지만, 현실은 계층을 엄격하게 구분하기 힘들다.

예를 들어 가수 인터페이스와 작곡가 인터페이스가 있다고 해보자. 가수와 작곡가는 계층을 구분하기 힘들다.

~~~
public interface Singer {
	AudioClip sing(Song s);
}
~~~

~~~
public interface Songwriter {
	Song compose(int chartPosition);
}
~~~

이와같은 상황에서 작곡도 하는 가수를 만들기 위해 Singer와 Songwriter 모두를 구현한 새로운 인터페이스를 정의할 수 있다.

~~~
public interface SingerSongwriter extends Singer, Songwriter {
	AudioClip sing();
	Song compose(int chartPosition);
}
~~~

위와 같이 인터페이스로 만들었을 시 각각을 조합해서 새로운 인터페이스를 만들 수 있다.

하지만 추상클래스로 만들게 된다면 속성이 n개일 때 n^2개의 조합을 지원해야되는 클래스폭발 현상이 일어날 수 있다.

~~~
public abstract class Singer {
	AudioClip sing(Song s);
}

public abstract class Songwriter {
	Song compose(int chartPosition);
}

public abstract class SingerAndSongwriter {
	AudioClip sing(Song s);
	Song compose(int chartPosition);
}
~~~

</br >

## 인터페이스와 추상 골격 구현 클래스

인터터페이스와 추상 골격 구현 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법이 있다.

- 인터페이스로 타입을 정의
- 골격 구현 클래스는 필요한 메서드들을 구현
- 이렇게 해두면 단순히 골격 구현을 확장하는 것만으로 이 인터페이스를 구현하는 데 필요한 일이 대부분 완료된다.(**템플릿 메서드 패턴**)

관례상 인터페이스의 이름이 interface라면 그 골격 구현 클래스의 이름은 AbstractInterface로 짓는다.(AbstractCollection, AbstractSet 등)

골격 구현 클래스는 추상 클래스처럼 구현을 도와주는 동시에 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서 자유롭다.

~~~
//인터페이스
public interface BeverageMachine {
    void boilWater();
    void pourInCup();
    void brew();
    void addCondiment();
    void prepareRecipe();
}
~~~

~~~
//골격 구현 클래스
public abstract class AbstractBeverageMachine implements BeverageMachine {

    @Override
    public void boilWater() {
        System.out.println("물 끓이는 중");
    }

    @Override
    public void pourInCup() {
        System.out.println("컵에 따르는 중");
    }

    @Override
    public void prepareRecipe() {
        boilWater();
        brew();
        addCondiment();
        pourInCup();
    }
}
~~~

~~~
//커피에서 사용할 때
public class Coffee extends AbstractBeverageMachine {
    @Override
    public void brew() {
        System.out.println("필터를 통해서 커피를 우려내는 중");
    }

    @Override
    public void addCondiment() {
        System.out.println("설탕과 우유를 추가하는 중");
    }
}

//차에서 사용할 때
public class Tea extends AbstractBeverageMachine {
    @Override
    public void brew() {
        System.out.println("차를 우려내는 중");
    }

    @Override
    public void addCondiment() {
        System.out.println("가루를 추가하는 중");
    }
}
~~~

골격 구현 클래스를 우회적으로 이용하는 방법도 있다.

인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 것이다.

골격 구현은 기본적으로 상속해서 사용하는 걸 가정하므로 설계 및 문서화를 해야 한다.

