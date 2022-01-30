# 아이템 8. finalizer와 cleaner 사용을 피하라

## 1) Java의 객체 소멸자

Java에서는 객체를 소멸하기 위해서 두 가지의 객체 소멸자를 제공한다.
<br>
이는 <b>finalizer와 cleaner</b>가 존재하며, 우선 이 둘이 정확하게 무엇인지 알아보겠다.

### - finalizer

finalizer는 Object 클래스에의해 제공이되며, GC가 수거 하기 전에 호출이 된다.
<br>
finalize()메소드를 우리는 finalizer라고 부른다. finalizer는 주로 JVM이 특정 객체가 GC에 의해 수거대상이라고 판별이 될 경우 호출이 된다.
<br>
하지만 finalizer의 주된 목적은 사용된 object가 메모리에서 제거되기 전에, resource를 해제하는 것이다.
<br>
즉, clean의 목적을 가지고 있다는 것을 알 수 있다. 그렇다면 이제 상세 코드 예시를 통해 살펴보겠다.

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;

public class Finalizable {
    private BufferedReader reader;

    public Finalizable() {
        InputStream input = this.getClass()
            .getClassLoader()
            .getResourceAsStream("file.txt");
        this.reader = new BufferedReader(new InputStreamReader(input));
    }

    public String readFirstLine() throws IOException {
        String firstLine = reader.readLine();
        return firstLine;
    }
}
```

위의 예시 코드에서 Finalizable 클래스가 reader라는 필드값을 가지고 있다.
<br>
reader는 자원을 받아올 수 있고, 닫을 수 있는 객체이다. 하지만 해당 코드에서는 reader가 close되지 않은 것을 볼 수 있다.
<br>

이렇게 reader를 닫지 않은 경우, finalize를 통해서 해당 작업을 수행할 수 있다.

``` java
    public void finalize () {
        try {
            reader.closer();
            System.out.println("Closed BufferedReader in the finalizer");
        }catch (IOException e){
            //...
        }
    }
```

위의 코드 처럼 그냥 일반적으로 메소드 선언하듯이 사용을 하면 된다.
<br>
하지만, <b>GC를 언제 호출하는 지는 JVM에 달려 있으며, 이는 시스템의 상황에 따라 변할 수 있기 때문에 프로그래머가 직접 제어를 할 수 없다.</b>
그래서 수동으로 GC를 직접 실행시키도록 흔하게 생각하는 System.gc 메소드가 있겠지만, 해당 메소드는 가급적 사용하지 않는게 좋다. 그 이유는 다음과 같다.

1. 무겁다
2. JVM이 즉시 GC를 실행하도록 호출을 하지 않고, 그저 시작해야 한다는 신호만 보낸다.
3. JVM이 언제 GC를 호출해야하는지 보다 더 잘 안다.

### - cleaner

java 9 의 새로운 클래스로, java.lang.ref.Cleaner 이다. cleaner는 객체가 finalized가 되어, 메모리에서 삭제를 할 수 있는 상태에 대해 알림을 주는 것이다. Cleaner 는
PhantomReference 와 ReferenceQueue를 사용하여, 변화를 알린다.

간단한 예를 보면, 다음과 같다.

```java
public class CleanerExample {
    public static void main(String[] args) {
        Cleaner cleaner = Cleaner.create();
        for (int i = 0; i < 10; i++) {
            String id = Integer.toString(i);
            MyObject myObject = new MyObject(id);
            cleaner.register(myObject, new CleanerRunnable(id));
        }

        //myObjects are not reachable anymore
        //do some other memory intensive work
        for (int i = 1; i <= 10000; i++) {
            int[] a = new int[10000];
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
            }
        }
    }

    private static class CleanerRunnable implements Runnable {
        private String id;

        public CleanerRunnable(String id) {
            this.id = id;
        }

        @Override
        public void run() {
            System.out.printf("MyObject with id %s, is gc'ed%n", id);

        }
    }
}

```
다음 코드의 실행 결과는 다음과 같이 나온다. 
>MyObject with id 7, is gc'ed <br>
>MyObject with id 6, is gc'ed <br>
>MyObject with id 4, is gc'ed <br>
MyObject with id 5, is gc'ed <br>
MyObject with id 9, is gc'ed <br>
MyObject with id 8, is gc'ed <br>
MyObject with id 2, is gc'ed <br>
MyObject with id 3, is gc'ed <br>
MyObject with id 0, is gc'ed <br>
MyObject with id 1, is gc'ed <br>




### finalizer 와 cleaner는 언제 사용해야 할까?

reosource의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안정망 역할을 할 때이다.
<br>
cleaner나 finalizer가 즉시 호출된다는 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것이 안 하는 것보다는 낫다. 안정망의 역할로 유용한 대표적인 자바 라이브러리로는
FileInputStream, FileOutputStrean, ThreadPoolExecutor가 대표적이다.
<br>
두 번째로, 네이티브 피어와 연결된 객체에서 유용하다.


> 네이티브 피어란?<br>
> JFrame을 자바 피어라고 할 수 있다. 실제 그래픽을 그릴 때, 네이티브 피어가 필요하다.
> JFrame의 경우 전부 삭제할 때, dispose()를 호출해야 한다. 그 이유는 GC가 손댈 수 없는 영역이므로,
> 명시적으로 네이티브 컴포넌트를 삭제해야 한다.
> <br><br>
> Java Native Interface JNI 는 자바 가상머신 위에서 실행되고 있는 자바 코드가 네이티브 응용
> 프로그램(하드웨어와 운영체제 플랫폼에 종속된 프로그램들) 그리고 C, C++
> 그리고 어샘블리 같은 다른 언어들로 작성된 라이브러리들을 호출하거나 반대로 호출
> 되는 것을 가능하게 하는 프로그래밍 프레임 워크이다.


다음 예시 코드를 살펴보자.

```java

import sun.misc.Cleaner;

public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {
        int numJunkPiles; // 방 (Room) 안의 쓰레기 수

        public State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        @Override
        public void run() {
            System.out.println("방청소");
            numJunkPiles = 0;
        }
    }

    // 방의 상태를 cleanabe과 공유
    private final State state;

    // cleanable 객체. 수거대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(State state, Cleaner.Cleanable cleanable) {
        this.state = new State(state.numJunkPiles);
        this.cleanable = cleaner.register(this, state);

    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}

```

위의 코드에서 Room의 cleaner는 단지 안정망으로 쓰였다. 클라이언트가 모든 Room 생성을 try-with-resources블록으로 감쌀 경우 자동 청소는 필요 없다.

```java
public class Adult {
    // 방청소가 반드시 실행됨
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕");
        }
    }
}
```

```java
public class Teenager {
    //방청소가 될수도있고 안될 수도 있다.
    public static void main(String[] args) {
        new Room(99);
        System.out.println("안녕");
    }
}
```

> 정리
> cleaner는 안정망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자.
> 물론 이때도, 불확실성과 성능 저하에 주의해야 한다.