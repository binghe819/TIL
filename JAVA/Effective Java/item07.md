> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 7. 다 쓴 객체 참조를 해제하라

C, C++처럼 메모리를 직접 관리해야 하는 언어가 아닌 자바는 GC가 자동적으로 메모리 관리를 해준다.

자바에 GC가 있기 때문에 메모리 관리에 대해 신경쓰지 않아도 될거라고 생각하기 쉽지만 그렇지 않다.

<br>

## 메모리를 직접 관리하는 객체

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
  
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
  
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }
  
    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

스택에 데이터를 계속 쌓다가 많이 빼냈다고 가정한다면, 그래도 스택이 차지하고 있는 메모리는 줄어들지 않는다.

그 이유는 위 스택의 구현체는 필요없는 객채에 대한 참조를 그대로 가지고 있기 때문이다. 

가용한 범위는 `size` 보다 작은 부분이고 그 값보다 큰 부분에 있는 값들은 필요없이 메모리를 차지하고 있는 부분이다.

( 새로운 값은 덮어씌우는 형식으로 사용된다. )

<br>

해법은 간단하다. 해당 참조를 다 썼을 때 `null`처리하면 된다.

```java
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
      
        Object value = this.elements[--size];
        this.elements[size] = null; // 데이터 참조 끊기
        return value;
    }
```

스택에서 꺼낼 때 그 위치에 있는 객체를 꺼내주고 그 자리를 `null` 처리해줌으로써 다음 GC가 발생할 때 참조를 정리하게 된다.

<br>

그렇다고 필요없는 객체를 볼 때 마다 `null`로 설정하는 코드를 작성하지는 말아라. **객체를 `null`로 설정하는 건 예외적인 상황에서나 하는것이지 평범한 일이 아니다.** 필요없는 객체 레퍼런스를 정리하는 최선책은 그 레퍼런스를 가리키는 변수를 특정한 범위(스코프) 안에서만 사용하는 것이다. (로컬 변수는 그 영역을 넘어가면 자동적으로 정리된다.) **변수를 가능한 가장 최소의 범위로 사용하면 자연스럽게 그렇게 된다.**

그럼 언제 참조를 `null`로 설정해야 할까? **메모리를 직접 관리 할 때**이다.

`Stack` 구현체처럼 `elements`라는 배열을 관리하는 경우에 GC는 어떤 객체가 필요없는 객체인지 알 수 없다. 오직 프로그래머만 `elemetns`에서 가용한 부분(size 보다 작은 부분)과 필요 없는 부분(size 보다 큰 부분)을 알 수 있다. 따라서, 프로그래머가 해당 참조를 `null`로 만들어서 GC한테 필요없는 객체들이라고 알려줘야 한다.

**메모리를 직접 관리하는 클래스는 프로그래머가 메모리 누수를 조심해야 한다.**

<br>

## 캐시

**캐시 역시 메모리 누수를 일으키는 주범이다.** 객체 참조를 캐시에 넣고 캐시를 비우는 것을 잊기 쉽다. 여러가지 해결책이 있는지만, 캐시의 키에 대한 참조가 캐시 밖에서 필요 없어지면 해당 엔트리를 캐시에서 자동으로 비워주는 `WeakHashMap`을 쓸 수 있다.

또는 특정 시간이 지나면 캐시값이 의미가 없어지는 경우에 백그라운드 쓰레드를 사용하거나 (아마도 `ScheduledThreadPoolExecutor`), 새로운 엔트리를 추가할 때 부가적인 작업으로 기존 캐시를 비우는 일을 할 것이다. (`LinkedHashMap` 클래스는 `removeEldestEntry`라는 메서드를 제공한다.)

<br>

## 콜백

**세번째로 흔하게 메모리 누수가 발생할 수 있는 지점은 리스너와 콜백이 있다.**

클라이언트 코드가 콜백을 등록할 수 있는 API를 만들고 콜백을 뺄 수 있는 방법을 제공하지 않는다면, 계속해서 콜백이 쌓이게 될 것이다. 이것 역시 `WeakHashMap`을 사용해서 해결할 수 있다.

**메모리 누수는 발견하기 쉽지 않기 때문에 수년간 시스템에 머물러 있을 수도 있다.** 코드 인스택션이나 `heap profiler` 같은 디버깅 툴을 사용해서 찾아야 한다. 따라서 **이런 문제를 예방하는 방법을 학습하여 미연에 방지하는 것이 좋다.**

<br>

## WeakHashMap이란

http://blog.breakingthat.com/2018/08/26/java-collection-map-weakhashmap/