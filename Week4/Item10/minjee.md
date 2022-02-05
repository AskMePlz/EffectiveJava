#### 아이템 10
# equals는 일반 규약을 지켜 재정의하라


### 다음 상황 중 하나에 해당하다면 equals 메서드는 재정의 하지 않는 것이 최선이다.
- 각 인스턴스가 본질적으로 고유하다.
  - 값을 표현하는 것이 아니라 동작하는 개체를 표한하는 클래스
  - 예 : Thread, Object의 equals 메서드는 이러한 클래스에 맞게 구현
- 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.
  - 값을 비교해서 동등한지 비교할 일이 없다면 논리적 동치성 검사가 필요없다는 것이고 기본적은 Object의 equals로도 충분하다.
  - 예 : java.util.regex.Pattern은 equals를 재정의 하여 두 pattern의 인스턴스가 같은 정규식을 나타내는지 확인하는 논리적 동치성 검사. 
        위와 같은 경우가 아닌 이상 필요없음.  
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다
- 클래스가 private이거나 pakage-private 이고 equals 메서드를 호출할 일이 없다.

### equals

##### 참고
논리적 동치성(logical equality)  :

##### 참고자료

