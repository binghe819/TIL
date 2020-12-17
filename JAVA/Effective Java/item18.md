> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 18. 상속보다는 조합을 사용하라

> 여기서 말하는 상속은 구현 상속을 의미한다. (인터페이스 상속과는 무관하다)

- [상속의 문제](#상속의-문제)
  * [메서드 호출과 달리 상속은 캡슐화를 깨뜨린다](#메서드-호출과-달리-상속은-캡슐화를-깨뜨린다)
- [상속 문제 해결방법](#상속-문제-해결방법)
  * [조합 (Composition)](#조합-composition)
- [래퍼 클래스의 단점](#래퍼-클래스의-단점)
- [그럼 언제 상속을 사용하는가?](#그럼-언제-상속을-사용하는가)
- [핵심 정리](#핵심-정리)

<br>

## 상속의 문제



### 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다

* **상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.**
  * 상위 클래스 설계자가 확장을 충분히 고려하고 문서화도 제대로 해두지 않으면 **하위 클래스는 상위 클래스의 변화에 발맞춰 수정돼야만 한다.**

* 예시

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
  // 추가된 원소의 수 (size와 달리 추가한 횟수를 저장하는 변수)
  private int addCount = 0; 
  
  ...
    
  @Override
  public boolean add(E e) {
    addCount++;
    return super.add(e);
  }
  
  @Override
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll();
  }
  
  public int getAddCount() {
    return addCount;
  }
}
```

```java
// 위 클래스에 아래와 같이 원소 3개를 더했다고 해보자.
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("틱", "탁탁", "펑"));
s.getAddCount(); // 6
```

원래는 3이 반환되야 정상인 것처럼 보이지만, 결과는 6이 나오게 된다.

그 이유는 HashSet의 addAll 메서드가 add 메서드를 사용해 구현된 데 있다.

```java
// HashSet의 addAll 메서드
public boolean addAll(Collection<? extends E> c) {
  boolean modified = false;
  for (E e: c) {
    if (add(e))
      modified = true;
  }
  return modified;
}
```

(즉, addAll에서 3개를 이미 더했는데, 상위 클래스인 HashSet의 addAll에서 add를 사용하므로 3개가 더 추가된 것.)

**이처럼 상위 클래스의 구현이 숨겨지므로 오류를 발생시키기 쉽다.**

<br>

## 상속 문제 해결방법

기존 클래스를 확장하는 대신 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하면 된다.

즉, **기존 클래스를 래핑하는 것이다.**



### 조합 (Composition)

* 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 의미에서 조합 (Composition)이라 부른다.

* **새로운 클래스의 인스턴스 메서드들은 기존 클래스에 대응하는 메서드를 호출해 그 결과를 반환한다.** 이를 전달(Forwarding)이라 하며, 새 클래스의 메서드들은 **전달 메서드**라고 부른다.

```java
// 래퍼 클래스 (ForwardingSet의 멤버인 Set을 래핑)
public class InstrumentedSet<E> extends ForwardingSet<E> {
  private int addCount = 0;
  
  ...
   
  @Override
  public boolean add(E e) {
    addCount++;
    return super.add(e);
  }
  
  @Override
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll();
  }
  ...
}

// 재사용 가능한 전달(forwarding) 클래스
public class ForwardingSet<E> implements Set<E> {
  private final Set<E> s;
  
  public ForwardingSet(Set<E> s) { this.s = s}
  public void clear() { set.clear(); }
  public boolean isEmpty() { return set.isEmpbty(); }
  public boolean add(E e) { return set.add(e); }
  public boolean addAll(Collection<? extends E> c) { return set.addAll(c); }
  ...
}
```

**중요한 점은 InstrumentedSet이 호출하는 add가 자신이 재정의한 add가 아닌 상위 클래스가 가지고 있는 멤버(`Set<E> e`)의 add를 호출함으로써 addCount의 문제가 발생하지 않게 된다.**

<br>

> **다른 Set 인스턴스를 감싸고(wrap) 있다는 의미에서 래퍼 클래스라고 부른다.**
>
> 또한, 컴포지션과 전달의 조합은 넓은 의미로 위임(delegation)이라고 부른다.

<br>

## 래퍼 클래스의 단점

래퍼 클래스는 단점이 거의 없다.

한 가지, 래퍼 클래스가 콜백 프레임워크와는 어울리지 않는다.

* SELF 문제 때문이다.
  * 콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 한다.
  * 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다.

> **또한, 전달 메서드가 성능에 주는 영향은 미묘하니 전혀 걱정하지 않아도 된다.**

<br>

## 그럼 언제 상속을 사용하는가?

상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다.

즉, 클래스 B가 클래스 A와 `is-a` 관계일 때만 클래스 A를 상속해야 한다.

* 만약 아니라면 A를 private 인스턴스로 두고, A와는 다른 API를 제공해야 하는 상황이 대다수다. (래핑)

<br>

## 핵심 정리

* 상속은 강력하지만 캡슐화를 해친다는 문제가 있다.
* 상속은 순수한 `is-a` 관계일 때만 써야 한다.
  * `is-a` 관계일 때도 안심할 수만은 없는 게, 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 여전히 문제가 될 수 있다.
* 상속의 취약점을 피하려면 상속 대신 래퍼과 전달을 사용하자.
  * 특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 그렇다.