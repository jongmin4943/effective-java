## 3장 모든객체의 공통 메서드

------------------

#### 아이템 13 . Comparable 을 구현할지 고려하라

compareTo 와 Object 의 equals 는 두가지만 빼면 같다. compareTo 는 단순 동치성 비교에 더해</br>
순서까지 비교할 수 있으며 제네릭하다. Comparable 을 구현했다는 것은 그 클래스의 인스턴스들에는</br>
자연적인 순서가 있음을 뜻한다. Comparable 을 구현한 객체들의 배열은 Arrays.sort(); 로 손쉽게</br>
정렬할 수 있다. 검색, 극단값 계싼, 자동 정렬되는 컬렉션 관리도 역시 쉽게 할 수 있다. 예컨대 다음</br>
프로그램은 명렬줄 인수들을 (중복은 제거하고) 알파벳순으로 출력한다. String 이 Comparable을</br>
구현한 덕분이다.

```java
public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```
Comparable 을 구현하여 수많은 제네릭 알고리즘과 컬렉션의 힘을 누릴 수 있다. 사실상 자바 플랫폼</br>
라이브러리의 모든 값 클래스와 열거타입이 Comparable 을 구현했다. 알파벳, 숫자, 연대 같이 순서가</br>
명확한 값 클래스를 작성한다면 반드시 Comparable 을 구현하자.

```java
public interface Comparable<T> {
    int compareTo(T t);
}
```
compareTo 메서드의 일반 규약은 equals 의 규약과 비슷하다.

이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음수, 같으면 0, 크면 양수를 반환한다.</br>
이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException 을 던진다.
- Comparable 을 구현한 클래스는 모든 x, y 에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x)) 여야한다.</br>
- Comparable 을 구현한 클래스는 추이성을 보장해야한다. (x.compareTo(y)) > 0 && (y.compareTo(z) > 0)이면 </br>
  x.compareTo(z) > 0 이다.
- CompareTo 를 구현한 클래스는 모든 z 에 대해 x.compareTo(y) == 0 이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))다
- 필수는 아니지만, (x.compareTo(y) == 0) == (x.equals(y)) 여야 한다. 이것이 지켜지지 않는경우 그 사실을 명시해야한다.

equals 메서드와 달리, compareTo 는 타입이 다른 객체를 신경쓰지 않아도 된다. 타입이 다른 객체가 주어지면</br>
간단히 ClassCastException 을 던져도 되며, 대부분 그렇게 한다. 물론, 이 규약에서는 다른 타입 사이의 비교도</br>
허용 되며, 대부분 그렇게 한다. 물론, 이 규약에서는 다른 타입 사이의 비교도 허용하는데, 보통은 비교할 객체들이</br>
구현한 공통 인터페이스를 매개로 이뤄진다.

compareTo 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못한다. 비교를 활용하는 클래스의 예로는 </br>
정렬된 컬렉션인 TreeSet 과 TreeMap, 검색과 정렬 알고리즘을 활용하는 Collections 와 Arrays 가 있다. </br>

compareTo 메서드로 수행하는 동치성 검사도 equals 규약과 똑같이 반사성, 대칭성, 추이성을 충족해야 함을 뜻한다.</br>
기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 규약을 지킬 방법이 없다. 객체 지향적 추상화의</br>
이점을 포기할 생각이 아니라면 말이다. 우회법도 같다. Comparable 을 구현한 클래스를 확장해 값 컴포넌트를 추가하고</br>
싶다면, 확장하는 대신 독립된 클래스를 만들고, 이 클래스에 원래 클래스의 인스턴스를 가기리키는 필드를 두자.</br>
그런 다음 내부 인스턴스를 반환하는 '뷰' 메소드를 제공하면 된다. 이렇게 하면 바깥 클래스에 우리가 원하는 compareTo</br>
메서드를 구현해 넣을 수 있다. 클라이언트는 필요에 따라 바깥 클래스의 인스턴스를 필드 안에 담긴 원래 클래스의</br>
인스턴스로 다룰수도 있고 말이다.

