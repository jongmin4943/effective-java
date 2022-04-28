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

<strong>반사성</strong></br>은 객체는 자기 자신과 같아야한다는 뜻이다.</br>
일부러 어기는 경우가 아니라면 만족시키지 못하기가 더 어렵다. 이 요건을 어긴 클래스의 인스턴스를 </br>
컬렉션에 넣은다음 contains 메서드를 호출하면 방금 넣은 인스턴스가 없다고 답할 것이다.</br></br>
<strong>대칭성</strong></br>은 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻이다.</br>
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

<strong>추이성</strong></br>은 첫번쨰 객체와 두번째 객체가 같고, 두번째 객체와 세번째 객체가 같다면</br>
첫번째 객체와 세번째 객체도 같아야 한다는 뜻이다. 이 요건도 간단하지만 자칫하면 어기기 쉽다. 상위 클래스에는 없는 </br>
새로운 필드를 하위 클래스에 추가하는 상황을 생각해보자. equals 비교에 영향을 주는 정보를 추가한것이다. </br>
간단히 2차원에서의 점을 표현하는 클래스를 예로 들어보자.
```java
@Override
public boolean equals(Object o) {
    if(!(o instanceof ColorPoint)) return false;
    return super.equals(o) && ((ColorPoint)o).color == color;
}
```
이 메서드는 일반 Point 를 ColorPoint 에 비교한 결과와 그 둘을 바꿔 비교한 결과가 다를 수 있다. </br>
Point 의 equals 는 색상을 무시하고, ColorPoint 의 equals 는 입력 매개변수의 클래스 종류가 다르다며 </br>
매번 false 만 반환할 것이다.
```java
Point p = new Point(1,2);
ColorPoint cp = new ColorPoint(1,2,Color.RED);
```
이제 p.equals(cp) 는 true를, cp.equals(p)는 false 를 반환한다. ColorPoint.equals 가 Point와 </br>
비교할때는 색상을 무시하도록 하면 해결될까?
```java
@Override
public boolean equals(Object o){
        if(!(o instanceof Point))return false;

        // o가 일반 Point 면 색상을 무시하고 비교한다.
        if(!(o instanceof ColorPoint))return o.equals(this);

        // o가 ColorPoint면 색상까지 비교한다.
        return super.equals(o)&&((ColorPoint)o).color==color;
}
```
이 방식은 대칭성은 지켜주지만 추이성을 깨버린다.
```java
ColorPoint p1 = new ColorPoint(1,2 Color.RED);
Point p2 = new Point(1,2);
ColorPoint p3 = new ColorPoint(1,2 Color.BULE);
```
이제 p1.equals(p2) 와 p2.equals(p3) 는 true 를 반환하는데, p1.equals(p3) 가 false 를 반환한다.</br>
추이성에 위배된다. p1과 p2, p2와 p3비교에서는 색상을 무시했지만, p1과 p3비교에서는 색상까지 고려했기때문이다. </br>
또한, 이 방식은 무한 재귀에 빠질 위험도 있다. Point 의 또 다른 하위 클래스로 SmellPoint 를 만들고, equals 는 </br>
같은 방식으로 구현한다음, myColorPoint.equals(mySmellPoint) 를 호출하면 StackOverflowError 가 난다.</br>

이 현상은 모든 객체 지향 언어의 동치관계에서 나타나는 근본적인 문제다. 구체 클래스를 확장해 새로운 값을 추가하면서</br>
equals 규약을 만족시킬 방법은 존재하지 않는다. 객체 지향적 추상화의 이점을 포기하지 않는 한 말이다.</br>
이 말은 얼핏, equals 안의 instanceof 검사를 getClass 검사로 바꾸면 규약도 지키고 값도 추가하면서 구체 클래스를</br>
상속할수 있다는 뜻으로 들린다.
```java
@Override
public boolean equals(Object o){
    if(o == null || o.getClass() != getClass()) return false;
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```
이번 equals 는 같은 구현 클래스의 객체와 비교할 때만 true 를 반환한다. 괜찮아 보이지만 실제로 활용할 수 없다.</br>
Point 의 하위 클래스는 정의상 여전히 Point 이므로 어디서든 Point 로써 활용될 수 있어야한다. 그런데 이 방식에서는</br>
그렇지 못하다. 예를들면 주어진 점이 반지름이 1인원 안에 있는지를 판변하는 메서드가 필요하다 해보자.
```java
private static final Set<Point> unitCircle = Set.of(
            new Point(1,0), newPoint(0,1), newPoint(-1,0), new Point(0,-1)
        );
public static boolean onUnitCircle(Point p) {
    return unitCircle.contains(p);
}
```
이 기능을 구현하는 가장 빠른 방법은 아니지만, 어쨌든 동작은 한다. 이제 값을 추가하지 않는 방식으로 Point를 확장하겠다.</br>
만들어진 인스턴스의 개수를 생성자에서 세보도록 하자.

