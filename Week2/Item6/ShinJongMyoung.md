# Item6. 불필요한 객체 생성을 피하라

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다.

### 1. String
```java
String s = new String("bikini");
```
- 위 문장은 실행 될때마다 새로운 String 인스턴스를 만든다. 생성자로 넘겨진 `"bikini"`자체가 이 생성자로 만들어내려는 String 과 기능적으로 완전히 똑같다.

```java
String s = "bikini";
```
- 위 코드는 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 사용한다.
- 나아가, 이 방식을 사용한다면 같은 가상 머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장된다.

### 2. 생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스
```java
public final class Boolean implements java.io.Serializable, Comparable<Boolean> {

    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);
    
    @Deprecated(since="9")
    public Boolean(boolean value) {
        this.value = value;
    }

 
    @HotSpotIntrinsicCandidate
    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
    ...
```
- `Boolean(boolean value) 생성자` 대신 `Boolean.valueOf(boolean b) 팩터리 메서드` 를 사용하는 것이 좋다.
- 생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 전혀 그렇지 않다.

### 3. 생성비용이 비싼 객체
```java
static boolean isRomanNumeralSlow(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
- 위 코드는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해서 사용하기엔 적합하지 않다.
- 메서드 내부에서 만들어지는 Pattern 인스턴스가 한번 쓰고 버려져 가비지 컬렉션 대상이 되기 때문이다.
- 여기서 Pattern 은 입력받은 정규표현식에 해당하는 유한상태머신을 만들기에 인스턴스 생성비용이 높다.

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    
    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```
- 위 처럼 정규표현식을 표현하는 Pattern 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱해두고, 나중에 `isRomanNumeral` 메서드가 호출될 때마다 이 인스턴스를 재사용한다.
- 이렇게 개선하면, `isRomanNumeral` 메서드가 빈번하게 호출되는 상황에서 성능을 상당히 끌어올릴 수 있다.
- 또, Pattern 인스턴스에 이름을 지어주어 코드의 의미가 훨씬 잘 드러난다.

### 4. 오토박싱
```java
public static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
}
```
- 위 코드는 sum 에 i 를 더할 때, 불필요한 Long 인스턴스가 약 2^31 개 만들어진다.
- 박싱 객체 사용보다는 기본 타입을 권장하며 의도치 않은 오토박싱이 일어나지 않도록 주의해야한다.

### 주의할 점
- `"객체 생성은 비싸니 피해야 한다"` 로 오해하면 안된다.
- JVM에서 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 것은 크게 부담이 되지 않는 일이다. 또한 프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것은 일반적으로 좋다. 

