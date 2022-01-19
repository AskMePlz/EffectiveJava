# [item4] 인스턴스화를 막으려거든 private 생성자를 사용하라


### 정적 메서드와 정적 필드만을 담은 class의 쓰임새

- `java.lang.Math`  (java8 기준 작성)
    
    ```java
    public final class Math {
        private Math() {};
    	
        public static final double PI = 3.14159265358979323846;
        public static double sin(double a) {
            return StrictMath.sin(a); // default impl. delegates to StrictMath
        }
        public static double cos(double a) {
            return StrictMath.cos(a); // default impl. delegates to StrictMath
        }
    
    	... 
    }
    ```
    
- `java.util.Collections`
    
    ```java
    public class Collections {
    	public static <T extends Comparable<? super T>> void sort(List<T> list) {
        	list.sort(null);
    	}
    			
    	public static <T> void sort(List<T> list, Comparator<? super T> c) {
            	list.sort(c);
        	}
    
    		...
    
    }
    ```
    
- `final클래스와 관련한 메서드들을 모아 놓을 때`
    
    예를 들어 라이브러리 사용하는데, final로 선언된 클래스에 메소드를 추가 하고 싶다면, 유틸클래스를 만들듯 만들고 싶은 메소드를 만들수 있다고 한다.
#

### 의도치 않게 인스턴스화가 가능할 때가 있다.

유틸클래스의 의도는 인스턴스를 만들어 쓰려고 만든게 아니다. 

그럼에도 불구하고, 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 생성한다. 컴파일러가 기본 생성자를 만듦으로써 (public으로)사용자는 이 생성자가 자동생성된 것인지 구분할 수 없다.  
#


### 추상클래스로 만들면 인스턴스화를 막을 수 있지 않을까?

아니다, 하위 클래스를 만들어서 인스턴스화 하면 그만이고, 상속해서 사용하라는 의미로 받아들일 수 있다.  


#
### 의외로 간단한 인스턴스화 막는 방법: *private 생성자를 추가하자!*

명시적 생성자를 **private**으로 생성하면 클래스 밖에서 접근할 수 없다. 때문에 인스턴스화를 막을 수 있다. 

이 코드는 상속을 불가능하게 하는 효과도 있다. 모든 생성자는 상위 클래스의 생성자를 호출하게 되어 있는데, private으로 선언함으로써 상위클래스의 생성자를 호출할수 없다. 

```java
public class UtilityClass {
	// 기본 생성자가 만들어 지는 것을 막는다(인스턴스화 방지용)
	private UtilityClass() {
		throw new Error();
	}
}
```
