> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 72. 표준 예외를 사용하라

<br>

<br>

## 표준 예외의 장점
* 내가 작성한 API가 다른 사람이 익히고 사용하기 쉬워진다.
* 내가 작성한 API를 사용한 프로그램도 낯선 예외를 사용하지 않게 되어 읽기 쉽게 된다.
* 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다.

> 장점이라고 써놓았지만.. 크게 장점으로 다가오지는 않는다. 너무 당연한 소리를 하기 때문일지도

<br>

## 많이 사용되는 표준 예외
|예외|주요 쓰임|
|---|---|
|IllegalArgumentException|허용하지 않는 값이 인수로 건네졌을 때<br>(null은 따로 NullPointerException으로 처리)|
|IllegalStateException|객체가 메서드를 수행하기에 적절하지 않은 상태일 때|
|NullPointerException|null을 허용하지 않는 메서드에 null을 건넸을 때|
|IndexOutOfBoundsException|인덱스가 범위를 넘어섰을 때|
|ConcurrentModificationException|허용하지 않는 동시 수정이 발견됐을 때<br>(단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려 할 때)|
|UnsupportedOperationException|호출한 메서드를 지원하지 않을 때<br>(클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때)|

<br>

## 주의 사항
* Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말자.
  * 추상 클래스라고 생각하길 바란다고 한다.
  * 그 이유는 이 클래스들은 여러 성격의 예외들을 포괄하는 클래스이므로 안정적으로 테스트할 수 없기 때문이다.
* 확장시 주의할 점
  * 예외는 직렬화할 수 있다는 사실을 기억하자([아이템 12](./item12.md))
  * 이 사실만으로도 나만의 예외를 새로 만들지 않아야 할 근거로 충분할 수 있다.
