> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 30. 이왕이면 제네릭 메서드로 만들라

- [아이템 30. 이왕이면 제네릭 메서드로 만들라](#아이템-30-이왕이면-제네릭-메서드로-만들라)
  - [제네릭 메서드가 필요한 이유](#제네릭-메서드가-필요한-이유)
  - [활용 - 제네릭 싱글턴 팩토리 패턴](#활용---제네릭-싱글턴-팩토리-패턴)
  - [활용 - 재귀적 타입 한정](#활용---재귀적-타입-한정)
  - [핵심 정리](#핵심-정리)

<br>

## 제네릭 메서드가 필요한 이유
제네릭 메서드를 통해 함수의 타입 안정성을 보장할 수 있다.

```java
// s1과 s2이 같은 타입이라는 것을 증명할 수 없다.
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```
매개변수화 타입 (제네릭)을 사용하지 않고, **로타입으로 메서드를 만들면 데이터 타입이 일치하지 않아 런타임 에러가 발생할 확률이 높다.**

제네릭 메서드로 수정하여 타입안정성을 보장해야한다.

```java
// 제네릭 메서드 (3개의 Set의 타입이 같음을 보장한다.)
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>();
    result.addAll(s2);
    return result;
```

<br>

## 활용 - 제네릭 싱글턴 팩토리 패턴
때때로 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있는데, 이때는 제네릭 싱글톤 팩토리를 만들면 된다. 
`Collections.reverseOrder`, `Collections.emptySet`이 좋은 예제이다.

```java
@SuppressWarnings("unchecked")
public static <T> Comparator<T> reverseOrder() {
    return (Comparator<T>) ReverseComparator.REVERSE_ORDER;
}
```
**만약 제네릭을 쓰지 않았다면 요청 타입마다 형변환하는 정적 팩토리를 만들었어야 할 것이다.**

**타입별로 정적메서드를 하나씩 만들어줘야한다..**

<br>

## 활용 - 재귀적 타입 한정
재귀적 타입 한정이라는 개념을 이용하면 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수도 있습니다. 

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어 있습니다.");
    
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result Object.requireNonNull(e);
    
    return result;
}
```
타입 한정인 `<E extends Comparable<E>>`는 "모든 타입 E는 자기자신과 같은 타입인 원소 모두와 비교할 수 있다"라고 해석할 수 있습니다.

<br>

## 핵심 정리
* 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다.
