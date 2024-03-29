## 4장 클래스와 인터페이스

------------------

#### 아이템 22 . 인터페이스는 타입을 정의하는 용도로만 사용하라

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다. 클래스가 어떤 인터페이스를<br/>
구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것이다. 인터페이스는 오직<br/>
이 용도로만 사용해야 한다.

이 지침에 맞지 않는 예로 소위 상수 인터페이스라는 것이 있다. 상수 인터페이스란 메서드 없이, 상수를 뜻하는<br/>
state final 필드로만 가득 찬 인터페이스를 말한다. 그리고 이 상수들을 사용하려는 클래스에서는 정규화된<br/>
이름을 쓰는 걸 피하고자 그 인터페이스를 구현하곤 한다.
```java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    // 블츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    // 전자 질량 (kg)
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```
상수 인터페이스 안티패턴은 인터페이스를 잘못 사용한 예다. 클래스 내부에서 사용하는 상수는 외부 인터페이스가<br/>
아니라 내부 구현에 해당한다. 따라서 상수 인터페이스를 구현하는 것은 이 내부구현을 클래스의 API 로 노출하는<br/>
행위다. 클래스가 어떤 상수 인터페이스를 사용하든 사용자에게 아무런 의미가 없다. 오히려 사용자에게 혼란을<br/>
주기도 하며, 더 심하게는 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속되게 한다. 그래서 다음<br/>
릴리스에서 이 상수들을 더는 쓰지 않게 되더라도 바이너리 호환성을 위해 여전히 상수 인터페이스를 구현하고<br/>
있어야 한다. final 이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위 클래스의 이름공간이 그<br/>
인터페이스가 정의한 상수들로 오염되어 버린다. java.io.ObjectStreamConstants 등 자바 플랫폼 라이브러리<br/>
에도 상수 인터페이스가 몇개 있으나, 잘못 활용한 예이니 따라해서는 안된다.

상수를 공개할 목적이라면 더 합당한 선택지가 몇가지 있다. 특정 클래스나 인터페이스와 강하게 연관된 상수라면<br/>
그 클래스나 인터페이스 자체에 추가해야한다. 모든 숫자 기본 타입의 박싱 클래스가 대표적으로, Integer 와<br/>
Double 에 선언된 MIN_VALUE 와 MAX_VALUE 상수가 이런 예다. 열거 타입으로 나타내기 적합한 상수라면<br/>
열거 타입으로 만들어 공개하면 된다. 그것도 아니라면, 인스턴스화 할수 없는 유틸리티 클래스에 담아 공개하자.<br/>
다음 코드는 앞서 보여준 PhysicalConstants 의 유틸리티 클래스 버전이다.
```java
public class PhysicalConstants {
    private PhysicalConstants() {} // 인스턴스화 방지
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    // 블츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    // 전자 질량 (kg)
    static final double ELECTRON_MASS = 9.109_383_56e-31;
    /*
     숫자 리터럴에 사용한 밑줄은 자바 7부터 허용되는 숫자 리터럴 값에는 아무런 영향을 주지않으면서, 읽기는
     훨씬 편하게 해준다. 고정 소수점 수든 부동소수점 수든 5자리 이상이라면 밑줄을 사용하는걸 고려해보자.
     십진수 리터럴도 밑줄을 사용해 세자리씩 묶어주는것이 좋다. 
     */
}
```
유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 클래스 이름까지 함께 명시해야 한다.<br/>
PhysicalConstants.AVOGADROS_NUMBER 처럼 말이다. 유틸리티클래스의 상수를 빈번히 사용한다면<br/>
정적 임포트하여 클래스 이름은 생략할 수 있다.

> 핵심정리<br/>
> 인터페이스는 타입을 정의하는 용도로만 사용해야한다. 상수 공개용 수단으로 사용하지 말자.

