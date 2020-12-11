> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 13. clone 재정의는 주의해서 진행하라

이번 아이템에서는 `clone()` 메서드의 알맞은 구현 방법과 언제 그렇게 해야 하는지를 알려주고, 가능한 다른 선택지에 관해 논의한다.

- [Cloneable 인터페이스](#cloneable-인터페이스)
  * [Cloneable 인터페이스의 문제](#cloneable-인터페이스의-문제)
  * [Cloneable 인터페이스의 역할](#cloneable-인터페이스의-역할)
- [clone 메서드 재정의](#clone-메서드-재정의)
- [clone 재정의시 주의할 점](#clone-재정의시-주의할-점)
  * [가변 객체를 참조하는 경우](#가변-객체를-참조하는-경우)
- [Cloneable 요약](#cloneable-요약)
- [꼭 Cloneable을 사용해야 하는가?](#꼭-cloneable을-사용해야-하는가)
  * [복사 생성자](#복사-생성자)
  * [복사 팩터리](#복사-팩터리)
- [결론](#결론)

<br>

## Cloneable 인터페이스

### Cloneable 인터페이스의 문제

```java
// 내부의 아무런 추상 메서드가 없다.
public interface Cloneable {}
```

`Cloneable`은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이다.

하지만 **문제는 `clone()` 메서드가 선언된 곳이 `Cloneable`이 아닌 `Object` 이고, 그 마저도 `protected`이다.**

**그래서 `Cloneable`을 구현하는 것만으로는 외부 객체에서 `clone()` 메서드를 호출할 수 없다.**

실무에서는 `clone()`을 public으로 오버라이딩하여 제공한다.

> 믹스인 : 클래스가 본인의 기능 이외에 추가로 구현할 수 있는 자료형으로, 어떤 선택적 기능을 제공한다는 사실을 선언하기 위해 쓰인다. (일반적이지 않은 인터페이스 사용방식)

<br>

### Cloneable 인터페이스의 역할

**이 인터페이스는 `Object`의 `protected` 메서드인 `clone()`의 동작 방식을 결정한다.**

`Cloneable`을 구현한 클래스의 `clone()`을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 `CloneNotSupportedException`을 던진다.

즉, `Cloneable`을 구현하지 않은 객체는 복사하려고 하면 예외가 던져진다.

<br>

## clone 메서드 재정의

```java
public Class PhoneNumber implements Cloneable {
    ...
      
      
    @Override
    public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); // 일어날 수 없는 일이다.
        }
    }
  
  	...
}
```

* `Object`의 `clone()` 메서드는 native로 되어있다. 그리고 `Cloneable`을 구현하는 하위 클래스에서는 `super.clone()` 을 호출하면 자동적으로 완벽한 복제본을 얻을 수 있다.
* `CloneNotSupportedException`은 `Object`의 `clone()`이 던지는 예외이다.

<br>

## clone 재정의시 주의할 점

clone메서드는 사실상 생성자와 같은 효과를 낸다. clone은 다음 사항을 지켜야한다.

* clone은 원본 객체에 아무런 해를 끼치지 않아야 한다.
* 복제된 객체의 불변식을 보장해야한다.

<br>

### 가변 객체를 참조하는 경우

`Object`의 `clone()`은 그저 값만을 복사한다. 즉, **참조 변수는 깊은 복사를 안하기 때문에 불변이 깨지게 되는 문제가 발생한다.**

이 문제를 해결하기 위해서는 아래와 같이 구현해야 한다.

```java
// 가변 상태를 참조하는 클래스용 clone메서드
public class Stack implements Cloneable{
    private Object[] elements; // 문제의 참조 변수
    private int size = 0;
    
    ...
      
    @Override
    public Stack clone() {
        try {
            Stack result = (Stack) super.clone(); // 얕은 복사
            result.elements = elements.clone(); // 참조 변수의 내용을 복사
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

> 배열 같은 경우는 `clone()`이 이미 잘 정의 되어있어서 안전하게 사용할 수 있다.

<br>

## Cloneable 요약

1. `Cloneable`을 구현하는 모든 클래스는 `clone()`을 재정의해야 한다.
   * 이때 접근 제한자는 public으로, 반환 타입은 클래스 자신으로 변경한다.
2. 가장 먼저 `super.clone()`을 호출한 후 필요한 필드를 전부 적절히 수정한다.
   * 객체의 내부 **'깊은 구조'에 숨어있는 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 해야한다.**
   * 이러한 내부 복사는 주로 `clone()`을 재귀적으로 호출해 구현한다. (최선인 것은 아니다.)

<br>

## 꼭 Cloneable을 사용해야 하는가?

**복사 생성자와 복사 팩터리라는 더 나은 객체 방식을 제공할 수 있다.**

더 나은 이유는 다음과 같다.

* 언어 모순적이고 위험천만한 객체 생성 메커니즘 (생성자를 쓰지 않는 방식)을 사용하지 않는다.
* 엉성하게 문서화된 규약에 기대지 않는다.
* 정상적인 final 필드 용법과도 충돌하지 않는다.
* 불필요한 검사 예외를 던지지 않고, 형변환도 필요치 않다.

또한, 다형성을 이용해서 복사뿐만 아니라 변환도 가능하다. 예를 들어, Collections를 매개변수로 받는다면 HashSet을 TreeSet으로 변환하여 복사할 수도 있다.

<br>

### 복사 생성자

```java
public Yum(Yum yum) {...}
```

기존의 객체를 생성자의 매개변수로 넘겨줘서 복사하는 방식

<br>

### 복사 팩터리

```java
public static Yum newInstance(Yum yum) { ... }
```

복사 팩터리는 복사 생성자를 모방한 static 팩토리 ([아이템1](./item01.md))이다.

<br>

## 결론

* 기본 원칙은 '복제 기능은 생성자와 팩터리를 이용하는 게 최고'라는 것.
  * 즉, Cloneable은 기본 자료형에 대한 복사에만 사용하는 것이 좋다는 의미. (예를 들어, 배열)
* 이외에는 가능한 복사 생성자나 복사 팩터리를 사용하자.