## 2장 객체 생성과 파괴


------------------

#### 아이템 5 . 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

정적 유틸리티를 잘못 사용한 예 - <strong>유연하지 않고 테스트하기 어렵다</strong>
```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    
    private SpellChecker(){} // 객체 생성 방지
    
    public static boolean isValid(String word) {}
    public static List<Long> suggestions(String typo) {}
}
```

싱글톤을 잘못 사용한 예 - <strong>유연하지 않고 테스트하기 어렵다</strong>
```java
public class SpellChecker {
    private final Lexicon dictionary = ...;
    
    private SpellChecker(...){} // 객체 생성 방지
    public static SpellChecker INSTANCE = new SpellChecker(...);
    
    public boolean isValid(String word) {}
    public List<Long> suggestions(String typo) {}
}
```

두 방식 모두 사전을 단 하나만 사용한다는 점에서 그리 훌륭하지 않다. 실전에선 사전을 언어별로 나누기도, 특수 어위용으로 나누기도 한다.<br/>
심지어 테스트용 사전도 필요할 수 있다.<br/>
여러 사전을 사용할 수 있또록 dictionary 필드에 final 한정자를 제거하고 다른 사전으로 교체하는 메서드(setter)를 추가할수 있지만 <br/>
어색하고 오류를 내기 쉬우며 결정적으로 멀티스레드 환경에서는 쓸 수 없다.<br/>
사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글톤방식이 적합하지 않다.<br/>

클래스(SpellChecker)가 여러 자원 인스턴스를 지원해야하며 클라이언트가 원하는 자원(dictionary)을 사용해야한다. <br/>
이 조건을 만족하는 패턴이 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식이다.

의존객체 주입은 유연성과 테스트 용이성을 높여준다.
```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    } // 객체 생성 방지

    public boolean isValid(String word) {}
    public List<Long> suggestions(String typo) {}
}
```
딱 하나의 자원만 사용하지만, 자원이 몇개든 의존 관계가 어떻든 상관없이 잘 작동한다. 또한 <br/> 
불변을 보장하여 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 하다.
의존객체 주입은 생성자, 적정팩터리, 빌더 모두에 똑같이 응용할 수 있다.

이 패턴의 쓸만한 변형으로, 생성자에 자원 팩터리를 넘겨주는 방식이 있다. <br/>
팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다. 즉, 팩터피 메서드 패턴(Factory Method pattern)이다<br/>
자바 8에서 소개한 Supplier<T> 인터페이스가 팩터리를 표현한 완벽한 예다. <br/>
입력으로 받는 메서드는 일반적으로 한정적 와일드카드 타입(bounded wildcard type) 을 사용해 팩터리의 타입 매개변수를 제한해야한다. <br/>
이 방식을 사용해 클라이언트는 자신이 명시한 타입의 하위타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다. <br/>
Mosaic create(Supplier<? extends Tile> tileFactory) {...}<br/>
<br/>
의존객체 주입이 유연성과 테스트 용이성을 개선해주긴 하지만, 의존성이 수천 개나 되는 큰 프로젝트에서는 코드를 어지럽게 하기도 한다. <br/>
Dagger, Guice, Spring 같은 프레임 워크를 사용하면 이러한점을 해소 할 수 있다.

>핵심정리<br/>
> 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는것이 좋다.<br/>
> 이 자원들을 클래스가 직접 만들게 해서도 안된다. 대신 필요한 자원을 (혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) <br/>
> 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해준다.

