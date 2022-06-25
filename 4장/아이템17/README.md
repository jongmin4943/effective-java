## 4장 클래스와 인터페이스

------------------

#### 아이템 17 . 변경 가능성을 최소화하라

불변 클래스란 간단히 말해 그 인스턴스의 내부 값을 수정할 수 없는 클래스다. 불변 인스턴스에 간직된 정보는 고정되어</br>
객체가 파괴되는 순간까지 절대 다라지지 않는다. String, 기본 타입의 박싱된 클래스들, BigInteger, BigDecimal</br>
이 여기 속한다. 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다</br>

클래스를 불변으로 만드려면 다음 다섯가지 규칙을 따르면 된다.
- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
- 클래스를 확장할 수 없도록 한다. 하위 클래스에서 부주의하게 혹은 나쁜의도로 객체의 상태를 변하게 만드는 사태를</br>
  막아준다. 상속을 막는 대표적인 방법은 클래스를 final 로 선언하는 것이다.
- 모든 필드를 final 로 선언한다. 시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러내는 방법이다.</br>
  새로 생성된 인스턴스를 동기화 없이 다른 스레드로 건네도 문제없이 동작하게끔 보장하는데도 필요하다.
- 모든 필드를 private 으로 선언한다. 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다.</br>
  기술적으로는 기본 타입필드나 불변 객체를 참조하는 필드를 public final 로만 선언해도 불변 객체가 되지만,</br>
  이렇게 하면 다음 릴리스에서 내부 표현을 바꾸지 못하므로 권하지는 않는다.
- 자신 외에는 내부의 가변 컴포넌트에서 접근할 수 없도록 한다. 클래스에 가변 객체를 참조하는 필드가 하나라도</br>
  있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야한다. 이런 필드는 절대 클라이언트가 제공한 객체</br>
  객체 참조를 가리키게 해서는 안되며, 접근자 메서드가 그 필드를 그대로 반환해서도 안된다. 생성자, 접근자, </br>
  readObject 메서드 모두에서 방어적 복사를 수행하라.

지금까지 아이템들에서 선보인 예제 클래스들은 대부분 불변이다. 각 속성을 반환하는 접근자만 제공할 뿐 변경자는</br>
없던 아이템 11의 PhoneNumber 도 이에 해당한다.
```java
public final class Complex {
    private final double re;
    private final double im;
    
    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    public double realPart() { return re; }
    public double imaginaryPart() { return im; }
    
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im, re * c.im + im + c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re + c.re + im + c.im) / tmp, (im * c.re - re * c.im) / tmp);
    }
    
    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof Complex)) return false;
        Complex c = (Complex) o;

        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    
    @Override
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }
    
    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

이 클래스는 복소수(실수부와 허수부로 구성된 수)를 표현한다. Object 의 메서드 몇개를 재정의했고, 실수부와 </br>
허수부 값을 반환하는 접근자 메서드 (realPart 와 imaginaryPart)와 사칙연산 메서드를 정의했다. 이 사칙연산</br>
메서드들이 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환하는 모습에 주목하자. 이처럼</br>
피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴을 함수형 프로그래밍이라한다.</br>
이와 달리, 절차적 혹은 명령형 프로그래밍에서는 메서드에서 피연사자인 자신을 수정해 자신의 상태가 변하게 된다.</br>
또한 메서드 이름으로 (add 같은) 동사 대신 (plus 같은) 전치사를 사용한 점에도 주목하자. 이는 해당 메서드가</br>
객체의 값을 변경하지 않는다는 사실을 강조하려는 의도다. 참고로, 이 명명 규칙을 따르지 않은 BigInteger 와</br>
BigDecimal 클래스를 사람들이 잘못 사용해 오류가 발생하는 일이 자주 있다.

함수형 프로그래밍에 익숙하지 않다면 조금 부자연스러워 보일 수도 있지만, 이 방식으로 프로그래밍하면 코드에서</br>
불변이 되는 영역의 비율이 높아지는 장점을 누릴 수 있다. 불변 객체는 단순하다. 불변객체는 생성된 시점의</br>
상태를 파괴될 때까지 그대로 간직한다. 모든 생성자가 클래스 불변식(class invariant)을 보장한다면 그</br>
클래스를 사용하는 프로그래머가 다른 노력을 들이지 않더라도 영원히 불변으로 남는다. 반면 가변 객체는</br>
임의의 복잡한 상태에 놓일 수 있다. 변경자 메서드가 일으키는 상태 전이를 정밀하게 문서로 남겨놓지 않는</br>
가변 클래스는 믿고 사용하기 어려울 수도 있다.