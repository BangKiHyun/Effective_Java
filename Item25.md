# Item25: 톱레벨 클래스는 한 파일에 하나만 담으라

자바 컴파일러는 소스 파일 하나에 톱레베 클래스를 여러개 선언하더라도 불평하지 않는다.

하지만 득이 없을 뿐더러 심각한 위험을 감수해야 한다.

## 예제코드

### Main.java

~~~java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
~~~

Main 클래스는 다른 톱레벨 클래스 2개(Utensil과 Dessert)를 참조한다.

</br >

### Utensil.java

~~~java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
~~~

Utensil.java라는 한 파일에 Utensil.class, Dessert.class 두 클래스가 정의되어 있다.

여기서 Main을 실행하면 정상적으로 pancake를 출력한다.

</br >

### Desssert.java

~~~java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
~~~

위와 같이 똑같은 두 클래스를 담은 Dessert.java라는 파일을 만들었을 때 **어느 소스 파일을 먼저 컴파일하냐에 따라 결과가 달라지는 문제가 발생한다.**

</br >

## 해결책

- 단순히 **톱레벨 클래스들을 서로 다른 소스 파일로 분리**하면 된다.
- 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶으면 정적 멤버 클래스를 사용하는 방법을 고민해볼 수 있다.
  - 다른 클래스에 딸린 부차적인 클래스일 때 사용하면 좋다.
  - private으로 선언하면 접근 범위도 최소로 관리할 수 있다.

~~~java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
    
    private static class Utensil {
    		static final String NAME = "pan";
}

		private static class Dessert {
    		static final String NAME = "cake";
		}
}
~~~