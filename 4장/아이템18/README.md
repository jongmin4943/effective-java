## 4장 클래스와 인터페이스

------------------

#### 아이템 18 . 상속보다는 컴포지션을 사용하라

상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다. 잘못 사용하면 오류를 내기 쉬운 소프트웨어를</br>
만들게 된다. 상위 클래스와 하위클래스를 모두 같은 프로그래머가 통제하는 패키지 안에서라면 상속도 안전한 방법</br>
이다. 확장할 목적으로 설계되었고 문서화도 잘 된 클래스도 마찬가지로 안전하다. 하지만 일반적인 구체 클래스를</br>
패키지 경계를 넘어, 즉 다른 패키지의 구체 클래스를 상속하는 일은 위험하다. 상기하자면, 이 책에서의 '상속'</br>
은 구현 상속을 말한다. 이번 아이템에서 논하는 문제는 인터페이스와는 무관하다.

메서드 호출과 달리 상속은 캡슐화를 깨뜨린다. 다르게 말하면 상위 클래스가 어떻게 구현되느냐에 따라 하위</br>
클래스의 동작에 이상이 생길 수 있다. 상위 클래스는 릴리스마다 내부 구현이 달라질 수 있으며, 그 여파로 코드</br>
한 줄 건드리지 않은 하위 클래스가 오동작할 수 있다는 말이다. 이러한 이유로 상위 클래스 설계자가 확장을</br>
충분히 고려하고 문서화도 제대로 해두지 않으면 하위 클래스는 상위 클래스의 변화에 발맞춰 수정되어야한다.</br>

예를들면, 우리에게 HashSet 을 사용하는 프로그램이 있다. 성능을 높이려면 이 HashSet 은 처음 생성된 이후</br>
원소가 몇 개 더해졌는지알 수 있어야 한다.(HashSet 의 현재 크기와는 다른 개념이다. 현재 크기는 원소가 제거</br>
되면 줄어든다.) 그래서 밑의 코드와 같이 변형된 HashSet 을 만들어 추가된 원소의 수를 저장하는 변수와</br>
접근자 메서드를 추가했다. 그런 다음 HashSet 에 원소를 추가하는 메서드인 add 와 addAll 를 재정의했다.
```java
public class InstrumentHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    private int addCount = 0;
    
    public InstrumentHashSet() {}
    public InstrumentHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }
    
    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    
    public int getAddCount() {
        return addCount;
    }
}
```
이 클래스는 잘 구현된 것처럼 보이지만 제대로 작동하지 않는다. 이 클래스의 인스턴스에 addAll 메서드로 원소</br>
3개를 더했다고 해보자. 다음 코드는 자바 9부터 지원하는 정적 팩터리 메서드인 List.of 로 리스트를 생성했다.</br>
그 전 버전은 Arrays.asList 를 사용하면 된다.
```java
InstrumentHashSet<String> s = new InstrumentHashSet<>();
s.addAll(List.of("틱","탁탁","펑"));
```
이제 getAddCount 메서드를 호출하면 3을 반환하리라 기대하겠지만, 실제로는 6을 반환한다. 어디서 잘못된것일까?</br>
그 원인은 HashSet 의 addAll 메서드가 add 메서드를 사용해 구현된 데 있다. 이런 내부 구현 방식은 HashSet</br>
문서에서 쓰여지 있지 않다. InstrumentHashSet 의 addAll 은 addCount 에 3을 더한 후 HashSet 의 </br>
addAll 구현을 호출했다. HashSet 의 addAll 은 각 원소를 add 메서드를 호출해 추가하는데, 이때 불리는 add</br>
는 InstrumentHashSet 에서 재정의한 메서드다. 따라서 addCount 에 값이 중복해서 더해져 6이 된다.</br>

이 경우 하위 클래스에서 addAll 메서드를 재정의하지 않으면 문제를 고칠 수 있다. 하지만 당장은 제대로 동작할지</br>
모르나, HashSet 의 addAll 이 add 메서드를 이용해 구현했음을 가정한 해법이라는 한계를 지닌다. 이처럼 자신의</br>
다른 부분을 사용하는 '자가사용(self-use)' 여부는 해당 클래스의 내부 구현 방식에 해당하며, 자바 플랫폼</br>
전반적인 정책인지, 다음 릴리스에서도 유지될지는 알 수 없다.

