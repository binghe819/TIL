> 본 자료는 [Modern Java In Action](https://www.amazon.com/Modern-Java-Action-functional-programming/dp/1617293563)를 바탕으로 작성되었습니다.

# Ch08 - 컬렉션 API 개선
> [자바 HashMap을 효과적으로 사용하는 방법](http://tech.javacafe.io/2018/12/03/HashMap/)

<br>

- [Ch08 - 컬렉션 API 개선](#ch08---컬렉션-api-개선)
- [컬렉션 팩토리](#컬렉션-팩토리)
  - [리스트 팩토리](#리스트-팩토리)
    - [Arrays.asList](#arraysaslist)
    - [List.of](#listof)
  - [집합 팩토리](#집합-팩토리)
    - [HashSet<>(List)](#hashsetlist)
    - [Set.of](#setof)
  - [맵 팩토리](#맵-팩토리)
    - [Map.of](#mapof)
    - [Map.ofEntries](#mapofentries)
- [리스트와 집합 처리](#리스트와-집합-처리)
  - [removeIf](#removeif)
  - [replaceAll](#replaceall)
- [맵 처리](#맵-처리)
  - [forEach](#foreach)
  - [정렬 메서드](#정렬-메서드)
  - [getOrDefault](#getordefault)
  - [계산패턴](#계산패턴)
    - [computeIfAbsent](#computeifabsent)
    - [computeIfPresent](#computeifpresent)
  - [삭제 패턴](#삭제-패턴)
  - [교체 패턴](#교체-패턴)
  - [합침](#합침)
- [ConcurrentHashMap](#concurrenthashmap)
  - [ConcurrentHashMap이란](#concurrenthashmap이란)
  - [리듀스와 검색](#리듀스와-검색)

<br>

# 컬렉션 팩토리
자바 9에서는 작은 컬렉션 객체를 쉽게 만들 수 있는 방법을 제공한다.
* 리스트 - `Arrays.asList`, `List,of`
* 

<br>

## 리스트 팩토리

<br>

### Arrays.asList
```java
// 사용하기 전
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");

// Arrays.asList
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
```
* 주의할 점
  * 고정 크기의 리스트를 만들었으므로 요소를 갱신할 순 있지만 **새 요소를 추가하거나 요소를 삭제할 순 없다.**
  * `UnsupportedOperationException` 예외 발생
  * 물론, 요소를 get해서 변경하는 것은 막지 못한다. (불변으로 만든다면 괜찮을 듯 하다)

> 오버로딩 vs 가변 인수
> * 가변 인수는 추가 배열을 할당해서 리스트로 감싼다.
> * 따라서 배열을 할당하고 초기화하며 나중에 GC 비용이 있으므로 성능에는 좋은 영향이 없다.

<br>

### List.of
```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
```
* 주의할 점
  * `List.of`도 `Arrays`와 동일하게 추가 삭제를 할 수 없다.

<br>

## 집합 팩토리

<br>

### HashSet<>(List)
```java
Set<String> friends = new HashSet<>(Arrays.asList("Raphael", "Olivia", "Thibaut"));
```
* 단점으로는 리스트를 생성하는 비용이 든다.

<br>

### Set.of
```java 
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
```
* 주의 할점
  * 중복된 요소를 제공해 집합을 만들려고 하면 `IllegalArgumentException`이 발생한다.

<br>

## 맵 팩토리

<br>

### Map.of
```java
Map<String, Integer> ageOfFriends = 
    Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
```
* 10개 이하의 키와 값 쌍을 가진 작은 맵을 만들 때 유용하다.

<br>

### Map.ofEntries
```java
Map<String, Integer> ageOfFriends = Map.ofEntries(
            Map.entry("Raphael", 30),
            Map.entry("Olivia", 25),
            Map.entry("Thibaut", 26));
```

<br>

# 리스트와 집합 처리
* removeIf, replaceAll, sort
  * 공통점: 새로운 결과를 만드는 스트림과 다르게 기존 컬렉션을 변경한다.

<br>

## removeIf
```java
// removeIf를 사용하지 않는 코드
for (Transaction transaction : transactions) {
    if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
        transactions.remove(transaction);
    }
}

// removeIf
transactions.removeIf(transaction -> Character.isDigit(transaction.getReferenceCode().charAt(0)));
```
* `Predicate`를 만족하는 요소를 제거한다. List나 Set을 구현하거나 그 구현을 상속받은 모든 클래스에서 이용할 수 있다.

<br>

## replaceAll
```java
// Stream 버전 (새 문자열 컬렉션을 만드므로 비용이 더 든다)
referenceCodes.stream()
              .map(code -> Character.toUpperCase(code.charAt(0)) + code.subString(1))
              .collect(Collectors.toList());

// replaceAll
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.subString(1));
```
* 리스트의 각 요소를 새로운 요소로 바꿀 수 있다.
* 리스트에서 이용할 수 있는 기능으로 `UnaryOperator` 함수를 이용해 요소를 바꾼다.

<br>

# 맵 처리
자바 8에서 Map 인터페이스에 몇 가지 디폴트 메서드를 추가했다.

> 디폴트 메서드의 의도
> * 기본적인 구현을 인터페이스에 제공하는 메서드.
> * 자주 사용되는 패턴을 개발자가 직접 구현할 필요가 없도록 메서드를 추가한 것.

<br>

## forEach
**맵의 모든 요소를 출력해야하는 경우**
```java
// Map.Entry
for (Map.Entry<String, Integer> entry : ageOfFriends.entrySet()) {
    String friend = entry.getKey();
    Integer age = entry.getValue();
    System.out.println(friend + " is " + age + " years old");
}

// forEach
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
```
* 훨씬 깔끔하다

<br>

## 정렬 메서드
```java
Map<String, String> favouriteMovies = 
    Map.ofEntries(
        entry("Raphael", "Star Wars"),
        entry("Cristina", "Matrix"),
        entry("Olivia", "James Bond")
    );

favouriteMovies
    .entrySet()
    .stream()
    .sorted(Entry.comparingByKey())
    .forEachOrdered(System.out::println);

// 결과
Cristina=Matrix
Olivia=James Bond
Raphael=Star Wars
```
* 다음 두 개의 새로운 유틸리티를 이용하면 맵의 항목을 값 또는 키를 기준으로 정렬할 수 있다.
  * `Entry.comparingByValue`
  * `Entry.comparingByKey`

<br>

## getOrDefault
```java
Map<String, Integer> favouriteMovies = Map.ofEntries(
            Map.entry("Raphael", "Star Wars"),
            Map.entry("Olivia", "James Bond"));

String favouriteOfOlivia = favouriteMovies.getOrDefault("Olivia", "Matrix");
```
* 키가 존재하지 않으면 `null`을 반환하는데 NPE를 방지하기 위해서 기본값을 설정하는 메서드다.

<br>

## 계산패턴

<br>

### computeIfAbsent
```java
// 사용 전
String friend = "Raphael";
List<String> movies = friendsToMovies.get(friend);
if (movies == null) {
    movies = new ArrayList<>();
    friendsToMovies.put(friend, movies);
}
movies.add("Star Wars");

// 사용 후
friendsToMovies.computIfAbsent("Raphael", name -> new ArrayList<>()).add("Star Wars");
```
* 제공된 키에 해당하는 값이 없으면 (없거나 null), 키를 이용해 새 값을 계산하고 맵에 추가한다.
  
<br>

### computeIfPresent
```java
for (String word : text.split(" ")) {
map.computeIfPresent(word, (String key, Integer value) -> ++value);
}
```
* 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.

> compute: 제공된 키로 새 값을 계산하고 맵에 저장한다.

<br>

## 삭제 패턴
```java
// remove 사용 전
String key = "Raphael";
String value = "Jack Reacher 2";
if (favouriteMovies.containsKey(key) && Objects.equals(favouriteMovies.get(key), value)) {
    favouriteMovies.remove(key);
    return true;
}
return false;

// remove 사용 후
favouriteMovies.remove(key, value);
```

<br>

## 교체 패턴
```java
Map<String, String> favouriteMovies = new HashMap<>();
favouriteMovies.put("Raphael", "Star Wars");
favouriteMovies.put("Olivia", "James Bond");
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```
* replaceAll: `BiFunction`을 적용한 결과로 각 항목의 값을 교체한다.
* Replace: 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 있다.

<br>

## 합침
```java
Map<String, String> family = Map.ofEntries(
    Map.entry("Teo", "Star Wars"),
    Map.entry("Cristina", "James Bond")
);
Map<String, String> friends = Map.ofEntries(
    Map.entry("Raphael", "Star Wars"),
    Map.entry("Cristina", "Matrix")
);

Map<String, String> everyone = new HashMap<>(family);
friends.forEach((k, v) -> everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
System.out.println(everyone);

// 결과: {Raphael=Star Wars, Cristina=James Bond & Matrix, Teo=Star Wars}
```
* merge는 중복된 키를 어떻게 합칠지 결정하는 `BiFunction`을 인수로 받는다.
  * `forEach`와 `merge`를 이용해 충돌을 해결할 수 있다.

# ConcurrentHashMap

<br>

## ConcurrentHashMap이란
* 동시성 친화적이며 최신 기술을 반영한 HashMap
* 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다.
  * 동기화된 HashTable버전에 비해 읽기 쓰기 연산 성능이 월등하다.

> 표준 HashMap은 비동기라고 한다.

<br>

## 리듀스와 검색
ConcurrentHashMap은 세 가지 새로운 연산을 지원하다.
* forEach: 각 (키, 값) 쌍에 주어진 액션을 실행
* reduce: 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
* search: 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용

> 이들 연산은 상태를 잠그지 않고 연산을 수행한다. 따라서 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서등에 의존하지 않아야 한다.

