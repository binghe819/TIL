# 목차

- [Optional](#optional)
  * [1 Optional이란](#1-optional이란)
  * [2 Optional을 만든 의도](#2-optional을-만든-의도)
  * [3 Optional의 사용법](#3-optional의-사용법)
    + [3-1 Optional 객체 생성](#3-1-optional-객체-생성)
    + [3-2 Optional 객체에 접근](#3-2-optional-객체에-접근)
      - [get(), isPresent()](#get-ispresent)
      - [orElse(), orElseGet(), orElseThrow()](#orelse-orelseget-orelsethrow)
  * [4 Optional 바르게 쓰기](#4-optional-바르게-쓰기)
- [참조](#참조)





# Optional



## 1 Optional이란

> `null` 값도 포함할 수 있는 컨테이너 객체.

`java.util.Optional<T>` 클래스는 `Integer` 나 `Double` 클래스처럼 `T` 타입의 객체를 포장해주는 래퍼 클래스이다.

`Optional` 객체를 사용하면 예상치 못한 `NullPointException` 예외를 제공되는 메서드로 간단히 회피할 수 있다.

즉, 복잡한 조건문 없이도 `null` 값으로 인해 발생하는 예외를 처리할 수 있게 된다.



## 2 Optional을 만든 의도

`Optional` is primarily intended for use as a method return type where there is a clear need to represent "no result," and where using `null` is likely to cause errors. A variable whose type is `Optional` should never itself be `null`; it should always point to an `Optional` instance.

**메서드가 반환할 결과값이 ‘없음’을 명백하게 표현할 필요가 있고, `null`을 반환하면 에러를 유발할 가능성이 높은 상황에서 메서드의 반환 타입으로 `Optional`을 사용하자는 것이 `Optional`을 만든 주된 목적이다.** `Optional` 타입의 변수의 값은 절대 `null`이어서는 안 되며, 항상 `Optional` 인스턴스를 가리켜야 한다.

원래 의도에 맞게 사용해야지 기존의 시스템에 해르 끼치지 않게 된다.



## 3 Optional의 사용법



### 3-1 Optional 객체 생성

```java
// of
public static <T> Optional<T> of(T value) { return new Optional<>(value);}

// ofNullable
public static <T> Optional<T> ofNullable(T value) { return value == null ? empty() : of(value);}
```

`of()` 나 `ofNullable()` 메서드를 사용하여 `Optional`을 생성할 수 있다.

* `of()`
  * `null` 이 아닌 명시된 값을 가지는 `Optional` 객체를 반환한다.
  * 만약 `null` 이 저장되면 `NPE` 예외가 발생한다.
* `ofNullable()`
  * `null` 이 될 가능성이 있다면, `ofNullable()` 을 사용한다.
  * `null` 이면 비어있는 `Optional` 객체를 반환한다. `NPE` 를 방지할 수 있다.

```java
Optional<String> test = Optional.ofNullable("Optional 테스트");
System.out.println(test.get());
```



### 3-2 Optional 객체에 접근

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



## 4 Optional 바르게 쓰기

http://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/









# 참조

* https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html

* http://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/
* http://tcpschool.com/java/java_stream_optional