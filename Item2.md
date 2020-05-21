## Item2 생성자 매개변수가 많은 경우에 빌더 사용을 고려해 볼 것

- static 팩토리 메서드와 pulbic 생성자 모두 매개변수가 많이 필요한 경우에 불편해진다. 
- 클래스는 몇몇 반드시 필요한 필드와 부가적인 필드를 가질 수 있는데, 그런 경우에 필수적인 매개변수를 가진 생성자에 부가적인 필드를 하나씩 추가하며 여러 생성자를 만들 수 있다.



### 해결책 1. 생성자

- 설정하고 싶은 매개변수를 최소한으로 사용하는 생성자를 사용해서 인스턴스를 만들 수 있다.
  (필수 매개변수 + 선택 매개변수)
- 선택 매개변수에서 원하는 매개변수를 포함한 생성자 중 가장 짧은 것을 골라 인스턴스를 생성한다.

~~~
public static void main(String[] args) {
     NutritionFacts cocaCola =
             new NutritionFacts(240, 8, 100, 0, 35, 27);

     NutritionFacts pocari =
             new NutritionFacts(150, 5, 70);
}
~~~

- 이런 코드는 매개변수가 많아질수록 코드를 작성하기도 어렵고 읽기도 어렵다.



### 해결책 2. 자바빈

- 아무런 매개변수를 받지 않는 생성자를 사용해서 인스턴스를 만들고, 세터를 사용해서 필요한 필드만 설정한다.

~~~
    public static void main(String[] args) {
        JavaBeans cocaCola = new JavaBeans();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
~~~

#### 단점

1. 일관성이 깨진다.
   - 인스턴스가 완성되기까지 여러번의 호출을 거쳐야 하기 때문에 자바빈이 중간에 사용되는 경우 안정적이지 않은 상태로 사용될 수 있다.
2. 불변으로 만들 수 없다.
   - getter 와 setter 가 있기 떄문에 불편 클래스로 만들지 못한다.
   - 스레드 간 공유 가능한 상태가 있음으로 스레드 안정성을 보장하려면 추가 작업이 필요하다.(locking)



### 해결책 3. 빌더 패턴

- 생성자의 안정성과 자바빈을 사용할때 얻을 수 있었던 가독성의 장점을 취했다.

~~~
public static void main(String[] args) {
        BuilderPatter builderPatter = new Builder(
                240, 8)
                .calories(100)
                .sodium(35)
                .carbohydrate(27)
                .build();
    }
~~~

- 빌더 패턴은 만들려는 객체를 바로 만들지 않고 클라이언트는 빌더에 필수적인 매개변수를 주면서 호출한다.
  여기서 빌더는 생성자 또는 static 팩토리가 될 수 있다.
- 호출을 통해 Builder 객체를 얻은 다음 빌더 객체가 제공하는 세터와 비슷한 메서드를 사용해서 부가적인 필드를 채워넣고
  build 메서드를 호출해 객체를 생성한다.



### 클래스 계층 구조에서의 빌터 패턴 활용

- 추상 빌더를 가지고 있는 추상 클래스를 만들고 하위 클래스에서는 추상 클래스를 상속받으면
  각 하위 클래스용 빌더도 추상 빌더를 상속받아 만들 수 있다.

~~~
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

        // seㅣf-type : 메서드 체이닝(형변환 없이 메서드 연쇄)을 가능하게 한다.
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
~~~

~~~
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

~~~
public static void main(String[] args) {
     NyPizza pizza = new NyPizza.Builder(SMALL)
             .addTopping(SAUSAGE)
             .build();
}
~~~

