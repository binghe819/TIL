> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 29. 이왕이면 제네릭 타입으로 만들라

<br>

> 요약 : 클라이언트가 사용할 때 제네릭을 사용하지 않으면 매번 형변환을 해야하고, 그 과정에서 런타임 에러가 나는 것을 방지하기 위해서 제네릭 타입으로 만들어야 한다.

<br>

## Object배열로 이뤄진 스택의 문제
```java
// Object 배열로 이뤄진 스택 정의
public class Stack {

    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        // ensureCapacity
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    // ...
}

// Object 배열로 이뤄진 스택 사용
int pushValue = 10;

Stack stack = new Stack();
stack.push(pushValue);

String result = (String)stack.pop(); // ClassCastException 예외 발생 (Integer -> String 형변환 예외)
```
Object 배열로 이뤄진 스택은 위와 같이 `pop`을 할 때마다 형변환을 해줘야 한다.

만약 **형변환 과정에서 예외가 발생한다면 런타임 예외가 발생한다.**

<br>

## 제네릭으로 이뤄진 스택
Object로 이뤄진 스택을 제네릭 타입으로 변경하면 아래와 같다.

<br>

### 실체화 문제
```java
// 컴파일되지 않는다.
public class Stack<E> {

    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY]; // 컴파일 에러 (실체화 불가)
    }

    public void push(E e) {
        // ensureCapacity
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    // ...
}
```
E와 같은 실체화 불가 타입으로는 배열을 만들 수 없기 때문에 컴파일 예외가 발생한다. ([아이템 28](./item28.md)참고)

이를 해결하는 방법은 두 가지 있다.

<br>

### 첫번째 해결법 : 제네릭 배열 생성 금지 제약을 우회
**Object 배열을 생성한 다음 제네릭 배열로 형변환을 하는 것.**

```java
// 이제 컴파일러는 오류 대신 경고를 내보내게 된다. 
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

이 방법은 **컴파일러가 컴파일할 수 있게 하지만 컴파일러가 타입 안전한지까진 검사하지 못한다. (그래서 경고 내보냄)**

**컴파일러가 타입 안전한지 확인할 순 없지만, 개발자는 확인할 수 있기 때문에 이렇게 사용을 해도 되긴 하다고 한다.**

```java
// 실제로 사용하고자 하면 컴파일러가 예외를 잡아준다.
Integer pushValue = 10;

Stack<Integer> stack = new Stack<>();
stack.push(pushValue);

String result = stack.pop(); // 컴파일 에러
```

<br>

위와 같이 타입 안전함이 직접 증명된다면 범위를 최소로 좁혀 `@SuppressWarnings` 애너테이션으로 해당 경고를 숨겨준다. ([아이템 27](./item27.md))
```java
// 배열 elements는 push(E)로 넘어온 E 인스턴스만 남는다.
// 따라서 타입 안전성을 보장하지만,
// 이 배열을 런타임 타입은 E[]가 아닌 Object[]다!
@SuppressWarnings("unchecked")
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

<br>

#### 장점과 단점
* 장점: 필드에 E 배열 타입을 가지고 있음으로써 이 Stack은 E 타입의 인스턴스만 받을 수 있음을 확신할 수 있도록 한다.
* 단점: **힙 오염**. 런타임타입: E, 컴파일타임 타입: Object


<br>

### 두번째 해결법 : 필드를 Object[] 으로 변경
```java
public class Stack<E> {
    private Object[] elements;
    ...

    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }

    ...

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size]; // 타입 에러! (Object를 E타입으로 형변환해줘야 한다.)
        elements[size] = null;
        return result;
    }
} 
```
elements를 아예 Object로 사용하는 방법이다.

`pop`에서 형변환으로 인한 컴파일 에러가 발생한다.

**pop 메서드에 형변환과 @SuppressWarnings 를 추가하면 해결된다.**

```java
public E pop() {
    if (size == 0)
        throw new EmptyStackException();
    @SuppressWarnings("unchecked") E result = (E) elements[--size];

    elements[size] = null;
    return result;
}
```

<br>

#### 장점과 단점
* 장점: 힙오염 없음
* 단점: 원소를 pop 할 때마다 형변환


<br>

## 핵심 정리
* 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.
  * 그러니 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하자.