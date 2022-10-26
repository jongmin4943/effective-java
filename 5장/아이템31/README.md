## 5장 제네릭

------------------

#### 아이템 31 . 한정적 와일드카드를 사용해 API 유연성을 높여라

매개변수화 타입은 불공변이다. 즉, 서로 다른 타입1과 타입2가 있을때 List<타입1> 은 List<타입2>의 하위타입<br/>
도 상위타입도 아니다. 직관적이지 않겠지만 List<String> 은 List<Object> 의 하위 타입이 아니라는 뜻인데,<br/>
곰곰이 따져보면 사실 이쪽이 말이된다. List<Object> 에는 어떤 객체든 넣을 수 있지만 List<String>에는<br/>
문자열만 넣을 수 있다. 즉, List<String> 은 List<Object>가 하는 일을 제대로 수행하지 못하니 하위타입이<br/>
될 수 없다. 리스코프 치환원칙에 어긋난다.

하지만 때론 불공변 방식보다 유연한 무언가가 필요하다. Stack 클래스를 떠올려보자.
```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```
여기에 일련의 원소를 스택에 넣는 메서드를 추가해야한다고 해보자
```java
public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);    
    }
}
```
이 메서드는 깨끗이 컴파일되지만 완벽하진 않다. Iterable src 의 원소 타입이 스택의 원소 타입과 일치하면<br/>
잘 작동한다. 하지만 Stack<Number> 로 선언한 후 Iterable<Integer>인 ints 로 pushAll(ints) 을 <br/>
호출하면 논리적으론 잘 작동 해야할것 같다. 하지만 실제로는 매개변수화 타입이 불공변이기에 incompatible <br/>
type 오류 메시지가 뜬다.

다행히 해결책은 있다. 자바는 이런 상황을 대처할 수 있는 한정적 와일드카드타입이라는 특별한 매개변수화 타입<br/>
을 지원한다. pushAll 의 입력 매개변수 타입은 'E 의 Iterable' 이 아니라 'E 의 하위 타입의 Iterable'<br/>
이어야 하며, 와일드 카드 타입 Iterable<? extends E> 가 정확히 이런뜻이다.

pushAll 과 짝을 이루는 popAll 메서드는 Stack 안의 모든 원소를 주어진 컬렉션으로 옮겨 담는다.
```java
public void popAll(Collection<E> dst) {
    while(!isEmpty()) {
        dst.add(pop());    
    }    
}
```
이번에도 주어진 컬렉션의 원소 타입이 스택의 원소 타입과 일치한다면 문제 없지만 addAll 과 같은문제가 있다.<br/>
이번에도 와일드카드 타입으로 해결 할 수 있다. 이번에는 popAll 의 입력 매개변수의 타입이 'E 의 Collection'<br/>
이 아니라 'E 의 상위 타입의 Collection' 이어야 한다. 모든 타입은 자기 자신의 상위 타입이다.
```java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
public void popAll(Collection<? super E> dst) {
    while(!isEmpty()) {
        dst.add(pop());
    }
}
```
메시지는 분명하다. 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라<br/>
한편, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없다. 타입을 정확히<br/>
지정해야 하는 상황으로, 아때는 와일드카드 타입을 쓰지 말아야 한다.
> 다음 공식을 외워두면 어떤 와일드카드 타입을 써야하는지 기억하는데 도움이 된다.<br/>
> PECS : producer-extends, consumer-super<br/>

즉, 매개변수화 타입 T 가 생산자라면 <? extends T>를 사용하고, 소비자라면 <? super T>를 사용하라.<br/>
Stack 예에서 pushAll 의 src 매개변수는 Stack 이 사용할 E 인스턴스를 생산하므로 적절한 타입은<br/>
Iterable<? extends E> 이다. 한편, popAll 의 dst 매개변수는 Stack 으로부터 E 인스턴스를 소비하므로<br/>
dst 의 적절한 타입은 Collection<? super E> 이다. PECS 공식은 와일드카드 타입을 사용하는 기본원칙이다.<br/>
나프탈린과 와들러는 이를 GetPut 원칙으로 부른다.

