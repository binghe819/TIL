# 목차

- [목차](#목차)
- [Optional](#optional)
  - [1 Optional이 나오기 전엔](#1-optional이-나오기-전엔)
    - [1-1 null의 늪](#1-1-null의-늪)
    - [1-2 NPE(NullPointerException)](#1-2-npenullpointerexception)
  - [2 Optional이란](#2-optional이란)
  - [3 Optional을 만든 의도](#3-optional을-만든-의도)
  - [4 Optional의 사용법](#4-optional의-사용법)
    - [4-1 Optional 객체 생성](#4-1-optional-객체-생성)
    - [4-2 Optional 객체에 접근](#4-2-optional-객체에-접근)
      - [get(), isPresent()](#get-ispresent)
      - [orElse(), orElseGet(), orElseThrow()](#orelse-orelseget-orelsethrow)
  - [5 Optional을 Optional답게 사용하자](#5-optional을-optional답게-사용하자)
    - [5-1 Optional의 잘못된 사용](#5-1-optional의-잘못된-사용)
    - [5-2 Stream처럼 사용하기](#5-2-stream처럼-사용하기)
      - [map](#map)
      - [filter](#filter)
    - [5-3 이외의 Optional 바르게 쓰기](#5-3-이외의-optional-바르게-쓰기)
  - [6 자바9에서 추가된 기능](#6-자바9에서-추가된-기능)
    - [6-1 or()](#6-1-or)
    - [6-2 ifPresentOrElse()](#6-2-ifpresentorelse)
- [참조](#참조)

<br>

# Optional

<br>

## 1 Optional이 나오기 전엔
> 이번 챕터는 https://www.daleseo.com/java8-optional-before/ 글 내용을 정리한 글입니다.

<br>

### 1-1 null의 늪
* `null` 개념은 1965년 Tony Hoare라는 영국 컴퓨터 과학자에 의해 처음 고안되었다.
* 당시 "존재하지 않는 값"을 표현할 수 있는 방법이 null이라고 생각함.
* **하지만 나중에 "10억불 짜리 큰 실수"였고, null 참조를 만든 것을 후회한다고 토로하였다고한다.**

<br>

### 1-2 NPE(NullPointerException)
NPE는 지뢰같은 녀석이다. 경력과 상관없이 누구든 마주칠 수 있는 예외이다.

<br>

```java
// 주문을 한 회원이 살고 있는 도시를 반환한다.
public String getCityOfMemberFromOrder(Order order) {
  return order.getMember().getAddress().getCity();
}
```
위 코드의 문제는 매개변수 `order`과 `getXXX()` 메서드마다 모두 NPE가 발생할 가능성이 있다는 것이다.

<br>

**전통적인 NPE 방어 패턴**
1. 중첩 null 체크
   ```java
   if (order != null)
      Member member = order.getMember();
      if (member != null) {
          Address address = member.getAddress();
          if (address != null) {
              String city = address.getCity();
              if (city != null) 
                  return city;
          }
      }
   ```
   * 보기만해도 끔찍하다..
2. 사방에서 return하기
   ```java
   if (order == null) {
       return "Seoul";
   }
   Member member = order.getMember();
   if (member == null) {
       return "Seoul";
   }
   ...
   ```
   * 이것도 끔찍하다..

그저 `어떤 주문을 한 회원이 어느 도시에 살고 있는지 알려주는 기능`을 구현하는데 핵심 로직보다는 `null` 처리가 더 복잡해지는 상황이 온다.

<br>

## 2 Optional이란

> `null` 값도 포함할 수 있는 컨테이너 객체.
> 
> "존재할 수도 있지만 안 할 수도 있는 객체"

자바 8부터 도입된 `java.util.Optional<T>` 클래스는 `Integer` 나 `Double` 클래스처럼 `T` 타입의 객체를 포장해주는 **래퍼 클래스**이다.

`Optional` 객체를 사용하면 예상치 못한 `NullPointException` 예외를 제공되는 메서드로 간단히 회피할 수 있다.

즉, 복잡한 조건문 없이도 `null` 값으로 인해 발생하는 예외를 처리할 수 있게 된다.

<br>

## 3 Optional을 만든 의도

`Optional` is primarily intended for use as a method return type where there is a clear need to represent "no result," and where using `null` is likely to cause errors. A variable whose type is `Optional` should never itself be `null`; it should always point to an `Optional` instance.

**메서드가 반환할 결과값이 ‘없음’을 명백하게 표현할 필요가 있고, `null`을 반환하면 에러를 유발할 가능성이 높은 상황에서 메서드의 반환 타입으로 `Optional`을 사용하자는 것이 `Optional`을 만든 주된 목적이다.** `Optional` 타입의 변수의 값은 절대 `null`이어서는 안 되며, 항상 `Optional` 인스턴스를 가리켜야 한다.

원래 의도에 맞게 사용해야지 기존의 시스템에 해를 끼치지 않게 된다.

<br>

## 4 Optional의 사용법

<br>

### 4-1 Optional 객체 생성

```java
// of
public static <T> Optional<T> of(T value) { return new Optional<>(value);}

// ofNullable
public static <T> Optional<T> ofNullable(T value) { return value == null ? empty() : of(value);}

// empty()
private static final Optional<?> EMPTY = new Optional<>();
public static<T> Optional<T> empty() {
    Optional<T> t = (Optional<T>) EMPTY;
    return t;
}
```

`of()` 나 `ofNullable()` 메서드를 사용하여 `Optional`을 생성할 수 있다.

* `of()`
  * `null` 이 아닌 명시된 값을 가지는 `Optional` 객체를 반환한다.
  * 만약 `null` 이 저장되면 `NPE` 예외가 발생한다.
* `ofNullable()`
  * `null` 이 될 가능성이 있다면, `ofNullable()` 을 사용한다.
  * `null` 이면 비어있는 `Optional` 객체를 반환한다. `NPE` 를 방지할 수 있다.
* `empty()`
  * `null`을 담고 있는 `Optional` 객체를 얻어온다.

```java
Optional<String> test = Optional.ofNullable("Optional 테스트");
System.out.println(test.get());

Optional<User> maybeUser = Optional.empty();
```

<br>

### 4-2 Optional 객체에 접근

#### get(), isPresent()

```java
// get
public T get() {
  if (value == null) {
    throw new NoSuchElementException("No value present");
  }
  return value;
}

// isPresent
public boolean isPresent() { return value != null;}
```

* `get()`
  * `Optional` 객체에 저장된 값에 접근할 수 있다.
  * 만약 `null` 값이라면 `NoSuchElementException` 을 발생시킨다.
* `isPresent()`
  * `null` 인지 아닌지 확인한다.

```java
// 사용 예시
Optional<String> test = Optional.ofNullable("test");
if(test.isPresent()){
  System.out.println(test.get());
}
```

<br>

#### orElse(), orElseGet(), orElseThrow()

```java
// orElse
public T orElse(T other) {return value != null ? value : other};

// orElseGet
public T orElseGet(Supplier<? extends T> supplier) {
  return value != null ? value : supplier.get();
}

// orElseThrow
public T orElseThrow() {
  if (value == null) {
    throw new NoSuchElementException("No value present");
  }
  return value;
}
```

* `orElse()`
  * 저장된 값이 존재하면 값을 반환하고, 값이 `null` 이면 인수로 전달된 값을 반환한다.
* `orElseGet()`
  * 저장된 값이 존재하면 그 값을 반환하고, 값이 `null` 이면 전달된 람다 표현식의 결괏값을 반환한다.
* `orElseThrow()`
  * 저장된 값이 존재하면 그 값을 반환하고, 값이 `null` 이면 인수로 전달된 예외를 발생시킨다.

<br>

## 5 Optional을 Optional답게 사용하자
> 이번 챕터는 https://www.daleseo.com/java8-optional-after/ 의 글을 정리한 것입니다.
 
<br>

### 5-1 Optional의 잘못된 사용
```java
String text = getText();
Optional<String> maybeText = Optional.ofNullable(text);
int length;
if (maybeText.isPresent()) { // if (text != null)
	length = maybeText.get().length();
} else {
	length = 0;
}
```
위 코드는 `isPresent()`를 통해 `null`인지 확인하고 `get`을 호출하는 예시다. 

이렇게 사용할거면 뭐하러 `Optional`로 감쌌는가??..

**`Optional`의 목적은 고통스러운 null 처리를 직접하지 않고 `Optional` 클래스에 위임하기 위함입니다.**

```java
// 제대로된 Optional 사용법
int length = Optional.ofNullable(getText()).map(String::length).orElse(0);
```

<br>

### 5-2 Stream처럼 사용하기
**`Optional`을 최대 1개의 원소를 가지는 있는 특별한 Stream이라고 생각하면 좋다.**

Stream과 직접적인 구현이나 상속관계는 없지만, 사용방법이나 기본 사상이 매우 유사하기 때문이다.

Stream 클래스가 가지고 있는 `map()`이나 `flatMap()`, `filter()`와 같은 메서드를 `Optional`도 가지고 있다.

<br>

#### map
```java
// 주문을 한 회원이 살고 있는 도시를 반환한다.
public String getCityOfMemberFromOrder(Order order) {
  return Optiona.ofNullable(order)
          .map(Order::getMember)
          .map(Member::getAddress)
          .map(Address::getCity)
          .orElse("Seoul");
}
```
* 위와 같이 Optional의 fluent API에 의해 단순한 메서드 체이닝을 통해 우아한 코드를 완성할 수 있다.

<br>

#### filter
주어진 시간 내에 생성된 주문을 한 경우에만 해당 회원 정보를 구하는 메서드
```java
// filter 사용 전
public Member getMemberIfOrderWithin(Order order, int min) {
	if (order != null && order.getDate().getTime() > System.currentTimeMillis() - min * 1000) {
		return order.getMember();
	}
}
```
```java
// filter 사용 후
public Optional<Member> getMemberIfOrderWithin(Order order, int min) {
	return Optional.ofNullable(order)
			.filter(o -> o.getDate().getTime() > System.currentTimeMillis() - min * 1000)
			.map(Order::getMember);
}
```

### 5-3 이외의 Optional 바르게 쓰기

http://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/

<br>

## 6 자바9에서 추가된 기능

<br>

### 6-1 or()
```java
String coffeeToMake = Optional
  .ofNullable(order)
  .map(Order::getCoffee)
  .or(() -> Optional.ofNullable(user.getFavoriteCoffee()))
  .orElse("Americano");
System.out.println("만들 커피: " + coffeeToMake);
```
* `or()`는 `orElseGet()`나 `orElse()`와 달리 Optional 객체를 리턴하고 싶을 때 사용한다.
* **이번에 추가된 or() 메서드는 주로 메서드 채인의 중간에서 Optionl 객체가 비어있을 경우 값을 채워주기 위한 용도로 사용할 수 있다.**
  
<br>

### 6-2 ifPresentOrElse()
```java
// ifPresent()
Optional.ofNullable(order)
  .map(Order::getCoffee)
  .ifPresent(coffee -> System.out.println("만들 커피: " + coffee));
```
```java
// ifPresentOrElse()
Optional.ofNullable(order)
  .map(Order::getCoffee)
  .ifPresentOrElse(
    coffee -> System.out.println("만들 커피: " + coffee),
    () -> System.out.println("만들 커피: " + user.getFavoriteCoffee())
  );
```
* `ifPresent()`는 Optional 객체가 값을 담고 있을 때 처리할 내용만 정의했다면, `ifPresentOrElse()`는 그와 더불어 Optional 객체가 비어있을 경우 처리할 내용까지 정의할 수 있다.
  * 첫번째 인자는 `ifPresent()`와 동일 (값이 존재할 경우)
  * 두번째 인자는 Optional이 비어있을 경우 수행될 람다 함수

<br>

# 참조

* https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html
* http://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/
* http://tcpschool.com/java/java_stream_optional
* https://www.daleseo.com/java9-optional/