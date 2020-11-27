# Item54: null이 아닌, 빈 컬렉션이나 배열을 반환하라

### 컬렉션이 비었으면 null을 반환 - 따라 하지 말 것

```java
private final List<Cheese> cheesesInStock;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 * 단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null
            : new ArrayList<>(cheesesInStock);
}
```

위 코드는 컬렉션이나 배열 같은 컨테이너가 비었을 때 null을 반환한다. 이렇게 되면 다음과 같이 항시 방어 코드를 넣어줘야 한다.

~~~java
    public static void main(String[] args) {
        List<Cheese> cheeses = shop.getCheeses();
        // null check
        if(cheeses != null && cheeses.contains(Cheese.STILTON))
            System.out.println("좋았어, 바로 그거야.");
    }
~~~

만약 클라이언트에서 방어 코드를 빼먹으면 오류가 발생할 수 있다.

</br >

## 빈 컨테이너 할당 비용 vs null을 반환

때로 빈 컨테이너를 할당하는 데도 비용이 드니 null을 반환하는 쪽이 낫다는 주장도 있다. 하지만 두 가지 면에서 틀렸다.

1. 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한 이 정도의 성능 차이는 신경 쓸 수준이 못된다.
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

### 빈 컬렉션을 반환하는 전형적인 코드 - 대부분의 상황에서 이렇게 하면 된다

~~~java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
~~~

빈 컬렉션 할당이 성능을 떨어뜨릴 수도 있다. 해결책으로 매번 똑같은 빈 **'불변'컬렉션을 반환**하는 것이다.

`Collections.emtpyList` 메서드가 그러한 예다.

### 최적화 - 빈 컬렉션 매번 새로 할당하지 않음

~~~java
    public List<Cheese> getCheeses() {
        return cheesesInStock.isEmpty() ? Collections.emptyList()
                : new ArrayList<>(cheesesInStock);
    }
~~~

</br >

## 배열을 쓸 때도 null을 반환하지 말고 길이가 0인 배열을 반환하라

다음 코드에서 `toArray` 메서드에 건넨 길이 0짜리 배열은 우리가 원하는 반환 타입을 알려주는 역할을 한다.

~~~java
    public Cheese[] getCheeses() {
        return cheesesInStock.toArray(new Cheese[0]);
    }
~~~

만약 성능을 떨어뜨릴 것 같다면 길이 0짜리 배열을 미리 선언해두고 매번 그 배열을 반환하면 된다.(길이 0인 배열은 모두 불변)

~~~java
    private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

    public Cheese[] getCheeses() {
        return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
    }
~~~

위 코드는 `cheesesInStock`이 비었을 때면 언제나 `EMPTY_CHEESE_ARRAY`를 반환하게 된다.

</br >

## 정리

- null이 아닌, 빈 배열이나 컬렉션을 반환하라
- null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다. 그렇다고 성능이 좋은 것도 아니다.