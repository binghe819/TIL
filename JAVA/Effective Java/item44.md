> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 44. 표준 함수형 인터페이스를 사용하라

<br>

- [아이템 44. 표준 함수형 인터페이스를 사용하라](#아이템-44-표준-함수형-인터페이스를-사용하라)
  - [1 람다가 지원하면서](#1-람다가-지원하면서)
  - [2 표준 함수형 인터페이스](#2-표준-함수형-인터페이스)
  - [3 함수형을 직접 작성해야 할 때는?](#3-함수형을-직접-작성해야-할-때는)
    - [3-1 표준 인터페이스가 없을때](#3-1-표준-인터페이스가-없을때)
    - [3-2 구조적으로 같아도 직접 작성해야할 때가 있다](#3-2-구조적으로-같아도-직접-작성해야할-때가-있다)
  - [4 함수형 인터페이스 주의점](#4-함수형-인터페이스-주의점)
  - [핵심 정리](#핵심-정리)

<br>

## 1 람다가 지원하면서
* 자바에 람다가 지원하면서
  * 템플릿 메서드 패턴의 매력이 크게 줄었다. 
  * **이를 대처하는 해법은 같은 효과의 함수 객체를 받는 정적 팩토리나 생성자를 제공하는 것이다.**
    * 즉, 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 한다.
    * 이때 함수형 매개변수 타입을 올바르게 선택해야 한다.

<br>

**예시**
```java
// removeEldestEntry를 재정의하는 예시
LinkedHashMap<String, String> map = new LinkedHashMap<>() {
    @Override
    protected boolean removeEldestEntry(Entry<String, String> eldest) {
        return size() > 2; // 가장 최근 원소 2개를 유지한다.
    }
};

map.put("1", "a");
map.put("2", "b");
map.put("3", "c");
map.put("4", "d");

System.out.println(map); // {3=c, 4=d}         a,b 가 없어진 것을 볼 수 있다.
```
* 맵은 `put`을 할 때 `removeEldestEntry`를 호출한다고 한다. 그리고 `true`가 반환되면 가장 오래된 원소를 제거한다.
* **재정의한 `removeEldestEntry`는 size 메서드를 호출하는데, 이는 인스턴스 메서드라 가능하다.**
  * **하지만 팩터리나 생성자를 호출할 때는 Map의 인스턴스가 존재하지 않아 Map 자신도 함수 객체에 넘겨주어야 한다.** 이를 함수형 인터페이스로 선언하면 아래와 같다.
```java
@FunctionInterface
interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}
```

<br>

## 2 표준 함수형 인터페이스
**필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하는 것이 좋다.**

`java.util.function` 패키지에는 총 43개의 인터페이스가 담겨있다.

그 중 기본 인터페이스 6개만 외우면 나머진 충분히 유추해낼 수 있다.


인터페이스 | 함수 | 의미 | 예
---------|----------|---------|---------
 `UnaryOperator<T>` | `T apply(T t)` | 반환값과 인수의 타입이 같은 함수, 인수는 1개 | `String::toLowerCase`
 `BinaryOperator<T>` | `T apply(T t1, T t2)` | 반환값과 인수의 타입이 같은 함수, 인수는 2개 | `BigInteger::add`
 `Predicate<T>` | `boolean test(T t)` | 한 개의 인수를 받아서 boolean을 반환하는 함수 | `Collection::isEmpty`
 `Function<T, R>` | `R apply(T t)` | 인수와 반환 타입이 다른 함수 | `Arrays::asList`
 `Supplier<T>` | `T get()` | 인수를 받지 않고 값을 반환, 제공하는 함수 | `Instant::now`
 `Consumer<T>` | `void accept(T t)` | 한 개의 인수를 받고 반환값이 없는 함수 | `System.out::println`
* 항
  * `Unary` : 단항의
  * `Binary` : 이항의
* 기본 타입 (int, long, double) 예시
  * `IntPredicate` : int를 받는 Predicate
  * `LongBinaryOperator` : long을 받환하는 BinaryOperator
  * `LongFunction<int[]>` : long 인수를 받아 int[]를 반환
* Function의 변형 (`SrcToResult`)
  * `LongToIntFunction` : long을 받아 int를 반환
    * `int applyAsInt(long value)`
* 인수를 2개씩 받는 변형
  * `BiPredicate<T, U>` : `boolean test(T t, U u)`
  * `BiFunction<T, U, R>` : `R apply(T t, U u)`
  * `BiConsumer<T, U>` : `void accept(T t, U u)`

> 이외에도 너무 많다. 필요할 때 찾아 사용하자.

<br>

**주의 할 점**

**대부분 표준 함수형 인터페이스는 기본 타입만 지원한다.**
 
**그렇다고 박싱된 기본 타입을 넣어 사용하면 안 된다.** 동작은 하겠지만 계산이 많아지는 경우 성능이 매우 느려질 수 있다. 물론 필요한 용도에 맞는 게 없다면 직접 구현하면 된다.

<br>

## 3 함수형을 직접 작성해야 할 때는?

<br>

### 3-1 표준 인터페이스가 없을때
* 표준 인터페이스 중 필요한 용도에 맞는 게 없다면 직접 작성해야 한다.
  * 예를 들어 매개변수 3개를 받는 Predicate, 혹은 검사 예외를 던지는 경우

<br>

### 3-2 구조적으로 같아도 직접 작성해야할 때가 있다
구조적으로 똑같은 표준 함수형 인터페이스가 있더라도 직접 작성해야만 할 때가 있다.
```java
// Comparator<T> 인터페이스는 구조적으로 ToIntBiFunction<T,U>와 동일하다.
// Comparator
@FunctionInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}

// ToIntBiFunction
@FunctionalInterface
public interface ToIntBiFunction<T, U> {
    int applyAsInt(T t, U u);
}
```
Comparator가 먼저 등장하긴 했지만, 그래도 표준형 함수형 인터페이스인 ToIntBiFunction을 사용하지 않았을 것이다. 

이유는 다음과 같다. 
1. 이름이 용도를 아주 훌륭히 설명해준다.
2. 구현하는 쪽에서 반드시 지켜야 할 규약을 담고 있다.
3. 비교자들을 변환하고 조합해주는 유용한 디폴트 메서드들을 가득 담고 있다.

**이처럼 3가지 중 하나 이상의 이유가 있다면 직접 함수형 인터페이스를 구현할지 고민해도 좋다.**

<br>

## 4 함수형 인터페이스 주의점
**서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중정의해서는 안 된다.**

클라이언트에게 모호함을 주며 문제가 발생할 소지가 많다.

```java
public interface extends Executor {
    ...
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
}
```
* `submit`은 `Callable<T>` 받는 것과 `Runnable` 받는 것을 다중정의했다.
  * 그래서 **호출하는 입장에서 올바른 메서드를 알려주기 위해 형변환해야 할 때가 왕왕 생긴다.**
* 이 문제를 해결하는 방법은
  * **서로 다른 함수형 인터페이스를 같은 위치의 인수로 사용하는 다중정의를 피하는 것이다.**

<br>

## 핵심 정리
* 함수형 인터페이스를 최대한 활용하라
* `java.util.function`패키지의 표준 함수형 인터페이스를 사용하는 것이 가장 좋은 선택이다.
* 단, 흔치는 않지만 직접 새로운 함수형 인터페이스를 만들어 쓰는 편이 나을 수도 있다.