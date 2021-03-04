# Item02: 생성자 매개변수가 많은 경우에 빌더 사용을 고려해 볼 것

클래스는 몇몇 반드시 필요한 부가적인 필드를 가질 수 있습니다.

그런 경우에 필수적인 매개변수를 가진 생성자에 부가적인 필드를 하나씩 추가하며 어려 생성자를 만들 수 있습니다.

- 필수 매개변수만 받는 생성자
- 필수 매개변수만 받는 생성자 + 선택 매개변수 1개를 받는 생성자
- 필수 매개변수만 받는 생성자 + 선택 매개변수 2개를 받는 생성자 등등

위와 같은 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식을 **점층적 생성자 패턴(telescoping constructor pattern)**이라 합니다.

## 점층적 생성자 패턴(Telescoping Constructor Pattern)

- 점층적 생성자 패턴은 설정하고 싶은 매개변수를 최소한으로 사용하는 생성자를 사용해서 인스턴스를 만들 수 있습니다. 
  - (필수 매개변수 + 선택 매개변수)
- 선택 매개변수에서 원하는 매개변수를 포함한 생성자 중 가장 짧은 것을 골라 인스턴스를 생성하면 됩니다.

### 예제 코드

~~~java
public class Constructor {
    private final int servingSize;  // (mL, 1회 제공량)     필수
    private final int servings;     // (회, 총 n회 제공량)  필수
    private final int calories;     // (1회 제공량당)       선택
    private final int fat;          // (g/1회 제공량)       선택
    private final int sodium;       // (mg/1회 제공량)      선택
    private final int carbohydrate; // (g/1회 제공량)       선택

    public Constructor(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public Constructor(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public Constructor(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }
  
    // ...

    public static void main(String[] args) {
        Constructor cocaCola =
                new Constructor(240, 8, 100, 0, 35, 27);

        Constructor pocari =
                new Constructor(150, 5, 70);
    }
}

~~~

### 단점

- 점층적 생성자 패턴은 **매개변수가 많아질수록 코드를 작성하기도 어렵고 읽기도 어렵습니다.**
- 코드를 읽을 때 각 값의 의미가 무엇인지 헷갈리고, 매개변수가 몇 개인지도 꼼꼼히 세어봐야 합니다.
- 클라이언트가 실수로 매개변수의 순서를 바꾸면 컴파일 오류는 내지 않지만, 런타임시 원치 않는 동작을 하게 됩니다.

</br >

## 자바빈즈 패턴(JavaBeans Pattern)

자바빈즈 패턴은 아무런 매개변수를 받지 않는 생성자를 사용해서 인스턴스를 만들고, **세터(Setter)를 사용해서 원하는 매개변수의 값을 설정하는 방식**입니다.

### 예제 코드

~~~java
public class JavaBeans {
    // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
    private int servingSize = -1; // 필수; 기본값 없음
    private int servings = -1; // 필수; 기본값 없음
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public JavaBeans() {
    }

    // Setter
    public void setServingSize(int val) { servingSize = val; }
    public void setServings(int val) { servings = val; }
    public void setCalories(int val) { calories = val; }
    public void setFat(int val) { fat = val; }
    public void setSodium(int val) { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }

    public static void main(String[] args) {
        JavaBeans cocaCola = new JavaBeans();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
~~~

### 단점

1. 일관성이 깨집니다.
   - 인스턴스가 완성되기까지 여러번의 호출을 거쳐야 하기 때문에 자바빈이 중간에 사용되는 경우 안정적이지 않은 상태로 사용될 수 있습니다.
2. 불변으로 만들 수 없습니다.
   - setter 가 있기 때문에 불변 클래스로 만들지 못합니다.
   - 스레드 간 공유 가능한 상태가 있음으로 스레드 안정성을 보장하려면 추가 작업이 필요합니다.(locking)

</br >

## 빌더 패턴(Builder Pattern)

빌더 패턴은 생성자의 안정성과 자바빈을 사용할 때 얻을 수 있었던 가독성의 장점을 취한 패턴입니다.

### 예제 코드

~~~java
public class BuilderPatter {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public BuilderPatter build() {
            return new BuilderPatter(this);
        }
    }

    private BuilderPatter(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        BuilderPatter builderPatter = new Builder(
                240, 8)
                .calories(100)
                .sodium(35)
                .carbohydrate(27)
                .build();
    }
}
~~~

- 빌더 패턴은 만들려는 객체를 바로 만들지 않고, 클라이언트는 빌더에 필수적인 매개변수를 주면서 호출합니다. 여기서 빌더는 생성자 또는 static 팩토리가 될 수 있습니다.
- 호출을 통해 Builder 객체를 얻은 다음 빌더 객체가 제공하는 Setter와 비슷한 메서드를 사용해서 부가적인 필드를 채워넣고
  build 메서드를 호출해 객체를 생성합니다.

</br >

## 클래스 계층 구조에서의 빌다 패턴 활용

- 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋습니다.
- 추상 빌더를 가지고 있는 추상 클래스를 만들고 하위 클래스에서는 추상 클래스를 상속받으면 각 하위 클래스용 빌더도 추상 빌더를 상속받아 만들 수 있습니다.

~~~java
public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}

    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build(); //하위 클래스 타입을 리턴할 때 타입 캐스팅을 없애기 위한 작업

        // seㅣf-type : 메서드 체이닝(형변환 없이 메서드 연쇄)을 가능하게 합니다.
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
~~~

~~~java
public class NyPizza extends Pizza {
    public enum Size {SMALL, MEDIUM, LARGE}

    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    @Override
    public String toString() {
        return toppings + "로 토핑한 뉴욕 피자";
    }
}

~~~

~~~java
public static void main(String[] args) {
     NyPizza pizza = new NyPizza.Builder(SMALL)
             .addTopping(SAUSAGE)
             .build();
}
~~~

