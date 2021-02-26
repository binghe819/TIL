> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 55. 옵셔널 반환은 신중하라

<br>

- [아이템 55. 옵셔널 반환은 신중하라](#아이템-55-옵셔널-반환은-신중하라)
  - [Optional이 나오기 전](#optional이-나오기-전)
  - [Optional이란](#optional이란)
  - [Optional을 사용해야하는 이유](#optional을-사용해야하는-이유)
    - [기본값 설정 가능](#기본값-설정-가능)
    - [원하는 예외를 던질 수 있다.](#원하는-예외를-던질-수-있다)
    - [항상 값이 채워져 있는 경우](#항상-값이-채워져-있는-경우)
    - [기본값을 설정하는 비용이 큰 경우](#기본값을-설정하는-비용이-큰-경우)
  - [Optional의 안티 패턴](#optional의-안티-패턴)
    - [Collection, Stream, 배열은 감싸지말자](#collection-stream-배열은-감싸지말자)
    - [Optional을 Map의 키나 값으로 사용하지 말자](#optional을-map의-키나-값으로-사용하지-말자)
    - [isPresent()보단 다른 대안을 사용하자](#ispresent보단-다른-대안을-사용하자)
  - [이외에](#이외에)
  - [핵심 정리](#핵심-정리)

<br>

## Optional이 나오기 전
자바 8의 `Optional`이 나오기 전 특정 조건에서 값을 반환할 수 없을때 취할 수 있는 선택지는 두 가지였다.
* 예외 던지기
  * 예외는 반드시 예외적인 상황에서만 사용해야 한다.
  * 예외는 실행 스택을 추적하고 생성하는 비용이 비싸다.
* null 반환
  * 클라이언트 측에서 null에 대한 처리가 필요하다.

<br>

## Optional이란
> 더 자세한 내용은 [여기](https://github.com/binghe819/TIL/blob/master/JAVA/%EB%AA%A8%EB%8D%98%20%EC%9E%90%EB%B0%94/Optional.md)를 참고.

<br>

## Optional을 사용해야하는 이유
**Optional은 검사 예외와 취지가 비슷하다.**
즉, 반환 값이 있을 수도 있고, 없을 수도 있음을 클라이언트에게 명확히 알려준다.

<br>

### 기본값 설정 가능
```java
String lastWordInLexicon = max(words).orElse("단어 없음..");
```

<br>

### 원하는 예외를 던질 수 있다.
```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```
* 값이 없는 경우 원하는 예외를 던질 수 있다.
  * 예외가 실제 발생하지 않는 한 예외 생성 비용이 들지 않는다.

<br>

### 항상 값이 채워져 있는 경우
```java
Element lasetNobleGas = max(Elements.NOBLE_GASES).get();
```
* 옵셔널에 항상 값이 있음을 확신할 때 사용해야 한다.
  * 값이 없다면 `NoSuchElementException`이 발생한다.

<br>

### 기본값을 설정하는 비용이 큰 경우
```java
Optional<Member> member = ...;
return member.orElseGet(Member::new); // member에 값이 없을 때만 new Member()가 실행됨.
```
* 기본값 설정 비뵵이 커서 부담이라면 `orElseGet`을 사용하자.
  * 값이 처음 필요할 때 `Supplier<T>`를 사용하여 생성하므로 초기 설정 비용을 낮출 수 있다.

<br>

## Optional의 안티 패턴

<br>

### Collection, Stream, 배열은 감싸지말자
`Optional<List<T>>`를 반환하는 것보다 빈 `List<T>`를 반환하는 것이 좋다.

즉, 두번이나 감쌀 필요가 없다는 의미다.

<br>

### Optional을 Map의 키나 값으로 사용하지 말자
만약 Optional을 Map에서 사용한다면 모호한 상황이 발생한다.
* Key 자체가 없는 경우
* Key는 있지만, 속이 빈 Optional인 경우

쓸데없이 복잡도만 높아지게 되고 전혀 쓸모없는 짓이니 사용하지 말자.

<br>

### isPresent()보단 다른 대안을 사용하자
```java
// isPresent() 코드
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID: " + (parentProcess.isPresent() ? String.valueOf(parentProcess.get().pid()) : "N/A"));

// orElse
System.out.println("부모 PID: " + ph.parent().map(h -> String.valueOf(h.pid()).orElse("N/A"));
```
* `isPresent()`를 사용하는것 보단 `orElse`를 사용하는 것이 훨씬 간결하다.

<br>

## 이외에
* 박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무거울 수밖에 없다.
  * 그러므로 `OptionalInt`, `OptionalLong`, `OptionalDouble`등을 준비했다.
* 값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야 하는 메서드라면 `Optional`을 반환해야 하는 상황일 수 있다.
* `Optional`을 반환 값 이외의 용도로 쓰는 경우는 거의 없다.

<br>

## 핵심 정리
* 값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야하는 메서드라면 옵셔널을 반환해야 할 상황일 수 있다.
* 하지만 옵셔널 반환에는 성능 저하가 뒤따르니, 민감하면 `null`을 반환하거나 예외를 던지는 편이 나을 수 있다.
* 옵셔널을 반환갑 이외의 용도로 쓰는 경우는 극히 드물다.