## 2장 객체 생성과 파괴


------------------

#### 아이템 8 . finalizer 와 cleaner 사용을 피하라

자바의 두가지 소멸자 중 하나인 finalizer 는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.<br/>
오동작, 낮은 성능, 이식성 문제의 원인이 되기도 한다. 그래서 기본적으로 쓰지 말아야 한다. <br/>
자바 9 부터는 deprecated API 로 지정하고 cleaner 를 대안으로 소개했다. cleaner 는 finalize 보다는<br/> 
덜 위엄하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요 하다.<br/>
finalizer 와 cleaner 는 즉시 수행된다는 보장이 없다. 즉, <strong>finalizer 와 cleaner 로는 제때 실행되어야 하는 <br/>
작업은 절대 할 수 없다.</strong> 예컨대, 파일 닫기를 finalizer 와 cleaner 에 맡기면 중대한 오류가 날 수 있다.<br/>
시스템이 동시에 열 수 있는 파일 개수에 한계가 있기 때문이다. 즉 새로운 파일을 열지 못해 프로그램이 실패할 수 있다.<br/>
finalizer 와 cleaner 는 전적으로 가비지 컬렉터 알고리즘에 의해 결정되며 이는 가바지 컬렉터 구현마다 천차만별이다.<br/>
이 둘의 수행 시점에 의존하는 프로그램의 동작 또한 마찬가지다. <br/>

클래스에 finalizer 를 달아두면 그 인스턴스의 자원 회수가 제멋대로 지연될 수 있다. finalizer 스레드는 다른 <br/>
어플리케이션 스레드보다 우선순위가 낮아서 실행될 기회를 제대로 얻지 못한 것이다. 어떤 스레드가 finalizer 를 수행할지<br/>
명시하지 않으니 이 문제를 예방할 보편적인 해법은 없다. 딱 하나 사용하지 않는 방법뿐이다. 한편 cleaner 는 스레드를 제어<br/>
할수 있다는 면에서 조금 낫다. 하지만 여전히 백그라운드에서 수행되며 가비지 컬렉터의 통제하에 즉각 수행되리라는 보장이 없다.<br/>

접근할 수 없는 일부 객체에 딸린 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수도 있다. 따라서 <br/>
프로그램 생애주기와 상관없는, <strong>상태를 영구적으로 수정하는 작업에서는 절대 finalizer 나 cleaner 에 의존해서는 안된다.</strong><br/>
데이터베이스 같은 공유 자원의 영구 lock 해제를 finalizer 나 cleaner 에 맡겨 놓으면 분산 시스템 전체가 서서히 멈출것이다.<br/>

System.gc 나 System.runFinalization 메서드는 실행될 가능성을 높여줄뿐, 보장해주진 않는다. 사실 이를 보장해주겠다는 <br/>
메서드가 2개 있다. System.runFinalizersOnExit 와 Runtime.runFinalizerOnExit 이다. 하지만 이 두개의 메서드는 <br/>
심각한 결함 때문에 수십년간 지탄받아왔다. (ThreadStop)

finalizer 의 부작용은 여기서 끝이 아니다. finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.<br/>
잡지 못한 예외 때문에 해당 객체는 자칫 마무리가 덜 된 상태로 남을 수 있다. 거기에 다른 스레드가 이처럼 훼손된 객체를 사용하려 한다면 <br/>
어떻게 동작할지 예측할 수 없다. 보통의 경우엔 잡지 못한 예외가 스레드를 중단시키고 Stack trace 를 출력하지만, 같은 일이 finalizer <br/>
에서 일어난다면 경고조차 출력하지 않는다. 그나마 cleaner 는 자신의 스레드를 통제하기 때문에 이러한 문제가 발생하지 않는다.

finalizer 와 cleaner 는 심각한 성능 문제도 동반한다.가비지 컬렉터의 효율을 떨어뜨리기 때문이다.<br/>
finalizer 를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안문제를 일으킬 수 있다. 공격원리는 생성자나 직렬화 과정에서 <br/>
예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer 가 수행될 수 있게 된다. 이 finalizer 는 정적 필드에 자신의 <br/>
참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있다. 객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만, <br/>
finalizer 가 있다면 그렇지도 않다. final 클래스들은 하위 클래스를 만들수 없으니 공격에 안전한다. final 이 아닌 클래스를 finalizer <br/>
공격으로부터 방어하려면 아무 일도 하지 않는 finalizer 메서드를 만들고 final 로 선언한자.<br/>

파일이나 스레드 등 종료해야 할 자원을 담고 있는 객체의 클래스에 <strong>AutoCloseable</strong> 를 구현해주고 클라이언트에서 인스턴스를 <br/>
다 쓰고 다면 close 메서드를 호출하면 된다. 일반적으로는 try-with-resource 를 사용한다.

자바 라이브러리의 일부 클래스는 안전망 역할의 finalizer 를 제공한다. FileInputSteam, FileOutStream, ThreadPoolExecutor <br/>
가 대표적이다. cleaner 나 finalizer 가 즉시 호출되리라는 보장은 없지만, 자원 회수를 늦게라도 해주는것이 아예 안하는것보단 <br/>
낫기 때문이다. 이런 안전망 역할의 finalizer 를 작성할 때는 그럴만한 값어치가 있는지 생각하자.

적절히 활용하는 두번째 예는 네이티브 피어(native peer)와 연결된 객체에서다. <small>(네이티브 피어가 무엇인지 잘 이해가 안간다)</small><br/>
네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다. 네이티브 피어는 객체가 아니니 가비지 컬렉터는 <br/>
그 존재를 알지 못한다. 그 결과 피어를 회수할 때 네이티브 객체까지 회수하지 못한다. cleaner 나 finalizer 가 나서서 처리하기 적당한 작업이다.<br/>
단, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때에만 해당된다. 성능 저하를 감당할 수 없거나 네이티브 피어가<br/>
사용하는 자원을 즉시 회수해야 한다면 앞서 설명한 close 메서드를 사용해야한다.

cleaner 예제

```java
import java.lang.ref.Cleaner;

public class Room implements AutoCloseable {
    private static final Cleaner CLEANER = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room 을 참조 해서는 안된다.
    private static class State implements Runnable {
        int numJunkPiles;

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        @Override
        public void run() {
            System.out.println("방 청소");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleaner 과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = CLEANER.register(this, state);
    }

    @Override
    public void close() {
        cleanable.clean();
    }
}
```
System.exit 을 호출할때의 cleaner 동작은 구현하기 나름이지만 청소가 이뤄질지는 보장하지 않는다.
System.gc() 를 추가하는 것으로 종료전에 "방 청소" 를 출력 할수 있을지도 보장되지 않는다.

> 핵심정리
> cleaner(자바 8까지는 finalizer) 는 안정망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자. <br/>
> 물론 이런 경우라도 불확실성과 성능 저하에 주의 해야한다.