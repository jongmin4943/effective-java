## 4장 클래스와 인터페이스

------------------

#### 아이템 23 . 태그 달린 클래스보다는 클래스 계층구조를 활용하라

두가지 이상의 의미를 표현할 수 있으며, 그중 현재 표현하는 의미를 태그 값으로 알려주는 클래스를 본적이</br>
있을 것이다.
```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };
    
    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;
    
    // 다음 필드들은 모양이 사각형일때만 쓰인다.
    doubl length;
    doubl width;
    
    // 다음 필드는 모양이 원일때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }
    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }
    
    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```
태그 달린 클래스에는 단점이 한가득이다. 우선 열거 타입 선언, 태그 필드, switch 문 등 쓸데없는 코드가</br>
많다. 여러 구현이 한 클래스에 혼합돼 있어서 가독성도 나쁘다. 다른 의미를 위한 코드도 언제나 함께 하니</br>
메모리도 많이 사용한다. 필드들을 final 로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서</br>
초기화 해야한다. 생성자가 태그 필드를 설정하고 해당 의미에 쓰이는 데이터 필드들을 초기화하는데 컴파일러가</br>
도와줄 수 있는건 별로 없다. 엉뚱한 필드를 초기화해도 런타임에야 문제가 드러날 뿐이다. 또 다른 의미를</br>
추가하라면 코드를 수정해야만 한다. 예를 들어 새로운 의미를 추가할 때마다 모든 switch 문을 찾아 새 의미를</br>
처리하는 코드를 추가해야 하는데, 하나라도 빠뜨리면 역시 런타임에 문제가 불거져 나올 것이다. 마지막으로,</br>
인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 전혀 없다. 태그 달린 클래스는 장황하고 오류를 내기</br>
쉽고, 비효울적이다.

다행히 자바와 같은 객체 지향 언어는 타입 하나로 다양한 의미의 객체를 효현하는 훨씬 나은 수단을 제공한다.</br>
바로 클래스 계층 구조를 활용하는 서브 타이핑이다. 태그 달린 클래스는 클래스 계층구조를 어설프게 흉내낸 아류</br>
일 뿐이다.

가장 먼저 계층구조의 루트가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트</br>
클래스의 추상 메서드로 선언한다. Figure 클래스에서는 area 가 이러한 메서드에 해당한다. 그런 다음</br>
태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다. 모든 하위 클래스에서</br>
공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다. Figure 클래스에서는 태그 값에 상관없는</br>
메서드가 하나도 없고, 모든 하위 클래스에서 사용하는 공통 데이터 필드도 없다. 그 결과 루트 클래스에는</br>
추상 메서드인 area 하나만 남게 된다.

다음으로, 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다. Figure 를 확장한 원 클래스와</br>
사각형 클래스를 만들면 된다. 각 하위 클래스에는 각자의 의미에 해당하는 데이터 필드들을 넣는다. 원에는</br>
반지름을, 사각형에는 길이와 너비를 넣으면 된다. 그 다음 루트 클래스가 정의한 추상 메서드를 각자의 </br>
의미에 맞게 구현한다. 다음은 Figure 클래스를 계층구조 방식으로 구현한 코드다.
```java
abstract class Figure {
    
}
class Circle extends Figure {
    final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}
class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
```
클래스 계층구조는 태그 달린 클래스의 단점을 모두 날려버린다. 간결하고 명확하며, 쓸데없는 코드도 모두</br>
사라진다. 각 의미를 독립된 클래스에 담아 관련없던 데이터 필드를 모두 제거했다. 살아 남은 필드들은 모두</br>
final 이다. 각 클래스의 생성자가 모든 필드를 남김없이 초기화하고 추상 메서드를 모두 구현했는지 </br>
컴파일러가 확인해준다. 실수로 빼먹은 case 문 때문에 런타임 오류가 발생할 일도 없다. 루트 클래스의</br>
코드를 건드리지 않고도 다른 프로그래머들이 독립적으로 계층구조를 확장하고 함께 사용할 수 있다. 타입이</br>
의미별로 따로 존재하니 변수의 의미를 명시하거나 제한할 수 있고, 또 특정 의미만 매겨변수로 받을 수 있다.</br>

또한, 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일타임 타입 검사 능력을</br>
높여준다는 장점도 있다. 태그 달린 클래스를 정사각형도 지원하도록 수정하려면 어디어디를 고쳐야 할지</br>
각자 한번 확인해보자. 클래스 계층구조에서라면 다음과 같이 정사각형이 사각형의 특별한 형태임을 아주 간단하게</br>
반영할 수 있다.

```java
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```
> 핵심정리 </br>
> 태그 달린 클래스를 써야 하는 상황은 거의 없다. 새로운 클래스를 작성하는데 태그 필드가 등장한다면
> 태그를 없애고 계층구조로 대체하는 방법을 생각해보자. 기존 클래스가 태그 필드를 사용하고 있다면
> 계층 구조로 리팩토링 하는걸 고민해보자.