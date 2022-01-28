#### 아이템7
# 다 쓴 객체 참조를 해제하라

###  가비지 컬렉터를 갖추더라도 메모리 관리에 신경쓰지 않아도 된다고 오해할 수 있는데 절대 사실이 아니다

### 메모리 누수가 일어나고 있는 스택 구현, 해결방안
- 메모리 누수 발생 지점
   - 스택이 줄어들 때 스택에서 꺼내진 객체들은 가비지 컬렉터가 회수하지 않음 : 이 스택이 그 객체들의 다 쓴 참조를 여전히 가지고 있기 때문
```
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
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
- 해당 참조를 다 썼을 때 null 처리(참조해제) 
  - null 처리한 참조를 실수로 사용하면 즉시 NullPointException 발생 및 종료가 되므로 추가적인 이점이 있다.
- 객체 참조를 null 처리하는 일은 예외적인 경우여야 하며 모든 객체에 사용하는 것은 바람직하지 않다.
```
   public Object pop() {
       if (size == 0)
           throw new EmptyStackException();
       Object result = elements[--size];
       elements[size] = null; // 다 쓴 참조 해제
       return result;
   }
```

#### 메모리 누수의 주범
- 자기 메모리를 직접 관리하는 클래스
  - 원소를 다 사용한 즉시 그 원소가 참조한 객체들을 다 null 처리 해준다.
- 캐시
  - 외부에서 key를 참조하는 동안 엔트리가 살아 있는 캐시가 필요한 상황이라면 WeakHashMap을 사용해 캐시를 만들자
  - 흔한 방식(시간이 지날 수록 엔트리의 가치를 떨어트리는 방식)을 사용하지 않는 엔트리는 
    백그라운드 스레드를 활용하거나(Scheduled ThreadPoolExecutor)  
    캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법이 있다. (LinkedHashMap은 removeEldestEntry 메서드를 사용 후자의 방식으로 처리)
- 리스너(listener) 혹은 콜백(callback)
  - 콜백을 약한 참조(weak reference)로 저장하면 가비지 컬렉터가 즉시 수거한다 (예를 들어 WeakHashMap)


##### 참고
WeakHashMap 관련 참고 문서 http://blog.breakingthat.com/2018/08/26/java-collection-map-weakhashmap/
