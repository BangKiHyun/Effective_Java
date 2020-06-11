## Item13: clone 재정의는 주의해서 진행하라

Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 *믹스인 인터페이스다.
Cloneable 인터페이스는 Object의 protected 메서드인 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다.

인터페이스를 구현한다는 것은 일반적으로 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위다. 그런데 Cloneable은 상위 클래스에 정의된 protected 메서드의 동작 방식을 변경한 것이다.

실무에서는 Cloneable을 구현한 클래스는 clone 메서드를 pulbic으로 제공하며, 사용자는 복제가 제대로 이뤄졌다는 가정하에 사용한다.

#### 믹스인

믹스인이란 클래스가 자신의 '주된 타입'에 추가하여 구현할 수 있는 타입으로써, '주된 타입' 외의 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.



### clone 메서드의 일반 규약

이 객체의 복사본을 생성해 반환한다. '복사'의 일반적인 의도는 어떤 객체 x에 대해 다음 식은 참이다.

1. x.clone != x, 복사 객체와 원본 객체는 서로 다른 객체다.

2. x.clone().getClass() == x.getClass(),  반드시 만족해야 하는 것은 아니다.

3. x.clone().equals(x), 필수는 아니다.

4. x.clone().getClass() == x.getClass(), 관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다. 이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다는 가정하에 이 식은 참이다.

   반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

clone 메서드가 super.clone이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 정상적으로 판단한다.
하지만 이 클래스의 하위 클래스에서 super.clone을 호출한다면 잘못된 클래스의 객체가 만들어질 것이다.

~~~
public class Parent implements Cloneable {
    @Override
    protected Parent clone() {
        try {
            return new Parent();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

class Child extends Parent {
    @Override
    public Parent clone() {
        return super.clone();
    }
}

public class Main {
    public static void main(String[] args) {
        Child child = new Child();
        System.out.println(child.clone() instanceof Parent); //true
        System.out.println(child.clone() instanceof Child);	 //false
    }
}
~~~

super.clone을 사용하여 다음과 같이 구현할 수 있다.

~~~
    @Override
    protected Parent clone() {
        try {
            return (Parent) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
~~~

Parent의 clone 메서드는 Parent를 반환하게 했다. 자바가 *공변 반환 타입(covariant return typing)을 지원하기 때문에 위와같이 사용할 수 있다.

super.clone()은 checked exception인 CloneNotSupportedException을 던지기 때문에 try-catch 블록으로 감싼것을 볼 수 있다. 하지만 Parent가 Cloneable을 구현하기 떄문에, super.clone이 성공할 것을 알고 있다. 따라서 checked exception이 아닌 unchecked exception이여야 한다.


#### 공변 반환 타입(covariant return typing)

메서드의 공변 반환 타입이란 그 메서드가 오버라이딩될 때 더 좁은(narrower, 서브클래스) 타입으로 교체할 수 있다.



### clone 재정의 시 참고사항

1. clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.
2. Cloneable 구조는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다. 그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.
3. Coleanble을 구현한 클래스는 clone 메서드를 재정의 할 때, 접근 제어자를 public으로 바꾸고, throws 절을 없애야 한다.
4. 상속용 클래스는 Cloneable을 구현하면 안된다. Object를 바로 상속할 때처럼 Cloneable구현 여부를 하위 클래스에서 선택하도록 해준다.

### 요약

1. Cloneable을 구현하는 모든 클래스는 clone을 재정의해야 한다.
2. 접근 제어자를 public으로, 반환 타입은 클래스 자신으로 변경한다.
3. super.clone()을 호출한 후 객체 내부의 모든 가변 객체를 복사하고,
   복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 해야한다.



### 복사 생성자와 복사 팩토리

Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야 하지만, 그렇지 않은 상황에서는
**복사 생성자와 복사 팩터리를 사용하자!**

~~~
public Yum(Yum yum) {...} //복사 생성자
public static Tum newInstance(Yum yun) {...} //복사 팩터리
~~~

#### 장점

- 언어 모순적이고 위험천만한 객체 생성 메커니즘(생성자를 쓰지 않는 방식)을 사용하지 않는다.
- 엉성하게 문서화된 규약에 기대지 않는다.
- 정상적인 final 필드 용법과 충돌하지 않는다.
- 불필요한 예외 검사를 던지지 않는다.
- 형변환할 필요가 없다.