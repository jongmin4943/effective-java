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