# 아이템 10. equals는 일반 규약을 지켜 재정의하라

우리가 사용하는 Object 클래스는 모든 클래스의 최상위 클래스로서, 여기에 선언된 final이 아닌 메서드 equals, hashCode, toString, clone,
finalize는 모두 final이 아니다. 그래서 재정의를 해서 사용을 염두하고 설계가 되었다. 이 중에서 먼저 equals 메서드부터 살펴보겠다.

## equals는 재정의 하지 말아라

equals 메서드는 재정의를 해서 각자 상황에 맞게 사용을 하려는 경우가 있겠지만, 곳곳에 위험이 있기 때문에 안하는 것이 권장된다. 
특히 다음과 같은 상황에서는 재정의를 하지 않는게 가장 좋은 방법이다.

1. 각 인스턴스가 본질적으로 고유하다. 
2. 인스턴스의 논리적 동치성 (logical equality)을 검사할 일이 없다.  
3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다. 
4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

### equals를 그렇다면 재정의를 해야 하는 상황은 언제일까? 
equals 메서드는 객체 식별성 (object identity; 두 객체가 물리적으로 같은가)의 상황이 아닌, 논리적 동치성을 확인해야 하는 경우이다. 즉, 
상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 하지 않을 때를 equals를 재정의 해야 한다. 대표적으로 값 클래스들이 여기에 해당된다.
값 클래스란, Integer, String 처럼 값을 표현하는 클래스를 의미한다. 
<br>
Integer나 String 처럼 값을 표현하는 클래스에서는 객체가 같은지보다는 값이 같은지를 비교하고 싶기 때문에 equals가 논리적 동치성을 확인하도록
재정의를 하면 인스턴스는 값 비교는 물론, Map의 키와 Set의 원소로 사용할 수 있게끔 한다. 

<I>cf) 인스턴스 통제 클래스 <br>
값 클래스라고 해도 equals를 재정의 할 필요 없는 클래스가 존재한다. 대표적으로 ENUM을 예로 들 수 있다. 이런 클래스는 어차피 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않으므로, 논리적 동치성과 객체 식별성이
사실상 같은 의미가 된다. 따라서 Object의 equals가 논리적 동치성까지 확인해준다.</I>

### equals 메서드를 재정의 하기 위해 따라야 하는 규약
equals 메서드는 동치관계 (equivalence relation)를 구현하며, 다음을 만족한다. 
1) 반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다. 
2) 대칭성(symmetry): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true이다.
3) 추이성(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z) true면, x.equals(z)도 true 이다.
4) 일관성(consistency): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
5) null - 아님 : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

### equals 메서드를 구현하는 방법
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다. 
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다. 
3. 입력을 올바른 타입으로 형변환한다. 
4. 입력 객체와 자기 자신의 대응되는 '핵심'필드들이 모두 일치하는지 하나씩 검사한다. 


    기본 타입 필드 비교 : ==
    참조 타입 필드 비교 : equals
    float, double 타입 필드 비교 : 각 정적 메서드 compare 사용

#### equals를 작성하고, 테스트 해주는 오픈소스 : AutoValue 프레임워크
클래스에 annotation을 추가하면, AutoValue가 메서드를 알아서 작성해준다. 대부분의 IDE도 제공을 하지만, AutoValue가 가장 깔끔하니 참고하면
좋다. 

    정리: equals 재정의는 꼭 필요한 경우에만한다. 만약 한다면, 다섯 가지 규약을 지켰는지 확인을 하고, 테스트를 하자.








