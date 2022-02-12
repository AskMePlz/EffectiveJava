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
b의 결과값은 예상과 다르게 null이다. 그 이유는 hashcode를 재정의 해주지 않으면 Object의 hashcode 함수를 사용할테니 물리적으로 다른 두 객체가 논리적으로 같은지 알 수 없다.
  ![Screen Shot 2022-02-12 at 5 25 33 PM](https://user-images.githubusercontent.com/24830023/153703682-bdc31cdb-59ef-41e4-935a-8dc1d03dc04f.png)

  ![Screen Shot 2022-02-12 at 5 25 44 PM](https://user-images.githubusercontent.com/24830023/153703686-ff768b77-9e38-47a1-85e8-dc2208544558.png)

  
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
  
 -  **좋은 해시코드 만들기**
    - 좋은 해시 함수는 서로 다른 인스턴스에 다른 해시코드를 만든다. 
    - 이상적인 해시 함수는 주어진 (서로 다른) 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.
    - 다른 필드로부터 계산해 낼 수 있는 필드는 모두 무시해도 된다.
    
  ```java
    @Override
    public int hashCode() {
        int c = 31;
        int result = Short.hashCode(areaCode);
        result = c * result + Short.hashCode(prefix);
        result = c * result + Short.hashCode(lineNumber);
        return result;
    }
 ```
</details>

<details>
<summary><b>Object.hash()를 쓰면 편하다.</b></summary> 
<br>
  
```java
  @Override public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
  }
```
하지만 내부적으로 auto boxing이 일어나서 성능이 떨어진다.
</details>
<details>
<summary><b>주의 사항</b></summary> 
<br>
  
- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면 매번 새로 계산하기 보다는 캐싱하는 방법을 쓰자.
- 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.
- hascode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. <br>
  그래야 클라이언트가 이 값에 의지하지 않게 되고 추후에 계산 방식을 바꿀 수도 있다.
</details>
