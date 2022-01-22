#### 아이템5
# 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

### 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티, 싱글턴 방식이 적합하지 않다. 
- 정적 유틸리티 사용을 잘못한 예
```
public class SpellCheck{
	private static final Lexicon dic=...;
	private SpellCheck() {} // 객체 생성 방지
}
```
- 싱글턴을 잘못 사용한 예
```
public class SpellCheck{
	private final Lexicon dic=...;
	private SpellCheck() {}
	public static SpellCheck INSTANCE = new SpellCheck(...);
}
```

>> 위 예시 모두 유연하지 않고 테스트가 어렵다. (맞춤법 검사기 예시)
>> 하나의 사전이 아닌 다양한 사전이 필요한데 사전 하나로 모든 쓰임에 대응하기 어려움

### 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식(의존 객체 주입의 한 형태)
- 생성자에 자원을 넘겨주는 방식 : 맞춤법 검사기 생성 시, 의존 객체인 사전을 주입
```
public class SpellCheck{
	private final Lexicon dic;
	
	public SpellCheck(Lexicon dic){
		this.dic = Object.requireNonNull(dic);
	}
} 
```
- 의존객체 주입은 생성자, 정적팩터리, 빌더 모두에 똑같이 응용 가능
- 위 예에서는 dic이라는 자원 하나를 사용하지만 자원에 몇개든 의존관계가 어떻든 상관없이 작동

### 팩터리 메서드 패턴
- 의존 객체 주입 패턴의 변형
- 생성자에 자원 팍터리를 넘겨주는 방식 ( Supplier<T> 인터페이스 )

