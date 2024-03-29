## 4장 클래스와 인터페이스

------------------

#### 아이템 20 . 추상클래스보다는 인터페이스를 우선하라

자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상 클래스, 이렇게 두가지다. 자바 8부터 인터페이스도<br/>
디폴트 메서드를 제공할 수 있게 되어, 이제는 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다.<br/>
한편, 둘의 가장 큰 차이는 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가<br/>
되어야 한다는 점이다. 자바는 단일 상속만 지원하니, 추상 클래스 방식은 새로운 타입을 정의하는 데 커다란<br/>
제약을 안게 되는 셈이다. 반면 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면<br/>
다른 어떤 클래스를 상속했든 같은 타입으로 취급된다.

기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다. 인터페이스가 요구하는 메서드를 추가하고,<br/>
클래스 선언에 implements 구문만 추가하면 끝이다. 자바 플랫폼에서도 Comparable, Iterable, AutoCloseable<br/>
인터페이스가 추가됐을 때 표준 라이브러리의 수많은 기존 클래스가 이 인터페이스들을 구현한 채 릴리스 됐다.<br/>
반면 기존 클래스위에 새로운 추상 클래스를 끼워넣기는 어려운게 일반적이다. 두 클래스가 같은 추상클래스를<br/>
확장하기 원한다면, 그 추상클래스는 계층구조상 두 클래스의 공통 조상이어야 한다. 안타깝게도 이 방식은<br/>
클래스 계층구조에 커다란 혼란을 일으킨다. 새로 추가된 추상 클래스의 모든 자손이 이를 상속하게 되는것이다.<br/>
그렇게 하는 것이 적절하지 않은 상황에서도 강제로 말이다.

인터페이스는 믹스인 정의에 안성맞춤이다. 믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한<br/>
클래스에 원래의 주된 타입 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다. 예컨대 Comparable 은<br/>
자신을 구현한 클래스의 인스턴스들끼리는 순서를 정할 수 있다고 선언하는 믹스인 인터페이스다. 이처럼 대상<br/>
타입의 주된 기능에 선택적 기능을 혼합한다고 해서 믹스인이라 부른다. 추상 클래스로는 믹스인을 정의할 수 없다.<br/>
기존 클래스에 덧씌울 수 없기 때문이다. 클래스는 두 부모를 섬길 수 없고, 클래스 계층구조에는 믹스인을<br/>
삽입하기에 합리적인 위치가 없기 때문이다.

인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다. 타입을 계층적으로 정의하면 수많은 개념을<br/>
구조적으로 잘 표현할 수 있지만, 현실에는 계층을 엄격히 구분하기 어려운 개념도 있다. 예를들어 가수 인터페이스<br/>
와 작곡가 인터페이스가 있다고 해보자.

```java
public interface Singer {
    AudioClip sing(Song s);
}

public interface Songwriter {
    Song compose(int chartPosition);
}
```
이 코드처럼 타입을 인터페이스로 정의하면 가수 클래스가 Singer 와 Songwriter 모두를 구현해도 전혀 문제되지<br/>
않는다. 심지어 Singer 와 Songwriter 모두를 확장하고 새로운 메서드까지 추가한 제 3의 인터페이스를 정의 할<br/>
수도 있다.
```java
public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip strum();
    void actSensitive();
} 
```
이 정도의 유연성이 항상 필요하지는 않지만, 이렇게 만들어준 인터페이스가 결정적인 도움을 줄 수도 있다.<br/>
같은 구조를 클래스로 만드려면 가능한 조합 전부를 각각의 클래스로 정의한 고도비만 계층구조가 만들어질 것이다.<br/>
속성이 n 개 라면 지원해야 할 조합의 수는 2^n개나 된다. 흔히 조합 포발이라 부르는 현상이다. 거대한 <br/>
클래스계층구조에는 공통 기능을 정의해놓은 타입이 없으니, 자칫 매개변수 타입만 다른 메서드들을 수없이<br/>
많이 가진 거대한 클래스를 낳을 수 있다.

