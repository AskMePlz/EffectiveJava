#### 아이템 11
# equals를 재정의하려거든 hashCode도 재정의하라.   

>> equals를 재정의한 클래스 모두에서 hashCode도 재정의 하지 않으면
>> hashCode 일반 규약을 어기게 되어
>> 해당 클래스의 인스턴스를 HashMap, HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으키는 것

### Object 명세 규약 
hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 아래에서 두번째, 
즉 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
- equals 비교에 사용되는 정보가 변경되지 않았다면, 그 객체의 hashCode 메서드는 몇 번을 호출하더라도 애플리케이션이 실행되는 동안 매번 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 값이 달라져도 상관없다.
- equals(Object)의 결과가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 같은 값을 반환해야 한다.
- equals(Object)의 결과가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode 가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시 테이블(hash table)의 성능이 좋아진다.

### 좋은 hashCode 작성하는 간단 요령
좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환, 이것이 바로 hashCode 세번째 규약이 요구하는 속성이다.

1. int 변수인 result를 선언한 후 값을 c로 초기화한다. 이 때, c는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드이다.
  - 여기서 핵심 필드는 equals 비교에 사용되는 필드를 말한다.
2. 해당 객체의 나머지 핵심 필드인 f 각각에 대해 다음 작업을 수행한다.
  - 해당 필드의 해시코드 c 를 계산한다.
      - 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본타입의 박싱 클래스다.
      - 참조 타입 필드면서, 이 클래스의 equals 메소드가 이 필드의 equals를 재귀적으로 호출하여 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다.
      - 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
  - 단계 2.1에서 계산한 해시코드 c로 result를 갱신한다.
      - 코드 예시 : result = 31 * result + c;
3. result를 반환한다.

### hasCode 구현 후 확인할 점
- 이 메서드가 동치인 인스턴스에 대해 똑같이 해시코드를 반활할지 자문해보자. 
  - 서로 다른 해시 코드를 반환한다면 원인을 찾아 해결
- 파생 필드는 해시코드 계산에서 제외해도 된다.
  - 다른 필드로부터 계산해낼 수 있는 필드는 모두 무시해도 된다.
- equals 비교에 사용되지 않은 필드는 '반드시' 제외 해야 한다.
  - 그렇지 않으면 hashCode 규약 두번째를 어기게 될 위험이 있다.

### 전형적인 hashCode 메서드
```
    @Override public int hashCode() {
        int result = Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(prefix);
        result = 31 * result + Integer.hashCode(lineNum);
        return result;
    }
```
- 인스턴스의 핵심 필드 3개만 사용, 간단한 계산만 수행 
  - 비결정적 요소는 전혀 없으므로 동치인 인스턴스들은 같은 해시코드를 가질 것이 확실.

### 한 줄 짜리 hashCode 메서드
Object 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 hash 제공 
```
@Override public int hashCode() { return Objects.hash(lineNum, prefix, areaCode); }
```
- 입력 인수를 담기위한 배열이 만들어지고 입력 중 기본 타입이 있다면 박싱/언박싱도 거쳐야 하기 때문에 속도가 느리다.

### 해시코드 지연 초기화 - 스레드 안전성까지 고려해야 한다.
- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면 __캐싱하는 방식__을 고려해야 한다.
  - 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해 둔다.
- 해시의 키로 사용되지 않는  경우라면 hasCode가 처음 불릴 때 계산하는 지연 초기화를 한다.
  - 필드를 지연 초기화 하려면 그 클래스가 thread-safe가 되도록 동기화에 신경쓰는 것이 좋다.

```
private int hashCode; //자동으로 0으로 초기화된다.

@Override public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```
__hashCode 필드의 초기값은 흔히 생성되는 객체의 해시코드와 달라야 한다.__

### 마지막 주의 사항
- 성능을 높인다고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수 있다.

##### 참고자료