```java
public class CounterPoint extends Point {
  private static final AtomicInteger counter = new AtomicInteger();
  
  public CounterPoint(int x, int y) {
      super(x,y);
      counter.incrementAndGet();
  }
  public static int numberCreated() {return counter.get();}
}
```
리스코프의 치환 원칙(LSP) 에 따르면, 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다. </br>
따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야한다. 이는 앞서의 "Point의 하위 클래스는 정의상</br>
여전히 Point 이므로 어디서든 Point 로써 활용될 수 있어야 한다." 를 격식있게 표현한 말이다.</br>
그런데 CounterPoint 의 인스턴스를 onUnitCircle 메서드에 넘기면 어떻게 될까? Point 클래스의 equals 를 </br>
getClass 를 사용해 작성했다면 onUnitCircle 은 false 를 반환할 것이다. CounterPoint 인스턴스의 x,y 값과는</br>
무관하게 말이다. 컬렉션 구현체에서 주어진 원소를 담고 있는지 확인하는 방법에 문제가 있다. onUnitCircle 에서 사용한</br>
Set을 포함한 대부분의 컬렉션은 이 작업에 equals 메서드를 이용하는데, CounterPoint 의 인스턴스는 어떤 Point</br>
와도 같을 수 없기 때문이다. 반면, Point 의 equals를 instanceof 기반으로 올바르게 구현했다면 CounterPoint</br>
인스턴스를 건네줘도 onUnitCircle 메서드가 제대로 동작할 것이다.</br>
구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만 괜찮은 우회 방법이 하나 있다. "상속대신 컴포지션을 사용하라"</br>
는 아이템 18의 조언을 따르면 된다. Point 를 상속하는 대신 Point 를 ColorPoint 의 private 필드로 두고,</br>
ColorPoint 와 같은 위치의 일반 Point 를 반환하는 뷰(view) 메서드를 public 으로 추가하는 식이다.

```java
public class ColorPoint {
  private final Point point;
  private final Color color;

  public ColorPoint(int x, int y, Color color) {
    point = new Point(x, y);
    this.color = Objects.requireNonNull(color);
  }
  // 이 ColorPoint 의 Point 뷰를 반환한다.
  public Point asPoint() {
      return point;
  }
  @Override
  public boolean equals(Object o) {
      if(!(o instanceof ColorPoint)) return false;
      ColorPoint cp = (ColorPoint) o;
      return cp.point.equals(point) && cp.color.equals(color);
  }
}
```
자바 라이브러리에도 구체 클래스를 확장해 값을 추가한 클래스가 종종 있다. TimeStamp 는 Date 를 확장한 후 </br>
nanoseconds 필드를 추가했다. 그 결과로 Timestamp 의 equals 는 대칭성을 위배하며, Date 객체와 한 컬렉션</br>
에 넣거나 서로 섞어 사용하면 엉뚱하게 동작할 수 있다. 그래서 Timestamp 의 API 설명에는 Date 와 섞어 쓸때의</br>
주의사항을 언급하고 있다. 둘을 명확히 분리해 사용하는 한 문제될 것은 없지만, 섞이지 않도록 보장해줄 수단은 없다.</br>
자칫 실수하면 디버깅하기 어려운 이상한 오류를 경험할 수 있으니 주의하자.
> 추상클래스의 하위클래스에서라면 equals 규약을 지키면서도 값을 추가할 수 있다.
> 태그 달린 클래스보다는 클래스 계층구조를 활용하라(아이템23) 의 조언을 따르는 클래스 계층구조에서는
> 아주 중요한 사실이다. 예컨대 아무런 값을 갖지 않는 추상 클래스인 Shape 를 위에두고, 이를 확장하여
> radius 필드를 추가한 Circle 클래스와, length 와 width 를 추가한 Rectangle 클래스를 만들 수있다.
> 상위 클래스를 직접 인스턴스로 만드는게 불가능하다면 지금까지 이야기한 문제들은 일어나지 않는다.

