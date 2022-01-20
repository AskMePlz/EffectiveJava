
# [item5] 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

하나 이상의 자원에 의존하는 클래스가 정적 유틸클래스나 싱글턴으로 구현한 모습을 많이 볼 수 있다. 하지만 이 방식은 유연하지 않고 테스트하기 어렵다. 

```java
public class SpellChecker {
	private static final Lexicon dictionary = ...;
		
	private SpellChecker() {} // 객체 생성 방지
		
	public static boolean isValid(String word) { ... }
	public static List<String> suggestion(String typo) { ... }
}
```

```java
public class SpellChecker {
	private final Lexicon dictionary = ...;
		
	private SpellChecker() {}
	public static SpellChecker INSTANCE = new SpellChecker(...);
		
	public static boolean isValid(String word) { ... }
	public static List<String> suggestion(String typo) { ... }
}
```
# 
### SpellCheck가 여러 사전을 사용할 수 있도록 만들어보기

**방법1.** 
**필드에서 final 한정자를 제거하고 다른 사전으로 교체하는 메서드를 추가해보자.**
기존 코드에서는 static final을 변수에 지정함으로서 해당데이터의 의미, 용도를 고정시켰다. 그런데 final을 제거하고 다른 사전으로 교체하는 메서드를 추가한다면 여러 사전을 사용할 수 있게 된다.

하지만, 이 방법은 적절한 방법은 아니다. 이 방법은 멀티 스레드 환경에서는 쓸 수 없고 어색하다. final이 아닌 필드에 대한 접근을 동기화로 보호하지 않았을 때 데이터 경쟁(data race)이 일어날 수 있기 때문이다.


**방법2**. 
**인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식을 사용해보자.**
의존 객체 주입 패턴을 사용하자 (인스턴스 생성시 생성자에 필요한 자원을 넘겨주자)

```java
public class SpellChecker {
	private final Lexicon dictionary;

	public SpellChecker(Lexion dictionary) {
		this.dictionary = Objects.requireNoneNull(dictionary);
	}

	public boolean isValid(String word) {...}
	public List<String> suggestions(String typo) {...}
}
```
#
### 의존 객체 주입 패턴의 변형, 팩토리 메서드 패턴

**팩토리**란 호출할때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다.

**[팩토리 메서드 패턴](https://ko.wikipedia.org/wiki/%ED%8C%A9%ED%86%A0%EB%A6%AC_%EB%A9%94%EC%84%9C%EB%93%9C_%ED%8C%A8%ED%84%B4#:~:text=%ED%8C%A9%ED%86%A0%EB%A6%AC%20%EB%A9%94%EC%84%9C%EB%93%9C%20%ED%8C%A8%ED%84%B4(Factory%20method,%ED%95%98%EB%8F%84%EB%A1%9D%20%ED%95%98%EB%8A%94%20%ED%8C%A8%ED%84%B4%EC%9D%B4%EA%B8%B0%EB%8F%84%20%ED%95%98%EB%8B%A4. )**

> Factory method는 부모(상위) [클래스](https://ko.wikipedia.org/w/index.php?title=%ED%81%B4%EB%9E%98%EC%8A%A4_(%EC%A0%84%EC%82%B0%ED%95%99)&action=edit&redlink=1)에 알려지지 않은 구체 클래스를 생성하는 패턴이며. 자식(하위) 클래스가 어떤 객체를 생성할지를 결정하도록 하는 패턴이기도 하다. 부모(상위) 클래스 코드에 구체 클래스 이름을 감추기 위한 방법으로도 사용한다
> 

**팩토리 메서드 패턴**의 예로는 `Supplier<T>` 인터페이스가 있다. 

#
### 결론

필요한 자원을 생성자에 넘겨주자. 의존 객체 주입 기법은 클래스의 유연성, 재사용성 테스트 용이성을 기맥히게 개선해준다.
