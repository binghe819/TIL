> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 45. 스트림은 주의해서 사용하라

<br>

- [아이템 45. 스트림은 주의해서 사용하라](#아이템-45-스트림은-주의해서-사용하라)
  - [1 스트림](#1-스트림)
    - [1-1 스트림 개념](#1-1-스트림-개념)
    - [1-2 스트림은 지연 평가된다](#1-2-스트림은-지연-평가된다)
  - [2 스트림은 과용하면 읽거나 유지보수하기 힘들어진다](#2-스트림은-과용하면-읽거나-유지보수하기-힘들어진다)
    - [2-1 스트림을 사용하지 않을 경우](#2-1-스트림을-사용하지-않을-경우)
    - [2-2 스트림을 과하게 사용할 경우](#2-2-스트림을-과하게-사용할-경우)
    - [2-3 스트림을 적절히 사용할 경우](#2-3-스트림을-적절히-사용할-경우)
  - [3 스트림을 사용해야 할 때와 사용하지 말아야 할 때](#3-스트림을-사용해야-할-때와-사용하지-말아야-할-때)
    - [3-1 스트림을 사용하지 못할 때 - 함수객체 사용 관점에서](#3-1-스트림을-사용하지-못할-때---함수객체-사용-관점에서)
    - [3-2 스트림이 적절할 때](#3-2-스트림이-적절할-때)
    - [3-3 스트림으로 처리하기 어려울 때](#3-3-스트림으로-처리하기-어려울-때)
  - [핵심 정리](#핵심-정리)

<br>

## 1 스트림

<br>

### 1-1 스트림 개념
* 스트림 추상 핵심 개념
  1. 스트림은 데이터 원소의 유한 혹은 무한 시퀀드(sequence)를 뜻한다.
  2. 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.
* 스트림의 원소들은 어디로부터든 올 수 있다.
  * 컬렉션, 배열, 파일, 정규표현식, 난수 생성기, 다른 스트림
* 스트림 파이프라인은 소스 스트림에서 시작해 최종 연산으로 끝나며, 그 사이에 하나 이상의 중간 연산이 있을 수 있다.
  1. 생성
  2. 중간 연산
  3. 최종 연산

<br>

### 1-2 스트림은 지연 평가된다
* 스트림 파이프라인은 지연 평가 (lazy evaluation)된다.
* 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다.
* 이러한 지연 평가는 무한 스트림을 다룰 수 있게 해주는 열쇠라고 한다.

<br>

## 2 스트림은 과용하면 읽거나 유지보수하기 힘들어진다
아나그램 그룹들을 출력하는 예시를 통해 스트림을 과용하면 어떻게 되는지 알아보자.

<br>

### 2-1 스트림을 사용하지 않을 경우
```java
// 사전 하나를 훑어 원소 수가 많은 아나그램 그룹들을 출력한다.
public static void main(String[] args) throws IOException {
    List<String> dictionary = Arrays.asList("staple", "petals", "Mark", "Java", "JS", "Python");
    int minGroupSize = 1;

    Map<String, Set<String>> groups = new HashMap<>();
    Iterator<String> iterator = dictionary.iterator();
    while(iterator.hasNext()) {
        String word = iterator.next();
        groups.computeIfAbsent(alphabetize(word),
            (unused) -> new TreeSet<>()).add(word);
    }

    for (Set<String> group : groups.values()) {
        if (group.size() >= minGroupSize) {
            System.out.println(group.size() + " : " + group);
        }
    }
}

private static String alphabetize(String s) {
    char[] a = s.toCharArray();
    Arrays.sort(a);
    return new String(a);
}

// 결과
2 : [petals, staple]
1 : [JS]
1 : [Python]
1 : [Java]
1 : [Mark]
```
* `computeIfAbsent(`key, Function<> mappingFunction`)
  * 키에 매핑된 값이 있으면 해당 키의 값을 반환
  * 키에 매핑된 값이 없으면 건네진 함수 객체를 키에 적용하여 값을 계산해낸 다음 그 키와 값을 매핑해놓고, 값을 반환한다.

<br>

### 2-2 스트림을 과하게 사용할 경우
```java
// 스트림을 과하게 사용했다. - 따라 하지 말 것!
public static void main(String[] args) throws IOException {
    List<String> dictionary = Arrays.asList("staple", "petals", "Mark", "Java", "JS", "Python");
    int minGroupSize = 1;

    dictionary.stream().collect(
        Collectors.groupingBy(word -> word.chars().sorted()
        .collect(StringBuilder::new,
            (sb, c) -> sb.append((char) c),
            StringBuilder::append).toString())
    ).values().stream().filter(group -> group.size() >= minGroupSize)
        .map(group -> group.size() + " : " + group)
        .forEach(System.out::println);
}
// 결과는 위와 동일하다.
```
* 딱봐도 스트림을 과용하여 프로그램이 읽거나 유지보수하기 어려워보인다.
  * **위 코드와 달라진 점은 `alphabetize` 메서드를 정의하지 않고, 모두 스트림으로 해결한 점이다.**

<br>

### 2-3 스트림을 적절히 사용할 경우
```java
// 스트림을 적절히 활용하면 깔끔하고 명료해진다.
public static void main(String[] args) throws IOException {
    List<String> dictionary = Arrays.asList("staple", "petals", "Mark", "Java", "JS", "Python");
    int minGroupSize = 1;

    dictionary.stream().collect(Collectors.groupingBy(word -> alphabetize(word))) // Map<String, List<String>>
        .values().stream() // Stream(List<String>)
        .filter(group -> group.size() >= minGroupSize)
        .forEach(g -> System.out.println(g.size() + " : " + g));
}
// alphabetize는 위와 동일.
// 결과는 위와 동일하다.
```
* 스트림과 일반 메서드(`alphabetize`)를 혼용하여 이해하기 좋은 코드가 완성되었다.
* 그리고 `alphabetize`를 스트림으로 해결하기 힘든 이유는
  * **자바는 char용 스트림을 지원하지 않기 때문이다.**
  * IntStream chars = "Hello".chars(); // IntStream이 반환된다.
  * **설령 `(char) s`처럼 형변환을 할 수 있지만 헷갈리고 가능한 사용하지 않는 편이 좋다고 한다.**

<br>

## 3 스트림을 사용해야 할 때와 사용하지 말아야 할 때

<br>

### 3-1 스트림을 사용하지 못할 때 - 함수객체 사용 관점에서
* 지역변수를 읽고 수정할 필요가 있을 때
  * 람다의 변수는 사실상 final이다.
* 람다는 return, break, continue 문이 불가능하다

<br>

### 3-2 스트림이 적절할 때
* 원소들의 시퀀스를 일관성 있게 변환할 때
* 원소들의 시퀀스를 필터링할 때
* 원소들의 시퀀스를 연산 후 결합할 때
* 원소들의 시퀀스를 모을 때
* 원소들의 시퀀스 중 특정 조건을 만족하는 원소를 찾을 때

<br>

### 3-3 스트림으로 처리하기 어려울 때
* 원본 스트림을 계속 써야할 때
  * **스트림은 중간연산을 지나고 나면 원래의 스트림을 잃는다. 파이프라인의 순서를 바꿈으로써 해결할 수 있는지 고민해보자.**

<br>

## 핵심 정리
* **스트림을 사용해야 멋지게 처리할 수 있는 일이 있고, 반복 방식이 더 알맞은 일도 있다.**
  * **그리고 수많은 작업이 이 둘을 조합했을 때 가장 멋지게 해결된다.**
* **어느 쪽을 선택하는 규칙은 없지만 스트림과 반복 중 어느 쪽이 나은지 둘 다 해보고 더 나은 쪽을 택하라.**