> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 34. int 상수 대신 열거 타입을 사용하라

<br>

- [아이템 34. int 상수 대신 열거 타입을 사용하라](#아이템-34-int-상수-대신-열거-타입을-사용하라)
  - [1 상수를 정의하는 다양한 방법](#1-상수를-정의하는-다양한-방법)
    - [1-1 final static](#1-1-final-static)
    - [1-2 클래스와 인터페이스 사용](#1-2-클래스와-인터페이스-사용)
    - [1-3 클래스 사용](#1-3-클래스-사용)
    - [1-4 Enum 사용](#1-4-enum-사용)
  - [2 Enum의 특징](#2-enum의-특징)
    - [2-1 Enum도 클래스다!](#2-1-enum도-클래스다)
    - [2-2 enum은 싱글턴을 일반화한 형태다](#2-2-enum은-싱글턴을-일반화한-형태다)
    - [2-3 enum생성자는 private](#2-3-enum생성자는-private)
  - [3 Enum 기본 메서드](#3-enum-기본-메서드)
  - [4 Enum 활용](#4-enum-활용)
    - [4-1 상수별 메서드 구현](#4-1-상수별-메서드-구현)
    - [4-2 전략 열거 타입 패턴](#4-2-전략-열거-타입-패턴)
  - [핵심 정리](#핵심-정리)
  - [참고](#참고)

<br>

## 1 상수를 정의하는 다양한 방법
책에서는 정수로 상수를 만드는 내용만 다루지만, enum을 더 자세히 이해하기 위해서 상수를 정의하는 여러가지 방법은 정리하였다.

<br>

### 1-1 final static
```java
public class Example {

    private final static int MONDAY = 1;
    private final static int TUESDAY = 2;
    private final static int WEDNESDAY = 3;
    private final static int THURSDAY = 4;
    private final static int FRIDAY = 5;
    private final static int SATURDAY = 6;
    private final static int SUNDAY = 7;

    private 

    public static void main(String[] args) {
        int day = MONDAY;
        
        switch (day) {
            case MONDAY:
                System.out.println("월요일");
                break;
            case TUESDAY:
                System.out.println("화요일");
                break;
            ...
        }
    }
}
```
* `final static`
  * `final`을 이용해 한번 지정하면 변경되지 못하게 하였으며, `static`을 이용해 메모리에 한 번만 할당되게 만든다.
  * 게다가 변수명을 통해 상수에 이름을 부여할 수 있다.
* 단점
  * 상수가 많아지면 코드가 너무 길어지며, 한눈에 상수들이 어떤 것에 관련된 것인지 보기 힘들다.
    * 우회 방법으로 `APPLE_FUJI`, `APPLE_PIPPIN`, `ORANGE_NAVEL`과 같이 앞에 접두어를 사용한다. (안티패턴)
  * 네임 충돌

<br>

### 1-2 클래스와 인터페이스 사용
클래스와 인터페이스를 이용하면 각각의 상수들을 집합으로 정의할 수 있다.
```java
interface DAY {
    int MONDAY = 1;
    int TUESDAY = 2;
    int WEDNESDAY = 3;
    int THURSDAY = 4;
    ...
}

interface MONTH {
    int JANUARY = 1;
    ...
}

public class Example {
    public static void main(String[] args) {

        if (DAY.MONDAY == MONTH.JANUARY)  // 다른 집합의 상수랑 비교하면 false가 나와야하지만 true가 나온다.
            System.out.println("두 상수가 같다.");
        
        int day = DAY.MONDAY;

        switch(day) {
            case DAY.MONDAY:
                System.out.println("월요일");
                break;
            case DAY.TUESDAY:
                System.out.println("화요일");
                break;
            case ...:
                ...
        }
    }
}
```
* 인터페이스
  * 특징을 갖는 상수 집합을 만들 수 있다. (DAY, MONTH, ...)
  * 다른 집합과 네임 충돌이 일어나지 않는다.
* 단점
  * 타입 안정성이 떨어진다 -> 컴파일시 형검사를 하지 않으므로 비교연산시 오류가 발생한다.
    * `DAY.MONDAY == MONTH.JANUARY // true`
  * 변수명을 가진 상수지만 값을 가지고 있기 때문에 비교 연산이 되므로 오류가 발생할 확률이 높다.

<br>

### 1-3 클래스 사용
```java
class Day {
    public final static Day MONDAY = new Day();
    public final static Day TUESDAY = new Day();
    ...
}

public Month {
    public final static Month JANUARY = new Month();
    ...
}

public class Example {
    public static void main(String[] args) {
        if (Day.MONTH == Month.JANUARY)     // 컴파일 에러 발생 (같은 타입이 아니므로 연산이 불가능)
            System.out.println("두 상수는 같다.")
        
        Day day = Day.MONDAY;

        // 클래스로 만들어진 day는 switch 연산이 불가하다.
        switch(day) {  // 컴파일 에러
            case Day.MONDAY:
                System.out.println("월요일");
                break;
            case Day.TUESDAY:
                System.out.println("화요일");
                break;
            ...
        }
    }
}
```
* 클래스를 이용한 상수
  * **자기 자신을 인스턴스화 한 값을 할당한다.**
  * 이로 인해 각각의 상수들은 서로 다른 데이터를 의마하게 된다. (참조 변수이므로)
  * **같은 집합의 상수들은 같은 데이터 타입을 가지므로 서로 다른 집합의 상수끼리 비교 연산을 컴파일 시에 예방할 수 있다.**
* 단점
  * `switch`문은 참조변수를 허용하지 않으므로 `switch`문을 사용할 수 없다.

<br>

### 1-4 Enum 사용
여러 상수를 활용한 문제들을 해결한 방법이 바로 `Enum`을 사용하는 것이다

```java
enum Day {
    MONDAY, TUESDAY, ...;
}

enum MONTH {
    JANUARY, FEBRUARY, ...;
}

public class Example {
    public static void main(String[] args) {
        Day day = Day.MONDAY;


    }
}
```
* Enum - 열거형
  * 서로 연관된 상수들의 집합
  * 이전의 상수 정의하는 문제들을 모두 해결한다.
* **해결한 문제들**
  * 타입 안정성 보장 (서로 다른 타입의 연산을 방지)
  * 네임 충돌 방지
  * `switch`문 사용 가능
  * 추가 혹은 수정으로 인한 예외를 컴파일 타임때 잡아준다.

<br>

## 2 Enum의 특징
Enum은 어떤 특징이 있는지 알아보자.

<br>

### 2-1 Enum도 클래스다!
```java
// class Day와 enum Day는 같다고 볼 수 있다.
class Day {
    public final static Day MONDAY = new Day();
    public final static Day TUESDAY = new Day();
    ...
}

enum Day {
    MONDAY, TUESDAY, ...;
}
```
* Enum은 아이디어는 단순하다 클래스에 상수 하나당 자신의 인스턴스를 하나씩 만들어 공개하는 것.
* Enum이 클래스이기 때문에 **아래와 같이 연관된 데이터와 메서드를 추가할 수 있다.**

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6);
    ...

    // enum의 연관된 데이터
    private final double mass;
    private final double radius;
    private final double surfaceGravity;

    // enum 상수
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    // Getter
    ...

    // 메서드
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```
* 열거 타입은 근본적으로 불변(상수이기 때문)이라 모든 필드는 `final`이어야 한다.

<br>

### 2-2 enum은 싱글턴을 일반화한 형태다
상수는 당연히 불변 객체이며, 하나만을 생성하여 재활용해야 한다.

싱글턴은 원소가 하나뿐인 열거 타입이라 할 수 있고, 거꾸러 **열거 타입은 싱글턴을 일반화한 형태라고 볼 수 있다.**

* enum은 상수 집합이므로, new를 통해 인스턴스를 생성할 수 없으며, 상속을 받을 수 없다.
* 즉, **enum타입은 인스턴스 생성을 제어하며, 싱글톤을 일반화한다.**
  * 이러한 특성 때문에, **enum 타입은 싱글톤을 구현하는 하나의 방법으로 사용되기도 한다.**

<br>

### 2-3 enum생성자는 private
* 자바에서 enum은 특별한 형태의 클래스다.
* **enum에서 생성자를 public으로 설정하면 컴파일 에러가 발생한다.**
  * 그 이유는 enum은 상수 집합이므로, 런타임이 아닌 컴파일타임에 모든 값을 알고 있어야 하기 때문이다.
  * 즉, **다른 패키지나 클래스에서 enum 타입에 접근해서 동적으로 어떤 값을 정해줄 수 없다.**
  * **이렇게 되면 외부에서 접근 가능한 생성자가 없으므로 enum 타입은 실제적으로 final과 다름 없다.**

<br>

## 3 Enum 기본 메서드
> 참고 : https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Enum.html

Enum은 상수 역할을 할 뿐만 아니라, 기본적인 메서드를 제공한다.
```java
public abstract class Enum<E extends Enum<E>> implements... {
  // 상태
  private final String name;
  private final String ordinal;
  
  // 생성자
  protected Enum(String name, int ordinal) {
    this.name = name;
    this.ordinal = ordinal;
  }
  
  // name, toString()
  public final String name() {
    return name;
  }
  
  // ordinal
  public final int ordinal() {
    return ordinal;
  }
  
  // valueOf (매개변수로 주어진 String와 열거형에서 일치하는 이름을 갖는 원소를 반환)
  public static <T extends Enum<T>> T valueOf(Class<T> enumType, String name) {
    T result = enumType.enumConstantDirectory().get(name);
    if (result != null)
      return result;
    if (name == null)
      throw new NullPointerException("Name is null");
    throw new IllegalArgumentException(
      "No enum constant " + enumType.getCanonicalName() + "." + name);
  }
  
  // compareTo
  
}
```
* 이외에도
  * `E[] values()` : 해당 enum의 요소들을 배열로 반환한다.

<br>

## 4 Enum 활용
Enum은 다양하게 활용할 수 있다.

<br>

### 4-1 상수별 메서드 구현
```java
// 값에 따라 분기하는 열거 타입
public enum Operation {
    PLUS,
    MINUS,
    TIMES,
    DIVIDE;

    public double apply(double x, double y) {
        switch (this) {
            case PLUS:
                return x + y;
            case MINUS:
                return x - y;
            case TIMES:
                return x * y;
            case DIVIDE:
                return x / y;
            default:
                throw new AssertionError("알 수 없는 연산: " + this);
        }
    }
}
```
* 딱 봐도 코드가 예쁘지 않다.
  * 새로운 상수를 추가한다면 case문도 추가해야하는데 까먹기 쉽다.

```java
// 상수별 클래스 몸체와 데이터를 사용한 열거 타입 - 상수별 메서드 구현
public enum Operation {
    PLUS("+") {public double apply(double x, double y) {return x + y;}},
    MINUS("-") {public double apply(double x, double y) {return x - y;}},
    TIMES("*") {public double apply(double x, double y) {return x * y;}},
    DIVIDE("/") {public double apply(double x, double y) {return x / y;}};
    
    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    public abstract double apply(double x, double y);
}
```
* 코드가 훨씬 깔끔하고, 새로운 상수를 추가해도 문제가 없다.

<br>

아래와 같이 람다식을 이용한 방법도 사용가능하다.
```java
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final BiFunction<Double, Double, Double> operation;

    Operation(String symbol, BiFunction<Double, Double, Double> apply) {
        this.symbol = symbol;
        this.operation = apply;
    }

    public double apply(double x, double y) {
        return this.apply(x, y);
    }
}
```

<br>

### 4-2 전략 열거 타입 패턴
기본 일당과 오버타임 비용을 계산하는 예제이다.
```java
// 값에 따라 분기하여 코드를 공유하는 열거 타입
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
    SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFY = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch(this) {
            case SATURDAY: case SUNDAY: // 주말
                overtimePay = basePay / 2;
                break;

            default:                    // 주중
                overtimePay = minutesWorked <= MINS_PER_SHIFY ?
                0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overTimePay;
    }
}
```
* 만약 휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 `case`문도 추가해줘야한다..
  * 자칫 까먹으면 휴가때와 주중의 오버타임 비용이 같아진다..

<br>

```java
// 전략 열거 타입 패턴
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFY ?
                    0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```
* 위와 같이 전략 열거 타입을 사용하면 `휴가`가 추가되어도 `VACATION`만 추가해주면 된다.
  * `switch`문을 제거하고 의존성 주입으로 문제를 해결한 예제라고 볼 수 있다.

<br>

## 핵심 정리
* 열거 타입은 다른 상수보다 뛰어나다. 더 읽기 쉽고 안전하고 강력하다.
* 드물게는 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있다.
  * 이런 열거 타입에서는 `switch`문 대신 상수별 메서드 구현을 구현하자.
* 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.

<br>

## 참고
* http://www.nextree.co.kr/p11686/
* 이펙티브 자바 Effective Java 3/E
* https://woowabros.github.io/tools/2017/07/10/java-enum-uses.html
* https://docs.oracle.com/javase/8/docs/api/java/lang/Enum.html