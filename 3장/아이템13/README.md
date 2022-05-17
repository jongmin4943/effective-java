## 3장 모든객체의 공통 메서드

------------------

#### 아이템 13 . clone 재정의는 주의해서 진행하라

Cloneable 은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스(mixin interface)지만, <br/>
아쉽게도 의도한 목적을 제대로 이루지 못했다. 가장 큰 문제는 clone 메서드가 선언된곳이 Cloneable이 <br/>
아닌 Object 이고, 그마저도 protected 라는데 있다. 그래서 Cloneable 을 구현하는것만으로는 외부 객체에서<br/>
clone 메서드를 호출할 수 없다. 리플렉션을 사용하면 가능하지만, 100% 성공하는 것도 아니다. 해당 객체가 접근이<br/>
허용된 clone 메서드를 제공한다는 보장이 없기 때문이다. 하지만 이를 포함한 여러 문제점에도 불구하고 <br/>
Cloneable 방식은 널리 쓰이고 있어서 잘 알아두는 것이 좋다. 이번 아이템에서는 clone 메서드를 잘 동작하게끔<br/>
해주는 구현 방법과 언제 그렇게 해야하는지를 알려주고, 가능한 다른 선택지에 관해 논의한다.<br/>

메서드 하나 없는 Cloneable 인터페이스는 놀랍게도 Object 의 protected 메서드인 clone 의</br>
동작 방식을 결정한다. Cloneable 을 구현한 클래스의 인스턴스에서 clone 을 호출하면 그 객체의 </br>
필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportException</br>
을 던진다. 이는 인터페이스를 상당히 이례적으로 사용한 예이니 따라하지는 말자. 인터페이스를 구현한다는</br>
것은 일반적으로 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위다. </br>
그런데 Cloneable 의 경우에는 상위 클래스에 정의된 protected 메서드의 동작 방식을 변경한 것이다.</br>

명세에서는 아이기하지 않지만 실무에서 Cloneable 을 구현한 클래스는 clone 메서드를 public 으로 </br>
제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다. 이 기대를 만족시키려면 그 클래스와 모든 </br>
상위 클래스는 복잡하고, 강제할 수 없고, 허술하게 기술된 프로토콜을 지켜야만 하는데, 그 결과로 깨지기 쉽고,</br>
위험하고, 모순적인 메커니즘이 탄생한다. 생성자를 호출하지 않고도 객체를 생성할 수 있게 되는것이다.</br>

가변상태를 참조하지 않는 클래스용 clone 메서드
```java
@Override
public PhoneNumber clone() {
    try{
        return (PhoneNumber) super.clone();    
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(); // 일어날 수 없는 일
    }    
}
```

이 메서드가 동작하게 하려면 PhoneNumber 의 클래스 선언에 Cloneable 을 구현한다고 추가해야 한다.</br>
Object 의 clone 메서드는 Object 를 반환하지만 PhoneNumber 의 clone 메서드는 PhoneNumber 을 반환하게 했다.</br>
자바가 공변 반환 타이핑(covariant return typing)을 지원하니 이렇게 하는것이 가능하고 권장하는 방식이기도 하다.</br>
재정의한 메서드의 반환타입은 상위클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다. 이 방식으로</br>
클라이언트가 형변환하지 않아도 되게끔 해주자. 이를 위해 앞코드에서는 super.clone 에서 얻은 객체를 반환</br>
하기전에 PhoneNumber 로 형변환 하였다.(절대 실패하지 않는다)

super.clone 호출을 try-catch 블록으로 감싼 이유는 Object 의 clone 메서드가 checked exception 인</br>
CloneNotSupportedException 을 던지도록 선언되었기 때문이다. 이 거추장스러운 코드는 사실</br>
unchecked exception 였어야 한다는 신호다.

