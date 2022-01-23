# Item5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

클래스 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.

- 정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.
```java
public class SpellChecker{
    private static final Lexicon dictionary = ...;
    
    private SpellChecker() {} // 객체 생성 방지
    
    public static boolean isValid(String word){ ... }
    public static List<String> suggestions(String typo){ ... }
}
```
- 싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다. 
```java
public class SpellChecker{
    private static final Lexicon dictionary = ...;
    
    private SpellChecker(...) {}
    public static SpellChecker INSTANCE = new SpellChecker(...);
    
    public static boolean isValid(String word){ ... }
    public static List<String> suggestions(String typo){ ... }
}
```
- 위 두 방식 모두 단 하나의 사전만 사용한다고 가정하는 점에서 훌륭하지 않다.

### 언어별 사전, 특수 어휘용 사전 등이 필요하다면 어떻게 해야할까? 
1. final 한정자를 제거하고 다른 사전으로 교체하는 메서드를 추가해보자
    ```java
    public class SpellChecker{
        private Lexicon dictionary;
        
        public SpellChecker(){}
        
        public setDictionary(Lexicon dictionary) {
            this.dictionary = dictionary;
        }
        ...
    }
    ```
    - 아쉽게도 이와 같은 방식은 어색하고 오류를 내기 쉬우며 멀티스레드 환경에서는 쓸 수 없다.
2. **인스턴스를 생성할 때 `생성자`에 필요한 자원을 넘겨주자**
    ```java
    public class SpellChecker{
        private final Lexicon dictionary;
    
        public SpellChecker(Lexicon dictionary){
            this.dictionay = Objects.requiredNonNull(dictionay);
        }
        ...
    }
    ``` 
    - 클래스의 유연성, 재사용성, 테스트 용이성을 개선 해준다.
    - 불변을 보장해 해당 자원을 사용하려는 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있다.
    - 의존 객체 주입은 생성자, 정적 팩터리, 빌더 아이템에 똑같이 응용할 수 있다.

### 정리 
- 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.
- 이 자원들을 클래스가 직접 만들게 해서도 안된다. 대신, 필요한 자원을 (혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주자. 
- 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해준다.
