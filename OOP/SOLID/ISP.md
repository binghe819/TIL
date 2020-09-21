# 목차

- [ISP - 인터페이스 분리 원칙](#isp---인터페이스-분리-원칙)
  * [개념](#개념)
  * [예제 1](#예제-1)
  * [예제 2](#예제-2)
  * [빈약한 상위 클래스 vs 풍성한 상위 클래스](#빈약한-상위-클래스-vs-풍성한-상위-클래스)
- [참고](#참고)





#  ISP - 인터페이스 분리 원칙

> * "클라이언트는 자신이 사용하지 않는 메서드에 의존 관계를 맺으면 안된다." - 로버트 C. 마틴
> * **인터페이스는 그 역할에 충실한 최소한의 기능만 공개하라는 것이 객체 지향 스승들의 가르침이다.**





## 개념

* **인터페이스 최소주의 원칙**
  * 자신이 사용하지 않는 인터페이스는 구현하지 말아야 한다는 원칙
  * 하나의 일반적인 인터페이스보다는, 여러 개의 구체적인 인터페이스가 낫다.
* SRP와 ISP는 같은 문제에 대한 두 가지 다른 해결이라고 볼 수 있다.
  * 상황에 맞게 설계하는 것이 좋다.

> 쉽게 말해 **하나의 인터페이스는 하나의 책임**을 수행해야 한다는 의미.



## 예제 1

![img](./image/1234544.png)

* 아무 원칙도 적용되지 않는 설계

![img](./image/151516141.png)

* SRP (단일 책임 원칙)
  * `남자`클래스를 토막내서 하나의 역할(책임)만 하는 다수의 클래스로 분할하는 것

![img](./image/515161512.png)

* ISP (인터페이스 분리 원칙)

  * `여자친구`를 만날 때는 `남자친구` 역할만 할 수 있게 인터페이스로 제한하고 ...

  * ```java
    남자친구 홍길동 = new 남자(); // 기념일챙기기(), 키스하기()밖에 사용이 안된다.
    아들 홍길동 = new 남자(); // 효도하기(), 안마하기()밖에 사용이 안된다.
    사원 홍길동 = new 남자();
    소대원 홍길동 = new 남자();
    ```

* 인터페이스 최소화

  * `남자친구`인터페이스가 `사격하기()`메서드를 제공할 필요도 없고 제공해서도 안된다.





## 예제 2

```java
// ISP를 적용하지 않은 예제
public interface multifunction {
  void copy();
  void fax(Address from, Address to);
  void print();
}

public class copyMachine implements multifunction {
  @Override
  public void copy() {
    System.out.println("### 복사 ###");
  }
  
  @Override
  public void fax(Address from, Address to) {
    // 사용하지 않는 인터페이스가 변경되어도 함께 수정이 일어난다.
  }
  
  @Override
  public print() {
    // 사용하지 않는 인터페이스가 변경되어도 함께 수정이 일어난다.
  }
}
```

* `multifunction`인터페이스에 모든 기능들을 한번에 넣었더니, `copyMachine`을 구현하는데 필요없는 `fax`과 `print`도 모두 구현해줘야한다.

```java
// ISP가 적용된 예제
public interface Print{
  void print();
}

public interface Copy {
  void copy();
}

public interface Fax {
  void fax(Address from, Address to);
}

public class copyMachine implements Copy {
  @Override
  void copy() {
    System.out.println("### 복사 ###");
  }
}
```

* ISP가 적용된 예제
  * `copyMachine`을 구현하는데 `copy`만을 구현해주면 되므로 낭비가 발생하지 않는다.





## 빈약한 상위 클래스 vs 풍성한 상위 클래스

![Screenshot](./image/1541916340635.png)

* 상위클래스가 풍성할 수록 캐스팅이 적게 일어나서 소스코드가 깔끔해진다.
  * 상위클래스가 풍성할수록 좋다.





# 참고

* [스프링 입문을 위한 자바 객체 지향의 원리와 이해](http://www.yes24.com/Product/Goods/17350624)