compareTo 의 마지막 규약은 필수는 아니지만 꼭 지키길 권한다. 예를들어 만약 equals 와 다르다면 빈 HashSet</br>
인스턴스에 new BigDecimal("1.0") 과 new BigDecimal("1.00") 을 차례로 추가하면 equals 메서드로 비교하면</br>
다르기 때문에 원소를 2개 갖게된다. 하지만 HashSet 대신 TreeSet 을 사용하면 원소를 하나만 갖게된다.

이 책의 2판에서는 compareTo 메서드에서 정수 기본 타입 필드를 비교할 때는 관계 연산자인 < 와 > 를, 실수 기본</br>
타입 필드를 비교할 때는 정적 메서드인 Double.compare 과 Float.compare 를 사용하라고 권했다. 그런데</br>
자바 7부터는 상황이 변했다. 박싱된 기본ㅂ 타입 클래스들에 새로 추가된 정적 메서드인 compare 를 이용하면 된다.</br>
compareTo 메서드에서 관계 연산자를 사용하는 이전 방식은 거추장스럽고 오류를 유발하니, 이제 추천하지 않는다.</br>

클래스에 핵심 필드가 여러개라면 어느것을 먼저 비교하느냐가 중요해진다. 가장 핵심적인 필드부터 비교해나가자.</br>
비교 결과가 0이 아니라면, 거기서 순서가 결정되면 끝이다. 가장 핵심이 되는 필드가 똑같다면, 똑같지 않은</br>
필드를 찾을 때 까지 그 다음으로 중요한 필드를 비교해나간다.
```java
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode); // 가장 중요한 필드
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix); // 두번째로 중요한 필드   
        if(result == 0)
            ... // 이하 반복
    }
}
```

자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄방식으로 비교자를</br>
생성할 수 있게 되었다. 그리고 이 비교자들을 Comparable 인터페이스가 원하는 compareTo 메서드를 구현하는데</br>
멋지게 활용할 수 있다. 하지만ㅇ 약간의 성능 저하가 있다.

```java
private static final Comparator<PhoneNumber> COMPARATOR = 
                        comparingInt((PhoneNumber pn) -> pn.areaCode)
                            .thenComparingInt(pn -> pn.prefix)
                            ... // 이하 반복

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);    
}
```
comparingInt 는 객체 참조를 int 타입 키에 매핑하는 키 추출함수를 인수로 받아, 그 키를 기준으로 순서를 정하는</br>
비교자를 반환하는 정적 메서드다.

이따금 '값의 차' 를 기준으로 첫번째 값이 두번째 값보다 작으면 음수를, 두 값이 같으면 0을, 첫번째 값이</br>
크면 양수를 반환하는 compareTo 나 compare 메서드와 마주할것이다.
```java
// 해시코드 값의 차를 기준으로 하는 비교자 -> 추이성 위배!
static Comparator<Object> hashCodeOrder = new Comparator<> (
        public int compare(Object o1, Object o2) {
            return o1.hashCode() - o2.hashCode();
        }
)
```
이 방식은 사용하면 안된다. 정수 오버플로를 일으키거나 부동소수점 계산방식에 따른 오류를 낼 수 있다.</br>
그렇다고 월등히 빠르지도 않다. 대신 다음의 두 방식중 하나를 사용하자.
```java
// 정적 compare 메서드를 활용하자.
static Comparator<Object> hashCodeOrder = new Comparator<> (
        public int compare(Object o1, Object o2) {
            return Integer.compare(o1.hashCode(), o2.hashCode());
        }
)
```
```java
// 비교자 생성 메서드를 활용하자.
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
)
```

> 핵심정리 </br>
> 순서를 고려해야하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고,
> 검색하고, 비교 기능을 제공하는 컬력센과 어우러지도록 해야한다. compareTo 메서드에서 필드의 값을 비교할 때
> <와 > 연산자는 쓰지 말아야한다. 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator
> 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.