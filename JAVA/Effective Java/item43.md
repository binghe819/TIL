> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 43. 람다보다는 메서드 참조를 사용하라

<br>

- [아이템 43. 람다보다는 메서드 참조를 사용하라](#아이템-43-람다보다는-메서드-참조를-사용하라)
  - [1 메서드 참조는 람다보다 간결하다.](#1-메서드-참조는-람다보다-간결하다)
    - [1-1 메서드 참조란?](#1-1-메서드-참조란)
    - [1-2 때론 람다가 더 간결할 때가 있다.](#1-2-때론-람다가-더-간결할-때가-있다)
  - [2 메서드 참조의 유형](#2-메서드-참조의-유형)
  - [핵심 정리](#핵심-정리)

<br>

## 1 메서드 참조는 람다보다 간결하다.

<br>

### 1-1 메서드 참조란?
```java
클래스이름::메서드이름
참조변수이름::메서드이름
```
* 메서드 참조란
  * 람다 표현식이 단 **하나의 메서드만을 호출하는 경우**에 해당 람다 표현식에서 **불필요한 매개변수를 제거**하고 사용할 수 있게 해준다.

<br>

```java
// 예시 1
map.merge(key, 1, (count, incr) -> count + incr); // 람다 표현식
map.merge(key, 1, Integer::sum);                  // 메서드 참조

// 예시 2
(base, exponent) -> Math.pow(base, exponent); // 람다 표현식
Math::pow;                                    // 메서드 참조

// 예시 3
MyClass obj = new MyClass;
Function<String, Boolean> func = (a) -> obj.equals(a); // 람다 표현식
Function<String, Boolean> func = obj::equals;          // 메서드 참조

// 예시 4
DoubleUnaryOperator oper;

oper = (n) -> Math.abs(n);                  // 람다 표현식
System.out.println(oper.applyAsDouble(-5)); // 5.0

oper = Math::abs;                           // 메서드 참조
System.out.println(oper.applyAsDouble(-5)); // 5.0
```

<br>

### 1-2 때론 람다가 더 간결할 때가 있다.
**메서드와 람다가 같은 클래스에 있을 땐** 람다 표현식이 더 간결할 때가 있다.

```java
// 다음 코드는 GoshThisClassNameIsHumongous 클래스 안에 있다고 가정한다.
service.execute(GoshThisClassNameIsHumongous::action); // 메서드 참조
service.execute(() -> action());                       // 람다 표현식
```
* 메서드 참조 쪽은 더 짧지도, 더 명확하지도 않다. 따라서 람다 쪽이 낫다.

<br>

## 2 메서드 참조의 유형

메서드 참조 유형 | 예 | 같은 기능을 하는 람다
---------|----------|---------
 정적 | `Integer::parseInt` | `str -> Integer.parseInt(str)`
 한정적(인스턴스) | `Instant.now()::isAfter` | `Instant then = Instant.now();`<br>`t -> then.isAfter(t)`
 비한정적(인스턴스) | `String::toLowerCase` | `str -> str.toLowerCase()`
 클래스 생성자 | `TreeMap<K, V>::new` | `() -> new TreeMap<K, V>()`
 배열 생성자 | `int[]::new` | `len -> new int[len]`

* 정적 메서드 참조
  * 가장 흔한 메서드 참조 유형
* 인스턴스 메서드 참조
  * 한정적 : 수신 객체(참조 대상 인스턴스)를 특정하는 메서드 참조
    * 정적 메서드 참조와 비슷하다.
    * 즉, 함수 객체가 받는 인수와 참조되는 메서드가 받은 인수가 똑같다. (Input, Output이 같음)
  * 비한정적 : 수신 객체(참조 대상 인스턴스)를 특정하지 않는 메서드 참조
    * 함수 객체를 적용하는 시점에 수신 객체를 알려준다.
    * 주로 스트림 파이프라인에서 매핑과 필터 함수로 쓰인다.

<br>

## 핵심 정리
* 메서드 참조는 람다의 간단명료한 대안이 될 수 있다.
* 메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하자.