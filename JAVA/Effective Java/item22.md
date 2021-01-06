> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

<br>

## 인터페이스

### 인터페이스의 의미
<p align="center"><img src="./image/image-20200619124809620.png" width="400"></p>

* interface = inter(사이, ~간의) + face(얼굴, 대상)
  * 두 대상(객체) 간의 '연결, 대화, 소통'을 돕는 **중간 역할**을 한다.

> 더 자세한 내용은 [다음](https://github.com/binghe819/TIL/blob/master/JAVA/%EC%9E%90%EB%B0%94%EC%9D%98%20%EC%A0%95%EC%84%9D/%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4.md)을 참고해주세요.

<br>

### 인터페이스의 역할
* 다형성
  * 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 **타입 역할**을 한다.
* 공개 API
  * 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 **무엇을 할 수 있는지를 외부 클라이언트에 알려주는 역할**을 한다.
  * **인터페이스는 오직 이 용도로만 사용해야 한다.**

<br>

## 안티패턴 - 상수 인터페이스
```java
public interface PhysicalConstants {
    // 아보가드로 (1/몰)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;

    // 볼츠만
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
}
```

메서드 없이 상수 필드만 가득한 상수 인터페이스를 만들지 말아야한다. 안티패턴이다.

**그 이유는 상수 인터페이스의 구현은 외부 인터페이스가 아닌 내부 구현에 해당한다. 즉, 내부 구현을 외부로 노출하는 꼴이다.**

이는 사용자에게 혼란을 주며 클라이언트 코드가 내부구현에 해당하는 상수에 종속적이게 될 수 있다.

## 굳이 상수로 사용하고 싶다면

상수를 공개하고 싶다면 상수전용 인터페이스를 구현하지 말고 다음 방법을 사용하도록 하자

* 상수와 연관된 클래스나 인터페이스에서 공개 (예시 : `MIN_VALUE`, `MAX_VALUE`)
* Enum 사용
* 인스턴스화할 수 없는 유틸리티 클래스에 담아 공개
  ```java
  public class PhysicalConstants {
      private PhysicalConstants() {} // 인스턴스화 방지

      public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

      ...
  }
  ```