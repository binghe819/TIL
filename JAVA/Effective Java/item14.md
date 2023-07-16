> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 14. Comparable을 구현할지 고려하라
이번 아이템에서는 Comparable에 대해서 다룬다.

<br>

- [아이템 14. Comparable을 구현할지 고려하라](#아이템-14-comparable을-구현할지-고려하라)
  - [Comparable 인터페이스](#comparable-인터페이스)
    - [Comparable 예시](#comparable-예시)
  - [Comparable 규약](#comparable-규약)
    - [반사성, 대칭성, 추이성](#반사성-대칭성-추이성)
    - [마지막 규약](#마지막-규약)
  - [compareTo 작성 요령](#compareto-작성-요령)
    - [compareTo는 컴파일타임에 타입이 정해진다](#compareto는-컴파일타임에-타입이-정해진다)
    - [compareTo 예시](#compareto-예시)
    - [핵심 필드가 여러개일때](#핵심-필드가-여러개일때)
  - [Comparator 비교자 생성 메서드](#comparator-비교자-생성-메서드)
    - [숫자 타입](#숫자-타입)
    - [객체 참조](#객체-참조)
    - [Comparator 작성 요령](#comparator-작성-요령)
  - [결론](#결론)

<br>

## Comparable 인터페이스

```java
public interface Comparable<T> {
  		public int compareTo(T o);
}
```

자바에서는 같은 타입의 인스턴스를 서로 비교해야만 하는 클래스들은 모두 `Comparable` 인터페이스를 구현하고 있다.

**`Comparable`은 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다.**

`Comparable`을 구현했다는 것은 그 클래스의 인스턴스들에는 **자연적인 순서**가 있음을 뜻한다.

<br>

### Comparable 예시
```java
// 배열 정렬
Arrays.sort(a); // a라는 배열이 Comparable이 구현되어 있다면 쉽게 정렬가능하다.

// TreeXXX은 정렬된 상태로 값을 저장한다 (String이 Comparable이 구현되어있기때문에 자동적으로 정렬된다)
Set<String> s = new TreeSet<>();
Collections.addAll(s, args);
System.out.println(s);
```
* 이외에도 자바 플랫폼 라이브러리 대부분의 값 클래스와 열거 타입은 `Comparable`이 구현되어있다.

<br>

## Comparable 규약

`Comparable`은 기본적으로 객체의 순서를 비교하여 정렬할 때 사용된다. 그리고 객체의 순서를 비교하는 방식은 아래와 같다.

* 이 객체와 주어진 객체의 순서를 비교한다.
  * 이 객체가 주어진 객체보다 작으면 : 음수
  * 이 객체가 주어진 객체와 같으면 : 0
  * 이 객체가 주어진 객체보다 크면 : 양수
  * 이 객체와 비교할 수 없는 타입의 객체가 주어지면 `ClassCastException`

<br>

### 반사성, 대칭성, 추이성

규약중 3가지는 `equals`의 반사성, 대칭성, 추이성과 비슷하다.

그러므로 기존 클래스를 확장한 구체 클래스에서 새로운 변수를 추가했을때의 주의사항도 [아이템 10](./item10.me)의 `equals`와 같다.

* 반사성 : 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.
  * 첫 번째 객체가 두 번째 객체보다 작으면, 두 번째가 첫 번째보다 커야 한다. (a1 > a2, a2 > a1)
  * 첫 번째가 두 번째와 크기가 같다면, 두 번째는 첫 번째와 같아야 한다. (a1 == a2, a2 == a1)
  * 첫 번째 객체가 두 번째 객체보다 크면, 두 번째가 첫 번째보다 작아야 한다. (a1 > a2, a2 < a1)
* 대칭성 : 세 객체 참조의 다음과 같은 결과가 나와야 한다.
  * 첫 번째 객체가 두 번째 객체보다 크고, 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야한다.
* 추이성 : 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.
  * 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째와 세 번째 객체가 같아야한다. (크다, 작다 모두 포함)

**`equals`와 다른 점이라면, `compareTo`는 타입이 다른 객체를 신경 쓰지 않는다는 것이다. 만약 서로 다른 타입이면 `ClassCastException`을 던지면 되기때문이다.**

<br>

### 마지막 규약

마지막 규약은 필수는 아니지만 꼭 지키는게 좋다고 한다. 바로 `(x.compareTo(y) == 0) == (x.equals(y))`다.

실제로 정렬된 컬렉션들은 동치성을 비교할 때 `equals`대신 `compareTo`를 사용한다.

즉, **`compareTo`에서 같다라고 한 두 객체가 `equals`를 수행하면 `true`가 나와야 한다는 의미이다.**

<br>

## compareTo 작성 요령

<br>

### compareTo는 컴파일타임에 타입이 정해진다
`compareTo`의 작성 요령은 `equals`와 비슷하다.

하지만 `Comparable` 자체가 제너릭을 이용하므로, `compareTo`의 인수 타입은 컴파일타임에 정해진다.

```java
public class Comparable<T> {
    public int compareTo(T o);
}
```

그러므로 **입력 인수의 타입을 확인하거나 형변환할 필요가 없다.**

<br>

### compareTo 예시
```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
    ...
}
```
**`CaseInsensitiveString`이 `Compare<CaseInsensitiveString>`을 구현한 것을 주목하자.**

CaseInsensitiveString의 참조는 CaseInsensitiveString 참고와만 비교할 수 있다는 의미이며, Comparable을 구현할 때 일반적으로 사용하는 패턴이다.

<br>

### 핵심 필드가 여러개일때
클래스의 핵심 필드가 여러 개라면 어느 것을 먼저 비교하느냐가 중요해진다.

이때는 **가장 핵심적인 필드부터 비교해나간다.**

```java
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);   // 가장 중요한 필드
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);       // 두 번째로 중요한 필드
        if (result == 0) {
            result = Short.compare(lineNum, pn.lineNum); // 세 번째로 중요한 필드
        }
    }
    return result;
}
```

<br>

## Comparator 비교자 생성 메서드

자바 8에서는 `Comparator` 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.

**이 방식은 간결하지만 약간의 성능 저하가 뒤따른다고 한다. (실제 저자는 10% 정도 느려졌다고 한다.)**

<br>

### 숫자 타입

```java
private static final Comparator<PhoneNumber> COMPARTOR = 
    comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```
`comparingInt`이외에도 `long`, `double`등 타입마다 모두 구현되어있다.

<br>

### 객체 참조
객체 참조용 비교자 생성 메서드도 준비되어 있다.
자세한 내용은 `Comparator` 인터페이스 코드를 보면 이해할 수 있다.

<br>

### Comparator 작성 요령
**값의 차를 이용하면 안된다**
```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode(); // '값의 차' 방법은 안좋다.
    }
}
```
* 정수 오버플로를 일으키거나 부동소수점 계산 방식에 따른 오류를 낼 수 있기 때문이다.

<br>

**다음 두 가지 방법은 사용하면 된다**
```java
// 정적 compare 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
```

<br>

```java
static Comparator<Object> hashCodeOrder = 
    Comparator.comparingInt(o -> o.hashCode());
```

<br>

## 결론
* 순서를 고혀해야 하는 값 클래스를 작성한다면 꼭 `Comparable`를 구현하자.
* `compareTo` 메서드에서 필드 값을 비교할 때 `<`와 `>` 연산자는 쓰지 말아야한다.
  * 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 `compare` 메서드나 `Comparator` 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.
    * `Double.compare` ...