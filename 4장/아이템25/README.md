## 4장 클래스와 인터페이스

------------------

#### 아이템 25 . 톱레벨 클래스는 한 파일에 하나만 담으라

소스 파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 불평하지 않는다. 하지만 아무런 득이<br/>
없을뿐더러 심각한 위험을 감수해야 하는 행위다. 이렇게 하면 한 클래스를 여러 가지로 정의 할 수 있으며, 그중<br/>
어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하냐에 따라 달라지기 때문이다. 다음 소스파일은 Main 클래스<br/>
하나를 담고 있고, Main 클래스는 다른 톱레벨 클래스 2개를 참조한다.
```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```
Utensil 과 Dessert 클래스가 Utensil.java 라는 한 파일에 정의 되어 있다고 해보자
```java
// Utensil.java
class Utensil {
    static final String NAME = "pan";
}
class Dessert {
    static final String NAME = "cake";
}
```
물론 Main 을 실행하면 pancake 를 출력한다. 이제 우연히 똑같은 두 클래스를 담은 Dessert.java 라는<br/>
파일을 만들었다고 해보자.
```java
// Dessert.java
class Utensil {
    static final String NAME = "pot";
}
class Dessert {
    static final String NAME = "pie";
}
```
운 좋게 javac Main.java Dessert.java 명령으로 컴파일 한다면 컴파일 오류가 나고 Utensil 과 Dessert<br/>
클래스를 중복 정의했다고 알려줄 것이다. 컴파일러는 가장 먼저 Main.java 를 컴파일 하고, 그 안에서<br/>
(Dessert 참조보다 먼저 나오는) Utensil 참조를 만나면 Utensil.jva 파일을 살펴 Utensil 과 Dessert 를<br/>
모두 찾아낼 것이다. 그런 다음 컴파일러가 두번째 명령줄 인수로 넘어온 Dessert.java 를 처리하려 할 때 같은<br/>
클래스의 정의가 이미 있음을 알게 된다.

한편, javac Main.java 나 javac Main.java Utensil.java 명령으로 컴파일하면 Dessert.java 파일을<br/>
작성하기 전처럼 pancake 를 출력한다. 그러나 javac Dessert.java Main.java 명령으로 컴파일하면<br/>
potpie 를 출력한다. 이처럼 컴파일러에 어느 소스파일을 먼저 건네느냐에 따라 동작이 달라지므로 반드시<br/>
바로 잡아야 할 문제다.

다행히 해결책은 아주 간단하다. 단순히 톱레벨 클래스들을 서로 다른 소스 파일로 분리하면 그만이다. 굳이<br/>
여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용하는 방법을 고민해볼수 있다. 다른 클래스에<br/>
딸린 부차적인 클래스라면 정적 멤버 클래스로 만드는 쪽이 일반적으로 더 나을 것이다. 읽기 좋고, private<br/>
으로 선언하면 접근 범위도 최소로 관리할 수 있기 때문이다. 다음 코드는 앞의 예를 정적 멤버 클래스로 바꿔봤다.
```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
    
    private static class Utensil {
        static final String NAME = "pan";
    }
    private static class Dessert {
        static final String NAME = "cake";
    }
}
```
> 핵심정리<br/>
> 소스 파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담자. 이 규칙을 따른다면
> 컴파일러가 한 클래스에 대한 정의를 여러개 만들어내는 일은 사라진다. 소스 파일을 어떤 순서로 컴파일하든
> 바이너리 파일이라 프로그램의 동작이 달라지는 일은 결코 일어나지 않을 것이다.