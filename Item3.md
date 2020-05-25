## private 생성자 또는 enum 타입을 사용해서 싱글톤을 보장하라

### 싱글톤

- 인스턴스를 오직 하나만 생성할 수 있는 클래스
- 단순히 함수를 사용하기 위한 유틸, 로깅, 설계상 유일해야 하는 시스템 컴포넌트를 보통 싱글턴 인스턴스로 관리
- 클래스를 싱글톤으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워 진다.



### 1) public static final 필드 방식의 싱글턴

~~~
public class Elvis1 {
    public static final Elvis1 INSTANCE = new Elvis1();

    private Elvis1() {
    }
}
~~~

- public static final Elvis1 INSTANCE = new Elvis1() 은 static 영역이 로딩될 때 딱 1번만 호출된다.
  즉, 전체 시스템에서 하나 뿐임을 보장한다.
- 해당 클래스가 싱글턴임이 API에 명백히 들어난다. final이니 절대로 다른 객체를 참조할 수 없다.



### 2) 정적 팩토리 메서드를 public static 멤버로 제공

```
public class Elvis2 {
    private static final Elvis2 INSTANCE = new Elvis2();

    private Elvis2() {
    }

    public static Elvis2 getInstance() {
        return INSTANCE;
    }
}
```

- Elvis2.getInstace() 는 항상 같은 객체의 참조를 반환하므로 싱글톤을 보장해준다.
- 이 방식이 위 방식에 비해 API 변경에 매우 유연하다. 메서드 레퍼런스로 Elvis2::getInstance 처럼 사용이 가능하다.



### 역직렬화 시 싱글턴 깨짐

- 위 두 방법 모드 역직렬화 할 때 같은 타입의 인스턴스가 여러개 생길 수 있다.
- 이미 직렬화된 Evlis 를 다시 역직렬화 할 때, private static final Elvis INSTANCE 이라면 여러 인스턴스가 생길 수 있는 것이다.
- 해결 방법으로는 transient 키워드를 추가하고, readResolve()를 구현해서 transient Elvis INSTANCE를 반환하도록 해야한다.

```
public class Elvis3 implements Serializable {
    private static final transient Elvis3 INSTANCE = new Elvis3();

    public static Elvis3 getInstance() {
        return INSTANCE;
    }

    public Object readResolve() {
        // 진짜 Elvis3를 반환하고, 가짜 Elvis3	는 GC가 처리한다.
        return INSTANCE;
    }
}
```



### 직렬화 역직렬화 알아가기

- 직렬화 : JVM의 메모리에 상주(힙 또는 스택)되어 있는 객체 데이터를 byte 형태로 변환하는 기술
- 역직렬화 : 직렬화된 byte 형태의 데이터를 객체로 변환해서 JVM으로 상주시키는 형태
- transient 
  - Serialize하는 과정에서 제외하고 싶은 경우 선언하는 키워드
  - 패스워드와 같은 보안정보가 직렬화 과정에서 제외하고 싶은 경우 적용 
- readResolve() : 역직렬화 하는 과정에 호출
  - 클래스의 멤버변수가 serializable 하지않을 경우, 이 멤버변수를 직렬화/역직렬화 해주기 위해 호출



### 	3) enum

~~~
public enum Elvis {
		INSTANCE;
}
~~~

- 1번 방식과 비슷하지만, 더 간결하고, 추가 노력 없이 직렬화 할 수 있다.
- 원소가 하나뿐인 열거 타입의 싱글톤을 만드는 가장 좋은 방법이다.
- 만들려는 싱글톤이 Enum외의 클래스를 상속해야 한다면 사용할 수 없다.