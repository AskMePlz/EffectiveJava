# [item 10] equals는 일반 규약을 지켜 재정의하라



## equals 메서드는 다음에서 열거한 상황중 하나에 해당하면 재정의 하지 않는 것이 최선이다.

1. 각 인스턴스가 본질적으로 고유하다.
2. 인스턴스의 논리적 동치성(logical equality)를 검사할 일이 없다.
3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
4. 클래스가 private이거나 package-private이고 equals메서드를 호출할 일이 없다. 

## 그렇다면 equals를 재정의 해야 할 때는 언제일까

객체 식별성(object identity; physically equals)이 아니라 논리적 동치성(logically equals)를 확인해야하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 되지 않았을 때다.

- 값 클래스 (Integer, String)
    
    equals의 재정의로는 값 클래스가 예가 될 수 있다. 값 클래스의 경우 객체가 아니라 값이 같은지 알고 싶은 경우가 대부분이기 때문에 equals가 논리적 동치성을 확인하도록 재정의해두면 그 인스턴스는 값을 비교할 수 있게 된다. 
    
    - enum
    
    하지만 같은 값 클래스라도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 재정의하지 않아도 된다. 
    

## equals 메서드 재정의 시 지켜야하는 일반 규약

equals 메서드는 동치관계를 구현하며, 다음을 만족한다.

1. 반사성(reflexivity)
2. 대칭성(symmetry)
3. 추이성(transitivity)
4. 일관성(consistency)
5. null 아님

### 이 규약을 지켜야하는 이유

수많은 클래스들은 전달받은 객체가 이 규약을 잘 지킨다는 가정하에 동작하기 때문에 이 규약을 어기면 프로그램이 이상하게 동작하거나 종료될 것이다. 

## 동치 관계(equivalence relation)란 무엇인가?

동치관계란 모든 원소 사이에서 동치율(같은 비율)이 성립하는 것을 말한다. 즉, 두 대상이 같은지 다른지를 구분하게 해주는 것이다.

> 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산이다.
$X = \{a,b,c\}$ 집합이 있고
> 
> 
> $R = \{(a,a),(a,a),(b,b),(c,c),(b,c),(c,b)\}$라는 릴레이션이 있다.  아래의 집합은 릴레이션 클래스의 동치클래스다. 
> 
> $[a] = \{a\}, [b] = [c] = \{b,c\}$
> 

 **예제** 

- "Is equal to" on the set of numbers. For example, ${\displaystyle {\tfrac {1}{2}}}$ is equal to ${\displaystyle {\tfrac {4}{8}}}$
- 도형에서 합동일 경우 동치관계라고 부른다.

## 조건 살펴보기

1. **반사성(reflexivity)**

$(a = a)$

객체는 자기 자신과 같아야 한다. 

2. **대칭성(symmetry)**

$(a=b) ⇒ (b=a)$

서로에 대한 동치여부에 똑같이 답해야한다.

```java
import java.util.Objects;

public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        }
        if (o instanceof String)
            return s.equalsIgnoreCase((String) o);
        return false;
    }
}

class Main {
    public static void main(String[] args) {
        CaseInsensitiveString caseString = new CaseInsensitiveString("test");
        String s = "test";
        System.out.println(caseString.equals(s)); // true
        System.out.print(s.equals(caseString)); // false
    }
}
```

위 코드에서 대칭성이 성립하려면 `caseString.equals(s)` 의 결과와 `s.equals(caseString)` 의 결과가 동일해야 한다.

**대칭성 성립을 위해서는 어떻게 바뀌어야할까?**

equals를 String과도 연동하겠다는 허황된 꿈을 버리고 이렇게 구현해보자

```java
@Override
    public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }
```

3. **추이성(transitivity)**

$(a=b) ∧ (b=c)⇒ (c=a)$

첫번째 객체와 두번째 객체가 같으면 첫번째 객체와 세번째 객체가 같다.                                                                                                                                

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point point = (Point) o;
        return x == point.x && y == point.y;
    }
}
```

```java
public class ColorPoint extends Point {
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
}
```

**위와 같이 구현된 상태에서 x, y, color 값 모두 같은지 확인하려면 어떻게해야 할까?** ColorPoint에 equals 메서드를 추가해보자.

```java
@Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```

하지만 이 코드는 대칭성(규약2)에 위배 된다. (a = b and b = a 가 일치하지 않는다) 아래 코드를 통해 확인 할 수 있다. 

```java
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);
System.out.println(p.equals(cp)); // true
System.out.println(cp.equals(p)); // false
```

**그렇다면 Point와 비교할 때는 색상을 무시하도록하면 해결되지 않을까?**

아래와 같이 바꿔보자. 이로써 대칭성 문제가 해결되었다!!! 

```java
@Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        
        if (! (o instanceof ColorPoint)) return o.equals(this);
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```

아래 코드를 통해 확인해보자.  대칭성 문제는 예상대로 해결했다. 하지만 추이성이 깨진다. p1, p2 비교, p2, p3 비교는 색상 상관없이 비교했지만 p1, p3는 색상까지 고려해 비교했기 때문에 false이다.

```java
// 대칭성
System.out.println(p1.equals(p2)); // true
System.out.println(p2.equals(p1)); // true

// 추이성
System.out.println(p1.equals(p2)); // true
System.out.println(p2.equals(p3)); // true
System.out.println(p1.equals(p3)); // false
```

**방법이 없을까?**

구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다. 하지만 괜찮은 우회 방법은 있다. **상속 대신 컴포지션을 사용**하는 것이다. 아래 코드를 보자.

> 컴포지션(composition)
> 
> 
> 기존 클래스를 확장하는 대신 새로운 클래스를 만들고 private필드로 기존 클래스의 인스턴스를 참조하게 하자. 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻으로 컴포지션이라 한다.
> 

```java
class ColorPoint {
    private final Point point;
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = color;
    }

    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) return false;

        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(o) && cp.color.equals(color);
    }
}
```

4. **일관성(consistency)**

두 객체가 같다면(어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 뜻이다. 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원을 끼지 말아라. 이를 어기면 일관성 조건을 만족시키기가 어렵다. 

5. **null 아님**

모든 객체가 null과 같지 않아야 한다. 일반 규약은 `NullPointerException`을 던지는 경우도 허용하지 않는다. 아래 코드의 묵시적 null 검사를 보자. equals가 타입을 확인하지 않으면 잘못된 타입이 인수로 주어졌을 때 ClassCastException을 던져서 일반 규약을 위배하게 된다. 그런데 instanceof는 첫번째 피연산자가 null이면 바로 false를 반환하여 명시적으로 검사할 필요가 없다.

```java
// 묵시적 null 검사
@Override
    public boolean equals(Object o) {
        if (!(o instanceof MyType)) return false;

				MyType mt = (MyType) o;
        ...
    }

// 명시적 null 검사
@Override
    public boolean equals(Object o) {
        if (o == null) return false;

        ...
    }
```

### 양질의 equals메서드 구현 방법

1. == 연산자를 사용해 입력이 자기 자신의 참조인가 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.

### 주의사항

- equals를 재정의할 땐 hashcode도 반드시 재정의
- 필드들의 동치성만 검사해도 equals규약을 어렵지 않게 지킬 수 있다.
- object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.

### reference

[https://en.wikipedia.org/wiki/Equivalence_relation](https://en.wikipedia.org/wiki/Equivalence_relation)
