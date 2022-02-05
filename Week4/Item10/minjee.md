#### 아이템 10
# equals는 일반 규약을 지켜 재정의하라

>> 꼭 필요한 경우가 아니라면 equals는 재정의 하지 말자.  
>> 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.


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

### equals 메서드 재정의 규약  ( Object 명세에 적힌 규약)
equals 메서드는 동치관계(equivalence relation)를 구현하며 다음을 만족한다
이 규약을 어기면 프로그램의 이상동작 또는 종료될 것이며 원인이 되는 코드를 찾기 어려울 것이다. 한 클래스의 인스턴스는 다른곳으로 빈번히 전달되며 컬력션 클래스를 포함, 수많은 클래스는 전달받은 equals 규약을 지킨다고 가정하에 동작한다.

- 반사성(reflexivity) : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
- 대칭성(symmetry) : null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)가 true면 y.equals(x)도 true다.
- 추이성(transitivity) : null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고, y.equals(z)도 true면, x.equals(z)도 true다.
- 일관성(consistency) : null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님 : null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false다.

### 동치관계를 만족시키기 위한 다섯요건 상세 내용
1. 반사성 : x.equals(x)는 true
    - 객체는 자기자신과 같아야 한다.
2. 대칭성 : x.equals(y)가 true면 y.equals(x)도 true
    - 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.
    - 아래 예시에서 equals 메서드의 대칭성 위배 부분(주석처리 부분)으로 실행시키면 각각 다른 결과가 나오는데 (main-print : true, false)
      CaseInsensitiveString는 Stirng의 equals를 알고 있지만 그 반대는 모르기 때문이다.
  ```
  public class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배!
    @Override 
    public boolean equals(Object o) {
       // 대칭성 위배
       /*if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String) 
            return s.equalsIgnoreCase((String) o);
        return false;*/
        // 수정
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }

    // 문제 시연 (55쪽)
    public static void main(String[] args) {
        CaseInsensitiveString a = new CaseInsensitiveString("Polish");
        String b = "polish";

        System.out.println("a.equals(b): " + a.equals(b)); // 수정 후 false
		    System.out.println("b.equals(a): " + b.equals(a)); // 수정 후 false
    }
  }
  ```
3. 추이성 : x.equals(y)가 true이고, y.equals(z)도 true면, x.equals(z)도 true
  - 객체1, 객체2가 같고 객체2, 객체3이 같으면 객체1, 객체3도 같아야 한다는 뜻 : 간단하지만 어기기 쉽다.
 
  ```
   public class Point {
   	private final int x;
    	private final int y;

   	public Point(int x, int y) {
		this.x = x;
        	this.y = y;
   	}

    	@Override public boolean equals(Object o) {
        	if (!(o instanceof Point)) return false;
        	Point p = (Point)o;
        	return p.x == x && p.y == y;
        }
   }
  ```
  ```
  public class ColorPoint extends Point {
  	private final Color color;

	public ColorPoint(int x, int y, Color color) {
        	super(x, y);
        	this.color = color;
    	}
  ```
  - 위와 같은 경우 Point 구현이 상속되어 색상정보를 무시한 채 equals가 수행된다.
    equals 규약을 어긴 것은 아니지만 중요한 정보를 놓칠 수 있으니 다른 equals를 생각해보자
  ```
  @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
   }
   .
   .
   .
   Point p = new Point(1, 2);
   ColorPoint cp = new ColorPoint(1, 2, Color.RED)
   System.out.println(p.equals(cp) + " " + cp.equals(p)); // true false
  ```
  - Point를 ColorPoint에 비교한 결과, 그 반대로 비교한 결과가 다를 수 있으므로 대칭성 위배
  ```
    @Override public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
	//  o가 일반 Point면 색상을 무시하고 비교한다.
        if (!(o instanceof ColorPoint)) return o.equals(this);
	// o가 ColorPoint면 색상까지 비교한다.
        return super.equals(o) && ((ColorPoint) o).color == color;
	.
	.
	.
	ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
        Point p2 = new Point(1, 2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
	// true true fasle
        System.out.printf("%s %s %s%n", p1.equals(p2), p2.equals(p3), p1.equals(p3));
    }
  ```
  -  추이성 위배, 무한 재귀에 빠질 수 있다.
  ```
    @Override public boolean equals(Object o) {
        if (o == null || o.getClass() != getClass()) return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
  ```
  - 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 수 있는 방법은 존재하지 않는다 (객체 지향적 추상화의 이점을 포기하지 않는 한)
  	-> equals 안에 instanceof 검사를 getClass 검사로 바꾸면 규약도 지키고 값도 추가하면서 구체 클래스를 상속한다는 뜻
  - 위 코드는 같은 구현 클래스의 객체와 비교할 때만 true 반환, 그러나 실제로 활용할 수 없음
  - Point 하위 클래스는 정의상 여전히 Point이므로 어디서든 Point로 활용할 수 있어야 한다.
  ```
  public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    /**
     * 이 ColorPoint의 Point 뷰를 반환한다.
     */
    public Point asPoint() {
        return point;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
  ```
  - 상속 대신 컴포지션을 사용하라(아이템18/다른객체의 인스턴스를 자신의 인스턴스 변수로 포함해서 메서드를 호출) 