앞서의 구현은 클래스가 가변객체를 참조하는 순간 재앙으로 변한다.
```java
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack implements Cloneable{
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 떄마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size +1);
    }
    
    @Override
    public Stack clone() {
        try{
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
elements.clone 의 결과를 형변환할 필요는 없다. 배열의 clone 은 런타임 타입과 컴파일타임 타입 모두가 </br>
원본 배열과 똑같은 배열을 반환한다. 따라서 배열을 복제할 때는 배열의 clone 메서드를 사용하라고 권장한다.</br>
사실 배열은 clone 기능을 제대로 사용하는 유일한 예라 할 수 있다.

그러나 elements 필드가 final 이었다면 이와같은 방식은 작동하지 않는다. final 필드에는 새로운 값을 할당할</br>
수 없기 때문이다. 이는 근본적인 문제로, 직렬화와 마찬가지로 Cloneable 아키텍처는 '가변 객체를 참조하는 필드는</br>
final 로 선언하라' 라는 일반 용법과 충돌한다. 단, 원본과 복제된 객체가 그 가변객체를 공유해도 안전하다면</br>
괜찮다. 그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.</br>

clone 을 재귀적으로 호출하는것만으로는 충분하지 않을 때도 있다. 이번에는 해시테이블용 clone 메서드를 생각해보자.</br>
해시테이블 내부는 버킷들의 배열이고, 각 버킷은 키-값 쌍을 담는 연결 리스트의 첫번째 엔트리를 참조한다. 그리고 성능을 위해</br>
LinkedList 대신 직접 구현한 경량 연결 리스트를 사용하겠다.
```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
    
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        
        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
    
    /** 잘못된 clone 메서드 - 가변상태 공유 */
    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = buckets.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두</br>
