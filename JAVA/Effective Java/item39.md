> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 39. 명명 패턴보다 애너테이션을 사용하라
이번 아이템에서는 애노테이션의 장점을 설명하기 위해 테스트 러너 프로그램을 예시로 들었다.

추후에 테스트 프레임워크 공부할 때 다시 한번 복습하며 코딩하면 좋을듯 하다.

<br>

- [아이템 39. 명명 패턴보다 애너테이션을 사용하라](#아이템-39-명명-패턴보다-애너테이션을-사용하라)
  - [1 명명 패턴의 단점](#1-명명-패턴의-단점)
  - [2 마커 애노테이션](#2-마커-애노테이션)
  - [3 매개 변수 가진 애노테이션](#3-매개-변수-가진-애노테이션)
  - [4 배열 매개변수를 받는 애노테이션 타입](#4-배열-매개변수를-받는-애노테이션-타입)
  - [5 반복 가능한 애노테이션](#5-반복-가능한-애노테이션)
  - [핵심 정리](#핵심-정리)

<br>

## 1 명명 패턴의 단점
> 명명 패턴 : JUnit 3까지 테스트 메서드 이름을 `test`로 시작하게끔 했다.
* 오타가 나면 큰일난다.
* 올바른 프로그램 요소에서만 사용되리란 보증할 방법이 없다.
  * 예) 메서드가 아닌 클래스 명을 `TestSafyMechanisms`으로 지어 `JUnit`에 던져주면 문제가 발생한다.
* 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.
  * 예외를 던져야 성공하는 테스트의 경우 테스트할 방법이 없다.

<br>

## 2 마커 애노테이션
> 애노테이션은 명명 패턴의 문제를 해결할 수 있다.

```java
// 마커 애노테이션 타입 선언
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```
* 메타 애노테이션 : 애노테이션 선언에 다는 애노테이션
  * 예) `@Retention`, `@Target`등
* `@Retention`
  * `RetiontionPolicy.RUNTIME` : `@Test`가 런타임에도 유지된다는 의미 (보존 정책)
* `@Target`
  * `ElementType.METHOD` : `@Test`가 메서드 선언에서만 사용되어야 한다는 의미 (적용 대상)
* **마커 애노테이션 : 아무 매개변수 없이 단순히 대상에 마킹한다.**
  * **대상 코드의 의미는 그대로 둔채 애노테이션에 관심 있는 도구에서 특별한 처리를 할 기회를 준다.**
  * **실제 클래스에 영향을 주지 않지만, 애노테이션에 관심있는 프로그램에 추가 정보를 제공한다.** (리플렉션 API를 통해)

<br>

```java
// 마커 애노테이션을 처리하는 프로그램
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;

        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                test++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + "실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
```
* 리플렉션 API를 활용한 예시
* 코드 해석
  * 테스트 러너로써 명령줄로부터 완전 정규화된 클래스 이름을 받아, 그 클래스에서 `@Test` 애노테이션이 달린 메서드를 차례로 호출한다.
  * 테스트 메서드가 예외를 던지면 리플렉션 매커니즘이 `InvocationTargetException`을 감싸서 다시 던진다.
    * 그래서 원래 예외에 담긴 실패 정보 추출하여 출력한다.
  * `Invocation...Exception`외의 예외가 발생한다면 `@Test` 애노테이션을 잘못 사용했다는 의미가 된다.

<br>

## 3 매개 변수 가진 애노테이션
> 예외를 던져야하는 테스트 애노테이션 예시를 통해 설명한다.
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
    Class<? extends Throwable> value();
}
```
* 애노테이션의 필드
  * 애노테이션에서 사용하는 옵션 값.
  * 호출시 : `@Test(value = "RuntimeException.class")

<br>

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
  tests++;
  try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (InvocationTargetException wrappedEx) {
    Throwable exc = wrappedEx.getCause();
    Class<? extends Throwable> excType =
      m.getAnnotation(ExceptionTest.class).value(); // 애너테이션의 매개변수의 값을 추출한다.
    if (excType.isInstance(exc)) { // 애노테이션에 담긴 예외 체크 
      passed++;
    } else {
      System.out.printf(
        "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
        m, excType.getName(), exc);
    }
  } catch (Exception exc) {
    System.out.println("잘못 사용한 @ExceptionTest: " + m);
  }
}
```
* 이전 코드와 비슷하다. 한가지 차이점은
  * **애노테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인하는데 사용한다.**
* **테스트 프로그램이 문제 없이 컴파일 되면 애노테이션 매개변수가 가리키는 예외가 올바른 타입이라는 의미**
  * 예외 클래스 파일이 컴파일타임에는 존재했으나 런타임에는 존재하지 않는 경우엔 `TypeNotPresnetException`을 던진다.

<br>

## 4 배열 매개변수를 받는 애노테이션 타입
> 예외를 여러 개 명시하고 그중 하나가 발생하면 성공하게 만드는 예시
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
```
* 배열 매개변수 애노테이션 사용
  * `@ExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class})`

<br>

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
  tests++;
  try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (Throwable wrappedExc) {
    Throwable exc = wrappedExc.getCause();
    int oldPassed = passed;
    Class<? extends Throwable>[] excTypes =
      m.getAnnotation(ExceptionTest.class).value(); // 배열 매개변수 추출.
    for (Class<? extends Throwable> excType : excTypes) { // 반복문을 돌며 확인
      if (excType.isInstance(exc)) {
        passed++;
        break;
      }
    }
    if (passed == oldPassed)
      System.out.printf("테스트 %s 실패: %s %n", m, exc);
  }
}
```
* for문을 추가함으로써 `Throwable` 배열을 검증한다.

<br>

## 5 반복 가능한 애노테이션
자바 8부터는 `@Repeatable` 사용이 가능하다.
* `@Repeatable` : 하나의 프로그램 요소에 여러 개의 애노테이션을 달 수 있다.
* 주의 할 점
  * `@Repeatable`을 단 애노테이션을 반환하는 '컨테이션 애노테이션'을 하나 더 정의하고, `@Repeatable`에 이 컨테이너 애노테이션의 class 객체를 매개변수로 전달해야 한다.
  * 컨테이너 애노테이션은 내부 애노테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.
  * 컨테이너 애노테이션은 타입에 적절한 `@Repeatable`과 `@Target`을 명시해야 한다.

<br>

```java
// 반복 가능한 애노테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}

// 컨테이너 애노테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
  ExceptionTest[] value();
}
```
* 적용방법은 다음과 같다.
  ```java
    @ExceptionTest(IndexOutOfBoundsException.class)
    @ExceptionTest(NullPointerException.class)
    public static void doublyBad() {}
  ```

<br>

```java
// 반복 가능 애노테이션 다루기
if (m.isAnnotationPresent(ExceptionTest.class)
    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
  tests++;
  try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (Throwable wrappedExc) {
    Throwable exc = wrappedExc.getCause();
    int oldPassed = passed;
    ExceptionTest[] excTests =
      m.getAnnotationsByType(ExceptionTest.class); // 매개변수 추출
    for (ExceptionTest excTest : excTests) {
      if (excTest.value().isInstance(exc)) {
        passed++;
        break;
      }
    }
    if (passed == oldPassed)
      System.out.printf("테스트 %s 실패: %s %n", m, exc);
  }
}
```
* **여러 개 달면 하나만 달았을 때와 구분하기 위해 해당 '컨테이너' 애노테이션 타입이 적용된다.**
* isAnnotationPresent : 반복 가능 애너테이션이 달렸는지 검사 (false : 컨테이너 애너테이션)
* getAnnotationsByType : 반복 가능 애너테이션과 컨테이너 애너테이션을 모두 가져옴

<br>

## 핵심 정리
* 애노테이션이 명명 패턴보다 우수하다.
* 소스코드에 추가 정보를 제공할 수 있는 도구를 만드는 일을 한다면 애노테이션 타입을 제공한다
* **애노테이션으로 할 수 있는 일을 명명패턴으로 처리할 이유가 없다.**
* 애노테이션 타입을 적극 활용하자.