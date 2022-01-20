# 정적 유틸리티 클래스나 싱글턴 클래스가 적절하지 않은 경우는?

### 💡 앞서 포스팅한 정적 유틸리티성 클래스나 싱클턴 클래스에 의존관계가 발생한다면 어떻게 될까


의존성이 있는 클래스를 아래와 같이 정적 클래스나 싱글턴으로 작성하게 되면 Lotto 호출시 하나의 고정된 LottoPolicy만 적용할 수 있다. 테스트 또한 어려워지고 유연하지 않다. 이렇게 의존 관계가 있을 경우 정적 클래스나 싱글턴은 적절하지 않다. 

```java

// 정적 유틸리티 클래스 잘못된 예 
public class Lotto {
    
    private static final LottoPolicy lottoPolicy = new LottoPolicy();
    private Lotto() {}
    
}
```

```java
// 싱글턴 잘못된 예 
public class Lotto {

    private final LottoPolicy lottoPolicy = new LottoPolicy();
    private Lotto() {}

    public static Lotto INSTANCE = new Lotto();

}
```

# 의존성이 있는 경우 자원 직접 명시 ❌ → 의존 객체 주입 ⭕

### 💡 먼저 자원을 직접 명시한다는 것과 의존 객체를 주입한다는 것 무엇인가?

자원을 직접 명시한다는 것은 아래의 Lotto1 클래스처럼 의존관계에 있는 LottoPolicy클래스를 내부에서 직접 생성하여 명시하는 것을 말한다. 반대로 의존 객체를 주입한다는 것은 Lotto2 클래스와 같이 Lotto2의 생성자 등을 통해 외부에서 생성한 LottoPolicy를 주입 받는것을 말한다. 

```java
public class Lotto1 {

    // 의존 객체 직접 명시 
    private LottoPolicy lottoPolicy = new LottoPolicy();

    public Result apply() {
	lottoPolicy.apply();
    }

}
```

```java
public class Lotto2 {
    private LottoPolicy lottoPolicy;

		// 의존 객체 주입 
    public Lotto2(LottoPolicy lottoPolicy) {
        this.lottoPolicy = lottoPolicy;
    }

    public Result apply() {
	lottoPolicy.apply();
    }
}
```

### 💡 여러 의존 관계가 있고 의존 관계에 따라 동작이 달라지는 클래스는 어떻게 작성하는게 좋을까?


Lotto1 클래스와 같이 의존 관계를 직접 명시하는 것이 아닌 Lotto2와 같이 의존 객체 주입을 활용하는게 좋다.

의존 관계 주입은 앞서 보았던 생성자 뿐만 아니라, 정적 팩토리, 빌더에 모두 응용 가능하다. 

이 패턴의 변형으로 생성자에 자원 팩토리를 넘겨주는 방식도 있다. Supplier 는 대표적인 팩토리 인터페이스이다. 

이를 활용하여 클라이언트인 Application에서 정의한 Supplier를 LottoFactory에 전달하여 의존 관계를 주입하면 유연성이 높고 테스트에 용이한 구조가 된다. 

```java
public class Application {
    public void play() {
        List<Lotto> lottos = LottoFactory.create(() -> LottoGame.of("yeonju"), 6);
    }
}

```

```java
public class LottoFactory {
    
    public static List<Lotto> create(Supplier<? extends Lotto> lottoFactory, int LottoSize) {
        List<Lotto> lottos = new ArrayList<>();
        for (int i = 0; i < LottoSize; i++) {
            lottos.add(lottoFactory.get());
        }
        return lottos;
    }
}

```

```java
public class LottoGame implements Lotto{

    private String user;

    public LottoGame(String user) {
        this.user = user;
    }

    public static Lotto of(String user) {
        return new LottoGame(user);
    }

    @Override
    public void setNumber() {

    }
}
```

```java
public interface Lotto {
    void setNumber();
}
```

### 💡 결론 
- 의존성 또한 커지면 커질수록 비용 소모가 크다. 이러한 경우 의존성 주입 프레임워크 (대표적 스프링) 등을 써서 해결한다. 스프링에서 사용하는 빈들 또한 스프링 컨테이너에서 생성한 빈들을 주입하는 형태이다.
- 클래스에 의존성이 존재하고 자원이 클래스에 영향을 준다면 정적 유틸리티 클래스나 싱글턴은 적절하지 않다
- 의존 관계에 있는 클래스나 해당 클래스를 만들어주는 팩토리를 생성자나 정적팩토리 빌더를 통해 주입 시키면 유연함과 테스트 용이함을 갖춘 구조가 된다.
