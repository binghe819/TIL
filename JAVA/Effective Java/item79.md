> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 79. 과도한 동기화는 피하라

<br>

<br>

## 과도한 동기화는 좋지 않다
과도한 동기화는 성능을 떨어뜨리고, 교착 상태에 빠드리고, 심지어 예측할 수 없는 동작을 낳기도 한다.

**응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안된다.**

* **동기화된 블록 안에서는 재정의 가능한 메서드를 호출해서는 안 된다.**
* **클라이언트가 넘겨준 함수 객체를 호출해서도 안된다.**

이러한 메서드들은 무슨 일을 할지 모르기 때문에 예외를 발생시키거나, 교착상태를 만들거나, 데이터를 훼손할 수 있다.

이러한 메서드를 **외계인 메서드**라 부르기도 한다.

<br>

## 외계인 메서드
예시를 통해 외계인 메서드를 살펴보자.

집합(set)을 감싸고 있는 래퍼 클래스이며, 집합에 원소가 추가되면 알림을 받는 옵저버 패턴을 사용한 예제 코드다.

```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element); // notifyElementAdded 호출
        return result;
    }
}
```
```java
@FunctionalInterface
public interface SetObserver<E> {
    // ObservableSet에 원소가 추가되면 호출된다.
    void added(ObservableSet<E> set, E element);
}
```
* `addObserver`와 `removeObserver`를 통해 관찰자를 추가하거나 제거한다. 

이제 이 외계인 메서드를 통해 잘못된 코드 예제를 살펴본다.

<br>

### 예외 발생
집합에 추가된 정숫값을 출력하다가, 그 값이 23이면 자기 자신을 제거(구독해지)하는 관찰자를 추가한 예시

```java
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
    set.addObserver(new SetObserver<>() {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if (e == 23)
                s.removeObserver(this);
        }
    });

    for (int i = 0; i < 100; i++)
        set.add(i;)
}
```
* 0부터 23까지 출력한 후 자신을 remove 한 후에 종료할 것이다.
* 하지만 실제로 실행해보면 23까지 출력한 다음 `ConcurrentModificationException`을 던진다.
  * **이유는 `added` 메서드 호출이 일어난 시점이 `notifyElementAdded`가 `Observer`들의 리스트를 순회하는 도중이기 때문이다.**
  * `added` 메서드에서 `ObservableSet.removeObserver` 메서드를 호출하고, 또 여기서 `observers.remove` 메서드를 호출하는데 여기서 문제가 발생한다. **순회하고 있는 리스트에서 원소를 제거하려고 하는 것이다.**

> 익명 클래스를 사용한 이유는 람다가 자기 자신을 참조할 수단이 없기 때문이다.

<br>

### 쓸데없는 백그라운드 스레드
실행자 서비스(ExecutorService)를 사용하여 다른 스레드가 Observer를 제거하도록 해보자.

```java
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

    set.addObserver(new SetObserver<>() {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if (e == 23) {
                ExecutorService exec = Executors.newSingleThreadExecutor();
                try {
                    // 여기서 lock이 발생한다. (메인 스레드는 작업을 기리고 있음)
                    // 특정 태스크가 완료되기를 기다린다. (submit의 get 메서드)
                    exec.submit(() -> s.removeObserver(this)).get();
                } catch (ExecutionException | InterruptedException ex) {
                    throw new AssertionError(ex);
                } finally {
                    exec.shutdown();
                }
            }
        }
    })

    for (int i = 0; i < 100; i++) {
        set.add(i);
    }
}
```
* 예상과 달리 예외는 발생하지 않고 교착상태(deadlock)에 빠진다.
  * 백그라운드 스레드가 `s.removeObserver`메서드를 호출하면 `Observer`를 잠그려 시도하지만 락을 얻을 수 없다.
  * 메인 스레드가 이미 락을 잡고 있기 때문이다.
* 외계인 메서드 예제를 정의한 코드를 보면 `removeObserver` 메서드에는 `synchronized` 키워드가 있기 때문에 실행 시 락이 걸린다. 
  * 동시에 메인 스레드는 백그라운 스레드가 `Observer`를 제거하기만 기다리는 중이다. 따라서 교착상태에 빠진다.

<br>

### 두 문제점의 해결방법
외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 된다.

```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized (observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot) {
        observer.added(this, element);
    }
}
```

조금 더 나은 방법으로는 자바의 동시성 컬렉션을 사용하는 것이다.

```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}

```


<br>

## 핵심 정리
* 교착상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자.
* 일반화해 말하면, 동기화 영역 안에서의 작업은 최소한으로 줄이자.
* 가변 클래스를 설계할 때는 스스로 동기화해야 할지 고민하자.
* 멀티코어 세상인 지금은 과도한 동기화를 피하는 게 과거 어느 때보다 중요하다.
* 합당한 이유가 있을 때만 내부에서 동기화하고, 동기화했는지 여부를 문서에 명확히 밝히자.
