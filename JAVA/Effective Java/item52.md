> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 52. 다중정의는 신중히 사용하라

<br>

- [아이템 52. 다중정의는 신중히 사용하라](#아이템-52-다중정의는-신중히-사용하라)
  - [1 다중정의는 정적, 재정의는 동적으로 동작한다](#1-다중정의는-정적-재정의는-동적으로-동작한다)
    - [1-1 다중정의는 정적으로](#1-1-다중정의는-정적으로)
    - [1-2 재정의는 동적으로](#1-2-재정의는-동적으로)
  - [2 같은 수의 매개변수를 명확히 구분하는 방법](#2-같은-수의-매개변수를-명확히-구분하는-방법)
    - [2-1 가능한 개매변수 수가 같은 다중정의를 만들지 말자](#2-1-가능한-개매변수-수가-같은-다중정의를-만들지-말자)
    - [2-2 메서드 이름을 다르게 지어주는 방법도 있다](#2-2-메서드-이름을-다르게-지어주는-방법도-있다)
    - [2-3 생성자의 다중정의 경우는 정적 팩토리를 사용할 수도 있다](#2-3-생성자의-다중정의-경우는-정적-팩토리를-사용할-수도-있다)
    - [2-4 근본적으로 다른 매개변수 타입 사용](#2-4-근본적으로-다른-매개변수-타입-사용)
  - [3 다중정의에 혼란을 일으키는 예시](#3-다중정의에-혼란을-일으키는-예시)
    - [3-1 리스트 remove 예시](#3-1-리스트-remove-예시)
    - [3-2 람다와 메서드 참조](#3-2-람다와-메서드-참조)
  - [4 다중정의된 메서드가 완벽히 같은 로직을 수행할 땐 forward](#4-다중정의된-메서드가-완벽히-같은-로직을-수행할-땐-forward)
  - [핵심 정리](#핵심-정리)

<br>

## 1 다중정의는 정적, 재정의는 동적으로 동작한다

<br>

### 1-1 다중정의는 정적으로
```java
public class CollectionClassifier {

    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections) {
            System.out.println(classify(c));
        }
    }
}
// 결과
그 외
그 외
그 외
```
* 위 코드의 결과를 보면 `그 외`만 세번 출력된 것을 볼 수 있다.
  * **그 이유는 다중정의(Overroading)에서 어느 메서드를 호출할지가 컴파일타임에 정해지기 때문이다.**
  * 즉, **다중정의한 메서드중 무엇을 호출할지는 정적(컴파일타임에)으로 선택된다.**
* 위 코드에서 for문 안의 `c`는 항상 `Collection<?>`타입이다.
  * 런타임에는 타입이 매번 달라지지만, 호출할 메서드를 선택하는 데는 영향을 주지 못한다.

<br>

**굳이 사용하고싶다면**
```java
public static String classify(Collection<?> c) {
    return c instanceof Set ? "집합" :
            c instanceof List ? "리스트" : "그 외";
}
```
* 위와 같이 변경하면 처음 의도한대로 결과가 나오게 된다.
  
<br>

### 1-2 재정의는 동적으로
```java
class Wine {
    String name() { return "포도주"; }
}

class SparklingWine extends Wine {
    @Override String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
    @Override String name() { return "샴페인"; }
}

public class Overriding {

    public static void main(String[] args) {
        List<Wine> wineList = List.of(
            new Wine(), new SparklingWine(), new Champagne()
        );

        for (Wine wine : wineList) {
            System.out.println(wine.name());
        }
    }
}
// 결과
포도주
발포성 포도주
샴페인
```
* 예상한 것처럼 포도주, 발포성 포도주, 샴페인을 차례로 출력한다.
* **메서드를 재정의(Overriding)했다면 해당 객체의 런타임 타입이 어떤 메서드를 호출할지의 기준이 된다.**
  * 즉, **재정의한 메서드중 무엇을 호출할지는 동적(런타임에 타입에 의해)으로 선택된다.**

<br>

## 2 같은 수의 매개변수를 명확히 구분하는 방법

<br>

### 2-1 가능한 개매변수 수가 같은 다중정의를 만들지 말자
* 다중정의는 어떤 메서드를 사용할 지 컴파일 타임에 결정되며, **보통 프로그래머가 기대한 대로 동작하지 않는다.**
  * 그러므로 헷갈릴 수 있는 코드는 작성하지 않는게 좋다.
* **안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자**
  * 이 규칙만 지키면 어떤 다중정의 메서드가 호출될지 헷갈릴 일이 없다.

<br>

### 2-2 메서드 이름을 다르게 지어주는 방법도 있다
* **또 다른 방법으론 메서드 이름을 다르게 지어주는 방법도 있다.**
  * ex. `writeBoolean(boolean)`, `writeInt(int)` ...

<br>

### 2-3 생성자의 다중정의 경우는 정적 팩토리를 사용할 수도 있다
* **생성자는 이름을 다르게 지을 수 없으니 두 번째 생성자부터는 무조건 다중정의가 된다.**
  * 하지만 정적 팩토리라는 대안을 활용할 수 있는 경우가 많다. ([아이템 1](./item01.md))
* 그리고 가능하면 다중정의하는 매개변수의 수를 다르게 하자.

<br>

### 2-4 근본적으로 다른 매개변수 타입 사용
매개변수 수가 같은 다중정의 메서드가 많더라도, 그중 어느 것이 주어진 매개변수 집합을 처리할 지 명확히 구분하는 방법이있다!
* **바로 매개변수 중 하나 이상이 "근본적으로 다르다"면 헷갈릴 일이 없다.**
  * **근본적으로 다르다 : 두 타입의 값을 서로 어느쪽으로든 형변환 할 수 없다는 뜻이다.**
* **이렇게 서로 형변환이 안되는 두 타입을 매개변수로 두면 런타임에 호출을 결정하도록 보이게 만들 수 있다.**

## 3 다중정의에 혼란을 일으키는 예시

<br>

### 3-1 리스트 remove 예시
```java
public static void main(String[] args) {
    Set<Integer> set = new TreeSet<>();
    List<Integer> list = new ArrayList<>();

    for (int i = -3; i < 3; i++) {
        set.add(i);
        list.add(i);
    }

    for (int i = 0; i < 3; i++) {
        set.remove(i);
        list.remove(i);
    }
    System.out.println(set + " " + list);
}
// 결과
set : [-3, -2, -1] list : [-2, 0, 2]
```
* 똑같은 Collection의 `add`와 `remove`를 호출했지만, 런타임에 호출된 메서드가 달라서 결과가 다르게 나온다.
  * set -> `remove(Object o)`를 호출
  * list -> `remove(int idx)`를 호출
* 이 문제의 원인은 List가 `remove`를 다중 정의했기 때문이다.
  * `remove(Object)`, `remove(int)`
* 해결방법
  * 이 문제는 `list.remove`의 인수를 `Integer`로 형변환하여 올바른 다중정의 메서드를 선택하게 하면 해결된다.
    * `list.remove(Integer.valueOf(i))`

<br>

### 3-2 람다와 메서드 참조
```java
new Thread(System.out::println).start();

Executors.newCachedThreadPool().submit(System.out::println); // 컴파일 에러
```
* 2번만 컴파일 오류가 난다
  * 원인은 바로 submit 다중 정의 메서드 중에는 Callable<T>를 받는 메서드도 있다는 데 있다.
  * `<T> Future<T> submit(Callable<T> task);`
  * println이 void를 반환하고, 반환 값이 없는 callable과 헷갈릴 리는 없다고 생각하지만, 다중정의 알고리즘은 그렇게 동작하지 않는다고한다.
* **핵심은 다중정의된 메서드들이 함수형 인터페이스를 인수로 받을 때, 비록 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생긴다는 것.**
  * **서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다.**

<br>

## 4 다중정의된 메서드가 완벽히 같은 로직을 수행할 땐 forward
이번 아이템의 지침을 어긴 사례가 있다. 바로 String이다

String은 자바 4부터 `contentEquals(StringBuffer)`가 있었는데, 

자바 5에서 StringBuffer, StringBuilder, String, CharBuffer등의 비슷한 부류의 타입을 위한 공통 인터페이스 `CharSequence`가 등장하였고, 자연스럽게 String도 이 인터페이스를 `contentEquals`에 다중정의했다.

하지만 **다행히 이 두 메서드는 같은 객체를 입력하면 완전히 같은 작업을 수행해주니 해로울 건 전혀 없다.**

```java
public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence) sb);
} 
```
**위와 같이 더 특수한 다중정의 메서드에서 덜 특수한(일반적인) 다중정의 메서드로 일을 넘기면(forward) 된다.**

<br>

## 핵심 정리
* 일반적으로 매개변수 수가 같을 때는 다중정의를 피하는 게 좋다.
* **상황에 따라, 특히 생성자라면 이 조언을 따르기가 힘들다.**
  * **그럴땐, 헷갈릴 만한 매개변수는 형변환하여 정확한 다중정의 메서드가 선택되도록 해야한다. (근본적으로 다른 타입을 넘기는 방식)**
* **이것이 불가능하면,기존 클래스를 수정해 새로운 인터페이스를 구현해야 할 때는 같은 객체를 입력받는 다중정의 메서드들이 동일하게 동작하도록 해야한다. (forward)**