<strong>일관성</strong></br>
은 두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 뜻이다.</br>
가변 객체는 비교 시점에 따라 서로 다를수도 혹은 같을 수도 있는 반면, 불변 객체는 한번 다르면 끝까지 달라야 </br>
한다. 클래스를 작성할 때는 불변 클래스로 만드는게 나을지를 심사숙고하자. 불변 클래스로 만들기로 했다면 </br>
equals 가 한번 같다고 한 객체는 영원히 같도록 해야한다.</br>
클래스가 불변이든 가변이든 equals 의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다. 이 제약을 어기면</br>
익관성 조건을 만족시키기가 아주 어렵다. 예컨대 URL 의 equals 는 주어진 URL 과 매핑된 호스트이 IP 주소를 </br>
이용해 비교한다. 호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야하는데, 그 결과가 항상 같다고 보장할 수 없다.</br>
이는 URL의 equals 가 일반 규약을 어기게하고, 실무에서도 종종 문제를 일으킨다. 하휘 호환성이 발목을 잡아 </br>
잘못된 동작을 바로잡을 수도 없다. 이런 문제를 피하려면 equals 는 항시 메모리에 존재하는 객체만을 사용한</br>
결정적(deterministic) 계산만 수행해야한다.</br>

<strong>null-아님</strong></br>
은 이름처럼 모든 객체가 null 과 같지 않아야 한다는 뜻이다.</br> 의도하지 않았음에도 o.equals(null)이 true를</br>
반환하는 상황은 상상하기 어렵지만, 실수로 NPE을 던지는 코드는 흔할 것이다. 이 일반규약은 이런 경우도 허용하지않는다.</br>
수많은 클래스가 다음코드처럼 입력이 null인지를 확인해 자신을 보호한다.
```java
// 명시적 null 검사 -> 필요없다.
@Override public boolean equals(Ojbect o) {
  if(o == null) return false
}
```
이러한 검사는 필요치 않다. 동치성을 검사하려면 equals 는 건네받은 객체를 적절히 형변환한 후 필수 필드들의 값을</br>
알아내야한다. 그러려면 형변환에 앞서 instanceof 연산자로 입력 매개변수가 올바른 타입인지 검사해야한다.
```java
@Override public boolean equals(Ojbect o) {
  if(!(o instanceof MyType)) return false;
  MyType mt = (MyType) o;
}
```
equals 가 타입을 확인하지 않으면 잘못된 타입이 인수로 주어졌을 때 ClassCastException 을 던져서 일반 규약을</br>
위배하게 된다. 그런데 instanceof는 첫번째 피연자가 null이면 false를 반환한다. 따라서 입력이 null이면 타입확인</br>
단계에서 false를 반환하기 때문에 null검사를 명시적으로 안해도 된다.

- equals 메서드 구현방법 정리</br>
  1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다. 성능 최적화용으로 자기자신이면 true를 반환한다.
  2. instanceof 연산자로 입력이 올바른 타입인지 확인한다. 그렇지 않다면 false를 반환한다. 이때의 올바른 타입은 equals가
  정의된 클래스인 것이 보통이지만, 가끔은 그 클래스가 구현한 특정 인터페이스가 될 수도 있다. 어떤 인터페이스는 자신을 구현한
  클래스끼리도 비교할 수 있도록 규약을 수정하기도 한다. 이런 인터페이스를 구현한 클래스라면 equals 에서
  해당 인터페이스를 사용해야 한다. Set, List, Map 등의 컬렉션 인터페이스들이 여기에 해당한다.
  3. 입력을 오바른 타입으로 형변환한다.
  4. 입력 객체와 자기 자신의 대응되는 '핵심'필드들이 모드 일치하는지 하나씩 검사한다. 모든 필드가 일치하면 true,
  하나라도 다르다면 false를 반환한다. 2단계에서 인터페이스를 사용했다면 입력의 필드값을 가져올때도 그 인터페이스의
  메서드를 사용해야한다. 타입이 클래스라면 해당 필드에 직접 접근할 수도 있다.

float 과 double 을 제외한 기본 타입 필드는 == 연산자로 비교하고, 참조 타입 필드는 각각의 equals 메서드로,</br>
float 과 double 필드는 각각 정적 메서드인 Float.compare(float, float) 와 Double.compare(double, double)</br>
로 비교한다. 이 둘이 특별한 이유는 Float.NaN, -0.0f, 특수한 부동소수값 등을 다뤄야 하기 때문이다.</br>
Float.equals 와 Double.equals 메서드를 대신 사용할 수도 있지만, 이 메서드들은 오토박승을 수반할 수 있으니</br>
성능상 좋지 않다. 배열 필드는 원소 각각을 앞서의 지침대로 비교한다. 배열의 모든 원소가 핵심필드라면 </br>
Arrays.equals 메서드들 중 하나를 사용하자.때론 null 도 정상 값으로 취급하는 참조 타입 필드도 있다. </br>
이런 필드는 Objects.equals(Object, Object)로 비교해 NPE 발생을 예방하자.

