> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 58. 전통적인 for 문보다는 for-each문을 사용하라

<br>

- [아이템 58. 전통적인 for 문보다는 for-each문을 사용하라](#아이템-58-전통적인-for-문보다는-for-each문을-사용하라)
  - [배열 및 컬렉션 순회](#배열-및-컬렉션-순회)
  - [for-each](#for-each)
  - [for-each는 Iterable 인터페이스를 구현한 객체라면 모두 순회 가능하다](#for-each는-iterable-인터페이스를-구현한-객체라면-모두-순회-가능하다)
  - [for-each를 사용할 수 없는 상황](#for-each를-사용할-수-없는-상황)
    - [파괴적인 필터링](#파괴적인-필터링)
    - [변형](#변형)
    - [병렬 반복](#병렬-반복)
  - [핵심 정리](#핵심-정리)

<br>

## 배열 및 컬렉션 순회
```java
// 컬렉션 순회
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    // 로직
}

// 배열 순회
for (int i = 0; i < a.length; i++) {
    // a[i] 로직
}
```
* 반복자와 인덱스 변수는 코드를 지저분할 뿐 우리가 진짜 필요한 원소들을 얻기 위한 부수적인 코드다.
  * 진짜 필요한 건 원소들 뿐이다.

<br>

## for-each
```java
// 기존의 for
for (int i = 0; i < arr.length; i++) {
    // arr[i]; 로직
}

// for-each
for (String e : arr) {
    // e 로직
}
```
* for-each문을 향상된 for문이라고 부른다.


```java
// for-each문
for (Transaction transaction : transactions) {
  if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
    transactions.remove(transaction);
  }
}

// for-each문의 실제 구동 코드
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext(); ) {
  Transaction transaction = iterator.next();
  if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
    transactions.remove(transaction);
  }
}
```


<br>

## for-each는 Iterable 인터페이스를 구현한 객체라면 모두 순회 가능하다
```java
// Iterable
public interface Iterable<E> {
  // 이 객체의 원소들을 순회하는 반복자를 반환한다.
  Iterator<E> iterator();
}
```

<br>

## for-each를 사용할 수 없는 상황

<br>

### 파괴적인 필터링
* 파괴적인 필터링: 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다. 그러므로 반복자를 사용해야 한다.
  * for-each에서 리스트 자체를 수정하게 되면 `ConcurrentModificationException`이 발생한다.
* 하지만, 자바 8에서 생긴 `Collection`의 `removeIf` 메서드를 사용하여 삭제할 수 있다.
* 변형: 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야한다.

<br>

### 변형
```java
@Test
void foreach_test() {
    // given
    List<String> list = new ArrayList<>();
    list.add("a"); list.add("b"); list.add("c");

    assertThatThrownBy(() -> {
        for (String s : list) {
            if (s.equals("a")) {
                list.remove(s);
            }
        }
    }).isInstanceOf(ConcurrentModificationException.class);
}
```
* 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.

<br>

### 병렬 반복
* 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자나 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

<br>

## 핵심 정리
* 전통적인 for 문과 비교했을 때 for-each문은 명료하고, 유연하고, 버그를 예방해준다.
* 성능 저하도 없다.
* 가능한 for-each문을 사용하자.
