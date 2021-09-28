# 목차

- [목차](#목차)
- [DTO, VO](#dto-vo)
  - [DTO](#dto)
    - [DTO란](#dto란)
    - [DTO 특징](#dto-특징)
    - [DTO를 사용하게 된 이유](#dto를-사용하게-된-이유)
  - [VO](#vo)
    - [VO란](#vo란)
    - [VO 특징](#vo-특징)
- [참고](#참고)



# DTO, VO

> 오래전 J2EE Patterns에서 DTO를 VO라고 표현하여 오늘날 많은 사람들이 VO를 DTO라고 생각한다고 한다.
>
> 그리고 다음 버전의 J2EE의 책에서는 이 내용이 곧바로 수정되었다.

스프링 공부를 하다 DTO와 VO의 차이점을 명확히 모르겠어서 이참에 정리하고자 한다.

DTO와 VO는 불변객체냐 아니냐에 차이와 비슷한 것 같다.

* 예시
  * String - 불변객체
  * StringBuilder - 가변객체



## DTO



### DTO란

🤔 DTO

* Data Transfer Object (데이터 전송 객체)
* DTO is just as stupid data container which is used to transport data between layers and tiers.
  * **계층 (Layer)간 데이터 교환을 위해 사용하는 객체.**
* DTOs are anemic in general and do not contain any business logic
  * **로직을 갖고 있지 않는 순수한 데이터 객체.** 
* `getter/setter` 메서드만을 가지고 있는다.
* 흔히 직렬화(`Serializable`)하여 전송할 때 사용.

```java
public class User {
  private String name;
  private int age;
  
  public void setName(String name){
    this.name = name;
  }
  
  public void setAge(int age){
    this.age = age;
  }
  
  public String getName(){
    return this.name;
  }
  
  public int getAge() {
    return this.age;
  }
}
```



### DTO 특징

* 데이터 접근 메서드 외에 기능을 가지지 않는다.
  * 정렬, 직렬화 등 데이터 표현을 위한 기능은 가질 수 있다.
* 데이터의 캡슐화를 통해 유연한 대응이 가능하다.
  * 데이터 요청 수 감소 효과



### DTO를 사용하게 된 이유

* **프로세스 간 통신**이 일반적으로 원격 인터페이스(웹 서비스)로 재정렬하면서 이루어지게 되는데 여기에서 각 호출의 비용이 많다는 점을 동기로 하여 이용하게 된다.
* 각 호출의 비용이 큰 것이 클라이언트와 서버 간 왕복 시간과 관련되기 때문에 호출의 수를 줄이기 위해 여러 호출에 의해 전송되는 데이터를 축적하면서 오직 하나의 호출만으로 서비스되는 객체인 DTO를 사용하는 것이다.





## VO



### VO란

🤔 VO

* Value Object (값 그 자체를 표현하는 객체)
* 서로 다른 이름을 가진 VO의 인스턴스가 모든 속성 값이 같다면 같은 객체. (동등성)
* 객체의 불변성을 보장한다.
* 로직을 포함할 수 있다.
* 예시
  * **`java.lang.Integer` (래퍼 클래스)**
  * 날짜



### VO 특징

* 값 자체로 의미를 가지는 객체
* 변하지 않는 값을 가지는 객체 (불변성) READ ONLY
  * 값이 변하지 않음을 보장하며 코드의 안정성과 생산성을 높임
  * **이러한 클래스는 중간에 값을 바꿀 수 없고 새로 만들어야 한다.**
* 값이 같다면 다른 이름을 가져도 동일한 객체
  * 같이 객체인지 판단하기 위해 각 속성들의 값을 비교한다.
    * `equals()`, `hashCode()`를 오버라이딩.

```java
public class Color {
  private final double red;
  private final double green;
  prviate final double blue;
  
  private Color(double red, double green, double blue){
    this.red = red;
    this.green = green;
    this.blue = blue;
  }
  
  // 빌더 패턴을 사용 (값을 변경하기 위해서는 새로운 객체를 만든다.)
  public static Color of(double red, double green, double blue){
    return new Color(red, green, blue);
  }
  
  // getter / setter
  
  public boolean equals(Color color){
    ...
  }
  
  public int hashCode(){
    ...
  }
}
```

* **값 자체만으로 의미를 가지므로 꼭 `equals` 와 `hashCode` 를 오버라이딩 해줘야 한다.**



# 참고

* https://adambien.blog/roller/abien/entry/value_object_vs_data_transfer

* https://martinfowler.com/bliki/ValueObject.html
* https://stackoverflow.com/questions/1612334/difference-between-dto-vo-pojo-javabeans

* https://www.youtube.com/watch?v=J_Dr6R0Ov8E

* https://github.com/benelog/blog/issues/27