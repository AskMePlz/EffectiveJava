> 객체를 올바르게 생성파고 적절히 파괴하는 법을 배우는 장의 마지막 아이템이다. 앞서 정적 팩터리 메서드로 생성자를 만든다거나, 생성자를 빌더로 만든다거나, 싱글턴 객체를 만드는 법을 배웠다. 언제 어떠한 방법으로 객체를 만들고 사용하는게 효율적인지 불필요한 객체 생성을 피하는 방법을 배운것이다. 불필요한 객체 생성을 피하는것 만큼 중요한것이 적절히 파괴하는 것이다. JVM을 통해 가비지 컬렉터가 주기적으로 사용하지 않는 객체는 알아서 정리하지만 이 GC 손이 닿지 못하는 곳에 메모리 누수가 발생할 수 있다. 그 지점을 찾아서 적절히 파괴해 주고, 괜히 사용하여 문제가 발생할 수 있는 자바의 기능까지 알아보자. 또한 이를 해결할 다양한 방법도 찾아보자
> 

## 더이상 사용하지 않는 참조를 찾아 적절히 파괴하자.

- 흔히 발생하는 케이스
    - 스택을 구현한 케이스에서 pop()을 통해 객체를 제거 하더래도 제거된 객체의 다 쓴 참조 주소가 여전히 존재한다. 스택에 쌓였다가 줄어들었을 때 스택에서 꺼내진 객체들의 참조는 남아있게 된다. 왜냐하면 pop()은 배열의 size를 줄여서 줄인 인덱스에 객체를 반환하고 있기 때문에 size 범위 밖에 객체들은 접근하지 못한 상태로 그대로 살아있기 때문이다. 가비지 컬렉터는 객체 참조가 살아 있으면 그 안에서 참조하는 다른 객체까지 제거하지는 못하기 때문에 잠재적으로 문제가 된다.
        
        ```java
        // 가장 최근 쌓인 객체 반환 (제거)
        public Object pop() {
            if(size == 0) {
                throw new EmptyStackException();
            }
            return elements[--size];
        }
        ```
        
- 해결책
    - 해당 객체가 **자기 메모리를 직접 관리하는 예외적인 경우일 때는** null처리 해주면 된다.
        
        ```java
        public Object pop_개선() {
            if(size == 0) {
                throw new EmptyStackException();
            }
            Object object = elements[--size];
            elements[size] = null; // 삭제한 객체는 지워주기 
            return object;
        }
        ```
        
        ```java
        // java.util.Stack 의 pop() 일부 
        public synchronized E pop() {
              E       obj;
              int     len = size();
        
              obj = peek();
              removeElementAt(len - 1);
        
              return obj;
          }
        
        public synchronized void removeElementAt(int index) {
            if (index >= elementCount) {
                throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                         elementCount);
        		} else if (index <0) {
                throw new ArrayIndexOutOfBoundsException(index);
        		}
        
            int j = elementCount - index -1;
        
        		if (j >0) {
                System.arraycopy(elementData,index +1,elementData,index,j);
        		}
        
        	  modCount++;
        		elementCount--;
        		elementData[elementCount] = null; /* to let gc do its work */
        }
        ```
        
    - 자기 메모리를 직접 관리하는 경우가 아니라면 일반적으로는 변수를 유효 범위 밖으로 밀어내는 것이 적절하다.
        
        ```java
        // sample 
        ```
        

## 시스템에 잠복할 수 있는 메모리 누수의 주범을 알아보자.

- 자기 메모리를 직접 관리하는 클래스에서 처리가 미흡한 경우
    - 위에서 본 Stack과 같은 경우 원소를 위한 공간을 미리 마련하고 증가시키면서 메모리를 관리하고 있는데 이러한 경우에 주의해야 한다.
    
    ```java
    // Stack이 상속받고 있는 java.util.Vector class 일부
    private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity <= 0) {
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            return minCapacity;
        }
        return (newCapacity - MAX_ARRAY_SIZE <= 0)
            ? newCapacity
            : hugeCapacity(minCapacity);
    }
    ```
    
- 캐시 관리가 제대로 안되는 경우
    - 객체 참조를 캐시에 넣고 다 쓴 후에도 그대로 놔두는 경우 메모리에 쌓여 있게 된다.
        - 키를 참조하는 동안만 캐시 엔트리가 살아있으면 되는 경우 WeakHashMap 을 통해 다쓴 엔트리 제거
            
            <aside>
            💡 WeakHashMap 
            : WeakHashMap 클래스는 Key에 해당하는 객체가 더이상 사용되지 않을 경우 해당 Element는 GC의 대상이 된다.
            
            </aside>
            
            ```java
            public static void WeakReferenceTest() throws Exception {
                    
                  WeakHashMap<MainClass, SubClass> map = new WeakHashMap<>();
                  MainClass mainClass = new MainClass();
                  SubClass subClass = new SubClass();
                  map.put(mainClass, subClass);
                  
                  System.out.println(map.size()); // 1 
                  mainClass = null;
                  System.gc();
                  Thread.sleep(2000);
                  System.out.println(map.size()); // 0 -> GC 동작하여 삭제됨 
            
                  HashMap<MainClass, SubClass> map2 = new HashMap<>();
                  MainClass mainClass2 = new MainClass();
                  SubClass subClass2 = new SubClass();
                  map2.put(mainClass2, subClass2);
            
                  System.out.println(map2.size()); // 1 
                  mainClass2 = null;
                  System.gc();
                  Thread.sleep(2000);
                  System.out.println(map2.size()); // 1  
            
            }
            ```
            
        - 보통은 캐시 유효기간을 정확히 정의하기가 어렵기 때문에 서서히 캐시 엔트리의 가치를 떨어뜨린다. 즉 쓰지 않는 엔트리를 청소 한다.
- 리스너와 콜백 관리가 제대로 안될 경우
    - 콜백 등록 후 해지하지 않는 경우 계속 쌓여감. 콜백을 약한 참조(weak reference)로 저장 즉 WeakHashMap에 키로 저장하는 방식으로 해결한다.
        
        ```java
        // example
        ```
        

## 피해야 하는 자바의 객체 소멸자! finalizer, cleaner

- 왜 ?
- 그렇다면 ?

## 자원을 close() 해야하는 경우 try-finally 대신 try-with-resources !

- 왜?

> 오랜시간 잠복해 있을 수 있는 잠재적인 메모리 누수 요소들을 살펴보고 해결할 수 있는 다양한 해결책들을 알아두어야
>
