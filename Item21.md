# Item21: 인터페이스는 구현하는 쪽을 생각해서 설계하라

Java8 이전에는 인터페이스를 해치지 않고 메서드를 추가할 방법이 없었지만 Java*에 와서 기존 인터페이스에 메서드를 추가할 수 있는 default 메서드가 생겼다.

하지만 **생각할 수 있는 모든상황에서 불변식을 해치지 않는 default 메서드를 작성하는 것은 어렵다.**

</br >

## Collection 인터페이스의 removeIf 메서드

자바 8에 새롭게 추가된 메서드다.

~~~
default boolean removeIf(Predicate<? super E> filter) {
	Objects.requireNonNull(filter);
	boolean removed = false;
	final Iterator<E> each = iterator();
	while (each.hasNext()) {
		if (filter.test(each.next())) {
			each.remove();
			removed = true;
		}
	}
	return removed;
}
~~~

위 코드는 Predicate가 true를 반환하는 모든 원소를 제거한다. 하지만 모든 Collection 구현체와 어우러지는건 아니다.

예로 아파치의 SynchronizedCollection 클래스가 있다. 이는 클라이언트가 제공한 객체로 락을 거는 능력을 추가로 제공한다. 즉, 모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스다.

그러나 removeIf에서는 동기화에 관해 아무것도 모르므로 락 객체를 사용할 수 없다. 그 결과 멀티 스레드 환경에서 removeIf를 호출할 경우 ConcurrentModificationException이 발생하거나 다른 예기치 못한 결과로 이어질 수 있다.

</br >

## 결론

- 기존 인터페이스에 default 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피하자.
- default 메서드를 통해 새로운 메서드를 추가하더라도 모든 기존 구현체들과 매끄럽게 연동되지 않을 수 있다.
- 인터페이스를 수정하는 것이 가능한 경우도 있겠지만, 그 가능성을 기대해서는 안 된다.

