# 목차

- [ThreadLocal](#threadlocal)
  * [1 ThreadLocal이란?](#1-threadlocal이란)
  * [2 ThreadLocal 사용해보기](#2-threadlocal-사용해보기)
    + [2-1 생성및 get/set](#2-1-생성및-getset)
    + [2-2 사용 팁](#2-2-사용-팁)
  * [3 ThreadLocal 주의할 점](#3-threadlocal-주의할-점)
    + [3-1 ThreadPool에서의 ThreadLocal](#3-1-threadpool에서의-threadlocal)
  * [4 Spring에서의 ThreadLocal](#4-spring에서의-threadlocal)
    + [4-1 SpringSecurity에서의 ThreadLocal](#4-1-springsecurity에서의-threadlocal)
    + [4-2 트랜잭션](#4-2-트랜잭션)





# ThreadLocal



## 1 ThreadLocal이란?

🤔 ThreadLocal?

* `Java.lang` 패키지에서 제공하는 쓰레드 범위 변수.  **한 스레드에서 공유할 변수.**
* **쓰레드 수준의 데이터 저장소이다.**
  * 같은 쓰레드 내에서만 공유
  * **따라서 같은 쓰레드라면 해당 데이터를 메서드 매개변수로 넘겨줄 필요 없다.**



## 2 ThreadLocal 사용해보기



### 2-1 생성및 get/set

```java
// 쓰레드 로컬 선언
ThreadLocal<String> THREAD_LOCAL = new ThreadLocal<String>();

// set
THREAD_LOCAL.set("Hello World");

// get
THREAD_LOCAL.get(); // Hello world
```



### 2-2 사용 팁

ThreadLocal을 제대로 사용하려면 유틸같이 Context를 만들어서 사용하는 것이 좋을 것 같다.

```java
class ThreadContext {
  // 선언
  private static final ThreadLocal<String> THREAD_LOCAL = new ThreadLocal<>();
  
  // set (이름)
  public static void setName(String name) {
    THREAD_LOCAL.set(name);
  }
  
  // get (이름)
  public static String getName() {
    return THREAD_LOCAL.get();
  }
  
  // set (별명)
  public static void setNickname(String nickname) {
    THREAD_LOCAL.set(nickname);
  }
  
  // get (별명)
  public static String getNickname() {
    return THREAD_LOCAL.get();
  }
  
}
```





## 3 ThreadLocal 주의할 점



### 3-1 ThreadPool에서의 ThreadLocal

> 너무 궁금한 내용들을 잘 정리한 사이트가 있어서 URL로 남긴다.

ThreadPool을 사용한다면 할당 받은 Thread로 Task를 모두 완료한다면 ThreadLocal 변수는 그대로 남는가??

* [Will the ThreadLocal object be cleared after thread returned to Thread Pool? - stackoverflow](https://stackoverflow.com/questions/30328437/will-the-threadlocal-object-be-cleared-after-thread-returned-to-thread-pool)
* [ThreadPool과 ThreadLocal을 같이 사용할 때 주의할 점](https://www.baeldung.com/java-threadlocal#threadlocalsand-thread-pools)





## 4 Spring에서의 ThreadLocal



### 4-1 SpringSecurity에서의 ThreadLocal

> [백기선님의 SpringSecurity]()강의를 참고한 내용입니다.

SpringSecurity에서는 ThreadLocal을 기본 전략으로 SecurityContextHolder를 사용한다.

서블릿은 기본적으로 쓰레드 기반으로 돌아가기 때문에 ThreadLocal을 사용하기 용이하다.

```java
// 한 요청(스레드)안에서 사용할 컨텍스트를 ThreadLocal로 구현한 클래스.
public class AccountContext {

  	// static이므로 애플리케이션 실행시 초기화 된다.
    private static final ThreadLocal<Account> ACCOUNT_THREAD_LOCAL = new ThreadLocal<>();

    public static void setAccount(Account account) {
        ACCOUNT_THREAD_LOCAL.set(account);
    }

    public static Account getAccount() {
        return ACCOUNT_THREAD_LOCAL.get();
    }

}
```

* `ThreadLocal` 를 사용한 `ccountContext`를 만들어줘서 한 요청(스레드)에 대해서 애플리케이션 내에 여러 위치에서 사용하고자 하는 변수를 담을 수 있다.



> ServletConfig도 ThreadLocal 형식으로 생성되는 것인가?? 라고 추측해본다..



### 4-2 트랜잭션

트랜잭션 매니저에서 transaction Context를 전파하는데 사용된다.





























