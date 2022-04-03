## 2장 객체 생성과 파괴


------------------

#### 아이템 7 . 다 쓴 객체 참조를 해제하라

자바는 C 나 C++ 처럼 메모리를 직접 관리하지 않아도 가비지 컬렉터 가 관리해준다. 그래서 자칫 메모리 관리에 더 이상<br/>
신경 쓰지 않아도 된다고 오해할 수 있는데, 절대 사실이 아니다.<br/>

주의해야할 것은 <strong>메모리누수</strong>로, 드문 경우긴 하지만 심할때는 디스크 페이징이나 OutOfMemoryError<br/>
를 일으켜 프로그램이 예기치 않게 종료되기도 한다.

```java
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack {
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
}
```

이 코드에서 메무리 누수는 스택이 커졌다가 줄어들 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다.<br/>
이 스택이 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문이다. 다 쓴 참조란 앞으로 다시<br/>
쓰지 않을 참조를 뜻한다. elements 배열의 활성 역역 밖의 참조들이 모두 여기에 해당한다. 활성 영역은 인덱스가 <br/>
size 보다 작은 원소들로 구성된다.<br/>

가비지 컬렉션 언어에서는 메모리 누수를 찾기가 아주 까다롭다. 객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체<br/>
뿐만 아니라 그 객체가 참조하는 모든 객체를 회수해가지 못한다. 그래서 단 몇개의 객체가 매우 많은 객체를 회수되지<br/>
못하게 할 수 있고 잠재적으로 성능에 악영향을 줄 수 있다.<br/>

해법은 참조를 다 썼을 때 null 처리를 하면 된다.
```java
    public Object pop() {
        if (size == 0)
        throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
```

null 처리한 참조를 사용하려 하면 NullPointerException 을 던진지는 이점이 생긴다.<br/>
객체 참조를 null 처리하는 일은 예외적인 경우여야한다. 다 쓴 참조를 해제하는 가장 좋은 방법은 <br/>
그 참조를 담은 변수를 scope 밖으로 밀어내는것이다. 변수의 범위를 최소가 되게 정의하면 좋다. <br/>
일반적으로 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야한다. <br/>

캐시 역시 메모리 누수를 일으키는 주범이다. 캐시 외부에서 키를 참조하는 동안만 엔트리가 살아있는 캐시가 <br/>
필요한 상황이라면 WeakHashMap 을 사용해 캐시를 만들자. 다 쓴 엔트리는 그 즉시 자동으로 제거가 될것이다.<br/>
(key 값인 객체가 gc 되면 map 객체 안의 값도 사라진다.)

메모리 누수의 세번째 주범은 바로 리스너(listener) 혹은 콜백(callback) 이다. <br/>
클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면, 뭔가 조치해주지 않는 한 콜백은 계속 쌓여갈 것이다.<br/>
이럴 때 콜백을 약한 참조 (weak reference)로 저장하면 가비지 컬렉터가 즉시 수거해간다.

> 핵심정리
> 메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 <br/>
> 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 <br/>
> 예방법을 익혀두는것이 매우 중요하다.