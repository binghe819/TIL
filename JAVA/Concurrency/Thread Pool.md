# 목차

<br>

<br>

# 들어가며

<br>

# ThreadPool이 등장하게 된 배경

<br>

❗️ **ThreadPool을 사용하지 않는다면**

많은 프로그램은 스레드를 이용하여 병렬 (혹은 병행)처리를 한다.

문제는 점차 필요한 스레드의 개수가 많아지면서, 잦은 스레드 생성과 스케줄링으로 인해 CPU와 메모리 사용량이 증가한다.

각 스레드는 메모리(RAM)과 같은 특정 수준의 컴퓨터 리소스를 사용하기 때문에 동시에 너무 많은 스레드가 활성된다면 컴퓨터의 속도가 저하된다.

또한 스레드의 생성 개수를 제어하기 힘들기 때문에 자칫하면 컴퓨터가 멈출 수도 있다.

> 예를 들어, 메모리(RAM)이 너무 많이 소모되어 OS가 RAM을 Disk로 스왑아웃하기 시작하면 속도는 현저히 느려진다.

스레드 생성뿐만 아니라 회수도 해야한다..

<br>

👍 **ThreadPool을 사용한다면**

**ThreadPool의 핵심은 `스레드 재사용`이다.**

ThreadPool을 사용하지 않는 기존의 Thread 전략은 한번 사용하면 종료되고 다시 만들어애했다.

하지만 ThreadPool은 정해진 숫자만큼 만들어둔 **기존 스레드를 종료시키지않고 계속해서 작업을 맡겨 실행하게 된다.**

> 스레드를 미리 만들어두고 작업이 들어올면 스레드들에게 작업을 적절히 분배한다.

<br>

🤔 **그래서 뭐가 좋다는건데? - Thread Pool의 장점**

결론적으론 정해진 숫자의 기존 스레드를 재사용하기 때문에 리소스 사용량을 절약하고, 총 처리량도 더 높일 수 있다.

1. 스레드 풀을 사용하면 한 번에 활성 상태인 스레드 수를 쉽게 제어할 수 있다. 이를 통해 스레드의 생성과 회수로 인한 부하를 줄일 수 있다.
2. 한 번에 많은 요청이 들어올 때 빠르고 효율적으로 동시 작업을 수행할 수 있다.

<br>

❗️ **그렇다고 좋은 것만은 아니다 - Thread Pool의 단점**

1. 메모리 낭비
   * ThreadPool은 Thread를 미리 만들어두고 재사용한다는 측면에서 굉장히 유용한 솔루션이다.
   * 하지만 **몇 개의 Thread를 미리 만들어둘지가 중요하다. 만약 너무 많은 스레드를 만들어둔다면 메모리만 차지하고 아무것도 하지 않는 스레드가 존재할 수도 있다.**
2. 놀고먹는 스레드 발생 가능성
   * 병렬적으로 처리요청을 한다면 한 스레드는 요청을 처리하느라 허덕이고 있는와중에 다른 스레드는 놀고있을 수 있다.
   * 이로인해 미리 만들어둔 스레드를 잘 활용하지 못할 수도 있는 문제가 있다.
   * 물론 이는 자바에서는 `ForkJoinPool`을 지원한다.
3. DeadLock
   * ThreadPool만의 문제라기보다는 멀티스레딩 환경에서 발생할 수 있는 대표적인 문제점은 데드락이다. ThreadPool도 데드락이 발생할 수 있다.
   * ThreadPool에 경우, 만약 하나의 Task의 결과를 기다리는 또 다른 Task를 Queue에 넣으면 데드락이 발생할 확률이 높다.
4. 스레드 없어짐 문제
   * ThreadPool에 미리 생성된 스레드가 요청을 처리하고나서 다시 대기상태가 안되고 제거될 수 있다. (재사용 불가)
   * 예를 들어, 스레드에 할당된 Task가 예외를 던졌는데 해당 예외를 잡지 못한다면 Thread 자체가 종료되기 때문에 ThreadPool에서 스레드의 개수가 하나씩 줄어들 수 있다.

<br>

# ThreadPool은 어떻게 동작하는가?

<p align="center"><img src="./image/thread_pool_workflow.png"><br>출처: https://www.logicbig.com/tutorials/core-java-tutorial/java-multi-threading/thread-pools.html </p>

사실 위 그림만봐도 ThreadPool이 어떻게 동작하는지 알 수 있다. 그래도 조금 글로 정리하자면 다음과 같다.

* ThreadPool에 요구에 맞는 스레드를 재사용하기 위해 미리 생성해둔다. (New ThreadPool)
* 새로운 작업이 들어올 때마다 TaskQueue에 넣는다. (New Task)
* TaskQueue에서 하나씩 꺼내서 분할 방식에따라 유휴 스레드에 작업을 할당한다. (Thread Assignment)
* 각 작업이 완료되면, 콜백 형태로 작업을 요청한 주체에게 결과를 알려준다. (Task Finished)

