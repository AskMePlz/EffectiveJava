#### 아이템6
# 불필요한 객체 생성을 피하라

>> 똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을  때가 많다.
>> 재사용은 빠르게 세련된다.

### 하지 말아야하는 극단적인 예 

```
	String s = new String("cat"); // 따라하지 말 것!!!***
	Stirng s = "cat";
```
- 전자 : 문장이 실행될 때마다 인스턴스 새로 생성
- 후자 : 하나의 Stirng 인스턴스 사용

### 정적 팩터리 메서드를 사용, 불필요한 객체 생성을 피할 수 있다.
- Boolean(String) 생성자 대신 Boolean.valueOf(Stirng) 팩터리 메서드 사용
	: 생성자는 호출할 때마다 새로운 객체를 만들지만 팩터리 메서드는 전혀 그렇지 않다.

### 생성비용이 비싼 객체가 반복해서 필요하다면 캐싱하여 재사용하길 권한다.

```
	static boolean isRomanNumeral(String s) {
		return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
	}
``` 
- String.matches : 정규표현식 문자열 형태를 확인하는 가장 쉬운 방법이지만 성능이 중요한 상황에서는 반복해 사용하기 적합하지 않다.
- 내부에서 만드는 정규표현식용 Pattern 인스턴스는 한번 쓰고 버려져 곧바로 가비지 컬렉션 대상이며 Pattern은 입력받은 정규 표현식에 해당하는 유한 상태 머신을 만들기 떄문에 인스턴스 비용이 높음

```
	private static final Pattern ROMAN = Pattern.compile(
		"^(?=.)M*(C[MD]|D?C{0,3})"+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

	static boolean isRomanNumeral(String s) {return ROMAN.matcher(s).matches();}
``` 
- Pattern 인스턴스를 클래스 초기화(정적 초기화) 과정에서 직접 캐싱, 나중에 isRomanNumeral 메서드가 호출될 때마다 이 인스턴스를 재사용

### 오토박싱 : 불필요한 객체를 만들어내는 또 다른 예
- 오토박싱은 프로그래머가 기본 타입, 박싱된 기본타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술
- 오토박싱은 기본 타입, 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만 완전히 없애주는 것은 아님

```
	private static long sum() {
		Long sum = 0L;
		for (long i = 0; i <= Integer.MAX_VALUE; i++) sum += i;
		return sum;
	}
``` 
- sum을 Long으로 선언하여 더해질때마다 불필요한 Long 인스턴스 생성	₩
- sum타입을 long으로 바꿔주면 저자의 컴퓨터에서는 6.3초에서 0.59초로 빨라짐
