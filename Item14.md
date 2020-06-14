## Item14: Comparable을 구현할지 고려하라

Comparable 인터페이스는 compareTo 메서드가 있다. compareTo 메서드는 단순 동치성 비교와 순서까지 비교할 수 있으며, 제네릭 하다.
클래스가 Comparable을 구현한다면 클래스의 인스턴스간에 자연적인 순서가 있음을 뜻한다.
알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.

</br >

### compareTo 메서드의 일반 규약

equals의 규약과 비슷하다.

- 이 객체가 주어진 객체의 순서를 비교한다.
- 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다.
- 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
- Comparable을 구현한 클래스는 모든 x,y에 대해 (x.compareTo(y) == -y.compareTo(x))여야 한다.
  즉, 두 객체의 참조 순서를 바꿔서 비교해도 예상한 결과가 나와야 한다.
- Comparable을 구현한 클래스는 추이성을 보장해야 한다.
  즉, x.compareTo(y) > 0 && y.compareTo(z) 이면 x.compareTo(z) > 0 이다.
- x.compareTo(y) == 0 이면, x.compareTo(z) == y.compareTo(z) 다.
  즉. 크기가 같은 객체는 어떤 객체와 비교하더라도 항상 같이야 한다.
- (x.compareTo(y) == 0) == (x.compareTo(y))여야 한다.
  즉, compareTo의 동치성 결과가 equals와 같아야 한다.
  이 규약은 필수는 아니지만 지키는걸 권한다. 이를 잘 지키면 compareTo로 줄지은 순서와 equals의 결과가 일관된다.

### 마지막 규약을 지키지 않았을때 문제점

compareTo의 순서와 equals의 결과가 일관되지 않았을 시 이 클래스의 객체를 정렬된 컬렉션에 넣으면 해당 컬렉션이 구현한 인터페이스(Collection,Set,Map)에 정의된 동작과 맞지 않을 수 있다. **정렬된 컬렉션들은 동치성을 비교할 때 equals대신 compareTo를 사용**하기 때문이다.

~~~
BigDecimal bigDecimal1 = new BigDecimal("1.0");
BigDecimal bigDecimal2 = new BigDecimal("1.00");

System.out.println(bigDecimal1.compareTo(bigDecimal2) == 0); //true
System.out.println(bigDecimal1.equals(bigDecimal2)); //false

Set<BigDecimal> hashSet = new HashSet<>();
Set<BigDecimal> treeSet = new TreeSet<>();

hashSet.add(bigDecimal1);
hashSet.add(bigDecimal2);

treeSet.add(bigDecimal1);
treeSet.add(bigDecimal2);

System.out.println(hashSet.size()); //2
System.out.println(treeSet.size()); //1
~~~

</br >

### compareTo 작성 요령

1. Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo메서드의 인수 타입은 컴파일타임에 정해진다.
   즉, 입력 인수의 타입을 확인하거나 형변환할 필요가 없다.
   인수 타입이 잘못 됐으면 컴파일되지 않는다.
2. null을 인수로 넣어 호출하면 NullPointerException을 던진다.

</br >

### 박싱된 기본 타입 클래스는 compare을 사용하자

compareTo 메서드에서 관계 연산자 <와>를 사용하는 방식보다 깔끔하고 정확하다.

클래스에 핵심 필드가 여러 개라면 가장 핵심적인 필드부터 비교하자.

~~~
    @Override
    public int compareTo(final PhoneNumber pn) {
        int result = Integer.compare(areaCode, pn.areaCode);
        if (result == 0) {
            result = Integer.compare(prefix, pn.prefix);
            if (result == 0) {
                result = Integer.compare(lineNum, pn.lineNum);
            }
        }
        return result;
    }
~~~

</br >

자바 8부터는 Comparator 인터페이스가 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다. 그리고 이 비교자들을 Comparable 인터페이스가 원하는 compareTo 메서드를 구현하는데 활용할 수 있다. 다만 약간의 성능 저하가 뒤따른다.

~~~
    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
            .thenComparingInt(pn -> pn.prefix)
            .thenComparingInt(pn -> pn.lineNum);
    
    public int compareTo(final PhoneNumber pn){
        return COMPARATOR.compare(this, pn);
    }
~~~

</br >

### 정리

- 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하자.
- compareTo 메서드에서 필드의 값을 비교할 때 <와> 연산자 대신 박싱된 기본 타입 클래스가 제공하는 compare메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.





