> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

> 리소스를 엮을 때는 직접 의존성을 생성하지 말고 의존성 주입을 선호하라.

많은 클래스가 하나  이상의 리소스에 의존한다. 책에서는 `SpellChecker`와 `Dictionary`를 예로 들고 있다.

`SpellChecker`가 `Dictionary`를 사용하고, 이를 의존하는 리소스 또는 의존성이라고 부른다. 

이때 다음과 같이 구현하면 부적절하다고 책에서는 강조한다.



## 부적절한 구현

의존성을 제대로 관리하지 못하도록 구현할 경우 부적절한 구현이 된다.



### static 유틸리티 클래스를 잘못 사용한 예

```java
// 부적절한 static 유틸리티 사용 예 - 유연하지 않고 테스트 할 수 없다.
public class SpellChecker {
    private static final Lexicon dictionary = new KoreanDictionary;  // 의존성 직접 생성
  
    private SpellChecker() {};  // 객체 생성 방지
  
    public static boolean isValid(String word) {...}
  
    public static List<String> suggestions(String typo) {...}
  
}

interface Lixicon {}

class KoreanDictionary implements Lexicon {}
```

* 부적절한 static 유틸리티를 사용한 이유

  * 의존성을 변경하기가 힘들다. -> 의존성을 직접 생성하고 주입할 수 있는 방법이 없으므로

  * 유연하지 않고 테스트 할 수 없다. -> 테스트용 사전을 사용하고 싶은 경우, 의존성을 변경할 수 없으므로 테스트가 힘들다.



### 싱글톤으로 구현한 예

```java
// 부적절한 싱글톤 사용 예 - 유연하지 않고 테스트 할 수 없다.
public class SpellChecker {
  
    private final Lexicon dictionary = new KoreanDictionary();
  
    private SpellChecker() {}
  
    public static final SpellChecker INSTANCE = new SpellChecker() {};
  
    public boolean isValid(String word) {...}
    public List<String> suggestions(String typo) {...}
}
```

사전을 하나만 사용할꺼라면 위와 같은 구현도 만족스러울 수 있겠지만, 실제로는 각 언어의 맞춤법 검사기는 사용하는 사전이 각기 다르다. 

또한 테스트 코드에서는 테스트용 사전을 사용하고 싶을 수도 있다.

<br>

**어떤 클래스가 사용하는 리소스에 따라 행동을 달리 해야 하는 경우에는 static 유틸리티 클래스와 싱글톤을 사용하는 것은 부적절하다.**

<br>

그런 요구 사항을 만족할 수 있는 간단한 패턴으로 생성자를 사용해서 새 인스턴스를 생성할 때 사용할 리소스를 넘겨주는 방법이 있다.



## 적절한 구현



### 생성자를 통한 의존성 주입

```java
// 적절한 구현 - 외부에서 의존성 주입.
public class SpellChecker {
  
    private final Lexicon dictionary;
  
  	// 생성자를 통해 외부에서 의존성을 주입.
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
  
  	public boolean isValid(String word) {...}
  	public List<String> suggestions(String typo) {...};
}
```

위와 같은 의존성 주입은 생성자, static 팩토리([아이템1]()) 그리고 빌더([아이템2]())에도 적용할 수 있다.

주로 사용되는 의존성 주입 방법으로는 세 가지가 있다.

1. 생성자 주입
2. setter 주입
3. 메서드 주입

<br>



### 의존성 주입의 단점

의존성 주입이 유연함과 테스트 용이함을 크게 향상 시켜주지만, 의존성이 많은 큰 프로젝트인 경우에는 코드가 장황해 질 수 있다.

이 경우에는 스프링 같은 프레임워크를 사용해서 해결할 수 있다.



## 요약

**의존하는 리소스에 따라 행동을 달리하는 클래스를 만들 때는 싱글톤이나 static 유틸 클래스를 사용하지 말자.** 

**그런 경우에는 리소스를 생성자나 팩토리로 전달하는 의존성 주입을 사용하여 유연함, 재사용성, 테스트 용이성을 향상 시키자.**



