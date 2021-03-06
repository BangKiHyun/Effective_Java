# Item57: 지역변수의 범위를 최소화하라

지연변수의 범위는 선언된 지점부터 그 지점을 포함한 볼록이 끝날때까지다.

실제 사용하는 블록 바깥에 선언된 변수는 그 블록이 끝난 뒤까지 살아 있게 된다. 그래서 실수로 의도한 범위 앞 혹은 뒤에서 그 변수를 사용하면 끔찍한 결과로 이어질 수 있다. 

## 가장 처음 쓰일 때 선언하라

지역변수의 범위를 줄이는 가장 강력한 기법은 가장 처음 쓰일 때 선언하기다.

- 사용하려면 멀었는데 미리 선언부터 코드가 어수선해져 가독성이 떨어진다.
- 변수를 실제로 사용하는 시점에 타입과 초깃값이 기억나지 않을 수도 있다.

</br >

## 거의 모든 지역변수는 선언과 동시에 초기화 하라

초기화에 필요한 정보가 충분하지 않다면 충분해질 때까지 선언을 미뤄야 한다.

### 예외 (try-catch)

- 변수를 초기화하는 표현식에서 검사 예외를 던질 가능성이 있다면 try 블록 안에서 초기화해야 한다.
  - 그렇지 않으면 예외가 블록을 넘어 메서드까지 전파됨.
- 변수 값을 try 블록 바깥에서도 사용해야 한다면 try 블록 앞에 선언해야 한다.

</br >

## while 문보다 for 문을 사용하자

반복문에서는 반복 변수의 범위가 **반복문의 몸체, `for` 키워드와 몸체 사이의 괄호 안**으로 제한된다.

따라서 **반복변수의 값을 반복문이 종료된 뒤에서 써야 하는 상황이 아니라면 `while` 문보다 `for` 문을 쓰는 편이 낫다.**

</br >

### 컬렉션이나 배열을 순회하는 권장 관용구

~~~java
for (Element e : c) {
    ...
}
~~~

</br >

### 반복자를 사용해야 하는 상황에서  `for-each` 문 대신 전통적인 `for` 문을 쓰는게 낫다

~~~java
for(Iterator<String> i = list.iterator(); i.hasNext()){
    final String s = i.next();
    // s와 i로 무언가를 함
}
~~~

</br >

### while 문 문제점

~~~java
        final Iterator<String> i = list.iterator();
        while (i.hasNext()){
            doSomething(i.next());
        }

        final Iterator<String> i2 = list2.iterator();
        while (i.hasNext()){ // 위에서 선언한 i사용. 버그발생
            doSomething(i2.next());
        }
~~~

- 두 번째 `while` 문에 버그가 발생한다.
  - 새로운 반복 변수 i2를 초기화했지만 실수로 이전 i를 다시 썼다.
- i의 유효 범위가 아직 끝나지 않았으므로 이 코드는 컴파일도 잘 되고 실행 시 예외도 던지지 않는다.
  - 하지만 두 번째 `while` 문을 순회하지 않고 곧장 끝난다.

**for 문을 사용하면 위와 같은 오류를 컴파일 타임에 잡아준다.** 유효 범위가 반복문 종료와 함께 끝나기 때문이다.

</br >

## 반복문 관용구 최적화

다음은 지역변수 범위를 최소화하는 또 다른 반복문 관용구다.

~~~java
for (int i = 0, k = expensiveComputation(); i < k; i++) {
    doSomething(i);
}
~~~

반복 여부를 결정짓는 변수 i의 한계값을 변수 n에 저장하여, **반복 때마다 다시 계산해야 하는 비용을 없앴다.**

값을 같은 반환하는 메서드를 매번 호출한다면 이 관용구를 사용하자.

</br >

## 메서드를 작게 유지하고 한 가지 기능에 집중하자

- 한 메서드에서 여러 가지 기능을 처리한ㄴ다면 그중 한 기능과만 관련된 지역변수라도 다른 기능을 수행하는 코드에서 접근할 수 있다.
- 해결책으로 단순히 메서드를 기능별로 쪼개면 된다.