addAll 메서드를 다른 식으로 재정의할 수도 있다. 예컨대 주어진 컬렉션을 순회하며 원소 하나당 add 메서드를</br>
한번만 호출하는 것이다. 이 방식은 HashSet 의 addAll 을 더 이상 호출하지 않으니 addAll 이 add 를 사용</br>
하는지와 상관없이 결과가 옳다는 점에서 조금은 나은 해법이다. 하지만 여전히 문제는 남는다. 상위 클래스의</br>
메서드 동작을 다시 구현하는 이 방식은 어렵고, 시간도 더 들고, 자칫 오류를 내거나 성능을 떨어뜨릴 수도 있다.</br>
또한 하위 클래스에서는 접근할 수 없는 private 필드를 써야하는 상황이라는 이 방식으로는 구현이 불가능하다.

하위 클래스가 깨지기 쉬운 이유는 더 있다. 다음 릴리스에서 상위 클래스에 새로운 메서드를 추가한다면 어떨까</br>
보안 때문에 컬렉션에 추가된 모든 원소가 특정 조건을 만족해야만 하는 프로그램을 생각해보자. 그 컬렉션을</br>
상속하여 원소를 추가하는 모든 메서드를 재정의해 필요한 조건을 먼저 검사하게끔 하면 될 것 같다. 하지만</br>
이 방식이 통하는 것은 상위 클래스에 또 다른 원소 추가 메서드가 만들어지기 전까지다. 다음 릴리스에서 우려한</br>
일이 생기면 하위 클래스에서 재정의하지 못한 그 새로운 메서드를 사용해 '허용되지 않은' 원소를 추가할 수 있다.</br>
실제로도 컬렉션 프레임워크 이전부터 존재하던 HashTable 과 Vector 를 컬렉션 프레임워크에 포함시키자 이와</br>
관련한 보안 구멍들을 수정해야 하는 사태가 벌어졌다.

이상의 두 문제 모두 메서드 재정의가 원인이었다. 따라서 클래스를 확장하더라도 메서드를 재정의하는 대신 새로운</br>
메서드를 추가하면 괜찮으리라 생각할 수도 있다. 이 방식이 훨씬 안전한 것은 맞지만, 위험이 전혀 없는 것은 아니다.</br>
다음 릴리스에서 상위 클래스에 새 메서드가 추가 됐는데, 운 없게도 하필 여러분이 하위 클래스에 추가한 메서드와</br>
시그니처가 같고 반환 타입이 다르다면 여러분의 클래스는 컴파일조차 되지 않는다. 반환 타입 마저 같다면 상위클래스</br>
의 새 메서드를 재정의한 꼴이니 앞서의 문제와 똑같은 상황이 벌어진다. 여러분이 만든 메서드는 상위 클래스의 메서드가</br>
존재하지도 않았으니, 여러분이 만든 메서드는 상위 클래스의 메서드가 요구하는 규약을 만족하지 못할 가능성이 크다.

다행히 이상의 문제를 모두 피해가는 묘안이 있다. 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private</br>
필드로 기존 클래스의 인스턴스를 참조하게 하자. 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한</br>
설계를 컴포지션(composition) 이라 한다. 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를</br>
호출해 그 결과를 반환한다. 이 방식을 전달(forwarding)이라 하며, 새 클래스의 메서드들을 전달 메서드 라 부른다.</br>
그 결과 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 매서드가</br>
추가되더라도 전혀 영향받지 않는다. 다음은 InstrumentedHashSet 을 컴포지션과 전달 방식으로 다시 구현한 코드다.</br>
하나는 집합 클래스 자신이고, 다른 하나는 전달 메서드만으로 이뤄진 재사용 가능한 전달 클래스다.
```java
// 래퍼 클래스 - 상속 대신 컴포지션을 사용
public class InstrumentHashSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    
    public InstrumentHashSet(Set<E> s) {
        super(s);
    }
    
    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    
    public int getAddCount() {
        return addCount;
    }
}
```
```java
// 재사용할 수 있는 전달 클래스
public class ForwardingSet<E> extends Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) {this.s = s;}

    public void clear() {s.clear();}
    // ... 이하 Set 메서드 구현 및 s. 로 호출
}
```
InstrumentedSet 은 HashSet 의 모든 기능을 정의한 Set 인터페이스를 활용해 설계되어 견고하고 유여하다.</br>
구체적으로는 Set 인터페이스를 구현했고, Set 의 인스턴스를 인수로 받는 생성자를 하나 제공한다. 임의의 Set에</br>
계측 기능을 덧씌워 새로운 Set 으로 만드는 것이 이 클래스의 핵심이다. 상속 방식은 구체 클래스 각각을 따로</br>
확장해야 하며, 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해줘야 한다. 하지만 지금</br>
선보인 컴포지션 방식은 한번만 구현해두면 어떠한 SEt 구현체라도 계측할 수 있으며, 기존 생성자들과도 함께 사용</br>
할 수 있다.