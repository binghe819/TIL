> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 83. 지연 초기화는 신중히 사용하라

<br>

- [아이템 83. 지연 초기화는 신중히 사용하라](#아이템-83-지연-초기화는-신중히-사용하라)
  - [지연 초기화](#지연-초기화)
    - [지연 초기화란](#지연-초기화란)
    - [최적화에서의 지연초기화](#최적화에서의-지연초기화)
    - [멀티 스레드 환경에서 지연 초기화](#멀티-스레드-환경에서-지연-초기화)
  - [필드 초기화 방법](#필드-초기화-방법)
    - [일반적인 인스턴스 필드 초기화](#일반적인-인스턴스-필드-초기화)
    - [인스턴스 필드의 지연 초기화](#인스턴스-필드의-지연-초기화)
    - [정적 필드용 지연 초기화 홀더 클래스 관용구](#정적-필드용-지연-초기화-홀더-클래스-관용구)
    - [인스턴스 필드 지연 초기화용 이중검사 관용구](#인스턴스-필드-지연-초기화용-이중검사-관용구)
    - [단일 검사 관용구](#단일-검사-관용구)
    - [단일 검사 관용구](#단일-검사-관용구-1)
  - [핵심 정리](#핵심-정리)

<br>

## 지연 초기화

<br>

### 지연 초기화란
* **필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법.**
* 값이 전혀 쓰이지 않으면 초기화도 결코 일어나지 않는 기법이다.
* 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결하는 효과가 있다.

<br>

### 최적화에서의 지연초기화
* 지연 초기화는 보통 최적화 용도로 사용된다.
* 지연 초기화는 양날의 검이다.
  * **클래스 혹은 인스턴스 생성 시의 초기화 비용은 줄지만, 지연 초기화하는 필드에 접근하는 비용이 커진다.**
  * 지연 초기화하려는 필드들 중 결국 초기화가 이뤄지는 비율에 따라, 실제 초기화에 드는 비용에 따라, 초기화된 각 필드를 얼마나 빈번히 호출하느냐에 따라 지연초기화가 성능을 느려지게 할 수 있기 때문이다.
* 지연 초기화가 필요한 시점
  * **클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화하는 비용이 크다면 지연 초기화가 제 역할을 해준다.**

<br>

### 멀티 스레드 환경에서 지연 초기화
* **지연 초기화 하는 필드들 둘 이상의 스레드가 공유한다면 반드시 동기화하자!**

<br>

## 필드 초기화 방법

<br>

### 일반적인 인스턴스 필드 초기화
```java
private final FieldType field = computeField();
```
* 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.

<br>

### 인스턴스 필드의 지연 초기화
```java
private FieldType field;

private synchronized FieldType getField() {
    if (field == null) {
        field = computeFieldValue();
    }
    return field;
}
```
* 지연 초기화가 초기화 순환성을 깨뜨릴 것 같으면 `synchronized`를 단 접근자를 사용하라

<br>

### 정적 필드용 지연 초기화 홀더 클래스 관용구
```java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() { 
    return FieldHolder.field;
}
```
* **성능때문에 정적 필드를 지연초기화해야 한다면 지연초기화 홀더 클래스(lazy initialization holder class) 관용구를 사용하자.**
* **클래스는 클래스가 처음 쓰일 때 비로소 초기화 된다는 특성이 있다.**

<br>

### 인스턴스 필드 지연 초기화용 이중검사 관용구
```java
private volatile FieldType field4;

private FieldType getField4() {
    FieldType result = field4;
    if (result != null)    // 첫 번째 검사 (락 사용 안 함)
      return result;

    synchronized(this) {
      if (field4 == null) // 두 번째 검사 (락 사용)
        field4 = computeFieldValue();
      return field4;
    }
}
```
* 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사(double-check) 관용구를 사용하자.
* 초기화된 필드에 접근할 때의 동기화 비용을 없애준다.
  * 첫번째 검사 => 두번째 검사(필드가 초기화 되지 않았을 때)
* `result` 지역 변수: 필드가 이미 초기화된 상태에서는 이 필드를 딱 한번만 읽도록 보장하는 역할을 한다.

<br>

### 단일 검사 관용구
```java
private volatile FieldType field5;

private FieldType getField5() {
    FieldType result = field5;
    if (result == null)
      field5 = result = computeFieldValue();
    return result;
}

private static FieldType computeFieldValue() {
    return new FieldType();
}
```
* 이중검사의 변종.
  * 반복해서 초기화해도 상관없는 인스턴스 필드를 지연 초기화하는 경우.
* 초기화가 중복해서 일어날 수 있다.

<br>

### 단일 검사 관용구
```java
private FieldType field5;

private FieldType getField5() {
    FieldType result = field5;
    if (result == null)
      field5 = result = computeFieldValue();
    return result;
}

private static FieldType computeFieldValue() {
    return new FieldType();
}
```
* 모든 스레드가 필드의 값을 다시 계산해도 상관없고 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 단일 검사의 필드선언에서 volatile을 없애도 된다.
* 어떤 환경에서는 필드 접근 속도를 높여주며, 초기화가 스레드당 최대 한번 더 이뤄질 수 있다.

<br>

## 핵심 정리
* 대부분의 필드는 지연시키지 말고 곧바로 초기화해야 한다.
* 성능 때문에 혹은 위험한 초기화 순환을 막기 위해 꼭 지연 초기화를 써야 한다면 올바른 지연 초기화 기법을 사용하자.
  * 인스턴스 필드에는 이중검사 관용구
  * 정적 필드에는 지연 초기화 홀더 클래스 관용구
  * 반복해 초기화해도 괜찮은 인스턴스 필드에는 단일검사 관용구

