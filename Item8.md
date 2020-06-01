## Item8: finalizer와 cleaner 사용을 피하라

### 두 가지 객체 소멸자

자바에서는 finalizer 메서드와 Cleaner 클래스의 객체 소멸자를 제공한다. 두 가지 모두 GC가 수행될 때 실행되는 구문이다.

그 중 finalizer는 예측할 수 없고, 위험하며, 대부분 불필요하다. 성능도 안좋아진다. 자바9부터는 deprecated가 되었고, 대안으로 cleaner를 소개했다. 그러나 cleaner 역시 finalizer보다 덜 위험하지만 여전히 문제는 비슷하다.



### finalizer의 단점

1. 실행 시점을 보장할 수 없다.
   - 어떤 객체가 더 이상 필요 없어진 시점에 즉시 finalizer나 cleaner 가 바로 실행된다는 보장이 없다.
     즉, finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다.
   - finalizer는 인스턴스 반납을 지연 시킬 수 있다. finalizer 쓰레드는 우선 순위가 낮아서 언제 실행될지 모른다.
     즉, finalizer에 들어간 작업을 쓰레드가 처리 못해 대기하고 있다면, 해당 인스턴스는 GC가 되지 않고 계속 쌓이다 OutOfMemoryError를 발생시킬 수 있다.
   - 실행 시점 뿐만 아니라 수행 여부조차 보장하지 않는다. Unrecheable 객체의 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수 있다.
     따라서 DB의 공유 자원의 영구 락 해제를 finalizer나 cleaner에게 맡겨 놓으면 분산 시스템 전체가 서서히 멈출 수 있다.
2. 성능의 저하를 유발한다.
   - AutoCloseable 객체를 생성하고 GC가 자원 반납을 하는데 걸리는 시간은 12ns 인데 반해(try-with-resource로 close 하도록 했다.), finalizer를 사용한 경우에 550ns가 걸렸다. 다시말해 finalizer를 사용한 객체를 생성하고 파괴하니 50배나 느렸다.
3. 심각한 보안 문제의 가능성
   - finalizer를 사용한 클래스는 finalizer 공격에 노출될 수 있다. 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 하위 클래스의 finalizer가 수행될 수 있게 된다.
     원래는 죽었어야 할 객체의 finalizer가 실행 되면, 그 안에서 해당 인스턴스의 레퍼런스를 기록할 수 있고, GC가 수집하게 못하게 막을 수 있다.
   - final 클래스는 하위 클래스를 만들 수 없기 때문에 공격에서 안전하다. 그 이외의 클래스를 finalizer공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언하여 오버라이딩하는 것을 막을 수 있다.



### 대안

AutoCloseable 인터페이스를 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메소드를 호출해 주거나, try-with-resource를 사용하면 된다. 참고로close 메소드는 현재 인스턴스의 상태가 이미 종료된 상태인지 확인하고, 이미 반납이 끝난 상태에서 close가 다시 호출됐다면 IllegalStateException을 던져야 한다.



### finalizer와 cleaner의 쓰임새

자원의 소유자가 close 메서드를 호출하지 않는 것에 대한 안정만 역할로 사용할 수 있다. 
finalizer나 cleaner가 즉시 호출되리라는 보장은 없지만, 안하는 것 보다는 낫다. 
자바 라이브러리 일부 클래스는 안정망 역할의 finalizer를 제공한다.(FileInputStream, FileOutputStream 등)