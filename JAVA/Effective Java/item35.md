> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

enum의 기본 필드에는 ordinal (순서)가 있다.

API 문서에 따르면 이 필드는 "EnumSet", "EnumMap"등에 사용될 목적으로 구현되어 있다고 한다.

그러므로, 아래와 같이 순서를 나타내는 필드로는 사용하면 안된다.

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() {return ordinal() + 1;}
}
```

이 문제를 해결하는 방법은 따로 필드를 추가하는 것이다.

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), NONET(9), DECTET(10);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() {return this.numberOfMusicians;}
}
```