4. 일관성 : x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환 
  - 두 객체가 같다면 수정되지 않는 한 앞으로 영원히 같아야 한다는 뜻이다.
  - 불변 클래스로 만들기로 했다면 equals가 한번 같다고 한 객체와 영원히 같다고 답하고 다르다고 한 객체와는 영원히 다르다고 답하도록 만들어야 한다.
  - 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.
  - equals를 사용할 때는 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행한다.
5. null-아님 : x.equals(null)은 false
  - 입력이 null인지 확인해 자신을 보호하는 것보다 instanceof 연산자로 입력 매개변수가 올바른 타입인지 검사한다. 
	(입력이 null이면 타입 확인 단계에서 false 반환 )
  ```
  @Override
  public boolean equals(Object o) {    // 명시적 null 검사
    if(o == null)
        return false;
  }

  @Override
  public boolean equals(Object o) {    //묵시적 null 검사
    if(!(o instanceof MyType))
        return false;
  }
  ```

### eauals 메서드 구현 방법
1. == 연산자를 사용, 입력이 자기 자신의 참조인지 확인
2. instanceof 연산자로 입력이 올바른 타입인지 확인
3. 입력을 올바른 타입으로 형변환
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사

### equals를 다 구현했다면 세가지를 자문하자
1. 대칭적인가 
2. 추이성이 있는가
3. 일관적인가
- 이 중 하나라도 실패한다면 원인을 찾아 고치자. 반사성과 null-아님 도 만족해야하지만 이 두가지가 문제되는 경우는 별로 없다.

### 주의사항
- equals를 재정의할 때는 hashcode도 반드시 재정의하자
- 너무 복잡하게 해결하려 들지말자
  필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있으며 오히려 너무 공격적으로 파고들다가 문제를 일으키기도 한다.
- object 외 타입을 매개변수로 받는 equals 메서드는 선언하지 말자
  입력타입이 Object가 아니면 재정의가 아니라 다중정의(아이템 52) 한 것이다.
- equals(hashcode 도 마찬가지) 작성 후 구글에서 만든 Auto Value 프레임워크를 활용하여 테스트 하자


##### 참고자료
도서 정리 자료  
https://parkadd.tistory.com/84  
https://catsbi.oopy.io/313829e4-e869-48fe-9fb4-adcca06de1c5  
https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/01/04/eqauls%EB%A5%BC-%EC%9E%AC%EC%A0%95%EC%9D%98-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95/  
equals, hashCode, == 연산자 비교  
https://jeong-pro.tistory.com/172  
