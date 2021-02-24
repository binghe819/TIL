> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

<br>

- [아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라](#아이템-54-null이-아닌-빈-컬렉션이나-배열을-반환하라)
  - [null을 반환하는 건 악몽이다..](#null을-반환하는-건-악몽이다)
  - [간혹 null이 합당하다고 하는 주장이 있는데..](#간혹-null이-합당하다고-하는-주장이-있는데)
  - [배열도 null을 반환하지 말고 길이가 0인 배열을 반환하라](#배열도-null을-반환하지-말고-길이가-0인-배열을-반환하라)
  - [핵심 정리](#핵심-정리)

<br>

## null을 반환하는 건 악몽이다..
```java
// 컬렉션이 비었으면 null을 반환한다. - 따라 하지 말것
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null 
        : new ArrayList<>(cheesesInStock);
}
```
* 재고가 없다고 특별 취급을 하면 안된다. 하지만 **null을 반환하는 것은 특별 취급이다.**
  * **클라이언트가 null 상황을 처리하는 코드를 추가로 작성해줘야하기 때문이다.**

```java
List<Cheese> cheese = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON) {
    System.out.println("좋았어, 바로 그거야!");
})
```
* **위와 같이 null을 반환하는 메서드를 사용할 때면 항시 이와 같은 방어 코드를 넣어줘야한다.**
  * `if (cheeses != null ...)`

<br>

## 간혹 null이 합당하다고 하는 주장이 있는데..
때로는 빈 컨테이너를 할당하는 데도 비용이 드니 **`null`을 반환하는 쪽이 낫다는 주장도 있다.**

**하지만 이는 틀린 주장이다.**
1. 성능 분석 결과 할당이 성능 저하의 주범이 아니라면 성능 차이는 별로 없다.
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다. (캐싱)
   * `Collections.emptyList`, `Collections.emptyMap`...

<br>

## 배열도 null을 반환하지 말고 길이가 0인 배열을 반환하라
```java
// 빈 배열을 반환해야하는 경우
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0]; // 캐싱

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```
* `getCheeses`는 항상 `EMPTY_CHEESE_ARRAY`를 인수로 넘겨 `toArray`를 호출한다.
  * **따라서 `cheesesInStock`이 비었을 때면 언제나 `EMPTY_CHEESE_ARRAY`를 반환한다.**

<br>

## 핵심 정리
* null이 아닌, 빈 배열이나 컬렉션을 반환하라.
  * null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다.
  * 그렇다고 성능이 좋은 것도 아니다.

