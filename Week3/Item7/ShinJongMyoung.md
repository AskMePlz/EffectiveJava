# Item7. 다 쓴 객체 참조를 해제하라

자바는 개발자가 메모리를 직접 관리하지 않고 가바지 컬렉터가 다쓴 객체를 알아서 회수해간다. 그래서 메모리 관리에 더 이상 신경쓰지 않아도 된다고 오해할 수 있는데, 절대 사실이 아니다.
다음 스택 예시를 보자.
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```
위 프로그램을 오래 실행하다 보면 점차 GC 활동과 메모리 사용량이 늘어나 결국 성능이 저하될 것이다.
상대적으로 드문 경우긴 하지만 심할 때는 디스크 페이징이나 OutOfMemoryError를 일으켜 프로그램이 예기치 않게 종료되기도 한다.

### 언제 메모리 누수가 발생할까?
Stack 이 커졌다가 줄어들었을 때 Stack 에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다. 프로그램에서 그 객체들을 더 이상 사용하지 않더라도 말이다.
Stack 이 그 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문이다. 여기서 다 쓴 참조란 문자 그대로 앞으로 다시 쓰지 않을 참조를 뜻한다.
elements 배열의 `활성 영역` 밖의 참조들이 모두 여기에 해당한다.

### 메모리 누수를 해결하는 방법은?
해당 참조를 다 썼을 때 null 처리(참조 해제) 하면 된다. 다음은 pop 메서드를 제대로 구현한 예시이다.
```java
public Object pop() {
    if (size == 0){
        throw new EmptyStackException();
    }
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```
여기서 주의할 점은 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다는 점이다.
다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내는 것이다.

### 또 다른 메모리 누수
- 캐시 
    - 객체 참조를 캐시에 넣고 나서, 이 사실을 까맣게 잊은 채 그 객체를 다 쓴 뒤로도 한참을 그냥 놔두는 일을 자주 접할 수 있다.
    - 운 좋게 캐시 외부에서 키(key)를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 상황이라면 `WeakHashMap` 을 사용해 캐시를 만들자.

- 리스너(listener) 혹은 콜백(callback)
    - 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면, 뭔가 조치해주지 않는 한 콜백은 계속 쌓여갈 것이다.
    - 이럴 때 콜백을 약한 참조(weak reference)로 저장하면 가비지 컬렉터가 즉시 수거해간다. 예를 들어 WeakHashMap에 키로 저장하면 된다.
  