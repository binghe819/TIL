> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 3. private 생성자나 열거 타입으로 싱글톤임을 보증하라

**싱글톤이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다**. 보통 함수 같은 Stateless 객체 또는 본질적으로 유일한 시스템 컴포넌트를 그렇게 만든다. 

**싱글톤을 사용하는 클라이언트 코드를 테스트하는게 어렵다**. 싱글톤이 인터페이스를 구현한게 아니라면 mock으로 교체하는게 어렵기 때문이다.

**싱글톤으로 만드는 두가지 방법**이 있는데, 두 방법 모두 생성자를 private으로 만들고, public static 멤버를 사용해서 유일한 인스턴스를 제공한다.

<br>

## final 필드

```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  
  private Elvis() {}
}
```

* 장점
  * 이런 API 사용이 static 팩토리 메서드를 사용하는 방법에 비해 더 명확하고 더 간단하다.
* 단점
  * 리플렉션을 사용해서 `setAccessible`을 통해 private 생성자를 호출할 수 있다.
  * 이 방법을 막고자 생성자 안에서 카운팅하거나 flag를 이용해서 예외를 던지게 할 수도 있다.

<br>

## static 팩토리 메서드

```java
public class Elvis {
  
  private static final Elvis INSTANCE = new Elvis();
  
  private Elvis {}
  
  // supplier 구현체처럼 사용될 수 있는 메서드
  public static Elvis getInstance() {
    return INSTANCE;
  }
}
```

* 장점
  * API(`getInstance()`)를 변경하지 않고도 싱글톤으로 쓸지 안쓸지 변경할 수 있다.
  * 처음엔 싱글톤으로 쓰다가 나중엔 스레드당 새 인스턴스를 만든다는 등 클라이언트 코드를 고치지 않고도 변경할 수 있다. -> `getInstance()`를 변경해주기만 하면 된다. 

* static 팩토리 메서드를 `Supplier<Elvis>`에 대한 메서드 레퍼런스로 사용할 수도 있다.

  ```java
  Supplier<Singleton> singletonSupplier = Singleton::Instance
  ```

<br>

## 직렬화

자바에서는 역직렬화를 할 때 직렬화한 코드를 다시 실행한다. 이 과정에서 같은 타입의 인스턴스가 여러 개 생길 수 있다.

이 문제를 해결하려면 모든 인스턴스 필드에 `transient`를 추가 (직렬화 하지 않겠다는 의미)하고, `readResolve`메서드를 다음과 같이 구현하면 된다.

* **역직렬화를 하면 `readResolve`를 항상 호출하게 된다. (있으면)** -> [참고](http://www.oracle.com/technetwork/articles/java/javaserial-1536170.html)

```java
public class Singleton {
  private static final transient Singleton INSTANCE = new Singleton();
  
  private Singleton() {}
  
  public static Singleton getInstance() {
    return INSTANCE;
  }
  
  // 역직렬화를 사용할 때 사용됨 (매번 역질렬화마다 동일한 인스턴스를 반환하도록 해준다.)
  private Object readResolve() {
  	return INSTANCE;
  }
}
```

<br>

## Enum

직렬화 문제, 리플렉션으로 호출되는 문제를 모두 해결하는 방법이 있다. 

```java
public enum Elvis {
  INSTANCE;
}
```

코드는 조금 불편하게 느껴지지만 싱글톤을 구현하는 최선의 방법이라 한다.

하지만, Enum 말고 다른 상위 클래스를 상속해야 한다면 사용할 수 없다.























