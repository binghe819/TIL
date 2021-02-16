> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 46. 스트림에서는 부작용 없는 함수를 사용하라

<br>

<br>

## 1 함수형 프로그래밍

<br>

### 1-1 스트림은 함수형 프로그래밍이다
* 일반적인 코드와 스트림을 사용한 코드를 보면 가독성외에는 왜 사용하는지 모르는 경우가 많다. (나..)
  * 즉, 스트림의 장점이 무엇인지 쉽게 와 닿지 않는다.
* 스트림은 함수형 프로그래밍에 기초한 패러다임이다.
  * **스트림이 제공하는 표현력, 속도, 병렬성을 얻으려면 함수형 패러다임을 받아들여야한다.**
  * 스트림 API는 데이터를 추상화하여 다루므로, **다양한 방식으로 저장된 데이터를 읽고 쓰기 위한 공통된 방법을 제공합니다.**
    * 따라서 스트림 API를 이용하면 배열이나 컬렉션뿐만 아니라 파일에 저장된 데이터도 모두 같은 방법으로 다룰 수 있게 됩니다.

<br>

### 1-2 순수함수
* **스트림의 핵심은 계산을 일련의 변환(transformation)으로 재구성하는 부분이다.**
  * **이때 각 변환 단계는 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다.**
  * 즉, **스트림의 모든 단계는 모두 순수 함수로 구성되어야 한다.**
* 순수 함수란
  * **부수 효과(side effect)가 없는 함수**
  * **오직 입력만이 결과에 영향을 주는 함수**
  * **다른 가변 상태 (ex. 인스턴스 변수)를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다.**
  * ex. 람다

<br>

**예시**
```java
// 비 순수 함수 - 따라 하지 말 것
List<String> words = Arrays.asList("hi", "hi", "hi", "hello", "hello", "java", "python");

Map<String, Long> freq = new HashMap<>();
words.forEach(word -> {
    freq.merge(word.toLowerCase(), 1L, Long::sum);
});

freq.entrySet().forEach(System.out::println);

// 결과
hi=3
python=1
java=1
hello=2
```
* 문제점 - 스트림 코드를 가장한 반복적 코드다.
  * `forEach`문 안에서 외부 상태 (빈도표)를 수정하는 람다를 실행하면서 문제가 생긴다. (**부수 효과**)

<br>

```java
// 순수 함수
List<String> words = Arrays.asList("hi", "hi", "hi", "hello", "hello", "java", "python");

Map<String, Long> freq = words.stream()
    .collect(groupingBy(String::toLowerCase, counting()));

freq.entrySet().forEach(System.out::println);
```
* 앞서와 같은 일을 하지만, 순수 함수이므로 함수형 패러다임에 더 적합하다.

<br>

> `forEach` 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자.

<br>

## 2 Collector
* Collector란?
  * 수집기
  * **축소 (reduction) 전략을 캡슐화한 블랙박스 객체**
    * **축소란 스트림의 원소들을 객체 하나에 취합한다는 의미이다.**
  * 인수로 전달되는 Collectors 객체에 구현된 방법대로 스트림의 요소를 수집한다.
* Collector를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다.

<br>

**예시**
```java
// 빈도표에서 가장 흔한 단어 2개를 뽑아내는 파이프라인
List<String> words = Arrays.asList("hi", "hi", "hi", "hello", "hello", "java", "python");

Map<String, Long> freq = words.stream()
    .collect(groupingBy(String::toLowerCase, counting()));

List<String> topTwo = freq.keySet().stream()
    .sorted(Comparator.comparing(freq::get).reversed())
    .limit(2)
    .collect(toList());

topTwo.forEach(System.out::println); // hi hello
```
> `toList`는 `Collectors`의 메서드다. 이처럼 `Collectors`의 멤버를 정적 임포트하여 쓰면 가독성이 좋아져, 흔히들 이렇게 사용한다.

<br>

## 3 Collectors

<br>

### 3-1 Collector란
* Collectors : 스트림 요소의 수집 용도로 사용하는 클래스. 미리 정의된 다양한 방법이 클래스 메서드로 정의되어 있다.
  * 스트림을 배열이나 컬렉션으로 변환 : toArray(), toCollection(), toList(), toSet(), toMap()
  * 요소의 통계와 연산 메서드와 같은 동작을 수행 : counting(), maxBy(), minBy(), summingInt(), averagingInt()
  * 요소의 소모와 같은 동작을 수행 : reducing(), joining()
  * 요소의 그룹화와 분할 : groupingBy(), partitioningBy()
  * 사용자가 직접 Collector 인터페이스를 구현하여 자신만의 수집 방법을 정의할 수 있다.

<br>

### 3-2 