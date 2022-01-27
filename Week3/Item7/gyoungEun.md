# [item7] 다 쓴 객체 참조를 해제하라
자바에서도 메모리 관리를 해줘야한다. 아래의 예를 보자. 어느 부분이 문제가 될까?

```java
class Stack {
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

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

프로그램에서 객체들을 더 이상 사용하지 않더라도 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 자비지 컬렉터가 회수하지 않는다. 이는 스택을 오래 실행하다 보면 점차 가비지 컬렉션 활동과 메모리 사용량이 늘어나서 결국 성능이 저하될 것이다. 심하면 OutOfMemoryError를 발생시킬 수 있다.

## GC언어에서 메모리 해지하기

 해당 참조를 다썼을 때 null처리 (참조 해제)하면 된다. 

```java
public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
```

## 메모리 누수의 주범 3가지

- **자기 메모리를 직접 관리하는 클래스** 

     예를 들어 스택의 경우 저장소 풀을 만들어서 원소들을 관리하는데 활성 영역에 속한 원소들이 사용되고 비활성 영역은 쓰이지 않는다. 여기서 문제는 GC는 이 사실을 알 수 없다는 것이다.  GC가 볼때는 비활성 영역에서 참조하는 객체도 똑같이 유효해 보이기 때문이다.  때문에 개발자가 다 쓴 참조를 해지해줘야한다.
    
- **캐시**
     
     객체 참조를 캐시에 넣어두고 다쓴 후에 참조를 해지하는 것을 까먹는 경우가 많다. 외부에 키를 참조하는 동안만 엔트리가 살아있는 캐시가 필요한 상황이라면 weakHasMap을 사용하는 것도 방법이다.
     캐시의 유효기간을 정확히 정의하기 어렵기 때문에 쓰지 않는 엔트리를 이따금 청소해줘야한다. (예. Scheduled ThreadPoolExecutor) 
   
- **리스너 혹은 콜백**   
    
    클라이언트가 콜백을 등록만하고 해지하지 않는다면, 뭔가 조치해주지 않는 한 콜백은 쌓여갈 것이다. 이럴때 콜백을 약한 참조로 저장하면 가비지 컬렉터가 즉시 수거해간다. (예. WeakHasMap에 키로 저장)
