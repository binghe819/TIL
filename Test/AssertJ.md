> 본 글은 [AssertJ 공식 문서](https://assertj.github.io/doc/#contributing-to-this-guide)를 번역하며 정리한 글입니다. 
> 
> 전체 내용을 모두 번역하진 않았으므로, 필요하신 분은 공식 문서를 참고바랍니다.

# AssertJ Core 정리

<br>

- [AssertJ Core 정리](#assertj-core-정리)
  - [1 AssertJ 란?](#1-assertj-란)
  - [2 Quick Start](#2-quick-start)
    - [2-1 Maven](#2-1-maven)
    - [2-2 Gradle](#2-2-gradle)
    - [2-3 Assertions](#2-3-assertions)
  - [3 주요 Assertions](#3-주요-assertions)
    - [3-1 간단한 예시](#3-1-간단한-예시)
    - [3-2 제공되는 타입 assertions](#3-2-제공되는-타입-assertions)
    - [3-3 Assertion description](#3-3-assertion-description)
    - [3-4 에러 메시지 재정의](#3-4-에러-메시지-재정의)
    - [3-5 잘못된 사용법을 피하라](#3-5-잘못된-사용법을-피하라)
    - [3-6 AssertJ 설정](#3-6-assertj-설정)
    - [3-7 String/CharSequence assertions](#3-7-stringcharsequence-assertions)
    - [3-8 contains](#3-8-contains)
      - [contains](#contains)
      - [containsOnly](#containsonly)
      - [containsExactly](#containsexactly)
      - [containsSequence](#containssequence)
      - [containsOnlyOne](#containsonlyone)
      - [containsAnyOf](#containsanyof)
    - [3-9 Satisfy](#3-9-satisfy)
    - [3-10 Match](#3-10-match)
    - [3-11 First/Last/Element](#3-11-firstlastelement)
    - [3-12 filter](#3-12-filter)

<br>

## 1 AssertJ 란?
AssertJ란 많은 assertion와 도움되는 에러 메시지를 제공하는 자바 라이브러리이다. 또한, 테스트 코드의 가독성을 높여주며, 좋아하는 IDE에서 쓰기 굉장히 쉽다.

<br>

**예시**
```java
// entry point for all assertThat methods and utility methods (e.g. entry)
import static org.assertj.core.api.Assertions.*;

// basic assertions
assertThat(frodo.getName()).isEqualTo("Frodo");
assertThat(frodo).isNotEqualTo(sauron);

// chaining string specific assertions
assertThat(frodo.getName()).startsWith("Fro")
                           .endsWith("do")
                           .isEqualToIgnoringCase("frodo");

// collection specific assertions (there are plenty more)
// in the examples below fellowshipOfTheRing is a List<TolkienCharacter>
assertThat(fellowshipOfTheRing).hasSize(9)
                               .contains(frodo, sam)
                               .doesNotContain(sauron);

// as() is used to describe the test and will be shown before the error message
assertThat(frodo.getAge()).as("check %s's age", frodo.getName()).isEqualTo(33);

// exception assertion, standard style ...
assertThatThrownBy(() -> { throw new Exception("boom!"); }).hasMessage("boom!");
// ... or BDD style
Throwable thrown = catchThrowable(() -> { throw new Exception("boom!"); });
assertThat(thrown).hasMessageContaining("boom");

// using the 'extracting' feature to check fellowshipOfTheRing character's names
assertThat(fellowshipOfTheRing).extracting(TolkienCharacter::getName)
                               .doesNotContain("Sauron", "Elrond");

// extracting multiple values at once grouped in tuples
assertThat(fellowshipOfTheRing).extracting("name", "age", "race.name")
                               .contains(tuple("Boromir", 37, "Man"),
                                         tuple("Sam", 38, "Hobbit"),
                                         tuple("Legolas", 1000, "Elf"));

// filtering a collection before asserting
assertThat(fellowshipOfTheRing).filteredOn(character -> character.getName().contains("o"))
                               .containsOnly(aragorn, frodo, legolas, boromir);

// combining filtering and extraction (yes we can)
assertThat(fellowshipOfTheRing).filteredOn(character -> character.getName().contains("o"))
                               .containsOnly(aragorn, frodo, legolas, boromir)
                               .extracting(character -> character.getRace().getName())
                               .contains("Hobbit", "Elf", "Man");

// and many more assertions: iterable, stream, array, map, dates, path, file, numbers, predicate, optional ...
```

<br>

## 2 Quick Start
maven과 gradle을 통해 쉽게 시작할 수 있다.

버전은 최신화가 계속 이루어지고 있으며 java 8 이전 버전의 경우 [공식 문서](https://assertj.github.io/doc/#contributing-to-this-guide)를 참고해서 사용하면 된다.

<br>

### 2-1 Maven
```xml
<dependency>
  <groupId>org.assertj</groupId>
  <artifactId>assertj-core</artifactId>
  <!-- use 2.9.1 for Java 7 projects -->
  <version>3.19.0</version>
  <scope>test</scope>
</dependency>
```

<br>

### 2-2 Gradle
```gradle
testImplementation("org.assertj:assertj-core:3.19.0")
```

<br>

### 2-3 Assertions
`Assertions` 클래스는 AssertJ를 사용하는 데 필요한 유일한 클래스로 필요한 모든 메서드들을 제공한다.

대안으로, 테스트 클래스가 `WithAssertions`를 구현하며 같은 메서드들에 접근할 수 있다.

```java
// Assertions
import static org.assertj.core.api.Assertions.*;
```
```java
// WithAssertions
import org.assertj.core.api.WithAssertions;

public class WithAssertionsExamples extends AbstractAssertionsExamples implements WithAssertions {

  // the data used are initialized in AbstractAssertionsExamples.

  @Test
  public void withAssertions_examples() {

    // assertThat methods come from WithAssertions - no static import needed
    assertThat(frodo.age).isEqualTo(33);
    assertThat(frodo.getName()).isEqualTo("Frodo").isNotEqualTo("Frodon");

    assertThat(frodo).isIn(fellowshipOfTheRing);
    assertThat(frodo).isIn(sam, frodo, pippin);
    assertThat(sauron).isNotIn(fellowshipOfTheRing);

    assertThat(frodo).matches(p -> p.age > 30 && p.getRace() == HOBBIT);
    assertThat(frodo.age).matches(p -> p > 30);
  }
}
```

<br>

## 3 주요 Assertions
이 섹션에서는 AssertJ에서 제공하는 Assertions와 이를 최대한 활용하기 위한 기타 유용한 기능을 설명한다.

[AssertJ Core javadoc](https://www.javadoc.io/doc/org.assertj/assertj-core/latest/index.html)은 대부분의 Assertions을 코드 예제로 설명하므로 특정 Assertions이 무엇인지 알고 싶다면 확인하면 된다.

<br>

### 3-1 간단한 예시
```java
public class SimpleAssertionsExample {
    @Test
    void a_few_simple_assertions() {
        assertThat("The Lord of the Rings").isNotNull()
                                           .startsWith("The")
                                           .contains("Lord")
                                           .endsWith("Rings");
    }
}
```
딱 봐도 가독성이 굉장히 좋다.

기본 assertion인 `isNotNull`을 제외하고, 다른 assertion은 테스트 대상 개체가 `String`이므로 문자열에만 한정된다.

<br>

### 3-2 제공되는 타입 assertions
자세한 내용은 [여기](https://assertj.github.io/doc/#assertj-core-supported-types)에서 확인.

<br>

### 3-3 Assertion description
AssertJ는 테스트 실패 시 나타낼 메시지를 `as`로 표현할 수 있다.

`as`는 검증 문 앞에 작성해야 하며, 뒤에 작성 시 무시된다.
```java
TolkienCharacter frodo = new TolkienCharacter("Frodo", 33, Race.HOBBIT);

// failing assertion, remember to call as() before the assertion!
assertThat(frodo.getAge()).as("check %s's age", frodo.getName())
                          .isEqualTo(100);
```
```java
// Error Message
[check Frodo's age] expected:<100> but was:<33>
```

<br>

### 3-4 에러 메시지 재정의
AssertJ는 이미 유용한 에러 메시지를 제공하지만, 사용자가 항상 재정의할 수 있다.
* `OverridingErrorMessage()`
* `WithFailMessage()`

```java
TolkienCharacter frodo = new TolkienCharacter("Frodo", 33, Race.HOBBIT);
TolkienCharacter sam = new TolkienCharacter("Sam", 38, Race.HOBBIT);
// failing assertion, remember to call withFailMessage/overridingErrorMessage before the assertion!
assertThat(frodo.getAge()).withFailMessage("should be %s", frodo)
                          .isEqualTo(sam);
```
```java
// Error Message
java.lang.AssertionError: should be TolkienCharacter [name=Frodo, age=33, race=HOBBIT]
```

<br>

**Lazy error message overriding**
만약 에러 메시지 생성 비용이 비싸다면, `String` 대신에 `Supplier(String)`을 사용해서 재정의하면 된다. 그러면 assertion fails에만 메시지가 생성된다.
```java
assertThat(player.isRookie()).overridingErrorMessage(() -> "Expecting Player to be a rookie but was not.")
                             .isTrue();

assertThat(player.isRookie()).withFailMessage(() -> "Expecting Player to be a rookie but was not.")
                             .isTrue();
```

<br>

### 3-5 잘못된 사용법을 피하라
```java
// BAD
assertThat(actual.equals(expected));

// GOOD
assertThat(actual).isEqualTo(expected);

assertThat(actual.equals(expected)).isTrue();
```
```java
// BAD
assertThat(1 == 2);

// GOOD
assertThat(1).isEqualTo(2);

assertThat(1 == 2).isTrue();
```
```java
// BAD
assertThat(actual).isEqualTo(expected).as("description");
assertThat(actual).isEqualTo(expected).describedAs("description");

// GOOD
assertThat(actual).as("description").isEqualTo(expected);
assertThat(actual).describedAs("description").isEqualTo(expected);
```
```java
// BAD
assertThat(actual).isEqualTo(expected).overridingErrorMessage("custom error message");
assertThat(actual).isEqualTo(expected).withFailMessage("custom error message");

// GOOD
assertThat(actual).overridingErrorMessage("custom error message").isEqualTo(expected);
assertThat(actual).withFailMessage("custom error message").isEqualTo(expected);
```
```java
// BAD
assertThat(actual).isEqualTo(expected).usingComparator(new CustomComparator());

// GOOD
assertThat(actual).usingComparator(new CustomComparator()).isEqualTo("a");
```

<br>

### 3-6 AssertJ 설정
자세한 내용은 [여기](https://assertj.github.io/doc/#assertj-core-configuration)에서 확인.

<br>

### 3-7 String/CharSequence assertions
이 섹션은 `CharSequence(including String, StringBuilder, StringBuffer, ...)`에 대해 사용할 수 있는 assertions을 다룹니다.

자세한 내용은 [javadoc](https://www.javadoc.io/static/org.assertj/assertj-core/3.19.0/org/assertj/core/api/AbstractCharSequenceAssert.html#method.summary)에서 확인가능합니다.

<br>

### 3-8 contains

<br>

**Checking iterables/arrays content - contains**
모든 메서드는 부정문이 존재한다. (`doesNotContains`)

<br>

#### contains
```java
 List<String> abc = Arrays.asList("a", "b", "c");

 // 성공
 assertThat(abc).contains("b", "a");
 assertThat(abc).contains("b", "a", "b");

 // 실패
 assertThat(abc).contains("d");
```

<br>

#### containsOnly
```java
List<String> list = Arrays.asList("one", "two", "three", "four");

// 순서와 상관없이 성공
assertThat(list).containsOnly("three", "two", "one", "four");
// 중복은 무시된다
assertThat(list).containsOnly("three", "two", "two", "one", "four");

// four가 없으므로 실패
assertThat(list).containsOnly("three", "two", "one");
// five가 list에 없으므로 실패
assertThat(list).containsOnly("three", "two", "one", "four", "five");
```

<br>

#### containsExactly
```java
List<String> list = Arrays.asList("one", "two", "three", "four");

// 성공
assertThat(list).containsExactly("one", "two", "three", "four");

// 실패 (순서가 다르다)
assertThat(list).containsExactly("four", "three", "two", "one");
```
* 순서와 상관 없이 테스트를 하고자 한다면 `containsExactlyInAnyOrder`을 사용.

<br>

#### containsSequence
```java
List<String> list = Arrays.asList("one", "two", "three", "four");

// 성공
assertThat(list).containsSequence("one", "two");
assertThat(list).containsSequence("one", "two")
                .containsSequence("three", "four");

// 실패
assertThat(list).containsSequence("one", "three")
                .containsSequence("two", "four");
```
* 순서대로 그룹핑을 하고 있나 확인하는 용도로 사용된다고 한다.
  * actual와 given이 모두 리스트(컬렉션)인 경우 `containsSubSequence`를 사용하자

<br>

#### containsOnlyOne
```java
// assertions will pass
 assertThat(newArrayList("winter", "is", "coming")).containsOnlyOnce("winter");
 assertThat(newArrayList("winter", "is", "coming")).containsOnlyOnce("coming", "winter");

 // assertions will fail
 assertThat(newArrayList("winter", "is", "coming")).containsOnlyOnce("Lannister");
 assertThat(newArrayList("Arya", "Stark", "daughter", "of", "Ned", "Stark")).containsOnlyOnce("Stark");
 assertThat(newArrayList("Arya", "Stark", "daughter", "of", "Ned", "Stark")).containsOnlyOnce("Stark", "Lannister", "Arya");
```
* actual 그룹이 given 값을 딱 한번만 가지고 있다는 테스트

<br>

#### containsAnyOf
```java
List<String> list = Arrays.asList("one", "two", "three", "four");

// 성공
assertThat(list).containsAnyOf("three")
                .containsAnyOf("two", "one")
                .containsAnyOf("one", "two", "three")
                .containsAnyOf("one", "two", "three", "four", "five")
                .containsAnyOf("four", "five", "six", "seven");

// 실패
assertThat(list).containsAnyOf("five")
                .containsAnyOf("five", "six", "seven", "eight");
```
* 요소의 값이 하나라도 있는지 확인하는 assertion

<br>

### 3-9 Satisfy
모든 요소나 혹은 아무 요소가 주어진 assertion을 만족하는지 확인할 수 있다. 

주어지는 assertion은 `Consumer`로 나타낸다.
* `allSatisfy`
* `anySatisfy`
* `noneSatisfy`

```java
List<Person> list = Arrays.asList(new Person("frodo", 5),
                                          new Person("pippin", 5),
                                          new Person("sam", 5));

// 모든 요소가 주어진 assertion을 모두 만족해야한다.
assertThat(list).allSatisfy(person -> {
    assertThat(person.getAge()).isEqualTo(5);
});

// 적어도 하나의 요소가 주어진 assertion을 만족해야한다.
assertThat(list).anySatisfy(person -> {
    assertThat(person.getAge()).isEqualTo(5);
    assertThat(person.getName()).isEqualTo("sam");
});

// 모든 요소가 주어진 assertion을 만족하면 안된다.
assertThat(list).noneSatisfy(person -> {
    assertThat(person.getName()).isEqualTo("none");
    assertThat(person.getAge()).isEqualTo(10);
});
```

<br>

### 3-10 Match
모든 요소나 혹은 아무 요소가 주어진 assertion에 부합한지 확인할 수 있다.

주어진 assertion은 `Predicate`로 나타낸다.
* `allMatch`
* `anyMatch`
* `noneMatch`

```java
List<Person> list = Arrays.asList(new Person("frodo", 5),
                                          new Person("pippin", 5),
                                          new Person("sam", 5));

assertThat(list).allMatch(person -> person.getAge() == 5, "age")
                .anyMatch(person -> person.getName().contains("fro"))
                .noneMatch(person -> person.getName().contains("b"));
```

<br>

### 3-11 First/Last/Element
`first`, `last`, `elements(index)`를 사용해서 요소를 가리킬 수 있다.

가리킨 후, 아래 예제에 표신된 것처럼 `Assert` 클래스 혹은 `InstanceOfAssertFactory`를 지정하지 않은 경우에만 객체 assertion을 할 수 있다.

```java
// 기본적인 사용 방법
List<String> list = Arrays.asList("frodo", "sam", "pippin");

// 기본적인 사용 방법
assertThat(list).first().isEqualTo("frodo");
assertThat(list).element(1).isEqualTo("sam");
assertThat(list).last().isEqualTo("pippin");

// 문자열 활용
assertThat(list).first(as(STRING))
                .startsWith("fro")
                .endsWith("do");
assertThat(list).element(1, as(STRING))
                .startsWith("sa")
                .endsWith("am");
assertThat(list).last(as(STRING))
                .startsWith("pipp")
                .endsWith("ppin");
```

<br>

**Single elements**
```java
List<String> list = Arrays.asList("frodo");

assertThat(list).singleElement()
                .isEqualTo("frodo");

assertThat(list).singleElement(as(STRING))
                .endsWith("do");
```
* `Single elements`로 하나의 요소만을 가지고 있음을 체크할 수 있다.

<br>

### 3-12 filter
