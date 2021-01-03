>  본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 10. equals는 일반 규약을 지켜 재정의하라

```java
// 기본적인 equals
public boolean equals(Object obj) {
    return (this == obj);
}
```

equals 메서드는 재정의하기 쉬워 보이지만 잘못하면 끔찍한 결과를 초래한다.

**가장 좋은 방법은 아예 재정의를 하지 않는 것이다.** 그냥 두면 해당 인스턴스는 오직 자기 자신과만 같게 된다.

<br>

## 재정의 하지 말아야 하는 상황

* **값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스인 경우**

  * 각 인스턴스가 본질적으로 고유하다. 인스턴스가 가지는 값보다 동작하는 개체임을 나타내는 게 더 중요 (hashcode)
  * 예 ) Thread

* **논리적 동치성을 검사할 일이 없는 경우**

  * 논리적 동치성 - 객체의 인스턴스 값이 같은가

* **상위 클래스에서 재정의한 equals가 하위 클래스에도 들어맞는 경우**

  * 예 ) Collection 구현체 (List, Set, Map)들은 상위 클래스인 AbstractXXX (List, Set, Map)으로부터 상속받아 그대로 사용한다.

* **값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 경우**

  * 예 ) Enum

* **클래스가 private이거나 패키지 전용 클래스라서 클래스의 equals 메서드가 호출 할 일이 없는 경우**

  * 만약 실수로라도 equals가 호출되는 것을 방지하기 위해서는 다음과 같이 코드를 구성하자.

    ```java
    @Override
    public boolean equals(Object o) {
        throw new Exception();  // 호출 금지 (예외 처리는 알아서)
    }
    ```

<br>

## equals를 재정의해야 할 때

* **객체 식별성 (object identity, 두 객체가 물리적으로 같은가)가 아닌 논리적 동치성 (값이 같은가)확인이 필요할 때**
  * 예 ) 두 객체를 객체가 가지고 있는 값을 통해 같은지를 확인하고 싶을때  (값 클래스 - Integer, String)
  * Integer, String은 객체가 같은지가 아니라 값이 같은지를 알고 싶어 할 것.

<br>

## equals 재정의 일반 규약

> 동치 관계란? 
>
> * 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산. => 같은 타입의 클래스( instanceof ) 

**equals 메서드는 동치 관계(equivalence relation)을 구현하며 다음과 같은 규약을 지켜야 한다.**

아래 규칙들을 보면 알겠지만, **`equals`는 많은 장애를 일으킬 확률이 높다.**

<br>

### 반사성

* 반사성 : `null`이 아닌 참조 x가 있을 때, `x.equals(x)`는 `true`를 반환한다.
  * 객체는 자기 자신과 같아야 한다는 의미. 어기기가 더 힘들다..

<br>

### 대칭성

* 대칭성 : `null`이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`가 `true`이면, `y.equals(x)`도 true다.
  * 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 의미.

<br>

### 추이성

* 추이성 : `null`이 아닌 모든 참조 값 x, y, z에 대해 `x.equals(y)`가 `true`이고, `y.equals(z)`가 `true`이면, `x.equals(z)`도 true다.
  * 어기기 쉬운 규칙이다. 
    * **구체 클래스를 확장해 새로운 값을 추가하면서 `equals` 규약을 만족시킬 방법은 존재하지 않는다. 그만큼 equals는 신경 쓰지 않으면 오류를 발생할 확률이 높다.** 
    * 대신 확장을 **상속 대신 컴포지션**하면 가능은 하다.
* 예시
  * `Point`라는 클래스는 `x, y`라는 인스턴스 멤버를 가지고 있고, `x, y` 값을 기준으로 `equals`를 구현했다고 해보자. 만약  `ColorPoint`가 `Point`를 상속받아 `color`라는 멤버를 추가한다면 `equals`는 `color` 값을 무시하고 `equals`연산을 진행하게 된다.
  * 실제로 `Timestamp`는 `Date`를 확장한 후 `nanoseconds` 필드를 추가했다. 그래서 API 설명에서도 둘을 섞어 쓸 때 주의를 해야 한다고 나와있다.

<br>

### 일관성

* 일관성 : `null`이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`를 반복해서 호출하면 항상 같은 결과를 반환한다.
  * 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.
* 예시
  * `URL`의 equals는 주어진 URL와 매핑된 호스트의 IP 주소를 이용해 비교한다. 호스트 이름을 IP 주소로 바꾸려면 네트워크를 통해야 하는데, 그 결과가 항상 같다고 보장할 수 있다.

<br>

### null-아님

* null-아님 : `null`이 아닌 모든 참조 값 x에 대해서 `x.equals(null)`은 `false`다.
  * 모든 객체는 `null`과 같지 않아야 된다는 조건.

```java
@Override
public boolean equals(Object o) {
    if (o == null)
        return false;
}
```

* 위와 같이 `null` 검사를 하지 않고, `instanceof`로 해도 된다.

<br>

## equals 메서드 구현 방법 - 중요

1. `==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
   * 자기 자신이면 `true`를 반환한다. 성능 최적화로도 좋다.
2. `instanceof` 연산자로 입력이 올바른 타입인지 확인한다.
   * `instanceof`를 이용해 항상 같은 타입인지 확인해야 한다. 아니라면 `false`를 반환
3. 입력을 올바른 타입으로 형변환한다.
   * `instanceof`를 검사했기 때문에 형변환은 문제가 없다. 꼭 형변환 해준다.
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
   * 모든 필드가 일치하면 `true`를, 하나라도 다르면 `false`를 반환한다.
   * 기본 타입은 `==`를 사용해도 되지만, 참조형이면 `equals`를 사용한다.

### 전형적인 equals
```java
// PhoneNumber
@Override public boolean equals(Object o) {
    if (o == this)
        return true;
    if (!(o instanceof PhoneNumber))
        return false;
    PhoneNubmer pn = (PhoneNumber)o;
    return pn.lineNum == lineNum 
        && pn.prefix == prefix
        && pn.areaCode == areaCode;
}
```
<br>

이외에도 `equals`를 구현했다면 세 가지만 자문해보자.

* 대칭적인가?
* 추이성이 있는가?
* 일관적인가?

<br>

## equals를 구현할 때 주의해야 할 몇가지 사항

* `equals`를 재정의할 땐 `hashcode`도 반드시 재정의 하자.

* 너무 복잡하게 해결하려 들지 말자.

* `equals`의 매개변수는 `Object`타입만을 사용하자.

  ```java
  // 잘못된 예 - 입력 타입은 반드시 Object여야 한다.
  public boolean equals(MyClass o) {
      ...
  }
  ```



## 결론

**꼭 필요한 경우가 아니라면 `equals`를 재정의하지 말자.**

**굳이 재정의해야 한다면 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.**