예기치 않게 동작할 가능성이 생긴다. 이를 해결하려면 각 버킷을 구성하는 연결리스트를 복사해야한다.
```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
        
        // 이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
        Entry deepCopy() {
            return new Entry(key, value, next==null ? null : next.deepCopy());
        }
    }

    /** 일반적인 해법 */
    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++) {
                if(buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();
            }
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
private 클래스인 HashTable.Entry 는 깊은 복사를 지원하도록 보강되었다. HashTable 의 clone 메서드는</br>
먼저 적절한 크기의 새로운 버킷 배열을 할당한 다음 원래의 버킷 배열을 순회하며 비지 않은 각 버킷에 대해</br>
깊은 복사를 수행한다. 이때 Entry 의 deepCopy 메서드는 자신이 가리키는 연결 리스트 전체를 복사하기 위해</br>
자신을 재귀적으로 호출한다. 이 기법은 간단하며 버킷이 너무 길지 않다면 잘 작동한다. 그러나 연결 리스트를 복제하는</br>
방법으로는 그다지 좋지않다. 재귀 호출때문에 스택오버플로를 일으킬 위험이 있기때문이다. 이 문제를 피하려면</br>
deepCopy 를 재귀 호출대신 반복자를 써서 순회하는 방향으로 수정해야한다.
```java
// 엔트리 자신이 가리키는 연결리스트를 반복적으로 복사한다.
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next) {
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    }
    retutn result
}
```

먼저 super.clone 을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한 다음, 원본 객체의 상태를 다시 생성하는</br>
고수준 메서드를 호출한다. HashTable 예 에서라면, buckets 필드를 새로운 버킷 배열로 초기화한다음 원본 테이블에</br>
담긴 모든 키-값 쌍 각각에 대해 복제본 테이블의 put(key, value) 메서드를 호출해 둘의 내용이 똑같게 해주면 된다.</br>
이처럼 고수준 API 를 활용해 복제하면 보통은 간단하고 제법 우아한 코드를 얻게되지만, 아무래도 저수준에서 바로처리</br>
할때보다는 느리다. 또한 Cloneable 아키텍처의 기초가 되는 필드 단위 객체 복사를 우회하기 때문에 어울리지 않는 방식이기도 하다</br>

생성자에서는 재정의 될 수 있는 메서드를 호출하지 않아야 하는데 clone 메서드도 마찬가지다. 만약 clone 이 하위클래스에서</br>
재정의한 메서드를 호출하면, 하위클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가</br>
달라질 가능성이 크다. 따라서 앞 문단에서 얘기한 put 메서드는 final 이거나 private 이어야 한다.</br>
Object 의 clone 메서드는 CloneNotSupportedException 을 던진다고 선언했지만 재정의한 메서드는 그렇지 않다.</br>
public 인 clone 메서드는 throws 절을 없애야한다. 그 메서드의 편의성을 위해서이다.

상속해서 쓰기 위한 클래스 설계 방식 두가지중 어느쪽에서든 상속용 클래스는 Cloneable 을 구현해서는 안된다.</br>
Object 방식을 모방해 제대로 작동하는 clone 메서드는 구현해 protected 로 둘 수도 있고, clone 을 동작하지않게</br>
구현해놓고 하위클래스에서 재정의하지 못하게 할 수도 있다.
```java
// 하위 클래스에서 Cloneable 을 지원하지 못하게 하는 clone 메서드
@Override
protected final Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException();    
}
```

Cloneable 을 구현한 스레드 안전 클래스를 작성할때는 clone 메서드 역시 적절히 동기화 해줘야 한다.</br>
Object 의 clone 메서드는 동기화를 신경 쓰지 않았다. 그러니 super.clone 호출 외에 다른 할 일이 없더라도</br>
clone 을 재정의하고 동기화해줘야 한다.

요약하자면, Cloneable 을 구현하는 모든 클래스는 clone 을 재정의 해야한다. 이때 접근 제한자는 public 으로</br>
반환 타입은 클래스 자신으로 변경한다. 이 메서드는 가장 먼저 super.clone 을 호출한 후 필요한 필드를 전부 적절히</br>
수정한다. 일반적으로 이 말은 그 객체의 내부 '깊은 구조'에 숨어 있는 모든 가변 객체를 복사하고, 복제본이 가진</br>
객체 참조 모두가 복사된 객체들을 가리키게 함을 뜻한다. 내부 복사는 주로 clone 을 재귀적으로 호출해 구현하지만,</br>
이 방식이 항상 최선인 것은 아니다. 기본 타입 필드와 불변 객체 참조만 갖는 클래스라면 아무필드도 수정할 필요가 없다.</br>
단 일련번호나 고유 ID 는 비록 기본 타입이나 불변일지라도 수정해줘야 한다.

이처럼 복잡한 경우는 드물다. Cloneable 을 이미 구현할 클래스를 확장한다면 어쩔 수 없이 clone 을 잘 작동하도록</br>
구현해야 한다. 그렇지 않은 상황에서는 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다.</br>
복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.</br>
복사 생성자와 그 변형인 복사 팩터리는 Cloneable 방식보다 나은면이 많다. 언어 모순적이고 위험천만한 객체 생성</br>
메커니즘을 사용하지 않으며, 엉성하게 문서화된 규약에 기대지 않고, 정상적인 final 필드 용볍과도 출돌하지 않으며</br>
불필요한 검사 예외를 던지지도 않고, 형변환도 필요치 않다. 복사 생성자와 복사 팩터리는 해당 클래스가 구현한 </br>
'인터페이스' 타입의 인스턴스를 인수로 받을 수 있다. 관례상 모든 범용 컬렉션 구현체는 Collection 이나 Map 타입을</br>
받는 생성자를 제공한다. 인터페이스 기반 복사 생성자와 복사 팩터리의 더 장확한 이름은 <strong>변환 생성자, 변환 팩터리</strong>다</br>
이들을 이용하면 클라이언트는 원본의 구현타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다. 예컨대</br>
HashSet 객체 s 를 TreeSet 타입으로 복제 할 수 있다. clone 으로는 불가능한 이 기능을 변환 생성자로는 간단히</br>
new TreeSet<>(s) 로 처리할 수 있다.

> 핵심정리
> Cloneable 이 몰고 온 모든 문제를 되짚어 봤을때, 새로운 인터페이스를 만들때는 절대 Cloneable 을 확장해서는 안되며
> 새로운 클래스도 이를 구현해서는 안된다. final 클래스라면 Cloneable 을 구현해도 위험이 크지 않지만, 성능 최적화
> 관점에서 검토한 후 별다른 문제가 없을 때만 드물게 허용해야한다. 기본원칙은 '복제 기능은 생성자와 팩터리를 이용하는게 최고'
> 라는 것이다. 단, 배열만은 clone 메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다.