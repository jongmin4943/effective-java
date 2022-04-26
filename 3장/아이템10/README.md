## 3장 모든객체의 공통 메서드

------------------

#### 아이템 10 . equals 는 일반 규약을 지켜 재정의하라
<strong>다음에 열거한 상황중 하나에 해당한다면 재정의하지 않는것이 최선이다.</strong>


- 각 인스턴스가 본질적으로 고유하다. 값을 표하는하는게 아니라 동작하는 개체를 표현하는 클래스. </br>
Thread가 좋은 예로, Object 의 equals 메서드는 이러한 클래스에 맞게 구현되어있다.
- 인스턴스의 논리적 동치성(logical equality)을 검사할 일이 없다.</br>
Pattern 은 equals 를 재정의해서 두 Pattern 의 인스턴스가 같은 정규표현식을 나타내는지를 검사하는</br>
즉 녹리적 동치성을 검사하는 방법도 있다. 하지만 설계자는 클라이언트가 이 방식을 원하지 않거나 애초에</br>
필요하지 않다고 판단할 수도 있다. 설계자가 후자로 판단했다면 Object 의 기본 equals 만으로 해결된다.
- 상위 클래스에서 재정의한 equals 가 하위 클래스에도 딱 들어맞는다.</br>
대부분의 Set 구현체는 AbstractSet 이 구현한 equals 를 상속받아 쓰고, List 구현체들은 AbstractList</br>
로부터, Map 구현체들은 AbstractMap 으로부터 상속받아 그대로 쓴다.
- 클래스가 private 이거나 package-private 이고 equals 메서드를 호출할 일이 없다.</br>
위험을 철저히 회피하는 스타일이라 eqauls 가 실수로라도 호출되는걸 막고싶다면
```java
@Override
public boolean equals(Object o) {
    throw new AssertionError(); // 호출시 바로 에러 발생
}
```

equals 를 재정의해야 할 때는 <strong>객체 실병성(object identity : 두 객체가 물리적으로 같은가) 이 아니라 </br>
논리적 동치성을 확인해야하는데, 상위 클래스의 equals 가 논리적 동치성을 비교하도록 재정의되지 않았을때다.</strong></br>
주로 Integer 과 String 같은 값 클래스들이 여기 해당한다. 두 값 객체를 equals 로 비교하는 프로그래머는</br>
객체가 같은지가 아니라 값이 같은지를 알고 싶어 할 것이다. equals 가 논리적 동치성을 확인하도록 재정의해두면</br>
그 인스턴스는 값을 비교하길 원하는 프로그래머의 기대에 부응함은 물론 Map 의 키와 Set 의 원소로 사용할 수 있게된다.</br>

값 클래스라 해도, 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스 라면 equals 를</br>
재정의 하지 않아도 된다. Enum 도 여기에 해당한다. 이런 클래스는 어차피 논리적으로 같은 인스턴스가 2개 이상</br>
만들어지지 않으니 논리적 동치성과 객체 식병성이 사실상 똑같은 의미가 된다. 따라서 Object 의 equals 가</br>
논리적 동치성까지 확인해준다고 볼 수 있다.

equals 메서드를 재정의할 때는 반드시 일반 규약을 따라야 한다. 다음은 Object 명세에 적힌 규약이다.
- equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.
  1. 반사성(reflexivity): null 이 아닌 모든 참조값 x 에 대해, x.equals(x) 는 true 다.
  2. 대칭성(symmetry): null 이 아닌 모든 참조값 x,y 에 대해, x.equals(y) 가 true 면 y.equals(x)도 true 다.
  3. 추이성(transitivity): null 이 아닌 모든 참조값 x,y,x 에 대해, </br> 
  x.equals(y)가 true 이고 y.equals(z)도 true 이면 x.equals(z)도 true 이다.
  4. 일관성(consistency): null 이 아닌 모든 참조값 x,y 에 대해, </br>
  x.equals(y)를 반복해서 호출하면 항상 true 를 반환하거나 항상 false 를 반환해야한다.
  5. null-아님: null 이 아닌 모든 참조값 x 에 대해, x.equals(null) 은 false 이다.

한 클래스들의 인스턴스는 다른 곳으로 빈번히 전달된다. 그리고 컬렉션 클래스들을 포함해 수많은 클래스는 </br>
전달 받은 객체가 equals 규약을 지킨다고 가정하고 동작한다.</br>

Object 명세에서 말하는 동치관계란 무엇일까?</br>
쉽게 말해, 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산이다. 이 부분집합을 동치류(동치클래스)</br>
라 한다. equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야한다.</br>

<strong>반사성</strong>은 객체는 자기 자신과 같아야한다는 뜻이다.</br>
일부러 어기는 경우가 아니라면 만족시키지 못하기가 더 어렵다. 이 요건을 어긴 클래스의 인스턴스를 </br>
컬렉션에 넣은다음 contains 메서드를 호출하면 방금 넣은 인스턴스가 없다고 답할 것이다.</br>
<strong>대칭성</strong>은 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻이다.</br>
반사성 요건과 달리 대칭성 요건은 자칫하면 어길 수 있다. 대소문자를 구별하지 않는 문자열을 구현한 다음 클래스를 예로 살펴보자.</br>
이 클래스에서 toString 메서드는 원본 문자열의 대소문자를 그대로 돌려주지만 equals 에서는 대소문자를 무시한다.</br>
```java
@Override
public boolean equals(Object o) {
    if(o instanceOf CaseInsensitiveString)
        return s.eqaulsIgnoreCase((CaseInsensitiveString) o).s;
    if(o instanceOf String) // 한 방향으로만 작동한다
        return s.eqaulsIgnoreCase((String) o);
    return false;
}
```
CaseInsensitiveString 의 equals 는 순힌하게 일반 문자열과도 비교를 시도한다. 다음처럼 </br>
CaseInsensitiveString 과 일반 String 객체가 하나씩 있다고 해보자.

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "Polish";
```

cis.equals(s) 는 true 를 반환한다. 문제는 CaseInsensitiveString 의 equals 는 일반 String 을 알고 있지만</br>
String 의 equals 는 CaseInsensitiveString 의 존재를 모른다는 데 있다. 따라서 s.equals(cis) 는 false 를</br>
반환하여, 대칭성을 명백히 위반한다. 이번에는 CaseInsensitiveString 를 컬렉션에 넣어보자.

```java
List<CaseInsensitiveString> list = new ArrayList<>();
list.add(cis)
```
이 다음에 list.contains(s) 를 호출하면 현재의 OpenJDK 에서는 false 를 반환한다. 하지만 이는 순전히</br>
구현하기 나름이라 OpenJDK 버전이 바뀌거나 다른 JDK 에서는 true 를 반환하거나 런타임 예외를 던질수도 있다.</br>
<strong>equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다.</strong></br>
이 문제를 해결하려면 CaseInsensitiveString 의 equals 를 String 과도 연동하겠다는 허황한 꿈을 버려야한다.</br>
```java
@Override
public boolean equals(Object o) {
    return o instanceOf CaseInsensitiveString && 
        ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

<strong>추이성</strong></br>
<strong>일관성</strong></br>
<strong>null-아님</strong></br>