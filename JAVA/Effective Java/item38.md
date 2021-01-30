> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

<br>

- [아이템 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라](#아이템-38-확장할-수-있는-열거-타입이-필요하면-인터페이스를-사용하라)
  - [1 타입 안전 열거 타입과 열거 타입의 차이](#1-타입-안전-열거-타입과-열거-타입의-차이)
    - [1-1 타입 안전 열거 타입과 열거 타입](#1-1-타입-안전-열거-타입과-열거-타입)
    - [1-2 열거타입은 확장하기 좋지 않다](#1-2-열거타입은-확장하기-좋지-않다)
  - [2 열거타입 확장](#2-열거타입-확장)
    - [2-1 열거타입이 인터페이스를 구현 (OCP)](#2-1-열거타입이-인터페이스를-구현-ocp)
    - [2-2 확장 가능한 열거 타입 흉내내는 방식의 문제점](#2-2-확장-가능한-열거-타입-흉내내는-방식의-문제점)
  - [핵심 정리](#핵심-정리)

<br>

## 1 타입 안전 열거 타입과 열거 타입의 차이

### 1-1 타입 안전 열거 타입과 열거 타입
우선 타입 안전 열거 타입과 열거 타입이 무엇인지 알아본다.

<br>

```java
// 타입 안전 열거 타입
class Day {
  public final static Day MONDAY = new Day();
  public final static Day TUESDAY = new Day();
  ...
    
}

class Month {
  public final static Day JANUARY = new Month();
  ...
}
```
```java
// 열거 타입
enum Day {
  MONDAY, TUESDAY, ...;
}

enum MONTH {
  JANURAY, FEBRUARY, ...;
}
```

<br>

### 1-2 열거타입은 확장하기 좋지 않다
열거 타입은 거의 모든 상황에서 타입 안전 열거 패턴(typesafe enum pattern)보다 우수하다고 한다.

하지만 **타입 안전 열거 패턴은 확장할 수 있으나 열거 타입은 그렇지 못하다.**

* 확장 타입의 원소는 기반 타입의 원소로 취급, 그러나 반대는 아니다 (이상함)
* 기반 타입과 확장 타입의 원소 모두 순회할 방법도 마땅치 않다.
* 확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 복잡하다.

<br>

## 2 열거타입 확장

<br>

### 2-1 열거타입이 인터페이스를 구현 (OCP)
열거타입 (이하 Enum)은 확장하는 건 좋은 생각이 아니다.

하지만 **연산코드와 같이 Enum을 이용한 확장을 하고 싶다면 인터페이스를 이용해라.**

```java
// 인터페이스를 이용해 확장 가능 열거 타입을 흉내냈다
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {

    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
    ... // toString()
}
```
* Enum이 인터페이스를 구현함으로써 새로운 사용자 확장 연산자를 확장할 수 있게 되었다. **(OCP)**
  * Enum인 `BasicOperation`은 확장할 수 없지만 인터페이스인 `Operation`은 확장할 수 있고, 인터페이스를 연산의 타입으로 사용하면 된다. (다형성)

```java
// 추가 사용자 정의 연산 (확장)
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) { return Math.pow(x, y)}
    },
    REMAINDER("%") {
        public double apply(double x, double y) { return x % y; }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    ...
}
```
* 이렇게 인터페이스를 구현함으로써 언제든 새로운 연산자를 추가할 수 있다.
* 새로 작성한 연산은 기존 연산을 쓰던 곳이면 어디든 사용이 가능하다.

<br>

### 2-2 확장 가능한 열거 타입 흉내내는 방식의 문제점
**열거 타입끼리 구현을 상속할 수 없다.**

* 디폴트 구현을 이용해 인터페이스에 추가하는 방법을 사용할 수 없다.
* 연산 기호를 저장하고 찾는 로직이 BasicOperation ,ExtendedOperation 모두에 들어가야한다.

<br>

## 핵심 정리
* Enum 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다.