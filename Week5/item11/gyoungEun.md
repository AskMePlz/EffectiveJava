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
<summary><b>논리적으로 같은 인스턴스는 같은 해시코드를 반환해야한다 (일반 규약 조건2)</b></summary> 
<br>
  
  ```java
import java.util.HashMap;

class AmbiguousInteger {
    private final int value;

    AmbiguousInteger(int value) {
        this.value = value;
    }
}

class Main {
    public static void main(String[] args) {
        HashMap<AmbiguousInteger, Integer> map = new HashMap<>();
        map.put(new AmbiguousInteger(1), 1); // a
        System.out.println(map.get(new AmbiguousInteger(1))); // b
    }
}
  ``` 
b의 결과값은 예상과 다르게 null이다. 그 이유는 hashcode를 재정의 해주지 않으면  Object의 hashcode 함수를 사용할테니 물리적으로 다른 두 객체가 논리적으로 같은지 알 수 없다.
 <br>
 <br>
 <b> 해결 방법?</b>
 ```java
 public int hashCode() { return 9; }
 ```
 위와 같이 hashCode를 재정의하면 해결이 되긴 된다. 위에서 본 규약의 3가지 조건에도 모두 만족한다. 그런데 문제가 있다. 
 hashMap에 추가되는 모든 노드의 해시값이 같기 때문에, 버킷의 동일한 index에 linkedList와 같은 형태로 저장되기 때문에 성능이 떨어지게 된다. (선형시간)

</details>

<details>
<summary><b>좋은 해시함수는 서로 다른 인스턴스면 다른 해시코드가 되게 하는게 좋다(일반 규약 조건3)</b></summary> 
<br>
 - 좋은 해시코드 만들기
</details>

<details>
<summary><b>equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.</b></summary> 
<br>
</details>
