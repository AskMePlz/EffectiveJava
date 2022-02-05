# Item10. equals 는 일반 규약을 지켜 재정의하라

### 재정의하지 않는 경우
- 각 인스턴스가 본질적으로 고유할 경우
    - 값이 아니라 동작하는 개체를 표현하는 클래스
    - ex. Thread
- 인스턴스의 논리적 동치성(logical equality) 을 검사하지 않을 경우
- 상위 클래스에서 재정의한 equals가 하위 클래스에서도 적합할 경우
    - ex. Set-AbstractSet List-AbstractList Map-AbstractMap
- 클래스가 private이거나 package-private이고, eqauls 메소드를 호출할 일이 없을 경우

### 재정의해야 하는 경우
- 객체 식별성(두 객체가 물리적으로 같은가)이 아니라 `논리적 동치성`을 확인해야 하는 경우
    - 값 클래스 (Integer, String)
        - 두 값 객체의 객체가 같은지가 아니라 값이 같은지를 비교하고 싶을 경우
        - Map의 키와 Set의 원소로 사용할 수 있음
    - 값 클래스라 해도, 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 equals를 재정의하지 않아도 됨
    - ex. Enum
    
### equals 메서드를 재정의 할 때 지켜야하는 일반 규약
1. 반사성
    - null이 아닌 모든 참조 값 x에 대해 x.equals(x)는 true다.
2. 대칭성
    - null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)가 true면 y.equals(x)도 true다.
    - 대칭성 위배 예시
    ```java
    public final class CaseInsensitiveString {
        private final String s;
        
        public CaseInsensitiveString(String s) {
            this.s = Objects.requireNonNull(s);
        }
        
        // 대칭성 위배!
        @Override
        public boolean equals(Object o) {
            if(o instanceof CaseInsensitiveString) {
                return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
            }
        
            if(o instanceof String) { // 한 방향으로만 작동한다.!!
              return s.equalsIgnoreCase((String) o);
            }
            return false;
        }
        ...
    }
    ```
}

3. 추이성
    - null이 아닌 모든 참조 값 x,y,z에 대해 x.equals(y)가 true이고, y.equals(z)가 true면 x.equals(z)도 true다.
4. 일관성
    - null이 아닌 모든 참조값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
5. null아님
    - null이 아닌 모든 참조값 x에 대해 x.equals(null)은 false다.

### equals 메서드 구현 방법
1. == 연산자를 사용해 자기 자신의 참조인지 확인
2. instanceof 연산자로 입력이 올바른 타입인지 확인
3. 입력을 올바른 타입으로 형변환
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사

### 주의 사항
- equals 를 재정의할 땐 hashCode 도 반드시 재정의하자(Item 11)
- 필드의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있다. 또한 일반적으로 별칭(alias)는 비교하지 않는게 좋다.
- object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자