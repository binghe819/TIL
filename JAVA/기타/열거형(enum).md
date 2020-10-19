# 목차

- [enum (열거형)](#enum-열거형)
  * [1 enum이란?](#1-enum이란)
  * [2 상수를 정의하는 다양한 방법](#2-상수를-정의하는-다양한-방법)
    + [2-1 final static](#2-1-final-static)
    + [2-2 클래스와 인터페이스 사용](#2-2-클래스와-인터페이스-사용)
    + [2-3 클래스 사용](#2-3-클래스-사용)
    + [2-4 Enum 사용](#2-4-enum-사용)
  * [3 생성자 활용](#3-생성자-활용)
    + [3-1 enum클래스에 추가 속성 부여](#3-1-enum클래스에-추가-속성-부여)
    + [3-2 On / Off](#3-2-on--off)
  * [4 enum 기본 메서드](#4-enum-기본-메서드)
  * [5 여러가지 enum이야기](#5-여러가지-enum이야기)
    + [5-1 enum생성자는 private](#5-1-enum생성자는-private)
    + [5-2 enum은 싱글톤](#5-2-enum은-싱글톤)
- [참조](#참조)



# enum (열거형)

> 여러 게시글을 [참조](#참고)하여 작성한 글입니다.



## 1 enum이란?

* **관련 있는 상수들의 집합**
  * 서로 관련 있는 상수들을 모아 심볼릿한 명칭의 집합으로 정의한 것
* Enum 클래스형을 기반으로 한 클래스형 선언
  * 클래스처럼 보이게 하는 상수
* 새로운 열거형을 선언하면, 내부적으로 Enum 클래스형 기반의 새로운 클래스형이 만들어진다.

> **C/C++의 경우 Enum은 결국 int값이지만, Java의 Enum은 완전한 기능을 갖춘 클래스다.**



## 2 상수를 정의하는 다양한 방법



### 2-1 final static

```java
public class Example {
  private final static int MONDAY = 1;
  private final static int TUESDAY = 2;
  private final static int WEDNESDAY = 3;
  private final static int THURSDAY = 4;
  private final static int FRIDAY = 5;
  private final static int SATURDAY = 6;
  private final static int SUNDAY = 7;
  
  public static void main(String[] args) {
    int day = MONDAY;
    
    switch(day) {
      case MONDAY:
        System.out.println("월요일");
        break;
      case TUESDAY:
        System.out.println("화요일");
        break;
      case ...:
        ...
    }
  }
}
```

* `final static`
  * `final`을 이용해 한번 지정하면 변경되지 못하게 하였으며, `static`을 사용해 메모리에 한 번만 할당되게 만들었다.
  * 게다가 변수명과 변수명을 통해 상수에 이름을 부여할 수 있다.
* **단점**
  * 상수가 많아지면 코드가 너무 길어지며, 한눈에 상수들이 어떤 것에 관련된 것인지 보기 힘들다.
  * 네임 충돌



### 2-2 클래스와 인터페이스 사용

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
    
    if(Day.MONDAY == MONTH.JANUARY) // 다른 집합의 상수랑 비교하면 false가 나와야하지만 true가 나온다
      System.out.println("두 상수가 같다.")
    
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

* **인터페이스**
  * 특징(DAY)을 갖는 상수 집합을 작성할 수 있다.
  * 다른 집합과 네임 충돌이 일어나지 않는다
* **단점**
  * **타입 안정성이 떨어진다. -> 컴파일 시 형검사를 하지 않으므로 비교연산시 오류 발생**
  * **변수명을 가진 상수지만 값을 가지고 있기 때문에 비교 연산이 되는 것.**



### 2-3 클래스 사용

```java
class Day {
  public final static Day MONDAY = new Day();
  public final static Day TUESDAY = new Day();
  ...
    
}

class Month {
  public final static Day JANUARY = new Month();
  ...
}

public class Example {
  public static void main(String[] args) {
    if(Day.MONTH == MONTH.JANUARY) // 에러 발생 (같은 타입이 아니다)
      System.out.println("두 상수는 같다.")
      
    Day day = Day.MONDAY;
    
    switch(day) {
      case DAY.MONDAY:
        System.out.println("월요일");
        break;
      case DAY.TUESDAY:
        System.out.println("화요일");
        break;
      ...
    }
  }
}
```

* 클래스를 이용한 상수
  * **자기 자신을 인스턴스화 한 값을 할당한다.**
  * 이로 인해, 각각의 상수들은 서로 다른 데이터를 의미하게 된다. (참조 변수이므로)
  * **같은 집합의 상수들은 같은 데이터 타입을 가지므로 서로 다른 집합의 상수끼리 비교 연산은 컴파일 시에 막을 수 있다.**
* 단점
  * switch문은 참조변수를 허용하지 않으므로 switch문을 사용할 수 없다. 



### 2-4 Enum 사용

여러 상수를 활용한 문제들을 해결 방법은 바로 Enum을 사용하는 것이다

```java
enum Day {
  MONDAY, TUESDAY, ...;
}

enum MONTH {
  JANURAY, FEBRUARY, ...;
}

public class Example {
  public static void main(String[] args) {
    Day day = Day.MONDAY;
    
    switch(day) {
      case MONDAY:
        System.out.println("월요일");
        break;
      ...
    }
  }
}
```

* Enum - 열거형
  * 서로 연관된 상수들의 집합.

```java
class Day {
  public final static Day MONDAY = new Day();
  public final static Day TUESDAY = new Day();
  ...
}

// class Day와 같은 기능을 한다.
enum Day {
  MONDAY, TUESDAY, ...;
}
```

* **클래스를 이용한 상수 패턴과 enum을 사용한 상수 방법은 똑같은 기능을 한다.**
* Enum의 장점은 아래와 같다
  * 코드가 단순해지며 가독성이 좋다
  * 인스턴스 생성과 상속을 방지한다
  * 키워드 enum을 사용하기 때문에 구현의 의도가 열거임을 분명하게 나타낼 수 있다.



## 3 생성자 활용



### 3-1 enum클래스에 추가 속성 부여

```java
public enum Brand {
    APPLE("애플"), SAMSUNG("삼성"), LG("엘지")
    ;

    private final String name; // 추가 속성

    private Brand(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

* enum의 상수에는 추가 속성을 부여할 수 있다.
  * `getXXX()`을 통해 속성에 접근할 수 있다.



### 3-2 On / Off

```java
public enum Power {
  ON("켜짐"),
  OFF("꺼짐");
  
  private String status;
  
  private Power(String status) {
    this.status = status;
  }
  
  public String getStatus() {
    return status;
  }
  
  public Power opposite() {
    if(this == Power.ON)
      return Power.OFF;
    else
      return Power.ON;
  }
}
```





## 4 enum 기본 메서드

참고 : https://docs.oracle.com/javase/8/docs/api/java/lang/Enum.html

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

* 기본 Enum은 name과 ordinal 상태를 포함하고 있다.



## 5 여러가지 enum이야기



### 5-1 enum생성자는 private

* 자바에서 enum은 특별한 형태의 클래스이다.
* **enum에서 생성자를 public으로 설정하면 에러가 발생한다.**
  * **그 이유는 enum은 상수의 집합이므로, 런타임이 아닌 컴파일타임에 모든 값을 알고 있어야 한다.**
  * 즉, 다른 패키지나 클래스에서 enum 타입에 접근해서 동적으로 어떤 값을 정해줄 수 없다.
  * 이렇게 되면 외부에서 접근 가능한 생성자가 없으므로 enum타입은 실제적으로 final과 다름 없다.



### 5-2 enum은 싱글톤

* enum은 상수 집합이다. 즉 new를 통해 인스턴스를 생성할 수 없으며, 상속을 받을 수도 없다.
* 즉, **enum타입은 인스턴스 생성을 제어하며, 싱글톤을 일반화한다.**
  * 이러한 특성 때문에, enum타입은 싱글톤을 구현하는 하나의 방법으로 사용되기도 한다.



# 참조

* http://www.nextree.co.kr/p11686/
* [이펙티브 자바 Effective Java 3/E](http://www.yes24.com/Product/Goods/65551284?OzSrank=1)
* https://woowabros.github.io/tools/2017/07/10/java-enum-uses.html
* https://docs.oracle.com/javase/8/docs/api/java/lang/Enum.html