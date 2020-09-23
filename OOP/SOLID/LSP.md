# 목차

- [LSP - 리스코프 치환 원칙](#lsp---리스코프-치환-원칙)
  * [개념](#개념)
  * [예제 1 - 잘못된 경우](#예제-1---잘못된-경우)
  * [예제 2 - 제대로된 경우](#예제-2---제대로된-경우)
- [참고](#참고)



# LSP - 리스코프 치환 원칙

> * "서브 타입은 언제나 자신의 기반 타입으로 교체할 수 있어야 한다." - 로버트 C. 마틴
> * 객체지향에서 상속은 조직도, 계층도가 아닌 **분류도**가 되어야 한다.
>   * 하위 클래스는 상위 클래스의 역할을 충실히 하면서 확장해나가야 한다는 것.





## 개념

* 리스코프는 만든이의 이름
* 아래 두 개의 문장대로 구현된 프로그램이라면 LSP를 지킨 것이다.
  * `하위 클래스 is a kind of 상위 클래스` - 하위 분류는 상위 분류의 한 종류다.
  * `구현 클래스 is able to 인터페이스` - 구현 분류는 인터페이스할 수 있어야 한다.
* 하위 클래스는 상위 클래스로 치환될 수 있다는 원칙
  * 업캐스팅을 해도 아무런 문제가 되지 않아야 한다는 것.
* `instanceof/downcasting`을 사용하는 것은 LSP를 위반하는 징조.
  * 기본적으로 하위 클래스가 상위 클래스로 치환될 수 있어야 하기때문.





## 예제 1 - 잘못된 경우

<img src="./image/1541915202328.png" alt="Screenshot" style="zoom:80%;" />

* 전형적인 계층도 구조
  * 딸은 아버지의 역할을 할 수 없다
  * `아버지 춘향이 = new 딸();` - 말이 안되지 않는가...





## 예제 2 - 제대로된 경우

<img src="./image/1541915209305.png" alt="Screenshot" style="zoom:80%;" />

* 전형적인 분류도 구조
  * 박쥐는 포유류의 역할을 할 수 있다.
  * `포유류 배트 = new 박쥐()` - 박쥐가 포유류의 역할을 하는 것은 전혀 문제가 없다.





# 참고

* [스프링 입문을 위한 자바 객체 지향의 원리와 이해](http://www.yes24.com/Product/Goods/17350624)