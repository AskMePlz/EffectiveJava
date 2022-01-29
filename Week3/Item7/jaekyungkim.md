# 아이템 7. 다 쓴 객체 참조를 해제하라

## 메모리 누수

### 1) 배열
java에서는 garbage collector(이하 gc)가 메모리를 자동으로 치워준다는 사실은 모두 알 것이다.<br>
그러나, gc가 수거해가지 못하는 상황도 존재하고 있다. <br>
그러므로, 이런 상황을 대비하여 객체를 수동으로 해제하는 방법도 알아야 한다.<br>

```java
import java.util.Arrays;
import java.util.EmptyStackException;

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

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

위의 예시 코드를 살펴보면, pop()메서드를 쓰는 과정에서 스택에서 꺼내진 객체는 gc가 회수를 하지 않는 것을 알 수 있다. <br>
이는 stack이 메모리를 직접 관리하기 때문에 발생하는 현상이며, gc가 다 쓴 객체인지 사용중인지 객체인지 판별을 할 수 없기 때문이다. <br>
그러므로, pop() 메서드에서 null 처리를 해주면 좋다.

```java
import java.util.EmptyStackException;

public class Stack {
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 객체 참조 해제
        return result;
    }
}
```
책에서는 null  처리를 일일히 하는 것은 번거롭기 때문에 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것을 권장하고 있다. 

### 2) 캐쉬

캐시를 사용할 때도 메모리 누수 문제를 조심해야한다. <br>
객체의 레퍼런스를 캐시에 넣어 놓고 캐시를 비우는 것을 잊기 쉽다. 
이에 대한 여러 해결 책 중 캐시의 키에 대해 레퍼런스가 캐시 밖에서 필요 없어지면 해당 엔트리를 캐시에서 자동으로 비워주는 WeekHashMap을 사용 할 수 있다.
<br>
GC는 WeekHashMap의 key 값을 Weak reference 로 감싸기 때문에 hard reference가 없어지면 GC의 대상이 된다. 

### 3) 콜백
콜백 또한 WeakHashMap을 사용해서 WeakHashMap에 키로 저장하면, GC가 알아서 수거해준다





