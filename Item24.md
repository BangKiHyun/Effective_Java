# Item24: 멤버 클래스는 되도록 static으로 만들어라

## 중첩 클래스

- 다른 클래스 안에 정의된 클래스를 말한다.
- **중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들자**
- [중첩 클래스](https://github.com/BangKiHyun/collect-knowledge/blob/master/Java/%EC%A4%91%EC%B2%A9%20%ED%81%B4%EB%9E%98%EC%8A%A4(Nested%20Class).md)

## 정적 멤버 클래스(static)

- 정적 멤버 클래스는 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다는 점만 제외하고는 일반 클래스와 똑같다.
  - 다른 정적 멤버와 똑같은 접근 규칙을 적용받는다.(예로 private으로 선언하면 바깥 클래스에서만 접근할 수 있는 식)
- 정적 멤버 클래스는 바깥 클래스와 함께 쓰일 때 유용한 **pulbic 도우미 클래스**로 쓰인다.
- private 정적 멤버 클래스는 흔히 바깥 클래스가 표현하는 한 부분(구성요소)을 나타낼 때 쓰인다.

</br >

## 비정적 멤버 클래스(non-static)

비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다.

- 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this(클래스명.this)를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 클래스의 인스턴스와 암묵적으로 연결된다.

  ~~~java
  public class Outer {
      private final int x;
      private final int y;
  
      public Outer(final int x, final int y) {
          this.x = x;
          this.y = y;
      }
      
      public void outerMethod(){
          System.out.println("암것도 없어");
      }
  
      public class Inner {
          private final int innerX;
          private final int innerY;
  
          public Inner(final int innerX, final int innerY) {
              this.innerX = innerX;
              this.innerY = innerY;
          }
  
          public int plus() {
              Outer.this.outerMethod();
              return Outer.this.x + Outer.this.y + this.innerX + this.innerY;
          }
      }
  }
  ~~~

- 따라서 개념상 **중첩 클래스의 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야 한다.**

  - 비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없기 때문이다.

- 비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 더 이상 변경할 수 없다.

  - 보통 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스 생성자를 호출할 때 만들어진다.

  - 드물게 바깥 인스턴스 클래스.new Member Calss(args)를 호출해 수동으로 만들기도 한다.

    ~~~java
    Outer outer = new Outer(1,2);
    Outer.Inner inner = outer.new Inner(1,2);
    ~~~

    이 경우 **비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 차지하면, 생성 시간도 더 걸린다.** 사용하지 말자.

- 비정적 멤버 클래스는 어댑터를 정의할 대 자주 쓰인다.

  - 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용하는 것이다.
  - 예로 Map 인터페이스의 구현체들은 보통 자신의 컬렉션 뷰(keySet, entrySet, values 등)를 반환할 때 사용한다.

</br >

## 결론

### 멤버 클래스에서 바깥 인스턴스에 접근하 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자

- static을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 된다.
- **숨은 외부 참조로 인해 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수가 생길 수 있다.**

