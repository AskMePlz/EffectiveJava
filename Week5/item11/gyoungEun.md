<details>
<summary><b>equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.</b></summary> 
  equals를 재정의한 클래스 모두에서 hashcode도 재정의해야 한다. 그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 컬렉션의 원소로 사용할 때 문제를 일으킨다. 
<br>
<br>
  
  [The general contranct of hashCode](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html#hashCode())

- **condition1** <br>
  equals **비교에 사용되는 정보가 변경되지 않았다면**, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드의 반환값은 **멱등성**을 보장해야한다. (단, 애플리케이션을 다시 수행한다면 값이 달라져도 상관없음)
- **condition2** <br>
  equals(Object)가 두 객체를 **같다고** 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야한다. (논리적으로 같은 인스턴스는 같은 해시코드를 반환해야한다)
- **condition3** <br>
  equals(Object)가 두 객체를 **다르다고** 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.
</details>

<details>
<summary><b>논리적으로 같은 인스턴스는 같은 해시코드를 반환해야한다 (일반 규약 조건2 위배)</b></summary> 
<br>
  
  ```java
import java.util.HashMap;

class AmbiguousInteger {
    private final int value;

    AmbiguousInteger(int value) {
        this.value = value;
    }

    // public int hashCode() { return 9; }
}

class Main {
    public static void main(String[] args) {
        HashMap<AmbiguousInteger, Integer> map = new HashMap<>();
        map.put(new AmbiguousInteger(1), 1); // a
        map.get(new AmbiguousInteger(1)); // b
    }
}
  ```
  
  a라인까지 실행한 그림
  
  ![Screen Shot 2022-02-03 at 12 16 46 PM](https://user-images.githubusercontent.com/24830023/153381678-7c0635b3-cdcf-48cb-b231-e54122ba0d64.png)

  
  b라인까지 실행하면 이렇다. hashcode를 재정의 해주지 않으면  Object의 hashcode 함수를 사용할테니 물리적으로 다른 두 객체가 논리적으로 같은지 알 수 없다. 
  ![Screen Shot 2022-02-03 at 12 19 11 PM](https://user-images.githubusercontent.com/24830023/153382122-344fffe3-f9f2-4dd8-b0c1-96d4dbcdda0f.png)
  
</details>

<details>
<summary><b>좋은 해시함수는 다른 인스턴스면 다른 해시코드가 되게하는게 좋다(일반 규약 조건3 위배)</b></summary> 
<br>
  
</details>

<details>
<summary><b>equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.</b></summary> 
<br>
</details>