래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다. 타입을 추상<br/>
클래스로 정의해두면 그 타입에 기능을 추가하는 방법은 상속뿐이다. 상속해서 만든 클래스는 래퍼 클래스보다<br/>
활용도가 떨어지고 깨지기는 더 쉽다.

인터페이스의 메서드중 구현 방법이 명백한 것이 있다면, 그 구현을 디폴트 메서드로 제공해 프로그래머들의 일감을<br/>
덜어줄 수 있다. 이 기법의 예는 removeIf 메서드를 보면 된다. 디폴트 메서드를 제공할 때는 상속하려는 사람을<br/>
위한 설명을 @implSpec 자바독 태그를 붙여 문서화 해야 한다.

디폴트 메서드에도 제약은 있다. 많은 인터페이스가 equals 와 hashCode 같은 Object 의 메서드를 정의하고<br/>
있지만, 이들은 디폴트 메서드로 제공해서는 안 된다. 또한 인터페이스는 인스턴스 필드를 가질 수 없고 public<br/>
이 아닌 정적 멤버도 가질 수 없다.(단, private 정적 메서드는 예외다.) 마지막으로, 여러분이 만들지 않은<br/>
인터페이스에는 디폴트 메서드를 추가할 수 없다.

한편, 인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공하는 식으로 인터페이스와<br/>
추상 클래스의 장점을 모두 취하는 방법도 있다. 인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드 몇개도<br/>
함께 제공한다. 그리고 골격 구현 클래스는 나머지 메서드들까지 구현한다. 이렇게 해두면 단순히 골격 구현을<br/>
확장하는 것만으로 이 인터페이스를 구현하는데 필요한 일이 대부분 완료된다. 바로 템플릿 메서드 패턴이다.

관례상 인터페이스 이름이 Interface 라면 그 골격 구현 클래스의 이름은 AbstractInterface 로 짓는다.<br/>
좋은 예로, 컬력센 프레임워크의 AbstractCollection, AbstractSet, AbstractList, AbstractMap 각각이<br/>
바로 핵심 컬렉션 인터페이스의 골격 구현이다. 제대로 설계했다면 골격 구현은 그 인터페이스로 나름의 구현을<br/>
만드려는 프로그래머의 일을 상당히 덜어준다. 다음 코드는 완벽히 동작하는 List 구현체를 반환하는 정적<br/>
팩터리 메서드로, AbstractList 골격 구현으로 활용했다.
```java
static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a);
    
    // 다이아몬드 연산자를 이렇게 사용하는건 자바 9부터 가능하다.
    // 더 낮은 버전을 사용한다면 <Integer> 로 수정하자
    return new AbstractList<>() {
        @Override
        public Integer get(int i) {
            retrun a[i];
        }
        
        @Override
        public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val;
            return oldVal;
        }
        
        @Override
        public int size() {
            return a.length;
        }
    };
}
``` 
List 구현체가 여러분에게 제공하는 기능들을 생각하면, 이 코드는 골격 구현의 힘을 잘 보여주는 인상적인<br/>
예라 할 수 있다. 이 예는 int 배열을 받아 Integer 인스턴스의 리스트 형태로 보여주는 어댑터이기도 하다.<br/>
int 값과 Integer 인스턴스 사이의 변환 때문에 성능이 그리 좋지 않다. 또한, 이 구현에서 익명 클래스형태를<br/>
사용했음에 주목하자.

골격 구현 클래스의 아름다움은 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때<br/>
따라오는 심각한 제약에서는 자유롭다는 점에 있다. 골격 구현을 확장하는 것으로 인터페이스 구현이 거의<br/>
끝나지만, 꼭 이렇게 해야하는 것은 아니다. 구조상 골격 구현을 확장하지 못하는 처지라면 인터페이스를 직접<br/>
구현해야 한다. 이런 경우라도 인터페이스가 직접 제공하는 디폴트 메서드의 이점을 여전히 누릴 수 있다.<br/>
또한, 골격 구현 클래스를 우회적으로 이용할 수도 있다. 인터페이스를 구현한 클래스에서 해당 골격 구현을<br/>
확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 것이다.<br/>
아이템 18에서 다룬 래퍼클래스와 비슷한 이 방식을 시뮬레이트한 다중 상속(simulated multiple inheritance)<br/>
라 하며, 다중 상속의 많은 장점을 제공하는 동시에 단점은 피하게 해준다.

