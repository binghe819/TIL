> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 78. 공유 중인 가변 데이터는 동기화해 사용하라

<br>

- [아이템 78. 공유 중인 가변 데이터는 동기화해 사용하라](#아이템-78-공유-중인-가변-데이터는-동기화해-사용하라)
  - [동기화 synchronized](#동기화-synchronized)
    - [동기화가 필요한 이유](#동기화가-필요한-이유)
    - [동기화의 의미](#동기화의-의미)
  - [동기화의 대한 오해](#동기화의-대한-오해)
    - [쓰기와 읽기 모두 동기화할 필요가 없다??](#쓰기와-읽기-모두-동기화할-필요가-없다)
  - [성능을 높이고 싶다면](#성능을-높이고-싶다면)
    - [volatile](#volatile)
    - [java.util.concurrent.atomic 패키지](#javautilconcurrentatomic-패키지)
  - [결론은 가변 데이터는 단일 스레드에서만 쓰도록 하자.](#결론은-가변-데이터는-단일-스레드에서만-쓰도록-하자)
  - [핵심 정리](#핵심-정리)

<br>

## 동기화 synchronized

<br>

### 동기화가 필요한 이유
<p align="center"><img src="./image/image-20200721035118255.png"></p>

* 실행 결과를 보면 잔고(balance)가 마이너스가 되는 것을 볼 수 있다.
  * 그 이유는 한 스레드가 if문의 조건식을 통과하고 출금하기 바로 직전에 다른 스레드가 끼어들어서 출금을 먼저 했기 때문이다.

<br>

### 동기화의 의미
* synchronized
  * synchronized 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다.
* 동기화의 단편적 의미 => **배타적 실행**
  * 많은 사람들은 **동기화를 배타적 실행, 즉 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도로만 생각한다.**
  * 즉, **동기화란 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다. 어떤 메서드도 상태가 일관되지 않은 순간을 볼 수 없다.**
* 위 설명도 맞지만, 중요한 기능이 하나 더 있다. => **스레드 간 통신 목적**
  * **동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.**
  * 즉, **동기화는 일관성이 깨진 상태를 볼 수 없게 한다.**

<br>

## 동기화의 대한 오해

<br>

### 쓰기와 읽기 모두 동기화할 필요가 없다??
* 언어 명세상 `long`과 `double` 외의 변수를 읽고 쓰는 동작은 원자적(atomic)이다.
  * 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 의미다.
* 이 말을 듣고 **성능을 높이려면 원자적 데이터를 읽고 쓸 때는 동기화하지 말아야겠다**란 오해를 한다.
  * **자바 언어 명세상 스레드가 필드를 읽을 때 항상 '수정이 완전히 반영된' 값을 얻는다고 보장하지만, 한 스레드가 저장한 값이 다른 스레드에게 '보이는가'는 보장하지 않는다.**
* 즉, **동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.**

<br>

**예시**
```java
public class StopThread {
    private static boolean stopRequest;

    public static void main(String[] args) {
        Thread backgoundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequest) {
                i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequest = true;
    }
}
```
* 위 코드를 실행하면 1초 후에 스레드가 종료될 줄 알지만, 계속해서 진행하게 된다.
* 원인을 동기화에 있다.
  * **동기화를 하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 될지 보증할 수 없다.**
  * 또한, 최적화에 따라 끌어올리기 기법으로 인해 아예 끝내기 못할 경우도 발생하게 된다.

```java
// 아래와 같이 동기화를 시켜주면 1초후에 종료된다. (동기화가 스레드 간 통신 목적으로만 사용되었다.)
public class StopThread {
    private static boolean stopRequest;

    private static synchronized void requestStop() {
        stopRequest = true;
    }

    private static synchronized boolean stopRequest() {
        return stopRequest;
    }

    public static void main(String[] args) {
        Thread backgoundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequest()) {
                i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```
* **쓰기 메서드 (`requestStop`)과 읽기 메서드 (`stopRequest`) 모두를 동기화했음에 주목하자!**
  * **쓰기와 읽기 모두가 동기화되지 않으면 동작을 보장하지 않는다.**

> 여기서 중요한 점은 동기화는 두 가지 역할을 한다는 것이다.
> * 배타적 수행
> * 스레드 간 통신

<br>

## 성능을 높이고 싶다면
매번 동기화는 비용이 크다고 생각든다면 다음의 대안을 사용해보자.

<br>

### volatile
```java
public class StopThread {
    private static volatile stopRequest;

    public static void main(String[] args) {
        Thread backgoundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequest) {
                i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequest = true;
    }
}
```
* **volatile은 배터적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.**
  * 더 자세한 내용은 [여기](https://github.com/binghe819/TIL/blob/master/JAVA/%EC%9E%90%EB%B0%94%EC%9D%98%20%EC%A0%95%EC%84%9D/Thread.md#8-5-volatile)를 참고.
* volatile 주의점
  * 안전 실패가 발생할 수 있다.

<br>

### java.util.concurrent.atomic 패키지
* 위 패키지안에 있는 객체들은 락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다.
  * 이 패키지의 객체들은 스레드 간 통신 뿐만 아니라 배타적 실행(원자성)까지 지원한다.

<br>

## 결론은 가변 데이터는 단일 스레드에서만 쓰도록 하자.
* **이번 아이템에서 언급한 문제들을 피하는 가장 좋은 방법은 애초에 가변 데이터를 공유하지 않는 것이다.**
  * **불변 데이터만 공유하거나 아무것도 공유하지 말자.**
  * 다시 말해, 가변 데이터는 단일 스레드에서만 쓰도록 하자.

<br>

## 핵심 정리
* 여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화 해야 한다.
  * 동기화하지 않으면 한 스레드가 수행한 변경을 다른 스레드가 보지 못할 수도 있다.
* 공유되는 가변 데이터를 동기화하는 데 실패하면 응답 불가 상태에 빠지거나 안전 실패로 이어질 수 있다.
  * 이는 디버깅 난이도가 가장 높은 문제에 속한다.
* 배타적 실행은 필요 없고 스레드끼리의 통신만 필요하다면 `volatile`한정자만으로 동기화할 수 있다.
  * 다만 올바로 사용하기가 까다롭다.
