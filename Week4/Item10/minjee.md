#### 아이템 10
# equals는 일반 규약을 지켜 재정의하라


### 다음 상황 중 하나에 해당하다면 equals 메서드는 재정의 하지 않는 것이 최선이다.
- 각 인스턴스가 본질적으로 고유하다.
  - 값을 표현하는 것이 아니라 동작하는 개체를 표한하는 클래스
  - 예 : Thread, Object의 equals 메서드는 이러한 클래스에 맞게 구현.
- 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.
  - 값을 비교해서 동등한지 비교할 일이 없다면 논리적 동치성 검사가 필요없다는 것이고 기본적은 Object의 equals로도 충분하다.
  - 예 : java.util.regex.Pattern은 equals를 재정의 하여 두 pattern의 인스턴스가 같은 정규식을 나타내는지 확인하는 논리적 동치성 검사. 
        위와 같은 경우가 아닌 이상 필요없음.  
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다
  - 상위에서 구현한 equals 로직으로 충분한 경우 재정의가 아닌 상위 클래스에 정의된 equals를 사용한다.
  - 예 : Set 구현체는 AbstractList / Map 구현체는 AbstracMap으로부터 상속받아 사용.
- 클래스가 private이거나 pakage-private 이고 equals 메서드를 호출할 일이 없다.
  - inner-class / 중첩 클래스(nested-class)
  - equals가 실수라도 호출되는 것을 막고 싶다면 다음과 같이 구현
    ```
    @Override public boolean equals(Object o){
      throw new AssertionError(); // 호출금지
    }
    ```

### equals 재정의, 언제하는가?
- 객체 식별성(두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야하는데 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때다.
  - 주로 값 클래스(Integer, String)가 해당
  - 값 클래스라도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스(아이템1) 라면 재정의 필요 없음.   
    (어짜피 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않으니 논리적 동치성과 객체 식별성이 사실상 같은 의미)
    
### Object 명세에서 말하는 동치관계
집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산, 이 부분 집합을 동치류(equivalence class, 동치 클래스)라 한다.
equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다.

### equals 메서드 재정의 규액 ( Object 명세에 적힌 규약)
equals 메서드는 동치관계(equivalence relation)를 구현하며 다음을 만족한다
이 규약을 어기면 프로그램의 이상동작 또는 종료될 것이며 원인이 되는 코드를 찾기 어려울 것이다. 한 클래스의 인스턴스는 다른곳으로 빈번히 전달되며 컬력션 클래스를 포함, 수많은 클래스는 전달받은 equals 규약을 지킨다고 가정하에 동작한다.

- 반사성(reflexivity) : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
- 대칭성(symmetry) : null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)가 true면 y.equals(x)도 true다.
- 추이성(transitivity) : null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고, y.equals(z)도 true면, x.equals(z)도 true다.
- 일관성(consistency) : null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님 : null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false다.

### 동치관계를 만족시키기 위한 다섯요건 상세 내용
1. 반사성 
  - 객체는 자기자신과 같아야 한다.
2. 대칭성
3. 추이성
4. 일관성
5. null-아님

##### 참고
논리적 동치성(logical equality)  :

##### 참고자료
도서 정리 자료
https://parkadd.tistory.com/84
https://catsbi.oopy.io/313829e4-e869-48fe-9fb4-adcca06de1c5
https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/01/04/eqauls%EB%A5%BC-%EC%9E%AC%EC%A0%95%EC%9D%98-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95/