이 공식을 기억해두고 이번 장의 앞 아이템에서 소개한 메서드와 생선자선언을 다시 살펴보자.
```java
public Chooser(Collection<T> choices)
```
이 생성자로 넘겨지는 choices 컬렉션은 T 타입의 값을 생산하기만 하니(그리고 나중을 위해 저장해둔다),<br/>
T 를 확장하는 와일드 타입을 사용해 선언해야 한다. 그렇게 수정하면 실질적인 차이가 생긴다. Chooser<Number><br/>
의 생성자에 List<Integer> 를 넘기고 싶다고 해보자. 수정 전 생성자로는 컴파일조차 되지 않겠지만,<br/>
한정적 와일드카드 타입으로 선언한 수정 후 생성자에서는 문제가 사라진다.

이번엔 union 메서드 차례다.
```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
```
s1 과 s2 모두 E 의 생산자이니 PECS 공식에 따라 다음처럼 바꿔야한다.
```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```
반환타입은 여전히 Set<E> 임에 주목해라. 반환 타입에는 한정적 와일드카드 타입을 사용하면 안된다.<br/>
유연성을 높여주기는 커녕 클라이언트 코드에서도 와일드카드 타입을 써야하기 때문이다.<br/>
제대로만 사용한다면 클래스 사용자는 와일드카드 타입이 쓰였다는 사실조차 의식하지 못할 것이다.<br/>
받아들여야 할 매개변수를 받고 거절해야 할 매개변수는 거절하는 작업이 알아서 이뤄진다. 클래스 사용자가<br/>
와일드카드 타입을 신경 써야 한다면 그 API 에 무슨 문제가 있을 가능성이 크다.<br/>
자바 7까지는 타입 추론 능력이 충분히 강력하지 못해서 문맥에 맞는 반환타입을 명시해야 했다. 컴파일러가<br/>
올바른 타입을 추론하지 못할 때면 언제든 명시적 타입 인수를 사용해서 타입을 알려주면 된다. 목표 타이핑<br/>
은 자바 8부터 지원하기 시작했는데, 그 전 버전에서도 이런 문제가 흔하진 않았다. 명시적 타입 인수는<br/>
코드를 지저분하게 하니, 이런 문제가 빈번하지 않은건 그나마 다행이었다.
```java
Set<Number> numbers = Union.<Number>union(integers, doubles);
```
인수(argument) 와 매개변수(parameter) 의 차이는 매개변수는 메서드 선언에 정의한 변수이고,<br/>
인수는 메서드 호출시 넘기는 실제값 이다.
```java
void add(int value) {} //매개변수
add(10) // 인수
```
이번에는 max 메서드를 보자. 원래 버전의 선언은 다음과 같다.
```java
public static <E extends Comparable<E>> E max(List<E> list); // before
public static <E extends Comparable<? super E>> E max(List<? extends E> list); // after
```
이번에는 PECS 공식을 두번 적용했따. 입력 매개변수에서는 E 인스턴스를 생산하므로 원래의 List<E>를<br/>
List<? extends E> 로 수정했다.<br/>
타입 매개변수 E 는 원래 선언에서는 E 가 Comparable<E> 를 확장한다고 정의했는데, 이때 Comparable<E><br/>
는 E 인스턴스를 소비한다.(그리고 선후 관계를 뜻하는 정수를 생산한다.) 그래서 매개변수화 타입 Comparable<E><br/>
를 한정적 와일드카드 타입인 Comparable<? super E> 로 대체했다. Comparable 은 언제나 소비자이므로,<br/>
일반적으로 Comparable<E> 보다는 Comparable<? super E>를 사용하는편이 낫다. Comparator 도 마찬가지다.<br/>
일반적으로 Comparator<E> 보다는 Comparator<? super E>를 사용하는편이 낫다.<br/>
수정된 버전의 max 는 이 책에서 가장 복잡한 메서드 선언일 것이다. 이렇게 까지 복잡하게 만들만한 가치가<br/>
있다. 그 근거로, 다음 리스트는 오직 수정된 max 로만 처리할 수 있다.
```java
List<ScheduleFuture<?>> scheduleFutures = ...;
```
수정 전 max 가 이 리스트를 처리할 수 없는 이유는 ScheduleFuture 가 Comparable<ScheduleFuture>를<br/>
구현하지 않았기 때문이다. ScheduleFuture 는 Delayed 의 하위 인터페이스이고, Delayed 는 Comparable<Delayed><br/>
를 확장했다. 다시 말해, ScheduleFuture 의 인스턴스는 다른 ScheduleFuture 인스턴스 뿐만아니라 Delayed<br/>
인스턴스와도 비교할 수 있어서 수정 전 max 가 이 리스트를 거부하는 것이다. 더 일반화 해서 말하면,<br/>
Comparable(혹은 Comparator)을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기<br/>
위해 와일드카드가 필요하다.
```java
public interface Comparable<E>{}
public interface Delayed extends Comparable<Delayed>{}
public interface ScheduledFuture<V> extends Delayed, Future<V>{}
```
와일드카드와 관련해 논의해야 할 주제가 하나 더 남았다. 타입 매개변수와 와일드카드에는 공통되는 부분이<br/>
있어서, 메서드를 정의할 때 둘 중 어느것을 사용해도 괜찮을 때가 많다. 예를 들어 주어진 리스트에서 명시한<br/>
두 인덱스의 아이템을 교환(swap) 하는 정적 메서드를 두 방식 모두로 정의해보자. 다음 코드에서 첫번째는<br/>
비한정적 타입 매개변수를 사용했고 두번째는 비한정적 와일드카드를 사용했다.
```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```
어떤 선언이 나을까? 더 나은 이유는 무엇일까? public API 라면 간단히 두번째가 더 낫다. 어떤 리스트든<br/>
이 메서드에 넘기면 명시한 인덱스의 원소들을 교환해줄 것이다. 신경 써야 할 타입 매개변수도 없다.<br/>

