> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 40. @Override 애노테이션을 일관되게 사용하라

<br>


## @Override를 안쓰면 생기는 문제
```java
// 똑같은 소문자 2개로 구성된 바이그램 26개를 10번 반복해 집합에 추가하는 예시
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second = second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }
        System.out.println(s.size());
    }
}
```
* `s.size()`의 결과가 26이여야하지만, 260이 나오게 된다.
* 문제
  * `equals`를 재정의(오버라이딩)한게 아니라 다중 정의(오버 로딩)해버렸다.
    * 기존의 `equals`는 매개변수로 `Object`를 받는다.
* 수정
  ```java
  @Override
  public boolean equals(Object o) {
      if (!(o instanceof Bigram)) {
          return false;
      }
      Bigram b = (Bigram) o;
      return b.first == first && b.second == second;
  }
  ```

<br>

## @Override를 의식적으로 달자
`@Override`를 붙이지 않으면 컴파일러가 오류를 찾아내기 힘들고, 원치 않는 결과가 나올 수 있다.

**그러므로 상위 클래스의 메서드를 재정의하려는 모든 메서드에 `@Override` 애노테이션을 달자.**

<br>

## 예외 - 추상 메서드를 재정의할 때
**구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때는 굳이 `@Override`를 달지 않아도 된다.**

구체 클래스인에 아직 구현하지 않았다면 컴파일러가 이를 알려주기 때문이다.

<br>

## 핵심 정리
* 재정의한 모든 메서드에 `@Override` 애노테이션을 의식적으로 달면 실수했을 때 컴파일러가 바로 알려준다.
  * 즉, **런타임에 발생할 문제는 컴파일때 바로 알 수 있다.**
* 예외는 한 가지뿐이다.
  * 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우엔 이 애노테이션을 달지 않아도 된다.
