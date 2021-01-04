> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 20. 추상 클래스보다는 인터페이스를 우선하라

- [아이템 20. 추상 클래스보다는 인터페이스를 우선하라](#아이템-20-추상-클래스보다는-인터페이스를-우선하라)
  - [다중 구현 매커니즘](#다중-구현-매커니즘)
    - [가장 큰 차이점 - 타입](#가장-큰-차이점---타입)
  - [인터페이스의 장점](#인터페이스의-장점)
    - [기존 클래스에 새로운 인터페이스를 구현해넣을 수 있다](#기존-클래스에-새로운-인터페이스를-구현해넣을-수-있다)
    - [인터페이스는 믹스인 정의에 안성맞춤이다](#인터페이스는-믹스인-정의에-안성맞춤이다)
    - [인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다](#인터페이스로는-계층구조가-없는-타입-프레임워크를-만들-수-있다)
    - [인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다](#인터페이스는-기능을-향상시키는-안전하고-강력한-수단이-된다)
  - [디폴트 메서드](#디폴트-메서드)
  - [추상 골격 구현 클래스](#추상-골격-구현-클래스)
    - [예시](#예시)
  - [핵심 정리](#핵심-정리)

<br>

## 다중 구현 매커니즘
자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상 클래스 두 가지이다.

### 가장 큰 차이점 - 타입
* 추상 클래스
  * 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다
  * 자바는 단일 상속만 지원하니, 추상 클래스 방식은 새로운 타입을 정의하는데 커다란 제약이 따른다.
* 인터페이스
  * 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취근한다.

<br>

## 인터페이스의 장점
인터페이스의 장점이 무엇이기에 인터페이스를 사용하라 하는 것일까?

### 기존 클래스에 새로운 인터페이스를 구현해넣을 수 있다
* 인터페이스 장점
  * **인터페이스가 요구하는 메서드를 추가하고, 클래스 선언에 `implements` 구문만 추가하면 끝이다.**
* 추상클래스 제약
  * 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어렵다.
  * 두 클래스가 같은 추상 클래스를 확장하길 원한다면, 추상 클래스는 계층구조상 두 클래스의 공통 조상이어야한다.
  * 이 방식은 클래스 계층구조에 커다란 혼란을 일으킨다.

<br>

### 인터페이스는 믹스인 정의에 안성맞춤이다
> 믹스인이란?
> * 믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.
> * **대상 타입의 주된 기능에 선택적 기능을 '혼합(mixed in)'한다고 해서 믹스인이라 부른다.**
> * 예시
>   * `Comparable`은 자신을 구현한 클래스의 인스턴스끼리 순서를 정할 수 있다고 선언한 믹스인 인터페이스이다.

* 인터페이스 장점
  * 인터페이스는 믹스인 정의를 당연히 제공한다.
* 추상클래스 제약
  * 추상 클래스로는 믹스인을 정의할 수 없다. 기존 클래스에 덧씌울 수 없기 때문이다. (다중 상속 불가)

<br>

### 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다
타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있지만, 현실에는 계층을 엄격히 구분하기 어려운 개념도 있다.
```java
// 가수
public interface Singer {
    AudioClip sing(Song s);
}

// 작곡가
public interface Songwriter {
    Song compose(int chartPosition);
}
```
가수와 작곡가가 있다면, 현실에는 작곡도 하고 노래도 부르는 가수가 있다.

이처럼 **인터페이스로 정의하면 가수 클래스가 Singer와 Songwriter 모두 구현해도 전혀 문제가 되지 않는다.**

심지어 둘을 확장하고 새로운 메서드까지 추가한 제 3의 인터페이스를 정의할 수도 있다.
```java
public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip strum();
    void actSensitive();
}
```
**같은 구조를 클래스로 만들려면 가능한 조합 전부를 각각의 클래스로 정의한 고도비만 계층구조가 만들어질 것이다.**

<br>

### 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다
래퍼 클래스 관용구([아이템 18](./item18.md))와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.
```java
// 래퍼 클래스 (ForwardingSet의 멤버인 Set을 래핑)
public class InstrumentedSet<E> extends ForwardingSet<E> {
  private int addCount = 0;
  
  ...
   
  @Override
  public boolean add(E e) {
    addCount++;
    return super.add(e);
  }
  
  @Override
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll();
  }
  ...
}

// 재사용 가능한 전달(forwarding) 클래스 (안전하고 강력)
public class ForwardingSet<E> implements Set<E> {
  private final Set<E> s;
  
  public ForwardingSet(Set<E> s) { this.s = s}
  public void clear() { set.clear(); }
  public boolean isEmpty() { return set.isEmpbty(); }
  public boolean add(E e) { return set.add(e); }
  public boolean addAll(Collection<? extends E> c) { return set.addAll(c); }
  ...
}
```
상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 깨지기 더 쉽다.

<br>

## 디폴트 메서드
* 디폴트 메서드란
  * 인터페이스의 메서드 중 구현 방법이 명백한 것을 디폴트 메서드로 구현하여 제공해 중복을 제거할 수 있다. (default)
* 디폴트 메서드의 제약
  * 인터페이스는 인스턴스 필드를 가질 수 없고, `public`이 아닌 정적 멤버도 가질 수 없다. (`private`은 가능) 
  * 많은 인터페이스가 `equals`와 `hashCode` 같은 `Object`의 메서드를 정의하고 있지만, 이들은 디폴트 메서드로 제공해서는 안 된다.
  * 본인이 만든 인터페이스가 아니라면 디폴트 메서드를 추가할 수 없다.

<br>

## 추상 골격 구현 클래스
자바8부터 인터페이스의 디폴트 메서드가 추가되면서 인터페이스의 중복되는 메서드들의 구현을 방지할 수 있게 되었다.

하지만 **디폴트 메서드도 제약이 있기 때문에 추상 골격 구현 클래스와 같이 사용하면 둘의 장점을 모두 취할 수 있다.**

간단히 말해, **인터페이스로는 타입을 정의하고, 메서드 구현이 필요한 부분은 추상 골격 클래스에서 구현하는 것이다.**

**맞다. 템플릿 메서드 패턴이다.**

<br>

### 예시
추상 골격 구현 클래스를 사용하지 않고 인터페이스만 사용한다면 아래와 같이 중복이 많이 발생하게 된다.
```java
public interface Player {
    void move();

    void jump();

    void attack();
}

public class Warrior implements Player {
    @Override
    public void move() {
        System.out.println("이동");
    }

    @Override
    public void jump() {
        System.out.println("점프");
    }

    @Override
    public void attack() {
        System.out.println("칼로 찌르기");
    }
}


public class Archer implements Player {
    @Override
    public void move() {
        System.out.println("이동");
    }

    @Override
    public void jump() {
        System.out.println("점프");
    }

    @Override
    public void attack() {
        System.out.println("활 쏘기");
    }
}
```

<br>

만약 추상 골격 구현 클래스 + 인터페이스를 활용한다면 아래와 같이 중복을 제거할 수 있다.
```java
public abstract class AbstractPlayer implements Player {
    @Override
    public void move() {
        System.out.println("이동");
    }

    @Override
    public void jump() {
        System.out.println("점프");
    }
}

public class Warrior extends AbstractPlayer {
    @Override
    public void attack() {
        System.out.println("칼로 찌르기");
    }
}

public class Archer extends AbstractPlayer {
    @Override
    public void attack() {
        System.out.println("활 쏘기");
    }
}
```
골격 구현 클래스의 아름다움은 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서는 자유롭다는 점이다.

<br>

## 핵심 정리
* 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다.
  * 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 고려해보자.
* 추상 클래스와 인터페이스를 이용해서 중복을 제거하는 것이 좋다.
  * 인터페이스의 디폴트 메서드를 이용한 골격 구현도 가능하지만, 인터페이스의 구현 제약상 추상 클래스와 같이 이용하는 경우가 더 흔하다.