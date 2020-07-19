# Item22: 인터페이스는 타입을 정의하는 용도로만 사용하라

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.

클래스가 어떤 인터페이스를 구현한다는 것은 **자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것**이다. 인터페이스는 오직 이 이 용도로만 사용해야 한다.

</br >

## 상수 인터페이스(안좋은 예)

상수 인터페이스란 **메서드 없이, 상수를 뜻하는 static final 필드로 가득 찬 인터페이스**를 말한다.

이 상수들을 사용하려는 클래스에서 정규화된 이름을 쓰는 걸 피하고자 그 인터페이스를 구현하곤 한다.

다음 코드는 상수 인터페이스 안티패턴이다.

~~~java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGARDROS_NUMBER = 6.022_140_857e27;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
~~~

### 문제점

- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아닌 내부 구현에 해당한다. 따라서 상수 인터페이스는 **내부 구현을 클래스의 API로 노출하는 행위**다.
- 클래스가 어떤 상수 인터페이스를 사용하던 사용자에게 아무런 의미가 없다. 오히려 혼란을 주기도 하거나, 이 상수들에 종속되게 한다.
- 다음 릴리즈에서 이 상수들을 쓰지 않더라도 바이너리 호환성을 위해 여전히 상수 인터페이스를 구현하고 있어야 한다.

</br >

## 상수 공개를 위한 선택지

- 특정 클래스나 인터페이스와 **강하게 연관되 상수라면 그 클래스나 인터페이스에 자체 추가**

  - ex Integer.MAX_VALUE, Integer.MIN_VALUE

- 열거 타입으로 나타내기 적합한 상수라면 **열거 타입으로 만들어 공개**

- 인스턴스화할 수 없는 **유틸리티 클래스에 담아 공개**

  ~~~
  public class PhysicalConstants {
      private PhysicalConstants() {} //인스턴스화 방지
  
      // 아보가드로 수 (1/몰)
      static final double AVOGARDROS_NUMBER = 6.022_140_857e27;
  
      // 볼츠만 상수 (J/K)
      static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
  
      // 전자 질량 (kg)
      static final double ELECTRON_MASS = 9.109_383_56e-31;
  }
  ~~~

</br >

## 결론

인터페이스는 타입을 정의하는 요도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.