> ThreadPool은 어렵고 복잡한 기술이라기보다는 그저 Thread를 잘 활용할 수 있게해주는 디자인패턴에 가깝다고 생각한다.

<br>

# Thread Pool In Java
자바는 5버전부터 자체적으로 Thread Pool을 지원하기 시작했다. `java.util.concurrent` 패키지에 ThreadPool과 관련된 클래스가 존재한다.

자바에서의 ThreadPool도 아래와 같이 동일하게 동작한다.

<p align="center"><img src="./image/thread_pool_in_java.png" width="500"><br>출처: https://www.baeldung.com/thread-pool-java-and-guava </p>

스레드 풀의 실제 구현에서 코드를 분리한 상태로 유지하고 애플리케이션 전체에서 이러한 인터페이스를 사용해야 합니다.

자바는 ThreadPool과 관련된 코드를 비즈니스 로직과 분리시키기위해 `Executor`과 `ExecutorService`라는 인터페이스로 추상화시켰으며, `Executors`라는 유틸성 클래스를 만들어 편의 메서드를 제공한다. 

<br>

## Executor vs ExecutorService

자바에서는 ThreadPool과 관련된 코드를 `Executor`과 `ExecutorService`로 추상화시켰다. 그렇다면 이 둘의 차이점은 무엇일까?

<br>

🤔 `Executor`

```java
public interface Executor {
    // Executes the given command at some time in the future. 
    // The command may execute in a new thread, in a pooled thread, or in the calling thread, at the discretion of the Executor implementation.
    void execute(Runnable command);
}
```

**`Executor`는 넘겨받은 Runnable Task를 실행하는 메서드를 가진 간단한 인터페이스다.**

인터페이스로 따로 빼둔 이유는 넘겨받은 Task를 어떻게 스레드가 실행할지, 스레드 스케줄링은 어떻게 할지등은 추상화시키기 위함이다.

쉽게 얘기하자면 기존엔 `new Thread(new RunnableTask()).start()`해야했던 작업을 아래와 같이 추상화 시킨것이다.

```java
Executor executor = anExecutor;
executor.execute(new RunnableTask1()); // 새로운 Task를 어떻게 처리할지는 Executor 구현체에 따라 다르다.
executor.execute(new RunnableTask2());
```

> 한마디로하면 OCP를 사용한 것.

<br>

**이렇게 추상화시킴으로써 아래와같이 상황에 맞게 Executor 구현체를 만들어 Task를 처리할 수 있기 때문이다.**

1. 동기적으로 Task 처리 (Thread를 활용한 비동기를 사용하지 않음)

```java
class DirectExecutor implements Executor {
    public void execute(Runnable r) {
        r.run();
    }
}
```

2. 매 Task마다 새로운 스레드를 생성하여 처리

```java
class ThreadPerTaskExecutor implements Executor {
    public void execute(Runnable r) {
        new Thread(r).start();
    }
}
```

3. Task를 특정 스레드에 스케줄링할 때 컴포지트 혹은 프록시 패턴과 같이 사용하여 처리할 수 있다.

```java
class SerialExecutor implements Executor {
    final Queue<Runnable> tasks = new ArrayDeque<Runnable>();
    final Executor executor;
    Runnable active;

    SerialExecutor(Executor executor) {
        this.executor = executor;
    }

    public synchronized void execute(final Runnable r) {
        tasks.offer(new Runnable() {
           try {
              r.run();
           } final {
              // 매 Task마다 끝나면 다음 Task 호출.
              scheduleNext();
           }
        });

        // 만약 현재 active(실행)되고있는 Task가 없다면 다음 Task 호출.
        if (active == null) {
           scheduleNext();
        }
    }

    // TaskQueue에서 다음 Task를 꺼내서 스레드에 스케줄링. (실행)
    protected synchronized void scheduleNext() {
        if ((active == tasks.poll() != null)) {
            executor.execute(active);
        }
    }
}
```

위와 같이 `Executor`라는 인터페이스를 분리시킴으로써 상황에 맞는 구현체를 만들거나 조합해서 다양하게 사용할 수 있다.

<br>

🤔 `ExecuteService`

```java
public interface ExecutorService extends Executor {

    void shutdown();

    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();

    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> Future<T> submit(Callable<T> task);

    <T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

`ExecuteService`는 Task를 관리하고 제어하는데 사용되는 Execute 인터페이스의 하위 인터페이스이다.

쉽게 말해 `Execute`보다 더 확장된 기능을 제공하기위한 인터페이스이다.

ThreadPool을 종료하는 메서드와 하나 이상의 비동기 작업을 추적하기 위한 `Future`를 반환하는 메서드를 제공한다.






<br>

# 참고
* https://en.wikipedia.org/wiki/Thread_pool
* http://tutorials.jenkov.com/java-concurrency/thread-pools.html
* https://www.javatpoint.com/java-thread-pool

* ThreadPool이 등장하게 된 배경
* ThreadPool은 어떻게 동작하는가?
* Thread Pool In Java
  * 
* Java로 ThreadPool 직접 구현해보기

