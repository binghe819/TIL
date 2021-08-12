# 목차

<br>

- [목차](#목차)
- [원시값 포장과 VO](#원시값-포장과-vo)
- [1 원시값 포장](#1-원시값-포장)
  - [1-1 원시값 포장이란](#1-1-원시값-포장이란)
  - [1-2 왜 원시값 포장을 해야하는가?](#1-2-왜-원시값-포장을-해야하는가)
- [2 VO](#2-vo)
  - [2-1 VO란?](#2-1-vo란)
  - [2-2 VO 특징](#2-2-vo-특징)
- [3 원시값 포장과 VO의 차이](#3-원시값-포장과-vo의-차이)
- [출처](#출처)

<br>

# 원시값 포장과 VO
이번 글에선 원시값 포장와 VO는 무엇이며, 두 개념의 차이점은 무엇인지에 대해서 정리해보고자한다.

<br>

# 1 원시값 포장

<br>

## 1-1 원시값 포장이란

[소트웍스 앤솔러지](http://www.yes24.com/Product/Goods/3290339?OzSrank=1)의 6장 객체 미용 체조에선 다음과 같이 말한다.

`모든 원시값과 문자열을 포장(wrap)한다`

포장하라는 것이 무슨 의미일까?? 아무 의미 없는 값을 객체로 포장하라는 의미이다.

**기본 타입형인 `int`값 하나 자체는 아무 의미 없는 스칼라 값일 뿐이다.** 

```java
int number = 1;
```

즉, **원시값은 값의 정의만 가질 뿐 별 다른 의미를 지니지 못한다.** 값 이상의 의미를 가질 수 없다는 것.

<br>

하지만, **다음과 같이 객체로 포장하면 의미를 가질 수 있다.**

```java
public class LottoNumber {
    int value;
    ...
}
```

이제 **타입을 정의해줌으로써 원시값에 의미를 부여할 수 있게 된다.**

> 원시값 포장은 말그대로 원시값을 객체로 포장한다는 의미.

<br>

## 1-2 왜 원시값 포장을 해야하는가?

그렇다면 **값에 의미를 부여하는 것 말고, 원시값 포장은 이점이 없을까? 물론 아니다.**

<br>

**1. 책임 분리**

**만약 로또 번호를 포장하지 않으면, 로또 번호를 사용하는 클라이언트 객체에선 매번 유효성 검사를 해줘야 한다.** (로또 번호는 1부터 45사이)

```java
// 포장되지 않은 로또 번호를 사용하는 클라이언트
public void doSomething(int lottoNumber) {
    if (lottoNumber < 0 || lottoNumber > 45) {
        throw new IllegalArgumentException();
    }
    ...
}
```

하지만 만약 포장한다면, 유효성 검사 대한 책임을 분리시킬 수 있다.

```java
// 로또 번호 포장
public class LottoNumber {
    private final int value;

    public LottoNumber(int value) {
        if (lottoNumber < 0 || lottoNumber > 45) {
            throw new IllegalArgumentException();
        }
        this.value = value;
    }
    ...
}

// 포장된 로또 번호를 사용하는 클라이언트
public void doSomething(LottoNumber lottoNumber) {
    ...
}
```

위와 같이 객체를 포장하면 모든 클라이언트 객체로부터 `LottoNumber`의 검증 책임을 분리시킬 수 있다.

<br>

**2. 자율적인 객체**

원시값을 포장하면 객체가 된다. 그렇다면 객체란 무엇일까?

`객체 = 상태 + 행동`

즉, 객체란 상태와 행위를 함께 지니며 스스로 자기 자신을 책임지는 객체를 의미한다.

다시 말해, 의미를 부여할 수 없는 원시값에 책임을 할당하고, 다른 객체와의 협력을 할 수 있게 된다.

<br>

**3. 테스트 용이성**

원시값을 객체로 포장하였으므로, 단위테스트를 할 수 있다.

심리적 안정감과 견고한 프로그램을 만들 수 있게 해준다.

<br>

**4. 캐싱**

원시값을 포장하면, 흔히 매번 객체를 만들어줘야하기 때문에 성능에 영향이 간다고 생각할 수 있다.

하지만, 캐싱을 이용한다면 이러한 부담을 줄일 수 있다. 이는 VO에도 적용된다.

```java
public class LottoNumber {
    public static final int MIN = 1;
    public static final int MAX = 45;

    private final int value;

    private LottoNumber(int value) {
        this.value = value;
    }

    public static LottoNumber valueOf(int value) {
        validateLottoNumber(value);
        return LottoNumberCache.CACHE[value];
    }

    // 유효성 검사
    private static void validateLottoNumber(int value) {
        if (value < LottoNumberCache.LOW || value > LottoNumberCache.HIGH) {
            throw new InvalidLottoNumberException();
        }
    }

    // 캐싱
    private static class LottoNumberCache {

        static final int LOW;
        static final int HIGH;
        static final LottoNumber[] CACHE;

        static {
            LOW = MIN;
            HIGH = MAX;
            CACHE = new LottoNumber[MAX + 1];
            for (int i = MIN; i <= MAX; i++) {
                CACHE[i] = new LottoNumber(i);
            }
        }
    }
    ...
}
```



<br>

# 2 VO

<br>

## 2-1 VO란?
VO(Value Object)는 말그대로 값 자체를 표현하는 객체이다.

다만, VO는 도메인에서 한 개 또는 그 이상의 속성들을 묶어서 특정 값을 나타내는 객체이다.

가장 대표적인 VO로 `String`이 있다.

문자열 자체를 하나의 값으로 보기 위해 char 배열을 감싼 형태이다.

<br>

## 2-2 VO 특징

<br>

**1. 동등성**

**VO는 값 자체를 표현하는 객체이기 때문에, 값을 기준으로 비교를 할 수 있어야 한다.**

<br>

💁‍♂️ 동등성 (equality) : 상태만을 이용해 두 값이 같은지 판단할 수 있는 성질

* 식별자가 없다.
* 불변 상태 (immutable state)이기 때문에 식별자가 필요 없다.
* **두 객체가 동일한 정보를 담고 있는 것을 동등성이라고한다.**
* `equals()`와 `hashCode()`를 재정의해줘야 한다.

<br>

**2. 불변**

VO는 속성값 모두를 식별값으로 사용한다.

만약 아래와 같이 주소를 복사하여, 복사한 참조값을 통해 내부 속성값을 변경하게 되면 기존 객체에 영향이 간다.

```java
public static void main(String[] args) {
    Address 철수주소 = new Address();
    철수주소.setCountry("Korea");
    철수주소.setCity("Seoul");

    Money 빙허주소 = 철수주소; // 참조 주소 복사

    빙허주소.setCountry("USA");
    빙허주소.setCity("New York");
}
```

위와 같이 VO를 불변으로 만들지않으면, 철수주소가 한국 서울이라는 예상과 다르게, 미국 뉴욕으로 나오게된다.

즉, VO(`Address`)를 불변으로 만들지 않으면, `주소 복사`로 인해 원치 않는 결과를 도출할 수 있다.

그래서 **VO는 불변이며, `setter`를 만들면 안된다.**

<br>

> **Entity는 따로 식별 값(PK)을 갖고 있기 때문에 내부 속성값들이 변경된다 하더라도 같은 객체로 인식할 수 있다.** 하지만 **VO의 경우 속성값 모두가 식별 값이므로**, 주소값 복사가 될 때 의도치 않는 객체들이 함께 변경되는 문제를 방지하기 위해 불변으로 만들어줘야한다.


<br>

# 3 원시값 포장과 VO의 차이
원시값 포장과 VO. 둘 다 어떠한 값을 포장하기 위해서 사용된다.

그렇다면 이 둘의 차이점은 무엇일까?? 둘을 같은 것으로 봐도 될까?? 물론 아니다.

|| 원시값 포장 | VO |
|----|----|----|
|단위| 원시값만 포장 | 값 표현을 위해 여러 개 포장 |
|용도| 책임 분리를 위한 원시값 포장 | 값 표현을 위한 포장 |
|동등성| 보장 할수도, 아닐 수도 | 무조건 보장 |
|불변| 불변 일수도, 아닐 수도 | 무조건 불변 |

<br>

# 출처
* [소트웍스 앤솔러지](http://www.yes24.com/Product/Goods/3290339?OzSrank=1)
* [현구막 블로그 - 원시값 포장과 VO는 서로 같은가?](https://hyeon9mak.github.io/is-same-wrapping-class-and-VO/)
