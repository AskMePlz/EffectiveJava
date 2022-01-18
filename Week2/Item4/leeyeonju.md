### 클래스의 인스턴스 생성 막기

static 변수와 static 메서드만 담은 클래스를 만드는 경우 즉 인스턴스 변수와 인스턴스 메소드가 필요없는 유틸리티성 클래스를 만드는 경우 해당 클래스의 인스턴스화를 막을 필요가 있다. 이러한 경우 인스턴스 생성이 무의미기 때문이다. 인스턴스 생성을 막아 인스턴스 변수와 메서드 없이 공통적으로 동작하게끔 만들어진 역할을 명확히 해주자. 

### 인스턴스 생성을 막기위한 바람직하지 않은 케이스

```java
public abstract class SampleUtils {
    public static void sampleMethod(){
        // ...
    }
}

class SubSampleUtils extends SampleUtils {
    public SubSampleUtils(){
        super(); 
		}
}

```

인스턴스 화를 막기 위하여 abstract 클래스로 변경 하여도 인스턴스화를 막을 순 있지만 해당 abstract 클래스를 상속 받아 구현하는 경우 상위 클래스의 생성자를 super()로 호출할 수 있기 때문에 바람직하지 않다. 또한 아무런 생성자를 만들지 않는 경우에도 컴파일러가 default 생성자를 기본적으로 만들기 때문에 인스턴스화를 막을 수 없다. 명시적으로 private 생성자를 추가하여 방지하자.

### private 생성자를 선언한 자바 API 예시

```java
public class Collections {

    // Suppresses default constructor, ensuring non-instantiability.
		private Collections() {}

    private static final int BINARYSEARCH_THRESHOLD   = 5000;
    private static final int REVERSE_THRESHOLD        =   18;
    private static final int SHUFFLE_THRESHOLD        =    5;
    ... 

}
```

```java
public final class Math {

	 /**
    * Don't let anyone instantiate this class.
    */
		private Math() {}

    public static final double E = 2.7182818284590452354;
    public static final double PI = 3.14159265358979323846;

    ... 
}
```

 

우리가 흔히 사용하는 Collections 클래스나 Math 클래스와 같은 유틸성 클래스를 열어보면 해당 클래스의 인스턴스 생성을 막기 위해 private 생성자를 선언해 놓고 있다. 또한 위와 같이 주석을 통해 이 선언이 인스턴스 생성을 막기 위함이라고 표기하고 있다. 개발자가 실제 유틸성 클래스를 만드는 경우에도 주석을 달아주는게 좋다. 

### 더 강력하게 막기

```java
public class SampleUtils {

    // Suppress default constructor for noninstantiability
    private SampleUtils () {
        throw new AssertionError();
    }
}
```

혹여나 호출되는 경우 위와 같이 에러를 발생시켜 생성을 더 강력하게 방지할 수도 있다.