골격 구현 작성은 조금 지루하지만 상대적으로 쉽다. 가장 먼저, 인터페이스를 잘 살펴 다른 메서드들의 구현에<br/>
사용되는 기반 메서드들을 선정한다. 이 기반 메서드들은 골격 구현에서는 추상 메서드가 될 것이다. 그다음으로,<br/>
기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공한다. 단, equals 와 hashCode<br/>
같은 Object 의 메서드는 디폴트 메서드로 제공하면 안된다는 사실을 항상 유념하자. 만약 인터페이스의 메서드<br/>
모두가 기반메서드와 디폴트 메서드가 된다면 골격 구현클래스를 별도로 만들 이유는 없다. 기반 메서드나<br/>
디폴트 메서드로 만들지 못한 메서드가 남아 있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어<br/>
남은 메서드들을 작성해 넣는다. 골격 구현 클래스에는 필요하면 public 이 아닌 필드와 메서드를 추가해도 된다.

간단한 예로 Map.Entry 인터페이스를 살펴보자. getKey, getValue 는 확실히 기반 메서드이며, 선택적으로<br/>
setValue 도 포함할 수 있다. 이 인터페이스는 equals 와 hashCode 의 동작 방식도 정의해놨다. Object<br/>
메서드들은 디폴트 메서드로 제공해서는 안되므로, 해당 메서드들은 모두 골격 구현 클래스에 구현한다.

```java
import java.util.Map;
import java.util.Objects;

public abstract class AbstractMapEntry<K, V> implements Map.Entry<K, V> {

    // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
    @Override
    public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    // Map.Entry.equals 의 일반 규약을 구현한다.
    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof Map.Entry)) return false;
        Map.Entry<?, ?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(), getKey())
                && Objects.equals(e.getValue(), getValue());
    }
    
    // Map.Entry.hashCode 의 일반 규약을 구현한다.
    @Override
    public int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }
    
    @Override
    public String toString() {
        return getKey() + "=" + getValue();
    }
}
```
- Map.Entry 인터페이스나 그 하위 인터페이스로는 이 골격 구현을 제공할 수 없다. 디폴트 메서드는
equals, hashCode, toString 같은 Object 메서드를 재정의 할 수 없기 때문이다.

골격 구현은 기본적으로 상속해서 사용하는 걸 가정하므로 아이템 19에서 이야기한 설계 및 문서화 지침을<br/>
모두 따라야 한다. 간략히 보여주기 위해 앞의 코드에서는 문서화 주석을 생략했지만, 인터페이스에 정의한<br/>
디폴트 메서드든 별도의 추상클래스든, 골격 구현은 반드시 그 동작을 잘 정리해 문서로 남겨야 한다.

단순 구현은 골격 구현의 작은 변종으로, AbstractMap.SimpleEntry 가 좋은 예다. 단순 구현도 골격<br/>
구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상 클래스가 아니란 점이 다르다. 쉽게 말해<br/>
동작하는 가장 단순한 구현이다. 이러한 단순 구현은 그대로 써도 되고 필요에 맞게 확장해도 된다.

> 핵심 정리 <br/>
> 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다. 복잡한 인터페이스라면 구현하는 수고를 덜어주는
> 골격 구현을 함께 제공하는 방법을 꼭 고려해보자. 골격 구현은 가능한 한 인터페이스의 디폴트메서드로 제공하여
> 그 인터페이스를 구현한 모든곳에서 활용하도록 하는 것이 좋다. 가능한 한 이라 한 이유는, 인터페이스에 걸려있는
> 구현상 제약 때문에 골격 구현을 추상클래스로 제공하는 경우가 더 흔하기 때문이다.