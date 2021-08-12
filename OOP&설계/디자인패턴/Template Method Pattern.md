# 목차

- [목차](#목차)
- [템플릿 메서드 패턴](#템플릿-메서드-패턴)
  - [1 개념](#1-개념)
  - [2 특징](#2-특징)
    - [2-1 상위 클래스가 흐름 제어 주체](#2-1-상위-클래스가-흐름-제어-주체)
    - [2-2 템플릿 메서드와 전략 패턴의 조합](#2-2-템플릿-메서드와-전략-패턴의-조합)
  - [3 고민](#3-고민)
- [참고](#참고)

<br>

# 템플릿 메서드 패턴

<br>

## 1 개념
```java
public abstract class Super {
  public void templateMethod(){
    // 기본 알고리즘 코드
    hookMethod();
    abstractMethod();
    ...
  }
  protected void hookMethod(){} // 선택적으로 오버라이드 가능한 훅 메서드
  public abstract void abstractMethod(); // 서브클래스에서 반드시 구현해야 하는 추상 메서드
}

public class Sub1 extends Super {
  protected void hookMethod(){ // 선택적 오버라이딩
    ...
  } 
  public void abstractMethod() { // 반드시 구현
    ...
  }
}
```
* Template : 견본, 본보기, 형판
* **상위클래스에 기본적인 로직의 흐름 (커넥션 가져오기, SQL 생성, 실행, 반환)을 만들고, 그 기능의 일부를 추상 메서드로 만든뒤 하위클래스에서 이런 메서드를 필요에 맞게 구현해서 사용하도록 하는 방법**
* 즉, 상속을 통해 상위 클래스의 기능을 확장할 때 사용하는 가장 대표적인 방법이다.
* **상위 클래스에서 큰 틀을 만들고, 하위 클래스에서 세부 사항을 변경하도록 강제시켜 사용하는 패턴**

> 훅 메서드는 기본 구현을 제공하면서, 하위 클래스에서 알맞게 선택적으로 확장할 수 있는 메서드를 의미한다.

<br>

## 2 특징
* 클래스의 역할
  * 상위 클래스: 실행 과정을 구현 (컨텍스트 - 부가 로직)
  * 하위 클래스: 일부 단계를 구현 (전략 - 핵심 로직)

<br>

### 2-1 상위 클래스가 흐름 제어 주체
**템플릿 메서드 패턴은 하위 클래스가 아닌 상위 클래스에서 흐름 제어를 한다.**

```java
// 일반적인 하위 타입
public class SuperCar extends ZetEngine {
    @Override
    public void turnOn() {
        // 하위 클래스에서 흐름 제어
        if (notReady) {
            beep();
        } else {
            super.turnOn();
        }
    }
}
```
**반면에 템플릿 메서드 패턴에서는 상위 타입의 템플릿 메서드가 모든 실행 흐름을 제어하고, 하위 타입의 메서드는 템플릿 메서드에서 호출되는 구조를 갖는다.**

<br>

### 2-2 템플릿 메서드와 전략 패턴의 조합
템플릿 메서드와 전략 패턴을 함께 사용하면 상속이 아닌 조립의 방식으로 템플릿 메서드 패턴을 활용할 수 있다.

```java
public <T> T execute(TransactionCallback<T> action) {
    // 일부 코드 생략
    TransactionStatus status = this.transactionManager.getTransaction(this);
    T result;
    try {
        result = action.doInTransaction(status);
    } catch (RuntimeException ex) {
        ...
    }
    ...
}
```
```java
// 사용
transactionTemplate.execute(new TransactionCallback<String>() {
    public String doInTransaction(TransactionStatus status) {
        // 트랜잭션 범위안에서 실행될 코드
    }
})
```
* 간단히 보면 핵심 로직을 만들어서 매개변수로 주입해준다고 보면 된다.

<br>

## 3 고민
* 훅 메서드와 템플릿 메서드의 순서를 하위 클래스에서는 어떻게 아는가??

<br>

# 참고
* [개발자가 반드시 정복해야 할 객체 지향과 디자인 패턴](http://www.yes24.com/Cooperate/Naver/welcomeNaver.aspx?pageNo=1&goodsNo=9179120)
