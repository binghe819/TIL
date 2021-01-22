> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 36. 비트 필드 대신 EnumSet을 사용하라

<br>

## 1 비트 필드 - 정수 열거 패턴
열거한 값들이 주로 단독이 아닌 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용했다.

```java
public class Text {
    public static final int STYLE_BOLD          = 1 << 0; // 1
    public static final int STYLE_ITALIC        = 1 << 1; // 2
    public static final int STYLE_UNDERLINE     = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int styles) {...}
}

// 사용
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```
* 문제점
  * 정수 열거 상수의 단점을 그대로 지닌다.

<br>

## 2 EnumSet을 사용하는 방법
비트 필드를 정수 열거 패턴으로 표현하는 문제는 `EnumSet`를 사용하면 쉽게 해결된다.
```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) {...}
}

// 사용
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```
깔끔하고 안전하다.