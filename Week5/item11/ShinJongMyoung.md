# Item11. equals를 재정의하려거든 hashcode도 재정의하라

### hashCode 일반 규약
- equals() 비교에 사용되는 필드가 변하지 않았다면, hashCode() 메서드는 몇번을 호출하든, 항상 같은 값을 반환해야 한다.
단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
- equals(Object)가 두 객체를 같다고 판단했으면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 
단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.
  
### hashCode 구현
1. 핵심 필드 (equals() 비교에 사용되는 필드) 에 대해 다음 작업을 수행한다.
- 기본 타입 필드면, Type.hashCode(f)를 수행한다. 여기서 Type이란, 기본 타입의 박싱 클래스다.
- 참조 타입 필드면서, 이 클래스의 equals() 메서드가 이 필드의 equals() 메서드를 재귀적으로 호출한다면, 이 필드의 hashCode()를 재귀적으로 호출한다.
    - 계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 표준형의 hashCode()를 호출한다.
    - 필드의 값이 null이면 0을 사용한다. (다른 상수도 괜찮지만 전통적으로 0을 사용한다.)
- 필드가 배열이라면, 핵심원소 각각을 별도 필드처럼 다룬다.
    - 배열에 핵심원소가 하나도 없다면 0(권장) 혹은 다른 상수를 사용한다.
    - 모든 원소가 핵심 원소라면 Arrays.hashCode()를 사용한다.
2. 위의 작업에서 계산한 해시코드로 result를 갱신한다.
```java
result = 31 * result + c
```
3. result를 반환한다.

### 구현된 전형적인 hashCode 메서드
```java
class PhoneNumber { 
    public final String areaCode; 
    public final String prefix; 
    public final String lineNum; 
    
    public PhoneNumber(String areaCode, String prefix, String lineNum) { 
        this.areaCode = areaCode; 
        this.prefix = prefix; 
        this.lineNum = lineNum; 
    } 
    
    @Override 
    public boolean equals(Object o) { 
        if (this == o) return true; 
        if (o == null || getClass() != o.getClass()) return false; 
        PhoneNumber that = (PhoneNumber) o; 
        return Objects.equals(areaCode, that.areaCode) && Objects.equals(prefix, that.prefix) && Objects.equals(lineNum, that.lineNum); 
    }

    @Override
    public int hashCode() {
        int result = areaCode.hashCode();
        result = 31 * result + prefix.hashCode();
        result = 31 * result + lineNum.hashCode();
        return result;
    }
}
```

### 한줄 짜리 hashCode 메서드
```java
@Override
public int hashCode() {
    Objects.hash(lineNum, prefix, areaCode);
}
```
- 입력 인수를 담기 위한 배열을 만들어지고, 입력 중 기본 타입이 있다면 박싱, 언박싱도 거쳐야 하기에 성능이 아쉽다.
- 성능에 민감하지 않은 상황에서만 사용하자.

<br>
<br>

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야한다.
### 해시코드를 지연초기화 하는 hashCode 메서드
```java
private int hashCode; 

@Override 
public int hashCode() { 
    int result = hashCode; 
    if(result == 0) { 
        result = Short.hashCode(areaCode); 
        result = 31 * result + Short.hashCode(prefix); 
        result = 31 * result + Short.hashCode(lineNum); 
        hashCode = result; 
    } 
    
    return result; 
}
```


