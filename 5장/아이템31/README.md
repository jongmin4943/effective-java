## 4장 클래스와 인터페이스

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