## 4장 클래스와 인터페이스

------------------

#### 아이템 21 . 인터페이스는 구현하는 쪽을 생각해 설계하라

자바 8 이후로 기존 인터페이스에 메서드를 추가할 수 있도록 디폴트 메서드가 소개되었다. 하지만 여기엔 위험이</br>
있다. 디폴트 메서드를 선언하면, 그 인터페이스를 구현한 후 디폴트 메서드를 재정의 하지 않은 모든 클래스에서</br>
디폴트 구현이 쓰이게 된다. 이는 모든 기존 구현체들과 매끄럽게 연동되리라는 보장이 없다. 디폴트 메서드는</br>
구현 클래스에 대해 아무것도 모른 채 합의없이 무작정 '삽입' 될 뿐이다.

자바 8 에서는 핵심 컬렉션 인터페이스들에 다수의 디폴트 메서드가 추가되었다. 자바 라이브러리의 디폴트</br>
메서드는 코드 품질이 높고 범용적이라 대부분 상황에서 잘 작동한다. 하지만 생각할 수 있는 모든 상황에서</br>
불변식을 해치지 않는 디폴트 메서드를 작성하기란 어려운법이다.

자바 8 의 Collection 인터페이스에 추가된 removeIf 메서드를 예로 생각해보자. 이 메서드는 주어진 불리언</br>
함수(Predicate)가 true 를 반환하는 모든 원소를 제거한다. 디폴트 구현은 반복자를 이용해 순회하면서 각</br>
원소를 인수로 넣어 Predicate 를 호출하고 true 를 반환하면 반복자의 remove 메서드를 호출해 그 원소를 제거한다.</br>
```java
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
        if (filter.test(each.next())) {
            each.remove();
            removed = true;
        }
    }
    return removed;
}
```
이 코드보다 범용적으로 구현하기도 어렵겠지만, 그렇다고 해서 현존하는 모든 Collection 구현체와 잘 </br>
어우러지는것은 아니다. 대표적인 예가 SynchronizedCollection 이다. 아파치 커먼즈 라이브러리의 이 클래스</br>
는 java.util 의 Collections.synchronizedCollection 정적 팩터리 메서드가 반환하는 클래스와</br>
비슷하다. 아파치 버전은 클라이언트가 제공한 객체로 락을 거는 능력을 추가로 제공한다. 즉, 모든 메서드에서</br>
주어진 락 객체로 동기화한 후 내부 컬렉션에 기능을 위임하는 래퍼 클래스다.

아파치의 SynchronizedCollection 클래스는 지금도 활발히 관리되고 있지만, 책이 쓰이던 시점엔 removeIf</br>
메서드를 재정의하고 있지 않다. 그러므로 자신이 한 약속을 더 이상 지키지 못하게 된다. 다시 말해 모든</br>
메서드 호출을 알아서 동기화해주지 못한다. removeIf 의 구현은 동기화에 관해 아무것도 모르므로 락 객체를</br>
사용할 수 없다. 따라서 SynchronizedCollection 인스턴스를 여러 스레드가 공유하는 환경에서 한 스레드가</br>
removeIf 를 호출하면 ConcurrentModificationException 이 발생하거나 예기치 못한 결과로 이어질 수 있다.

자바 플랫폼 라이브러리에서도 이런 문제를 예방하기 위해 일련의 조치를 취했다. 예를 들어 구현한 인터페이스의</br>
디폴트 메서드를 재정의하고, 다른 메서드에서는 디폴트 메서드를 호출하기 전에 필요한 작업을 수행하도록 했다.</br>
예컨대 Collections.synchronizedCollection 이 반환하는 package-private 클래스들은 removeIf 를</br>
재정의하고, 이를 호출하는 다른 메서드들은 디폴트 구현을 호출하기 전에 동기화를 하도록 했다. 하지만 자바</br>
플랫폼에 속하지 않은 제 3의 기존 콜렉션 구현체들은 이런 언어 차원의 인터페이스 변화에 수정될 기회가 없었으며,</br>
그 중 일부는 여전히 수정되지 않고 있다.

디폴트 메서드는 컴파일에 성공하더라도 기존 구현체에 런타임 오류를 일으킬 수 있다. 그러므로 기존 인터페이스에</br>
디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야한다. 추가하려는 디폴트 메서드가</br>
기존 구현체들과 충돌하지는 않을까 심사숙고해야 함도 당연하다. 반면, 새로운 인터페이스들을 만드는 경우라면</br>
표준적인 메서드 구현을 제공하는데 아주 유용한 수단이며, 그 인터페이스를 더 쉽게 구현해 활용할 수 있게끔 해준다.

한편, 디폴트 메서드는 인터페이스로부터 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도가 아님을</br>
명심해야 한다. 이런 형태로 인터페이스를 변경하면 반드시 기존 클라이언트를 망가뜨리게 된다.

핵심은 명백하다. 디폴트 메서드라는 도구가 생겼더라도 인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야</br>
한다. 디폴트 메서드로 기존 인터페이스에 새로운 메서드를 추가하면 커다란 위험도 딸려온다. 인터페이스에 내재된</br>
작은 결함도 사용자 입장에서는 짜증나는 일인데, 심각하게 잘못된 인터페이스라면 이를 포함한 API 에 어떤</br>
재앙을 몰고 올지 알 수 없다.

새로운 인터페이스라면 릴리스 전에 반드시 테스트를 거쳐야한다. 수많은 개발자가 그 인터페이스를 나름의 방식으로</br>
구현할 것이니, 여러분도 서로 다른 방식으로 최소한 세가지는 구현해봐야 한다. 또한 각 인터페이스의 인스턴스를</br>
다양한 작업에 활용하는 클라이언트도 여러 개 만들어봐야한다. 새 인터페이스가 의도한 용도에 잘 부합하는지를</br>
확인하는 길은 이처럼 험난하다. 이런 작업들을 거치면 여러분은 인터페이스를 릴리스하기 전에, 즉 바로잡을 기회가</br>
아직 남았을 때 결함을 찾아낼 수 있다. 인터페이스를 릴리스한 후라도 결함을 수정하는게 가능한 경우도 있겠지만,</br>
절대 그 가능성에 기대서는 안된다.