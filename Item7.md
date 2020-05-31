## Item7: 다 쓴 객체 참조를 해제하라

GC는 다 쓴 객체를 알아서 회수해 가기 때문에 메모리 관리에 더 이상 신경쓰지 않아도 된다고 오해할 수 있다. GC가 처리할 수 없는 자원이 생기지 않게 주의해야한다.

### 메모리 직접 관리

~~~
public class Stack {

    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        this.ensureCapacity();
        this.elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        return this.elements[--size]; // 메모리 누수의 원인
    }

    private void ensureCapacity() {
        if (this.elements.length == size) {
            this.elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
~~~

이 코드에서 스택이 커졌다 줄어들었다 해도 스택에서 깨내진 객체들은 GC가 회수하지 않는다. 왜냐하면 스택이 객체들의 다 쓴 참조를 여전히 갖고 있기 때문이다. '활성 영역'의 범위는 size보다 작은 부분이고, size값보다 큰 부분의 값들은 필요없이 메모리를 차지하고 있는 부분이다.

해결 방법으로는 해당 참조를 다 썼을 때 null 처리(참조 해제)하면 된다. 다음과 같이 수정하면 된다.

~~~
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        Object value = this.elements[--size];
        this.elements[size] = null;
        return value;
    }
~~~

스택에서 꺼낼 때 꺼낸 객체의 자리를 null로 설정한다. null 처리한 참조를 실수로 사용하려 프로그램은 즉시 NullPointerException을 발생시킨다. 다 쓴 객체를 돌려주는 것보다 예외를 던져주는 게 잘못된 일의 수행을 예방할 수 있어서 더 낫다.

하지만 이와같이 **객체 참조를 null 처리하는 일은 예외적인 경우**여야 한다. 모든 객체를 다 쓰자마자 일일이 null 처리하는데 혈안이 되기도 하고, 프로그램을 필요 이상으로 지저분하게 만들기 때문이다.

다 쓴 객체를 해제하는 가장 좋은 해결책은 **그 참조를 가리키는 변수를 특정 범위(스코프) 안에서만 사용하는 것**이다.(지역 변수는 그 영역이 넘어가면 GC의 대상이 된다.) 변수의 범위를 최소가 되게 정의하면 자연스럼게 이뤄지는 일이다.

그렇다면 **객체의 null 처리는 언제 해야 할까? 메모리를 직접 관리할 때**이다. Stack 클래스는 elements 배열로 자기 메모리를 직접 관리하기 때문에 배열의 활성 영역에 속한 원소들만 사용되고 비활성 영역은 쓰이지 않는걸 GC가 알 수 없다. GC의 입장에서는 활성 영역과 비활성 영역 모두 유효한 객체이다.

**자기 메모리를 직접 관리하는 클래스는 프로그래머는 항시 메모리 누수에 주의해야 한다.**



### 캐시

캐시 역시 메모리 누수를 일으키는 주범이다. 객체 참조를 캐시에 넣고, 이 사실을 잊은 채 그 객체를 다 쓴 후 비우지 않는다.

해법은 여러 가지가 있지만, 캐시의 키(key)에 대한 참조가 캐시 외부에서 필요 없어지면 해당 엔트리를 자동으로 제거해주는 WeakHashMap을 쓸 수 있다.

WeakHashMap은 특정 key 값이 더이상 사용되지 않는다고 판단되면 그 값을 제거해준다.



### 리스너와 콜백

리스너와 콜백 또한 메모리 누수의 주범이다. 클라이언트가 콜백을 등록만 하고 해지하지 않는다면, 콜백은 계속 쌓일 것이다. 이 때 콜백을 약한 참조(weak reference)로 저장하면 GC가 처리해 준다.

캐시와 같은 방법인 WeakHashMap에 키로 콜백을 저장하면 된다.

