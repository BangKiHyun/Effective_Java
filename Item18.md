# Item18: 상속보다는 컴포지션을 사용하라

여기서의 상속은 클래스가 다른 클래스를 확장하는 구현 상속을 말한다. 인터페이스 상속과는 무관하다.

## 상속의 문제점

메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.

상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다. 다시 말하자면 상위 클래스의 내부 구현이 달라졌을 시, 아무것도 건드리지 않은 하위 클래스가 오동작할 수 있다.

### HashSet 예제

~~~
public class InheritanceHashSet<E> extends HashSet<E> {

    private int addCount = 0;

    public int getAddCount() {
        return addCount;
    }

    @Override
    public boolean add(final E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(final Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
}
~~~

~~~
        InheritanceHashSet<String> s = new InheritanceHashSet<>();
        s.addAll(Arrays.asList("1", "2", "3"));

        System.out.println(s.getAddCount()); //결과: 6
~~~

위 코드는 실행했을 때 addCount의 값이 3이 나올 것으로 예상했지만 실제로는 6이 반환된다.

HashSet의 addAll 메서드가 자기 자신의 add 메서드를 사용해서 그렇다.

~~~
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
~~~

</br >

## 해결책(컴포지션)

메서드의 재정의하는 것보다 새로 만드는 게 나을 수 있다.

훨씬 안전한 방법이긴 하지만 위험 요소가 전혀 없는 것은 아니다. 만약 하위 클래스에서 추가한 메서드의 명과 파라미터가 같고 리턴 타입만 다르다면 그 클래스는 컴파일조차 되지 않는다.

기존 클래스를 확장하는 대신에 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하면 된다.

- Composition: 기존 클래스가 새로운 클래스의 구성요소로 쓰이는 것
- Forwarding: 새로운 클래스의 인스턴스 메서드들이 기존클래스에 대응하는 메서드를 호출해 그 결과를 반환하는 것

### 예제

~~~
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    public ForwardingSet(Set<E> s) { this.s = s;}

    public void clear() { s.clear(); }
    public boolean contains(Object o) {return s.contains(o); }
    public boolean isEmpty() { return s.isEmpty(); }
    public int size() { return s.size(); }
    public boolean add(E e) { return s.add(e); }
    public boolean remove(Object o) {return s.remove(o); }
    public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
    ...
}
~~~

~~~
public class CompositionHashSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public CompositionHashSet(final Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(final E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(final Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
~~~

위 코드는 다른 Set 인스턴스를 감싸고 있다는 뜻에서 **래퍼 클래스**라고 한다.

</br >

## 언제 상속을 사용해야하나?

상위 클래스와 하위 클래스의 관계가 **is-a관계**일때만 사용해야 한다.

즉, 반드시 **하위 클래스가 상위 클래스의 진짜 하위타입인 상황**에서만 쓰여야 한다. 그렇지 않다면 위와 같이 컴포지션을 사용하면 된다.

컴포지션을 사용해야할 상황에서 상속을 사용하는건 내부 구현을 불필요하게 노출하며, 상위 클래스의 결함까지도 그대로 계승한다. 왠만하면 컴포지션을 사용하자!