> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 81. wait와 notify보다는 동시성 유틸리티를 애용하라

<br>

- [아이템 81. wait와 notify보다는 동시성 유틸리티를 애용하라](#아이템-81-wait와-notify보다는-동시성-유틸리티를-애용하라)
  - [wait과 notify는 올바르게 사용하기 까다롭다](#wait과-notify는-올바르게-사용하기-까다롭다)
  - [java.util.concurrent](#javautilconcurrent)
  - [동시성 컬렉션](#동시성-컬렉션)
    - [동시성 컬렉션의 특징](#동시성-컬렉션의-특징)
    - [상태 의존적 메서드](#상태-의존적-메서드)
  - [동기화 장치](#동기화-장치)
  - [wait과 notify 메서드](#wait과-notify-메서드)

<br>

## wait과 notify는 올바르게 사용하기 까다롭다
자바 5에서 고수준의 동시성 유틸리티 도입되었다.

그러므로 **지금은 `wait`과 `notify`를 사용해야 할 이유가 많이 줄었다.**

그리고 **`wait`과 `notify`는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자.**

<br>

## java.util.concurrent
`java.util.concurrent` 패키지는 고수준의 동시성 유틸리티를 제공한다.

크게 세 범주로 나눌 수 있다.

1. 실행자 프레임워크
2. 동시성 컬렉션
3. 동기화 장치

<br>

## 동시성 컬렉션

<br>

### 동시성 컬렉션의 특징
* 동시성 컬렉션이란
  * List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다.
  * **높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다.**
* **동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.**

<br>

### 상태 의존적 메서드
* 상태 의존적 메서드이란
  * 동시성 컬렉션은 동시성을 무력화하지 못한다. 그러므로, 여러 메서드를 원자적으로 묶어 호출하는 일 역시 불가능하다.
  * 그래서 여러 동작을 하나의 원자적 동작으로 묶는 상태 의존적 메서드가 추가되었다.

**ex. `putIfAbsent`**
```java
private static final ConcurrentHashMap<String, String> map = 
    new ConcurrentHashMap<>();

// String.intern
public static String intern(String s) {
    String result = map.get(s);
    if (result == null) {
        result = map.putIfAbsent(s, s);
        if (result == null) {
            result = s;
        }
    }
    return result;
}
```
* `putIfAbsent`
  * Map의 디폴트 메서드이며, 인자로 넘겨진 key가 없을 때 value를 추가한다.
  * 기존 값이 있었다면 그 값을 반환하고, 없었다면 `null`을 반환한다.
* 이 메서드 덕에 위와 같이 스레드 안전한 정규화 맵을 쉽게 구현할 수 있다.

> 동기화된 컬렉션보다는 동시성 컬렉션을 사용하는 것이 훨씬 좋다고 한다.

<br>

## 동기화 장치
* 동기화 장치란?
  * 동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다.
* 동기화 장치의 종류
  * CountDownLatch
  * Semaphore
  * CyclicBarrier
  * Exchanger
  * Phaser

> 아래 부분은 이해가 안된다.. 추후에 보완할 예정

<br>

## wait과 notify 메서드
새로운 코드라면 가능한 동시성 유틸리티를 사용하는 것이 좋다.

하지만 `wait`과 `notify`를 사용해야하는 경우라면, 항상 반복문 안에서 사용해야 한다.

```java
synchronized(obj) {
    while (조건이 충족되지 않았다) {
        obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
    }

    ... // 조건이 충족됐을 때의 동작을 수행한다.
}
```
* 반복문의 역할
  * `wait` 호출 전후로 조건이 만족하는지를 검사하는 역할
  * 대기 전에 조건을 검사하여 조건이 충족되었다면 `wait`를 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치다.
  * 만약 조건이 이미 충족되었는데 스레드가 `notify` 또는 `notifyAll` 메서드로 먼저 호출한 후 대기 상태로 빠지면, 그 스레드를 다시 깨우지 못할 수 있다.
  
> 일반적으로 `notify`보다는 `notifyAll`을 사용하는 것이 안전하며, `wait`는 항상 `while`문 내부에서 호출하도록 해야 한다.


