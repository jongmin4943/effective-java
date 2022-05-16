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