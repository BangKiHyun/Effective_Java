# Item58: 전통적인 for 문보다는 for-each 문을 사용하라

`for-each` 문은 반복자와 인덱스 변수를 사용하지 않아 코드가 깔끔해지고 오류가 날 일도 없다.

하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어 어떤 컨테이너를 다루는지는 신경 쓰지 않아도 된다.

### 컬렉션과 배열을 순회하는 올바른 관용구

~~~java
for (Element e : elements) {
    doSomething(e);
}
~~~

여기서 콜론(:)은 "안의(in)"라고 읽으면 된다. 'elements 안의 각 원소 e에 대해'라고 읽는다.

</br >

## 컬렉션 중첩 순회

컬렉션을 중첩 순회할 때 `for-each` 문의 이점이 더욱 커진다.

### 두 개의 주사위로 나올 수 있는 모든 경우의 수 구하기 (버그)

~~~java
    enum Face {ONE, TWO, THREE, FOUR, FIVE, SIX}

    public static void main(String[] args) {
        EnumSet<Face> faces = EnumSet.allOf(Face.class);

        for (Iterator<Face> i = faces.iterator(); i.hasNext();){
            for(Iterator<Face> j = faces.iterator(); i.hasNext();){
                System.out.println(i.next() + " " + j.next());
            }
        }
    }
~~~

두 개의 주사위로 나올 수 있는 경우는 총 36가지이다. 하지만 위 코드는 6가지만 출력하고 끝난다.

### 수정된 코드 (for-each문)

~~~java
    public static void main(String[] args) {
        EnumSet<Face> faces = EnumSet.allOf(Face.class);

        for (Face face1 : faces) {
            for (Face face2 : faces) {
                System.out.println(face1 + " " + face2);
            }
        }
    }
~~~

`for-each` 문을 사용하여 간단히 해결 할 수 있다.

</br >

## for-each 문을 사용할 수 없는 세 가지 상황

### 파괴적인 필터링

- 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 `remove` 메서드를 호출해야 한다.
- 자바 8부터 Collection의 `removeIf` 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.

### 변형

- 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열을 인덱스를 사용해야 한다.

### 병렬 방복

- 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 명시적으로 제어해야 한다.

