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