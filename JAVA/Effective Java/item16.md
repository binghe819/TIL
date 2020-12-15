> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라



## 안 좋은 예시

```java
public class Point {
  public double x;
  public double y;
}
```

이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.

* API를 수정하지 않고는 내부 표현을 변경 불가능.
* 불변식을 보장할 수 없다.
* 외부에서 필드에 접근할 때 부수 작업을 수행할 수 없다.
  * 만약 누군가 0을 나누거나 하면 예외가 발생해 프로그램이 멈출 수도 있다.

<br>

## getter/setter

위 문제를  해결하는 가장 기본적인 방식은 접근자와 변경자를 제공하는 것이다.

```java
public class Point {
  private double x;
  private double y;
  
  public Point(double x, double y) {
    this.x = x;
    this.y = y;
  }
  
  public double getX() { return x; }
  public double getY() { return y; }
  
  public void setX(double x) { this.x = x; }
  public void setY(double y) { this.y = y; }
}
```

<br>

## 핵심 정리

* public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
* 불변 필드(final)라면 노출해도 덜 위험하지만 완전히 안심할 수 없다.
  * 예를 들어 참조값이면 불변을 보장할 수 없다.