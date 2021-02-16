> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 1. 생성자 대신 static 팩토리 메서드를 고려하라

> 1장은 객체의 생성과 파괴를 다룬다.

**1장의 학습 목표**

1. 객체를 만들어야할 때와 만들지 말아야 할 때는 구분하는 방법.
2. 올바른 객체 생성 방법을 알 수 있다.
3. 불필요한 생성을 피하는 방법
4. 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요령.

<br>

## 인스턴스를 생성하는 방법

* 인스턴스를 생성하는 두 가지 방법
  * public 생성자
  * **public static 팩토리 메서드를 사용해서 해당 클래스의 인스턴스를 만드는 방법.**

```java
public static Boolean valueOf(boolean b) {
  return b ? Boolean.TRUE : Boolean.FALSE;
}
```

* 정적 팩토리 메서드란?
  * **객체를 생성하는 메서드를 만들고, static으로 선언하는 기법.**

> **여기서 얘기하는 정적 팩토리 메서드는 디자인 패턴에서의 팩터리 메서드 패턴와 다르다.**

<br>

## 팩토리 메서드의 장점과 단점

<br>

#### :+1: 1. 이름을 가질 수 있다.

```java
public class Person {
  String name;
  
  public Person(String name) {
    this.name = name;
  }
  
  public static Person withName(String name) {
    return new Person(name);
  }
  
  public static void main(String[] args) {
    Person person1 = new Person("binghe"); // "binghe"가 무엇을 의미하는지 설명하지 못한다.
    Person person1 = Person.withName("binghe"); // 훨씬 가독성이 좋다.
  }
}
```

* 생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못할 경우
  * **static 팩토리 메서드를 이용하여 반환된 객체의 특성을 쉽게 묘사할 수 있다.**
  * 예 ) `BigInteger.probablePrime`

```java
public class Person {
  String name;
  String address;
  
  // ...기본 생성자
  
  public Person(String name) {
    this.name = name;
  }
  
  //public Person(String address) { // 에러 발생 (동일한 시그니처로 생성자를 하나 이상 만들 수 없다.)
  //  this.address = address;
  //}
  
  public static Person withName(String name) {
    return new Person(name);
  }
  
  public static Person withAddress(String address) {
    Person person = new Person();
    person.address = address;
    return person;
  }
}
```

* 하나의 시그니처로는 생성자를 하나만 만들 수 있다.
  * static 팩토리 메서드를 이용하면 하나 이상의 생성자 역할을 하는 메서드를 만들 수 있다.

<br>

#### :+1: 2. 반드시 새로운 객체를 만들 필요가 없다.

* **불변(immutable) 클래스인 경우나 매번 새로운 객체를 만들 필요가 없는 경우에 미리 만들어둔 인스턴스 또는 캐시해둔 인스턴스를 반환할 수 있다.** `Boolean.valueOf(boolean)` 메서드도 그 경우에 해당한다.
  * 인스턴스 통제 클래스라고도 한다.
    * **싱글턴으로도 만들 수 있고,**
    * **인스턴스화 불가로 만들 수도 있고,**
    * **불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다.**

<br>

#### :+1: 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

* **클래스에서 만들어 줄 객체의 클래스를 선택하는 "엄청난 유연함"이 있다.**
  * 반환할 객체의 클래스를 자유롭게 선택할 수 있는 "엄청난 유연성"을 가진다.
* **리턴 타입의 하위 타입의 인스턴스를 만들어 줘도 된다.**
  * **리턴 타입은 인터페이스로 지정하고, 그 인터페이스의 구현체는 API로 노출시키지 않고, 그 구현체의 인스턴스를 만들어 줄 수 있다.**
  * 예 ) `java.util.Collections`

<br>

#### :+1: 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

* 3번 장점의 연장선이다.
  * **반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.**
* `EnumSet` 클래스는 생성자 없이 public static 메서드, `allOf()`, `of()` 등을 제공한다.
  * 그 안에서 리턴하는 객체의 타입은 enum 타입의 개수에 따라 달라진다.

<br>

#### :+1: 5. static 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

* 3, 4번 장점과 비슷하게 유연성을 제공하는 장점
  * 이러한 유연성을 제공하는 static 팩토리 메서드는 `서비스 프로바이더` 프레임워크의 근본이다.
  * 대표적인 예시 ) JDBC

* 서비스 프로바이더 프레임 워크의 구성 (JDBC의 경우)
  * 프로바이더 등록 API : 구현체를 등록하는데 사용된다. (`DriverManager.registerDriver()`)
  * 서비스 액세스 API : 클라이언트가 해당 서비스의 인스턴스를 가져갈 때 사용된다. (`DriverManager.getConnection()`)
  * 서비스 프로바이더 인터페이스 : 서비스 인터페이스의 인스턴스를 제공한다. (`Driver`)

<br>

#### :-1: 1. 상속을 하려면 public / protected 생성자가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.

* 따라서 `Collections`클래스는 상속을 할 수 없다.
* 이 제약은 상속보다 합성을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 오히려 장점이 될 수 있다.

<br>

#### :-1: 2. static 팩토리 메서드는 프로그래머가 찾기 어렵다

* 생성자는 Javadoc 상단에 모아서 보여주지만 static 팩토리 메서드는 API 문서에서 특별히 다뤄주지 않는다.
* 따라서 클래스나 인터페이스 문서 상단에 팩토리 메서드에 대한 문서를 제공하는 것이 좋다.

<br>

## 정적 팩토리 메서드의 흔히 사용되는 명명 방식

* `from` : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
  * `Date d = Date.from(instant)`

* `of` : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
  * `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`
* `valueOf` : from과 of의 더 자세한 버전
  * `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);`
* `instance` 혹은 `getInstance` : 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.
  * `StackWalker luke = StackWalker.getInstance(options);`
* `create` 혹은 `newInstance` : instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
  * `Object newArray = Array.newInstance(classObject, arrayLen);`
* `getType` : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.
  * `FileStore fs = Files.getFileStore(path);`
* `newType` : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.
  * `BufferedReader br = Files.newBufferedReader(path)`
* `type` : getType과 newType의 간결한 버전
  * `List<Complaint> litany = Collections.list(legacyLitany);`