앞서의 CaseInsensitiveString 예처럼 비교하기가 아주 복잡한 필드를 가진 클래스도 있다. 이럴때는 그 필드의</br>
표준형(canonical form)을 저장해둔 후 표준형끼리 비교하면 훨씬 경제적이다. 이 기법은 특히 불변 클래스에 제격이다.</br>
가변 객체라면 값이 바뀔 때마다 표준형을 최신 상태로 갱신해줘야한다.</br>
어떤 필드를 먼저 비교하느냐가 equals 의 성능을 좌우하기도 한다. 최상의 성능을 바란다면 다를 가능성이 더 크거나</br>
비교하는 비용이 싼 필드를 먼저 비교하다. 동기화용 lock 필드 같이 객체의 논리적 상태와 관련 없는 필드는 비교하면</br>
안된다. 핵심 필드로부터 계산해낼 수 있는 파생 필드 역시 굳이 비교할 필요는 없지만, 파생 필드를 비교하는 쪽이 더 </br>
빠를때도 있다. 파생 필드가 객체 전체의 상태를 대표하는 상황에서 그렇다. 예컨대 자신의 영역을 캐시해두는 Polygon 클래스</br>
가 있다고 해보자. 그렇다면 모든 변과 정점을 일일이 비교할 필요 없이 캐시해둔 영역만 비교하면 결과를 곧바로 알 수있다.</br>

equals 를 다 구현했다면 세가지만 자문해보자. 대칭적인가? 추이성이 있는가? 일관적인가? 자문에서 끝내지 말고 </br>
단위 테스트를 작성해 돌려보자. 단, equals 메서드를 AutoValue 를 이용해 작성했다면 테스트를 생략해도 안심할 수 있다.</br>
세 요건중 하나라도 실패한다면 원인을 찾아서 고치자. 물론 나머지 요건인 반사성과 null-아님 도 만족해야한다.

```java
//전형적인 equals 메서드의 예
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;
    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNum = rangeCheck(lineNum, 999, "가입자 번호");
    }
    
    private static short rangeCheck(int val, int max, String arg) {
        if(val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }
    
    @Override
  public boolean equals(Object o) {
        if(o == this) return true;
        if(!(o instanceof PhoneNumber)) return false;
        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    }
}
```

- 주의사항
  1. equals 를 재정의 할땐 hashCode 도 반드시 재정의하자(아이템 11)
  2. 너무 복잡하게 해결하려 들지 말자. 필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있다.
  오히려 너무 공격적으로 파고들다가 문제를 일으키기도 한다. 일반적으로 별칭(alias)은 비교하지 않는게 좋다.
  예컨대 File 클래스라면, 심볼릭 링크를 비교해 같은 파일을 가리키는지를 확인하려 들면 안된다.
  3. Object 외의 타입을 매겨변수로 받는 equals 메서드는 선언하지 말자.

equals(hashCode 도 마찬가지로) 를 작성하고 테스트하는 일은 지루하고 항상 뻔하다. 다행히 이 작업을 대신해줄</br>
오픈 소스가 있으니, 바로 구글의 AutoValue 프레임워크다. 클래스에 애너테이션 하나만 추가하면 AutoValue 가 </br>
이 메서드들을 알아서 작성해주며, 여러분이 직접 작성하는 것과 근본적으로 같은 코드를 만들어 줄것이다.</br>
대부분의 IDE 도 같은 기능을 제공하지만 생성된 코드가 AutoValue 만큼 깔끔하거나 읽기 좋지는 않다. 또한 </br>
IDE 는 나중에 클래스가 수정된걸 자동으로 알아채지는 못하니 테스트 코드를 작성해둬여한다. 이런 단점을 감안하더라도</br>
사람이 직접 작성하는 것보다는 IDE에 맡기는 편이 낫다. 적어도 사람처럼 실수를 저지르지는 않으니 말이다.

> 핵심정리 </br>
> 꼭 필요한 경우가 아니면 equals 를 재정의하지 말자. </br>
> 많은 경우에 Object의 equals가 여러분이 원하는 비교를 정확히 수행해준다.</br>
> 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야한다.