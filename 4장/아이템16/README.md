## 4장 클래스와 인터페이스

------------------

#### 아이템 16 . public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현방식을 언제든 바꿀수<br/>
있는 유연성을 얻을 수 있다. public 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로<br/>
내부 표현 방식을 마음대로 바꿀 수 없게 된다

package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 문제가 없다.<br/>
그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다. 이 방식은 클래스 선언 면에서나 이를 사용하는<br/>
클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다. 클라이언트 코드가 이 클래스 내부 표현에<br/>
묶이기는 하나, 클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다.<br/>
따라서 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다. private 중첩<br/>
클래스의 경우라면 수정 범위가 더 좁아져서 이 클래스를 포함하는 외부 클래스까지로 제한된다.

자바 플랫폼 라이브러리에도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종있다.<br/>
대표적인 예가 java.awt.package 패키지의 Point 와 Dimension 클래스다. 이 클래스들을 흉내<br/>
내지 말고, 타산지석으로 삼길 바란다. 내부를 노출한 Dimension 클래스의 심각한 성능문제는 오늘날까지<br/>
해결되지 못했다.

public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 여전히 결코 좋은 생각이<br/>
아니다. API 를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는<br/>
단점은 여전하다. 단, 불변식은 보장할 수 있게 된다.

```java
// 불변 필드를 노출한 public 클래스
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOURS = 60;
    
    public final int hour;
    public final int minute;
    
    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY) 
            throw new IllegalArgumentException("시간 : " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOURS) 
            throw new IllegalArgumentException("분 : " + minute);
        this.hour = hour;
        this.minute = minute;
    }
    
}
```

> 핵심정리
> public 클래스는 절대 가변 필드를 직접 노출해서는 안된다. 불변 필드라면 노출해도 덜 위험하지만
> 완전히 안심할 수는 없다. 하지만 package-private 클래스나 private 중첩 클래스에서는 종종
> 필드를 노출하는 편이 나을때도 있다.