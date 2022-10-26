## 5장 제네릭

------------------

#### 아이템 30 . 이왕이면 제네릭 메서드로 만들라

클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다. 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통<br/>
제네릭이다. 예컨대 Collections 의 알고리즘 메서드는 모두 제네릭이다. 제네릭 메서드 작성법은 제네릭 타입<br/>
작성법과 비슷하다. 다음은 두 집합의 합 집합을 반환하는 문제가 있는 메서드다.
```java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```
컴파일은 되지만 unchecked 경고가 두개 발생한다. 경고를 없애려면 이 메서드를 타입 안전하게 만들어야 한다.<br/>
메서드 선언에서의 세 집합의 원소 타입을 타입 매개변수로 명시하고, 메서드 안에서도 이 타입 매개변수만<br/>
사용하게 수정하면 된다. 타입 매개변수들을 선언하는 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에<br/>
온다. 타입 매개변수 목록은 <E> 이고 반환 타입은 Set<E> 이다. 타입 매개변수의 명명 규칙은 제네릭 메서드나<br/>
제네릭 타입이나 똑같다.
```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```
단순한 제네릭 메서드라면 이정도면 충분하다. 이 메서드는 경고 없이 컴파일 되며, 타입 안전하고, 쓰기도 쉽다.<br/>
union 메서드는 집합 3개의 타입이 모두 같아야한다. 이를 한정적 와일드카드 타입을 사용하여 더 유연하게 개선<br/>
할 수 있다.

때때로 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있다. 제네릭은 런타임에 타입정보가 소거<br/>
되므로 하나의 객체를 어떤 타입으로든 매개변수화할 수 있다. 하지만 이렇게 하려면 요청한 타입 매개변수에<br/>
맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다. 이 패턴을 제네릭 싱글턴 팩터리라 하며<br/>
Collections.reverseOrder 같은 함수나 Collections.emptySet 같은 컬렉션용으로 사용한다.

이번에는 항등함수를 담은 클래스를 만들고 싶다고 해보자. 자바 라이브러리의 Function.identity 를 사용하면<br/>
되지만, 공부를 위해 직접 한번 작성해보자. 항등함수 객체는 상태가 없으니 요청할 때마다 새로 생성하는 것은<br/>
낭비다. 자바의 제네릭이 실체화 된다면 항등함수를 타입별로 하나씩 만들어야 했겠지만, 소거 방식을 사용한 덕에<br/>
제네릭 싱글턴 하나면 충분하다.
```java
// 제네릭 싱글턴 팩토리 패턴
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;    
}
```
형변환하면 비검사 형변환 경고가 발생한다. T 가 어떤 타입이든 UnaryOperator<Object> 는 <T> 가 아니기<br/>
때문이다. 하지만 항등함수란 입력 값을 수정없이 그대로 반환하는 특별한 함수이므로, T 가 어떤 타입이든<br/>
타입 안전하다. 우리는 이 사실을 알고 있으니 이 메서드가 내보내는 비검사 형변환 경고는 숨겨도 안심할 수 있다.<br/>

상대적으로 드물긴 하지만, 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다.<br/>
바로 재귀적 타입 한정이라는 개념이다. 재귀적 타입 한정은 주로 타입의 자연적 순서를 정하는 Comparable <br/>
인터페이스와 함꼐 쓰인다.
```java
public interface Comparable<T> {
    int compareTo(T o);
}
```
여기서 타입 매개변수 T 는 Comparable<T>를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다. 실제로<br/>
거의 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있다. 따라서 String 은 Comparable<String>을<br/>
구현하고 Integer 는 Comparable<Integer> 를 구현하는 식이다.

Comparable 을 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 그 원소들을 정렬 혹은 검색하거나,<br/>
최소값, 최대값을 구하는 식으로 사용된다. 이 기능을 수행하려면 컬렉션에 담긴 모든 원소가 상호 비교될 수<br/>
있어야 한다.
```java
// 재귀적 타입 한정을 이용해 상호 비교할 수 있음을 표현했다.
public static <E extends Comparable<E>> E max(Collection<E> c);
```
타입 한정인 <E extends Comparable<E>>는 "모든 타입 E 는 자신과 비교할 수 있다" 라고 읽을 수 있다.<br/>
상호 비교 가능하다는 뜻을 아주 정확하게 표현했다고 할 수 있다. 다음은 컬렉션에 담긴 원소의 자연적 순서를<br/>
기준으로 최대값을 계산하며 컴파일 오류나 경고가 발생하지 않는 코드다.
```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어있다.");
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Obejects.requireNonNull(e);
    return result;
}
```
재귀적 타입 한정은 훨씬 복잡해질 가능성이 있긴 하지만, 다행히 그런 일은 잘 일어나지 않는다. 이번 아이템에서<br/>
설명한 관용구, 여기에 와일드카드를 사용한 변형, 그리고 시뮬레이트한 셀프 타입 관용구를 이해하고 나면<br/>
실전에서 마주치는 대부분의 재귀적 타입 한정을 무리없이 다룰 수 있을 것이다.
> 핵심정리<br/>
> 제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다
> 제네릭 메서드가 더 안전하며 사용하기도 쉽다. 타입과 마찬가지로, 메서드도 형변환 없이 사용할 수 있는
> 편이 좋으며, 많은 경우 그렇게 하려면 제네릭 메서드가 되어야 한다. 역시 타입과 마찬가지로, 형변환
> 해줘야 하는 기존 메서드는 제네릭하게 만들자. 기존 클라이언트는 그대로 둔 채 새로운 사용자의 삶을
> 훨씬 편하게 만들어 줄 것이다.