기본 규칙은 이렇다. 메서드 선언에 타입 매개변수가 한번만 나오면 와일드카드로 대체하라 이때 비한정적<br/>
타입 매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다.<br/>
하지만 두번째 swap 선언에는 문제가 하나 있는데, 다음과 같이 아주 직관적으로 구현한 코드가 컴파일<br/>
되지 않는다는 것이다.
```java
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));    
}
```
이 코드를 컴파일하면 Object cannot be converted to ... 오류 메시지가 나온다.<br/>
방금 꺼낸 원소를 리스트에 다시 넣을 수 없는것이다. 원인은 리스트의 타입이 List<?> 인데, List<?>에는
null 외에는 어떤 값도 넣을 수 없다는 데 있다. 다행히 런타임 오류를 낼 가능성이 있는 형변환이나<br/>
리스트의 raw 타입을 사용하지 않고도 해결할 길이 있따. 바로 와일드카드 타입의 실제 타입을<br/>
알려주는 메서드를 private 도우미 메서드로 따로 작성하여 활용하는 방법이다. 실제 타입을 알아내려면<br/>
이 도우미 메서드는 제네릭 메서드여야 한다.
```java
public static void(List<?> list, int i, int j) {
    swapHelper(list, i, j);    
}
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));    
}
```
swapHelper 메서드는 리스트가 List<E> 임을 알고 있다. 즉, 이 리스트에서 꺼낸 값의 타입은<br/>
항상 E 이고, E 타입의 값이라면 이 리스트에 넣어도 안전함을 알고 있다. 다소 복잡하게 구현했지만<br/>
이제 깔끔히 컴파일된다. 이상으로 swap 메서드 내부에서는 더 복잡한 제네릭 메서드를 이용했지만, 덕분에<br/>
외부에서는 와일드카드 기반의 멋진 선언을 유지할 수 있었다. 즉, swap 메서드를 호출하는 클라이언트는<br/>
복잡한 swapHelper 의 존재를 모른 채 그 혜택을 누리는 것이다.<br/>
도우미 메서드느의 시그니처는 앞에서 public API 로 쓰기에는 너무 복잡하다 는 이유로 버렸던 첫번째<br/>
swap 메서드의 시그니처와 완전히 똑같다.
> 핵심정리<br/>
> 조금 복잡하더라도 와일드카드 타입을 적용하면 API 가 훨씬 유연해진다. 그러니 널리 쓰일 라이브러리를
> 작성한다면 반드시 와일드카드 타입을 적절히 사용해줘야 한다. PECS 공식을 기억하자. 생산자(producer)는
> extends, 소비자(consumer)는 super 를 사용한다. Comparable 과 Comparator 는 모두 소비자라는
> 사실도 잊지 말자.