> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 53. 가변인수는 신중히 사용하라

<br>

- [아이템 53. 가변인수는 신중히 사용하라](#아이템-53-가변인수는-신중히-사용하라)
  - [가변인수란](#가변인수란)
  - [가변인수만 존재할 경우 발생하는 문제](#가변인수만-존재할-경우-발생하는-문제)
  - [성능에 민감할 경우](#성능에-민감할-경우)
  - [핵심 정리](#핵심-정리)

<br>

## 가변인수란
```java
// 입력한 숫자의 합을 구하는 메서드
static int sum(int... args) {
    int sum = 0;
    for (int arg : args)
        sum += arg;
    return sum;
}
```
* 가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다.
* **가변인수 메서드를 호출하면 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 메서드에 건내준다.**

<br>

## 가변인수만 존재할 경우 발생하는 문제
```java
// 인수가 1개 이상이어야 하는 가변인수 메서드 - 잘못 구현한 예!
static int min(int... args) {
    if (args.length == 0) {
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    }
    int min = args[0];
    for (int i = 1; i < args.length; i++) {
        if (args[i] < min) {
            min = args[i];
        }
    }
    return min;
}
```
* 위 코드에서 인수를 0개 넣어 호출하면 런타임에 실패하게 된다.
  * 인수 개수는 런타임에 알 수 있으므로, 컴파일 타임때 예방하지도 못한다..
* 또한, 최소값을 구하는데 필요한 시드 값을 설정하기 힘들다.

<br>

```java
// 인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs) {
        if (arg < min) {
            min = arg;
        }
    }
    return min;
}
```
* 위와 같이 매개변수를 2개 받도록하면 0개 넣어 호출할 때 발생하는 문제가 말끔히 사라진다.

<br>

## 성능에 민감할 경우
성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있다.

가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화하기 때문이다!

이 문제를 해결하는 방법은 다중정의를 하는 것이다.

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```
`EnumSet`도 정적 팩토리로 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다.

<br>

## 핵심 정리
* 인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하다.
* 메서드를 정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자.
