# 목차

<br>

- [목차](#목차)
- [개요](#개요)
- [`java.lang.Class<T>`](#javalangclasst)
- [클래스 정보 조회](#클래스-정보-조회)
  - [`Class<T>` 접근](#classt-접근)
  - [`Class<T>`를 통해 정보 조회](#classt를-통해-정보-조회)
  - [애노테이션과 리플렉션](#애노테이션과-리플렉션)
- [인스턴스 생성 및 정보 수정](#인스턴스-생성-및-정보-수정)
- [클래스 정보 실행](#클래스-정보-실행)
- [리플렉션 단점과 주의할 점](#리플렉션-단점과-주의할-점)
- [참고](#참고)

<br>

# 개요
리플렉션은 자바의 여러 특징중 하나다. (C, C++와 같은 언어는 가지고 있지 않는 특징중 하나이기도 하다.)

리플렉션을 사용하면 실행 중인 자바 프로그램을 자체적으로 검사하거나 내부 속성들을 조작할 수 있다.

또한, 리플렉션을 통해 인스턴스 생성, 메서드 실행, 필드 조회 및 수정 등등 여러 조작을 할 수 있다.

실제로 스프링(DI, Component Scan, ...)와 여러 라이브러리(Object Mapper, ...)에서 리플렉션을 잘 활용하고 있다.

이번 글에선 리플랙션이란 무엇이며, 어떻게 사용하는지에 대해서 다뤄보고자 한다.

<br>

# `java.lang.Class<T>`
리플렉션에 대해서 다루기 전에 가장 먼저 알아야 하는 클래스가 있다. 바로 `java.lang.Class<T>`이다.

리플렉션은 `Class<T>`를 통해서 동작한다.

<br>

🤔 `Class<T>`
* `Class` 클래스는 자바 애플리케이션에서 동작하는 모든 클래스와 인터페이스를 표현하는 클래스이다.
  * 자바 애플리케이션에서 돌아가는 모든 것은 클래스에 속한다.
    * `enum`, `annotation`은 인터페이스
    * 배열과 기본 자료형 모두 `Class`로 표현된다.

<br>

❗️ `Class<T>`는 public 생성자가 없다.
* 그 이유는 사용자가 `Class`를 인스턴스화하지 않고, JVM이 인스턴스화해주기 때문이다.
  * 더 정확히는 JVM의 클래스 로더가 바이트 코드(`.class`)를 읽고 로딩 단계가 끝나면 해당 클래스 타입의 `Class`객체를 생성하여 `Heap 영역`에 저장한다.
  * JVM에서 `defineClass`라는 메서드를 호출하여 생성한다고 한다.

<br>

# 클래스 정보 조회
클래스에 대한 정보를 가져오기 위해선 `Class<T>`를 사용해야 한다.

<br>

## `Class<T>` 접근
> 자바 런타임때 `Class<T>`는 어떻게 가져올 까?

1. `타입.class`
   * 모든 클래스는 JVM을 통해 로딩된 후 `Class`객체를 생성한다. 그러므로, `타입.class`를 통해 접근할 수 있다.
2. `getClass()`
   * 모든 클래스의 인스턴스는 `getClass()`를 가지고 있다. 즉, 모든 인스턴스를 통해 해당 클래스를 접근할 수 있다.
3. `Class.forName("FQCN")`
   * 클래스 이름(Fully Qualified Class Name)을 통해 가지고 올 수 있다.
   * 찾는 클래스가 없으면 `ClassNotFoundException`이 던져짐.

<br>

## `Class<T>`를 통해 정보 조회
`Class<T>`를 통해서 클래스에 대한 정보를 가져올 수 있다.

필드, 메서드, 상위 클래스, 인터페이스, 애노테이션, 생성자 등등 모두 가져올 수 있다.

보통 다 `getXXX`을 통해 가져오며, 아래와 같은 특징이 있다.

* `getXXX()`와 `getDeclaredXXX()`
  * `getXXX()`는 public만 가져올 수 있다. name을 통한 검색도 동일하다.
  * `getDeclaredXXX()`는 접근 제어자와 상관없이 모든 속성과 메서드를 가져올 수 있다.
* 접근 제어자 - `Modifier`
  * 접근 제어자가 붙을 수 있는 정보(ex. 필드, 메서드...)는 모두 Modifier를 가진다.
  * Modifier를 통해서 접근 제어자가 무엇인지 알 수 있다. 
  * ex. `isPrivate(field.getModifier())`

<br>

## 애노테이션과 리플렉션
리플렉션을 통해 애노테이션을 다루기 위해선 몇 가지 더 알아야 하는 부분이 있다.

우선 애노테이션을 위한 속성 애노테이션이다.

* `@Retention`: 해당 애노테이션을 언제까지 유지할 것인지? ex. `RetentionPolicy.SOURCE`(코드), `CLASS`(바이트코드), `RUNTIME`(런타임)
* `@Inherit`: 해당 애노테이션을 하위 클래스까지 전달할 것인지? 이 애노테이션을 붙여진 클래스의 하위 클래스까지 이 애노테이션이 붙은 것 처럼 동작한다.
* `@Target`: 해당 애노테이션을 어디에 사용할 수 있는지? ex. `ElementType.FIELD`, `ElementType.METHOD`...

> 자세한 애노테이션 사용법은 [여기](../자바의%20정석/Annotation.md)서 확인가능하다.

<br>

또 하나 중요한 점은 `@Inherit`에 따른 조회다.

* `getAnnotation()`: 상속받은 (`@Inherit`)애노테이션까지 조회된다.
* `getDeclaredAnnotations()`: 자기 자신에만 붙어있는 애노테이션을 조회된다.

<br>

# 인스턴스 생성 및 정보 수정
리플렉션을 통해 `Class<T>`를 인스턴스화할 수도 있다.

두 가지 방법이 존재한다.

1. `Class.newInstance()` -> 현재 deprecated되어서 권장하지 않는다. (사용시 예외가 발생한다.)
2. 생성자를 통한 생성 -> 권장하는 인스턴스 만드는 방법이다.

<br>

**간단한 예시를 통해 살펴보자.**
```java
public class User {

    private String name;

    private int age;

    public User() {
    }

    public User(String name) {
        this.name = name;
    }

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // getter / setter
}


@Test
void 생성자를_통한_인스턴스_만들기()
    throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException, NoSuchFieldException {
    // given
    Class<?> classOfUser = User.class;

    // when
    // String을 매개변수로 가진 가진 생성자 가져옴.
    Constructor<?> constructor = classOfUser.getConstructor(String.class);
    User user = (User) constructor.newInstance("binghe");

    // private Field 접근 후 수정
    Field age = classOfUser.getDeclaredField("age"); // 필드 접근
    age.setAccessible(true);
    age.set(user, 10);                              // 필드 수정(변경하고자 하는 객체, 값)

    // then
    assertThat(user.getName()).isEqualTo("binghe");
    assertThat(user.getAge()).isEqualTo(10);
}
```
* **필드 값 접근 및 설정하기**
  * 특정 인스턴스가 가지고 있는 값을 가져오고 수정하는 것이기 때문에 인스턴스가 필요함.
  * 필드 접근: `Field.get(object)`
  * 필드 수정: `Field.set(object, value)`
    * Static 필드는 가져올 때 인스턴스(object)가 필요 없기 때문에 `null`을 넘기면 된다.

<br>

# 클래스 정보 실행
마지막으로 리플랙션을 통해 메서드 실행을 해보자.

`Object Method.invoke(object, params);`

```java
public class User {

    private String name;

    public User() {
    }

    public User(String name) {
        this.name = name;
    }

    public String sayHi() {
        return "hi";
    }
}

@Test
void 메서드_실행()
    throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
    // given
    Class<?> classOfUser = User.class; 

    // when
    // 객체 생성
    Constructor<?> constructor = classOfUser.getConstructor(String.class);
    User user = (User) constructor.newInstance("binghe");

    // 메서드 불러오기
    Method sayHi = classOfUser.getMethod("sayHi");

    // then
    // 메서드 실행 및 테스트
    assertThat(sayHi.invoke(user)).isEqualTo("hi");
}
```

<br>

# 리플렉션 단점과 주의할 점
> 아래 문제점은 잘못 사용할 경우에만 해당된다. 잘 사용하면 강력한 기능이다!
1. 지나친 사용은 성능 이슈를 야기할 수 있다.
   * 반드시 필요한 경우에만 사용할 것은 추천한다.
2. 컴파일 타임에 확인되지 않고 런타임 시에만 발생하는 문제를 만들 가능성이 있다.
3. 접근 지시자를 의도적으로 무시할 수 있기 때문에 자칫하면 보안적 이유가 발생할 수 있다.

<br>

# 참고
* https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html
* https://docs.oracle.com/javase/tutorial/reflect/index.html
* https://www.inflearn.com/course/the-java-code-manipulation
